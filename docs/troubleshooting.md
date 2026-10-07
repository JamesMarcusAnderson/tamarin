# Tamarin troubleshooting

Every distinct failure encountered during bring-up, in the order the
project hit them, with the fix that resolved it.

![DAP topology](images/dap-topology.svg)
*Keep the TAP → DAP → target references straight — most config errors are this.*

![SWD switch sequence](images/swd-timing.svg)
*What happens on the wire before the `tamarin.c:325` abort.*

## Config syntax errors

### `Error: invalid command name "swj_newdap"`
`swj_newdap` is the pre-0.10 spelling. Current OpenOCD renamed it.
**Fix:** use `swd newdap`:
```tcl
swd newdap t8015 cpu -irlen 6 -expected-id 0x4ba02477
```

### `Error: target requires -dap parameter instead of -chain-position`
`target create` and `dap create` take different references:
- `dap create <name>.dap -chain-position <chip>.<tap>` (DAP → TAP)
- `target create <name>.cpu aarch64 -dap <name>.dap` (target → DAP)

**Fix:** never pass `-chain-position` to `target create`.

### `Error: invalid subcommand "driver tamarin"`
`adapter driver` exists only in OpenOCD ≥ 0.11. The Tamarin fork used
here is 0.10/0.12-era.
**Fix:** use the period spelling in the interface config:
```tcl
interface tamarin
```

### `Error: session transport is "swd" but your config requires JTAG`
A JTAG-only command (e.g. `jtag_rclk`) is present while the transport
is SWD.
**Fix:** remove all JTAG-only commands from SWD configs.

### `Error: invalid subcommand "serial /dev/tty.usbmodem..."`
The `tamarin` driver takes no `serial` subcommand.
**Fix:** delete the line. If multiple probes are attached, select by USB
location instead (`adapter usb location`).

### `Warn: Transport "swd" was already selected`
Harmless. `transport select swd` appears twice (interface + target
file). Can be silenced by keeping it in one file only.

### `Error: -chain-position is invalid` (on `dap create`)
The referenced TAP (`t8015.cpu`) does not exist when `dap create` runs —
i.e. `swd newdap` in the interface file failed or never ran.
**Fix:** fix the interface file first; the target file cannot work
without the TAP.

## Build / install problems

### `Error: Can't find interface/picoprobe.cfg` / missing `aarch64.cfg`
`sudo make install` did not complete; the `tcl/` tree is incomplete.
**Fix:** rerun `sudo make install` from a clean build and verify
`tcl/interface` is populated. Do not hand-copy single files.

### picoprobe CMake failure: FreeRTOS has no RP2040 port
The FreeRTOS submodule is missing `portable/ThirdParty/GCC/RP2040`.
**Fix:** `git submodule update --init --recursive` in the picoprobe
tree and rebuild. The prebuilt `picoprobe.uf2` avoids this entirely.

### `unable to find CMSIS-DAP device` (macOS)
The Pico's CMSIS-DAP interface is claimed by something else, or the
wrong VID:PID is probed.
**Fix:** pass the IDs explicitly and skip the generic cfg:
```sh
openocd -c "adapter driver cmsis-dap" \
        -c "cmsis-dap vid_pid 0x2e8a 0x000c" \
        -c "transport select swd" \
        -f target/t8015.cfg
```
Do not try to unload Apple's USB kernel drivers (`kextunload` on
`com.apple.driver.AppleUSBHost` fails — the bundle id does not exist
as a loadable kext).

### `DEPRECATED!` warnings (`adapter_khz`, `cmsis_dap_vid_pid`, `-ctibase`, `interface`)
Version-skew warnings, not errors. Migration map:
`adapter_khz` → `adapter speed`, `cmsis_dap_vid_pid` → `cmsis-dap vid_pid`,
`-ctibase` → `-baseaddr`, `interface` → `adapter driver` (≥ 0.11 only).

## Probe / target communication failures

### `unsupported FTDI chip type: 0x0600`
The DCSD ALEX cable's FTDI is not in OpenOCD's FTDI driver table.
**Fix:** stop using the cable as the JTAG adapter; use the Pico
(Tamarin firmware or picoprobe) as the adapter instead. The cable is
still fine as a serial/DCSD transport.

### `Info: SWD DPIDR 0x00000001`
The DP is not answering — dormant debug port, target powered down, or
wiring fault. The expected T8015 DPIDR is `0x0bc11477`.
**Fix checklist:** confirm demotion (`mdw 0x2102BC000 1` → `0x00000000`),
reseat SWDIO/SWCLK/GND, lower `adapter_khz` to 1000, confirm the phone
is in the demoted standby state (not fully booted, not off).

### `Assertion failed: (false), function tamarin_swd_switch_seq, tamarin.c:325`
The Tamarin driver's SWD switch sequence got an unexpected ACK and hit
a bare `assert(false)`. This is a driver bug meeting a signaling
problem: the assert turns a recoverable error into a crash.
**Fix:** update to the stabilized `openocd-tamarin` commit; if patching
locally, replace the assert with a proper error return:
```c
if (response != EXPECTED_SWD_ACK) {
    LOG_ERROR("SWD sequence failed (got 0x%02x)", response);
    return ERROR_FAIL;
}
```
Then debug the underlying cause (usually the DPIDR `0x00000001`
condition above).

### GDB drops: `Core.0 failed to shut down`
Watchdogs revive the cores out from under the debugger.
**Fix:** the `reset-init` event in `t8015.cfg` halts and kills the
E-core watchdogs (`mww 0x2102BC000 0x0`, `mww 0x2102BD000 0x0`) on every
reset. If GDB still drops, the kills are not firing — confirm the event
is attached to the right target name.

## Command-line mistakes

### `Unexpected command line argument: mdw ...`
`-c` takes exactly one argument. Quote the whole command string:
```sh
openocd -f openocd/tamarin.cfg -f openocd/t8015.cfg \
  -c "init; aarch64 dbginit; reset halt; mdw 0x2102BC000 1; shutdown"
```
Better: run OpenOCD plain and drive it from `telnet localhost 4444`.
