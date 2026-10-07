# Tamarin — technical review

Review of the reconstructed project against the 30 archived conversations.
Corrections below were applied to the files in this repo; flags are things
I could not verify from the archive and should be treated as suspect until
tested on hardware.

## Corrections applied

1. **`swj_newdap` → `swd newdap`** in all configs. The old spelling is
   rejected by the OpenOCD versions used here; the archive shows this
   exact error and its fix.
2. **`-chain-position` vs `-dap` untangled.** `dap create` references the
   TAP via `-chain-position <chip>.<tap>`; `target create` references the
   DAP via `-dap <chip>.dap`. The threads mixed these up (producing
   "target requires -dap parameter instead of -chain-position"); the
   reconstructed files keep them straight.
3. **`adapter driver tamarin` → `interface tamarin`.** The `adapter driver`
   spelling is 0.11+; the fork in use is 0.10/0.12-era, where it fails
   with `invalid subcommand`. Comment in `tamarin.cfg` records this.
4. **Removed `init` / `reset halt` / `mdw` / `shutdown` from `t8015.cfg`.**
   The "working" config in the threads baked session commands into the
   target file. That works once and then fights every scripted or `-c`
   invocation. Config files define; the operator drives (telnet/GDB).
5. **Parameterized `-chain-position`.** The working config hardcoded
   `-chain-position t8015.cpu` while parameterizing everything else;
   now `-chain-position $_CHIPNAME.cpu`.
6. **Completed the truncated SEP target** in `t8015-cores.cfg`
   (`-ap-num 4 -cti ...sep.cti`, with a TODO for the unknown `dbgbase`).
   The Bonobo reference pasted in the threads cuts off mid-target.
7. **`tamarin.c:325` assert.** Recommending the error-return replacement
   and the stabilized upstream commit rather than leaving a bare
   `assert(false)` that aborts the process on a recoverable error.
8. **Kept `-ctibase`** (with a deprecation note) instead of silently
   migrating to `-baseaddr`, since the fork predates the rename and the
   threads show `-ctibase` working.

## Flags — unverified, treat as suspect

- **Pico↔iPhone wiring pinout.** The archive contains no definitive
  Tamarin-cable-to-iPhone-X pinout. One assistant message claims Pico
  GPIO0→SWDIO / GPIO1→SWCLK / GND→Pin 5; picoprobe convention is
  GPIO2/GPIO3. These disagree and neither is confirmed. **Do not trust
  either without the tamarin-firmware docs or a continuity check.**
- **DPIDR `0x0bc11477` vs CPUTAPID `0x4ba02477`.** The threads converge
  on both, but the only value ever observed on the wire is DPIDR
  `0x00000001` (dormant). The "expected" values are plausible but
  unconfirmed — `-expected-id` will abort the session if they're wrong.
- **All core/CTI/SEP base addresses** in `t8015-cores.cfg`
  (`0xc8010000`–`0xc8520000`, SEP `0x242020000`) are Bonobo's values,
  never validated against this bench. The file is marked UNVERIFIED
  throughout for this reason.
- **Watchdog kill addresses** (`0x2102BC000`, `0x2102BD000`) come from
  assistant messages, not from a captured successful halt. Plausible
  (same page as the demotion register) but unconfirmed.
- **Demotion register semantics** (`0x00000000` = demoted,
  `0x00010206` = not) are asserted in the threads without a captured
  before/after pair. Treat as a hypothesis until both states are read.
- **Several assistant claims in the threads are unreliable.** Examples:
  `aarch64 dbginit` "bypassing Apple secure debug authentication"
  (no such bypass is demonstrated), `target smp ... sep` (SEP is not
  an SMP peer of the app cores), and version-migration advice that
  contradicts the fork's actual vintage. I did not propagate these.

## Open questions needing hardware

1. Why does the DP answer `0x00000001`? Dormant port, power domain
   down, or wiring? This is the single blocking question.
2. What is the correct Tamarin-cable pinout for the iPhone X debug port?
3. Does `reset halt` actually hold with the watchdog kills, or does GDB
   still drop?
4. Are the Bonobo core/SEP base addresses valid on this T8015 revision?
5. What does `mdw 0x2102BC000 1` return on a known-demoted vs
   known-not-demoted device?

## Note: diagrams

The media image-generation pipeline's tool namespace is not available
in this subagent environment, so no pipeline-generated images were
used. Instead, five hand-authored SVG diagrams were added under
`docs/images/` (wiring, OpenOCD stack, bring-up flowchart, SWD switch
timing, DAP topology) plus `docs/glossary.md`. SVGs were chosen
deliberately: they stay sharp, remain editable in the repo, and every
label is exact (generated raster diagrams routinely garble technical
labels). They can be replaced with pipeline-generated art later if
desired — the markdown references (`docs/images/*.svg`) will keep
working if filenames are preserved.
