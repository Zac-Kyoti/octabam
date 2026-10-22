# `sidechain-compressor-bw` -- SIDECHAIN_COMPRESSOR_BW

Sidechain Compressor B-W: stock COMPRESSOR with a side-chain KEY from any of T1-T8 and a base-width key filter. Page 2 adds MON (hear the processed key instead), KEY (OFF or T1-T8), KGN (the key's gain, about -24..+24 dB), KBAS (the key's high-pass, 18 dB/oct: OFF, 20 Hz-19.6 kHz, shown in Hz) and KWID (the key's low-pass, 12 dB/oct, this many octaves above KBAS: 0.5-9.9 or OFF, shown in octaves; with KBAS OFF a plain low-pass from 20 Hz), beside stock's RMS. The filter isolates the key better than SIDECHAIN_COMPRESSOR's one-knob KFLT (a kick's fundamental without its sub-sonic rumble, say), which reduces distortion and keeps the compression smooth at short attack and release settings.

Built from [Zac-Kyoti/octatrack-kyoti-fw](https://github.com/Zac-Kyoti/octatrack-kyoti-fw)
(submodule `upstream/`, pinned to `9abc70e`). `Kind.DSP_EFFECT` with `MenuEntry(replaces="COMPRESSOR", stock_dsp=True)`: the row is COMPRESSOR's and its dispatch entry stays stock's. One linked unit (`patch_sidechain_bw.s`, 600 B; ROM cave or DRAM by the `PLACE` setting, default CAVE: MON's, KEY's, KBAS's and KWID's formatters, KEY's list widget, and `sc_norm`, reached by two `jsr` detours at the first instruction of both page-2 copiers, `0x4000cae8` and its twin `0x40003d1c`); page-2 slots 7-11 as raw descriptor words; four per-payload `DspHook`s (`sctap`, `scdet`, `moncommit`, and `scinit` on COMPRESSOR's init, which resets the key filters' state at r7+$1c..$2d); the per-core values as `DspSection.subst`; the 16 + 49 table words as one `ptable`.
`upstream/octabam-modules/sidechain-compressor-bw/README.md` is the full description, with what was measured and what was inferred.

## Measured

- Every build re-links `scbw_cf` and compares it with the author's own bytes (`reference`); a difference refuses the build.
- `upstream/tools/emu_sc_bw_dsp3.py --layout octabam` (this manifest's addresses): the key filters bit-exact against `upstream/tools/sc_keyfilt.py`, the MON crossfade, the KEY hold, `scinit`.
- `make check REMIX=sidechain-compressor-bw` (`remixes/test/sidechain-compressor-bw/`) and `make check REMIX=kyoti-mute-sidechain-bw` build and boot the image under the ColdFire port.
- DSP section 677 words (612 code + the 65-word `ptable`), in SPRING REV's words.

## On the unit

- 22 Oct 2026, on the author's MKI, in the author's standalone image (`140C_SCKF4`): page 2 and its readouts, a kick fundamental keyed at short attack and release, MON and KEY declick, the load with 1 + 7 and 4 + 4 COMPRESSOR / DJ EQ per core. The author's KYOTI V1.6 image (with MUTE_MODES' SC_KEY) ran on the unit the same day.
- This octabam form has not been flashed.

## Notes

- One or the other with SIDECHAIN_COMPRESSOR, the light version of the same effect: both replace COMPRESSOR and share its FX2 id; this module names it in `conflicts`, so the ledger refuses the pair.
- Like SIDECHAIN_COMPRESSOR: the keybus and MON words at core-private `Y:$7f0-$9ff`, and the cross-core window (the last `$202` words of each core's half of the shared window), so it is refused beside BusDelay, BusVerb and WAVE (as the light version is; a pairwise survey on `ff9c5f84`).
- For a composed caller, `scmonbody` uses a, b, x0, x1, y1, r1, n1 and r3.
