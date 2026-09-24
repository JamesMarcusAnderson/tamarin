# SOURCES — Tamarin project evidence map

Every factual claim in this draft traces to James Anderson's own chat messages
(REQUEST fragments) or his pasted terminal output in the DeepSeek archive
(`~/workspace/deepseek-archive/conversations.json`). Assistant text was never
used as evidence. Full conversation IDs below.

## Verified facts and where they come from

| Claim | Conversation | Evidence |
|---|---|---|
| Pico flashed with stacksmashing's tamarin-firmware; toolchain ready (openocd, pico-sdk, picoprobe, picotol, tamarin-firmware "downloaded, installed, compiled and ready to run") | `b08c1022-cbf3-4f4d-bbf3-e932891ae923` — "Using Debug Probes for SWD on Pico W" (2025-03-27) | James's own message |
| Tamarin Cable USB profile: 3 endpoints / 2 serial ports (control + DCSD/monitor); needs Tamarin-aware OpenOCD fork; `openocd -f interface/tamarin.cfg -f t8015.cfg` | `b08c1022-cbf3-4f4d-bbf3-e932891ae923` (same) | James pasted the tamarin-firmware README "Usage" section |
| Tamarin firmware source | `b08c1022-cbf3-4f4d-bbf3-e932891ae923` (same) | James: "its so i can do this https://github.com/stacksmashing/tamarin-firmware" |
| Tamarin Cable enumerates on his Mac: `/dev/tty.usbmodem313371`, `/dev/tty.usbmodem313374`; `ID 2b3e:0004 2b3e Tamarin Cable Serial: 31337` | `dd46d068-aa36-4b76-b8b6-8bbf96663dde` — "Fixing OpenOCD SWD Configuration Error" (2025-04-15) | James's own `ls -l /dev/tty.usbmodem*` and system_profiler output |
| **SWD DPIDR 0x4ba02477 from the iPhone X** via `interface/tamarin.cfg` + `t8015.cfg`; CoreSight targets enumerated (ecore0: 6 hw breakpoints, 4 watchpoints); GDB/telnet servers up | `dd46d068-aa36-4b76-b8b6-8bbf96663dde` (same) | James's own full OpenOCD terminal session (see `sessions/iphone-x-swd-session.log`) |
| `iphone.sep` target reachable in OpenOCD telnet (full `help iphone.sep` command tree) | `dfb4eda9-f4b2-451f-bdf0-d54f13fb9adb` — "1BitSquared and Apple Hacking Inquiry" (2025-05-22) | James pasted his own telnet `help` output, then: "i got those commands by myself when i got into the sep debug with a breakout board and a pico" |
| "i got JTAG into the iphone x ios 15.5 sep debug using only a $7.88 lightning connector breakout board and a $5.99 pi pico" | `9012de49-988a-47da-9b4b-3b468f43b2c5` — "Upgrade iPhone X and MacBook Air for Hacking" (2025-05-23) | James's own words |
| Target phone: iPhone X, iOS 15.5, jailbroken with palera1n | `41bc39fe-1576-45ce-898d-81ba21309fea` — "Jailbreak iPhone X iOS 15.5 with palera1n" (2025-03-27); `b08c1022` (same, msg 17) | James's own messages |
| Lightning breakout pinout (SWDIO→GP2, SWCLK→GP3, GND→GND, UART→GP0/GP1), labeled "(Confirmed)" | `dd46d068-aa36-4b76-b8b6-8bbf96663dde` (same) | James's own pasted pinout table |
| Tristar 1.8 V source; IDBUS lines 3.0 V-tolerant per his oscilloscope; level-shifter warning | `dd46d068-aa36-4b76-b8b6-8bbf96663dde` (same) | James's own message ("according to my oscilloscope") |
| `openocd/tamarin.cfg` in this repo | `dd46d068-aa36-4b76-b8b6-8bbf96663dde` (same) | James's own pasted config (cleaned: header comment added, otherwise verbatim) |
| Target config derived from Bonobo's published A11 configs | `6d7c3a9a-7a20-4955-8f90-525383cf079e` — "Creating OpenOCD Fork for iPhone X SWD" (2025-04-02) | James: "we can scrape some of the info here https://docs.bonoboswd.com/…" (t8015.cfg content is Bonobo's, linked not copied) |
| Pico-as-Picoprobe baseline: RP2040 target, `SWD DPIDR 0x0bc12477`, Cortex-M0+ examined, GDB server up | `65aaf874-aaf0-4407-891e-b2defa7fbbd0` — "Setting Up Picoprobe with Raspberry Pi Pico" (2025-04-01) | James's own OpenOCD terminal session |

## Deliberately NOT claimed (and why)

- **Memory reads/dumps:** `mdw` attempts in-session returned `Target not examined yet`; cores reported `powered down`. Not achieved — stated as a non-claim in the README.
- **DCSD console:** the cable's second serial port prints mode status; no DCSD console interaction was evidenced. Excluded per project boundaries.
- **Custom firmware:** none written; Tamarin firmware used unmodified. Stated explicitly.
- **Demotion mechanism:** discussed as a prerequisite in chats, but the mechanism James used was not evidenced in a verified form; not claimed.
- **Bypass/unlock/activation work:** other conversations discuss it, but nothing verified was achieved and it is out of scope for this repo. Excluded.
- **Astris/Kong-probe threads** (`5cf2a7d9`, `a977deee`, `a9566bb5`, `a7fe9ce5`) concern Apple's internal tooling and a different probe — not Tamarin, not included.
- **Assistant-generated hype** (e.g. "wiring cheatsheets", monetization/0-day content in some RESPONSE fragments) was ignored entirely — not James's verified work.

## Redactions

- Host paths in the archive were already anonymized to `/Users/user/` and `user@hostname-2` by the export; kept as-is.
- No MAC addresses, keys, tokens, or personal serials appear in the included materials. (A Pico CMSIS-DAP serial seen in an unrelated baseline session was omitted.)
