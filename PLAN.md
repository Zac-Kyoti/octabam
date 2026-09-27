# The plan

octabam is a remixer for the Octatrack's OS: a mod is a module, a remix is
a selection of modules, and the build turns a remix and the user's own
1.40C into one image. `docs/remixer/PLACEMENT.md` is the architecture
record for where code goes; `remixes/<name>/README.md` describes each remix;
`CHANGELOG.md` records each flashed image and what main carries beyond it.

## Where it stands (27 Sep 2026)

- **On hardware.** The rig (`bamsep26`) on Sam's MKII: image 43
  (`OCTABAM43`, 21 Sep 2026) is the last road build; images 44–99 were
  probes (the T1 frame bursts, below; 99 is on the card). `usb-audio` ran
  as image 64 (25 Sep: 16 channels, 9.6 minutes clean after each stream's
  first 1.6 s); `usb-lean` image 90 on Bryan T's MKII. `ok-ms` on midisc's
  author's unit as OKMS2; octalab (nordseele) on an MKI since 11 Sep;
  REPITCH as OCTABAM81; `recfix` as OCTABAM83/84.
- **Main is unheard since 23 Sep.** Image 53 built; everything after it
  is port-gated only: the host pages drawn like the SEND tracks (DEL / REV
  on slots 0 / 1, the rest on the TEMPO window), the DEL / REV split,
  TEMPO BUS, SCENES P2, per-sample knob ramps, BusVerb's wet limiter and
  new SHFT table, RLEN PLEN, RECORDER HOLD. The two voicing items from
  20 Sep are answered in code and not yet by ear: Character's DRV drives
  the curve's input (23 Sep), BusVerb's wet is limited before the ×2
  (27 Sep). **A project saved before needs a stamp before play**
  (`ot_project.py host <project>`, or `stamp-defaults … --all
  --keep-mode`; `remap-slot` for SHFT).
- **The platform** (`tools/remix/`): linked GNU-as units, detours, pokes
  and table growth wired by symbol and asserted against stock; recipe-built
  DRAM runtimes; the loader (derived from Octakit's, N payloads,
  hash-gated); the arena reserve (10 MiB off the bottom of the 85.5 MB
  sample pool); the ledger and compatibility matrix; the ColdFire port
  (`tools/emu/ot_emu`) that boots every remix and, with a project
  (`OT_PROJECT` or `~/.octabam_project`), plays it and reads every window
  back (`verify_set`, `verify_modedefaults`). The module table and the
  remix index are rendered from the manifests (`make docs`, `verify_docs`).

## The ground

Measured under the port unless marked (`docs/remixer/PLACEMENT.md`).

| where | how much | status |
|---|---|---|
| ROM: the OS image's free zero runs | ~8.4 KB, shared by every ROM cave and the chooser clones | measured |
| RAM | 128 MB at `0x40000000`; `0x48000000..` is the same memory uncached | boot code + hardware |
| the audio page arena `0x40a955e0..0x46025de0` | 85.56 MiB; Octakit the top 528 pages, octamax the bottom 64, the platform reserve the bottom 1,707 | measured |
| the top window `0x47fc7410..0x47fe0000` | 101,360 B; Octakit's boot-time stage; the engine task's sector bounce buffers land at `0x47fc8fe4..` at project load | port (PIO path); DMA-card path unexercised |
| the delay rings `0x47502c10..0x47fc7410` | 10.8 MB, cleared at boot through the alias; not free | static + port + hardware |
| `0x46025de0..0x4763d580` | stock's globals and object pool; not free | static |

## Next

1. **Flash main** as the next image on a fresh project (or a stamped one)
   and hear the 23–27 Sep changes: the host pages and TEMPO BUS, DRV,
   the limited wet, the ramps. Note what is heard in `CHANGELOG.md`.
2. Tag that build (`OCTABAM<N>`), close the `Unreleased` section, `v0.1.0`.
3. **The T1 frame bursts** (`docs/remixer/FAILURE_MODES.md` "Bursts of
   garbage on the reverb host's frame", 🔴 cause open): 23–53 samples of
   near-full-scale junk in T1's stereo block, ~3.5 per six minutes, only
   while the delay's DSP code runs past its preamble on T1; not the
   delay's audio output, not any shared-window access, not the ColdFire
   side. Open: where between T1's block after proc and the read-back
   words at `X:0x2600` it appears. Ten-minute takes only.
4. **The settings store**: one shared OTX store per project for every
   module's settings (`docs/proposals/OTX_PROJECT_PROPOSAL.md`, nordseele,
   draft 2; `docs/remixer/MODULES.md` "Settings on the card"). Not
   implemented.

## Open, not scheduled

- The Kit write protocol: midisc's Part save/reload hooks against Octakit's
  LOAD/SAVE KIT menus are unmeasured (`modules/octakit/README.md`).
- Tell Em what the port saw at `0x47fc8fe4` (`PLACEMENT.md`, the top window).
- Measure `0x46000000..0x47502c10` with samples loaded and the recorder
  running before anyone places there.
- Under the port the transport start re-applies the saved bank's FX ids for
  T1-T3, T7 and T8 only; `verify_set` stages the tested bank as bank A.
  Cause open.
- octamax (mxldyn): ported on branch `octamax-deferred` (`d952976`), parked
  pending a conversation with the author.
- PR #468 (bryantysinger, USB AUDIO OUT: host audio into inputs A-D), draft.
- Each module's README, `## Open`.

## Gates and rules

- `make check REMIX=<name>` is the floor for every remix touched.
- A change to the build proves it changed nothing: `scripts/refhash.sh save`
  on a tree you trust, then `scripts/refhash.sh check`.
- The author's build is the oracle: `pinned`, `reference(addr)`,
  `Linked.reference` and a `Runtime` recipe's identities are four forms of
  one rule.
- Measured beats inferred, and says which it is (markers as in
  `docs/firmware/CHIP.md`; a retraction propagates to every document that
  repeated the number).
- Never an Elektron byte in the repo (`CONTRIBUTING.md` for what that
  covers); `.incbin` from the user's stock image at build time.

```sh
make modules                     # the index, the compatibility matrix, the remixes
make image REMIX=bamsep26 BUILD=N    # a card-flashable image, version-stamped
make check REMIX=bamsep26        # everything that can be checked without hardware
make remix                       # the TUI remixer
scripts/refhash.sh check         # after a change to the build itself
```
