# `kyoti-mute-sidechain-bw` -- MUTE_MODES with SIDECHAIN_COMPRESSOR_BW

Two modules by Zac Kyoti and the stock effects, less SPRING REV.

## What is in it

- **MUTE_MODES** -- [`modules/mute-modes/README.md`](../../../modules/mute-modes/README.md). Beside SIDECHAIN_COMPRESSOR_BW it builds its SC_KEY variant: a muted KEY track keeps feeding the compressor.
- **SIDECHAIN_COMPRESSOR_BW** (Sidechain Compressor B-W) -- [`modules/sidechain-compressor-bw/README.md`](../../../modules/sidechain-compressor-bw/README.md).
- 13 of the 14 stock FX2 effects. SPRING REV gives up its words for SIDECHAIN_COMPRESSOR_BW's DSP section.

## Status

Each ColdFire unit's bytes are the author's: `reference` re-links them at the author's addresses every build and compares (MUTE_MODES' SC_KEY variant against the KYOTI V1.0 image). The author's KYOTI V1.6 image (MUTE_MODES with SC_KEY and SIDECHAIN_COMPRESSOR_BW, among others) ran on the author's MKI, 22 Oct 2026. This remix has not been flashed.

## Build

```bash
make image REMIX=kyoti-mute-sidechain-bw BUILD=1
```
