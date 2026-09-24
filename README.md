# Tamarin — Pico SWD/JTAG probe for iPhone X (A11)

A hardware debug-probe project: a Raspberry Pi Pico flashed with
[stacksmashing's Tamarin firmware](https://github.com/stacksmashing/tamarin-firmware)
(unmodified), used as an SWD/JTAG adapter through OpenOCD to reach the debug
port of an iPhone X (Apple A11 / T8015).

This is debug-port bring-up and nothing more: prove the probe enumerates,
prove SWD talks to the target, enumerate what CoreSight exposes. No firmware
was written, no console was accessed, nothing was dumped.

## Hardware

| Part | Role |
|---|---|
| Raspberry Pi Pico | Probe — runs Tamarin firmware, presents as "Tamarin Cable" over USB |
| 16-pin Lightning connector breakout board | Physical tap into the phone's IDBUS lanes |
| iPhone X (A11 / T8015), iOS 15.5, jailbroken (palera1n) | Target |
| macOS host | Runs OpenOCD |

## Wiring (confirmed)

Lightning breakout → Pico, as used in this project:

| Breakout pin | Signal | Lane | Pico |
|---|---|---|---|
| Pin 1 | GND | Ground | GND |
| Pin 2 | L0+ | Lane 0 positive | USB0 P |
| Pin 3 | L0− | Lane 0 negative | USB0 N |
| Pin 4 | ID0 | Lane 0 ID/control | **SWCLK → GP3** |
| Pin 5 | VCC | 5V power | VBUS |
| Pin 6 | L1− | Lane 1 negative | UART TX → GPO #1¹ |
| Pin 7 | L1+ | Lane 1 positive | UART RX → GP1 |
| Pin 8 | ID1 | Lane 1 ID/control | **SWDIO → GP2** |

> **Voltage warning (measured):** Tristar runs on a 1.8 V source and the IDBUS
> lines are 3.0 V-tolerant (verified with an oscilloscope). Do not drive IDBUS
> with 5 V-tolerant devices (e.g. some Arduinos) without a level shifter.
>
> ¹ James's notes read "GPO #1" — presumed GP0 typo; the UART lines were not
> used for SWD in this project.

## Software setup

1. Flash the Pico with Tamarin firmware (upstream release, unmodified):
   it enumerates as USB `2b3e:0004` "Tamarin Cable", serial `31337`, exposing
   three USB endpoints — two serial ports (control + monitor).
2. Build OpenOCD from source with the Tamarin probe driver
   (the upstream Tamarin project ships an OpenOCD fork with `adapter driver tamarin`).
3. Target config: based on Bonobo's published `t8015.cfg`
   ([docs.bonoboswd.com](https://docs.bonoboswd.com)) — A11 DAP, CoreSight CTIs,
   `iphone.dbg` / `iphone.mem` / `iphone.sep` targets.
4. Interface config: [`openocd/tamarin.cfg`](openocd/tamarin.cfg) (this repo).

Connect:

```sh
sudo openocd -f openocd/tamarin.cfg -f t8015.cfg
```

Full redacted session: [`sessions/iphone-x-swd-session.log`](sessions/iphone-x-swd-session.log).

## Verified results

- **Probe enumerates:** `/dev/tty.usbmodem313371` + `/dev/tty.usbmodem313374`
  appear on the Mac; `system_profiler` shows `ID 2b3e:0004 2b3e Tamarin Cable`.
- **SWD reaches the A11:** `Info : SWD DPIDR 0x4ba02477` — the iPhone X debug
  port answers over SWD through the Pico probe (IDCODE as captured in the
  session log below).
- **CoreSight enumerates:** OpenOCD instantiates `iphone.ecore0` (6 hardware
  breakpoints, 4 watchpoints), `ecore1–3`, `pcore0–1`, `iphone.dbg`,
  `iphone.mem`, and `iphone.sep`, and opens GDB/telnet/Tcl servers.
- **Sanity baseline:** the same bench (Pico as CMSIS-DAP Picoprobe) debugs an
  RP2040 target cleanly — `SWD DPIDR 0x0bc12477`, both Cortex-M0+ cores
  examined, GDB server up. Proves the method before pointing it at the phone.

## Explicit non-claims

- **No custom firmware.** The Pico runs stacksmashing's Tamarin firmware
  unmodified. Nothing here is a firmware project.
- **No DCSD console.** The cable's monitor port prints mode status only; the
  DCSD serial console was not used.
- **No memory access.** Cores reported `powered down` / `missing UTT
  configuration`; `mdw` attempts returned `Target not examined yet`. No halting,
  no reads, no dumps.
- **No bypass work.** No activation, iCloud, or lock-screen work of any kind.
  Debug-port bring-up only.

## Credit

Probe design and firmware by [stacksmashing](https://github.com/stacksmashing)
(Tamarin). Target definitions derived from Bonobo's published A11 OpenOCD
configs. Bench work, wiring, voltage measurements, and bring-up are original.
