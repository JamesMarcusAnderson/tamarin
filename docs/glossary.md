# Tamarin glossary

Terms as used in this project. Definitions are grounded in the bring-up;
broader meanings are noted where they differ.

- **SWD (Serial Wire Debug)** — ARM's two-pin debug protocol (SWDIO +
  SWCLK), the only debug transport the iPhone X exposes. Used instead of
  JTAG throughout this project.
- **SWDIO / SWCLK** — The SWD data and clock lines. On a picoprobe-flashed
  Pico these are GPIO2 and GPIO3; the Tamarin cable's own pinout is
  unverified (see REVIEW.md).
- **TAP (Test Access Port)** — The debug port's protocol endpoint. Created
  here by `swd newdap t8015 cpu`; OpenOCD names it `t8015.cpu`.
- **IDCODE** — The 32-bit TAP identification register. This project's
  converged T8015 value is `0x4ba02477`, enforced via `-expected-id`.
- **DP (Debug Port)** — The SWD-level access point to the debug fabric.
- **DPIDR (Debug Port Identification Register)** — Identifies the SWD DP
  itself. Expected `0x0bc11477` for T8015; the bench only ever read
  `0x00000001` (dormant DP) — the project's open problem.
- **DAP (Debug Access Port)** — OpenOCD object (`t8015.dap`) binding the
  TAP to one or more Access Ports. Created with `dap create ...
  -chain-position t8015.cpu`.
- **AP (Access Port)** — A bus slave behind the DAP. AP 1 = debug bus,
  AP 4 = system/memory bus in the Bonobo mapping used here.
- **MEM-AP (Memory Access Port)** — An AP giving direct memory access
  (`t8015.mem`, `mdw`/`mww`) without halting a core.
- **CTI (Cross Trigger Interface)** — Per-core trigger routing block; the
  full core config pairs each core target with its CTI (`-cti`) so
  halts/breakpoints propagate.
- **SRST (System Reset)** — The only reset line the iPhone X debug port
  exposes (`reset_config srst_only`); TRST is not wired.
- **Demote / demotion** — Irreversible palera1n operation (`--demote`)
  that disables the Secure Enclave on checkm8-vulnerable iPhones,
  unlocking the debug port. Verified via `mdw 0x2102BC000 1`:
  `0x00000000` = demoted, `0x00010206` = not.
- **palera1n** — checkm8-based jailbreak for A11 and older; the
  `--demote` flag performs the SEP demotion.
- **checkm8** — Unpatchable BootROM exploit for A5–A11; what makes the
  whole project possible.
- **Tamarin** — Three related things: (1) the Pico firmware
  (`stacksmashing/tamarin-firmware`) implementing the probe protocol,
  (2) the Tamarin Cable hardware product, (3) the `tamarin` OpenOCD
  adapter driver in the `stacksmashing/openocd-tamarin` fork.
- **Picoprobe** — Raspberry Pi's official CMSIS-DAP firmware for the
  Pico; the alternate probe path used in this project
  (`interface/cmsis-dap.cfg`, VID:PID `0x2e8a:0x000c`).
- **CMSIS-DAP** — ARM's standard probe protocol; what picoprobe speaks.
- **Bonobo** — Third-party SWD probe/config ecosystem
  (docs.bonoboswd.com); source of the core/CTI/SEP base addresses in
  `t8015-cores.cfg` (unverified).
- **OpenOCD** — Open On-Chip Debugger. This project requires the
  `stacksmashing/openocd-tamarin` fork; mainline has no `tamarin`
  driver.
- **`swd newdap`** — OpenOCD command creating an SWD DAP/TAP instance.
  (The pre-0.10 spelling `swj_newdap` is rejected by current OpenOCD.)
- **`aarch64 dbginit`** — OpenOCD post-init for AArch64 targets; run
  after `init`, before `reset halt`.
- **`reset-init`** — Tcl event hook in `t8015.cfg` that halts and kills
  the E-core watchdogs on every reset so GDB sessions survive.
- **Watchdog kill** — `mww 0x2102BC000 0x0` / `mww 0x2102BD000 0x0`;
  stops the E-core watchdogs reviving halted cores (the "Core.0 failed
  to shut down" GDB drop).
- **DCSD (Direct Connect System Debugger)** — Apple's diagnostic
  protocol/transport. The HAM DCSD ALEX cable provides it over FTDI
  serial; bypassed as a JTAG adapter (unsupported FTDI chip `0x0600`)
  but still usable as a serial console.
- **GDB** — Attaches to OpenOCD's gdb server (`:3333`) for interactive
  debugging once the target is halted.
- **Tcl** — The scripting language of OpenOCD config files (`.cfg`).
- **`adapter_khz`** — Period spelling of `adapter speed` in this fork's
  vintage; sets the SWD clock (10 MHz working, 1 MHz fallback).
