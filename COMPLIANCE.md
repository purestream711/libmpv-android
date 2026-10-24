# LGPL Compliance — NostalgiaTV build of libmpv-android

This branch (`lgpl-1.0.0`) is a fork of jarnedemeulemeester/libmpv-android
at tag v1.0.0, modified ONLY to produce an LGPL-licensed native stack,
plus a CI trigger line for this branch, a fork-distinct artifact version,
and one upstream bug-fix cherry-pick.

## What was changed

Four patch sets, all in the build configuration:

1. **FFmpeg 8.1 built LGPL-only** — `--enable-{gpl,version3}` reduced to
   `--enable-version3`; `--enable-nonfree` is never used. FFmpeg is
   therefore licensed LGPL v3 (`--enable-version3` is retained because
   Mbed TLS is Apache-2.0). The `libpostproc.so` symlink is dropped —
   libpostproc is GPL-only and is neither built into the AAR nor shipped.
2. **Filter allow-list** — upstream disables all filters
   (`--disable-filters`). Six are re-enabled, all LGPL-clean in FFmpeg
   8.1: `dynaudnorm` (audio normalization) with `aresample`, `aformat`
   and `anull` (the audio-filter plumbing it needs), plus `bwdif` and
   `yadif` (deinterlacing). `vf_bwdif.c` and `vf_yadif.c` both carry
   LGPL-2.1+ headers; yadif was relicensed GPL→LGPL in 2014 (FFmpeg
   commit `194ef56b`) and bwdif was LGPL from the start. Neither appears
   in FFmpeg's `LICENSE.md` GPL enumeration nor among configure's
   GPL-gated filter dependencies.
   Since v1.0.1-lgpl the list also carries `scale`, `format` and `null`:
   the converters libavfilter inserts on its own when a filter does not
   take the incoming pixel format (`bwdif` / `yadif` take no NV12, which
   is what MediaCodec copy-back decoding produces), so deinterlacing works
   on those frames too. All three are LGPL-2.1+ (`vf_scale.c`,
   `vf_format.c`, `vf_null.c`) and not GPL-gated.
3. **mpv 0.41.0 built with `-Dgpl=false`** → LGPL v2.1 or later.
4. **One muxer** (since v1.0.1-lgpl) — upstream disables all muxers
   (`--disable-muxers`); `spdif` is re-enabled. mpv's audio passthrough
   (`--audio-spdif`) wraps AC-3 / E-AC-3 / DTS / TrueHD in IEC 61937
   frames with it; without it passthrough produces no audio.
   `libavformat/spdifenc.c` is LGPL-2.1+ and not GPL-gated.

Also on this branch, not license-related:

- `.github/workflows/build.yaml`: one line adding `lgpl-1.0.0` to the
  push triggers.
- `.github/workflows/publish.yaml`: deleted. It fires on any `v*` tag and
  publishes to Maven Central, for which this fork has no credentials and
  wants none; leaving it would make every release tag a failed run.
- `libmpv/build.gradle.kts`: artifact version `1.0.0` → `1.0.1-lgpl`
  (`1.0.0-lgpl` before the muxer / converter additions), so a fork build
  can never be mistaken for an upstream artifact.
- `libmpv/src/main/java/dev/jdtech/mpv/MPVLib.kt`: upstream commit
  `bf5e0d80` ("fix: make `destroy` idempotent") cherry-picked verbatim
  from upstream `main`. At the v1.0.0 tag a second `destroy()` call
  throws; upstream fixed that after tagging.

`changes.diff` in this directory is the complete build-configuration diff
against upstream v1.0.0, generated with:

    git diff v1.0.0..lgpl-1.0.0 -- buildscripts .github libmpv/build.gradle.kts

The only changed file it does not cover is the `MPVLib.kt` cherry-pick
above, which is upstream's own commit applied unmodified. The branch
itself is the authoritative complete source.

## What is shipped

Exact dependency versions are pinned in `buildscripts/include/depinfo.sh`.
The AAR carries 10 shared objects per ABI, across four ABIs (arm64-v8a,
armeabi-v7a, x86, x86_64):

    libmpv.so       libplayer.so     libc++_shared.so
    libavcodec.so   libavdevice.so   libavfilter.so   libavformat.so
    libavutil.so    libswresample.so libswscale.so

The (L)GPL-licensed components are shipped as those separate,
dynamically loaded shared objects: **mpv 0.41.0** (LGPL v2.1+,
`libmpv.so`) and **FFmpeg 8.1** (LGPL v3, the `libav*` / `libsw*` set).
Each can be replaced and relinked by an end user — the LGPL §6 right this
fork exists to serve.

mpv's dependencies are statically linked into `libmpv.so`, and FFmpeg's
into the FFmpeg libraries:

| component | version | license | linkage |
|---|---|---|---|
| libplacebo | 7.360.1 | LGPL v2.1+ | static in `libmpv.so` |
| GNU FriBidi | 1.0.16 | LGPL v2.1+ | static in `libmpv.so` |
| libass | 0.17.4 | ISC | static in `libmpv.so` |
| FreeType | 2.14.3 | FTL | static in `libmpv.so` |
| HarfBuzz | 14.1.0 | MIT (Old) | static in `libmpv.so` |
| fontconfig | 2.17.1 | MIT-style (X11/HPND) | static in `libmpv.so` |
| libunibreak | 6.1 | zlib | static in `libmpv.so` |
| Lua | 5.2.4 | MIT | static in `libmpv.so` |
| libxml2 | 2.15.2 | MIT | static in FFmpeg libs |
| dav1d | 1.5.3 | BSD 2-Clause | static in FFmpeg libs |
| Mbed TLS | 3.6.6 | Apache-2.0 | static in FFmpeg libs |

The two statically linked LGPL components — libplacebo and FriBidi — are
covered by the same offer: complete source for the exact pinned versions
is provided, and `libmpv.so` can be rebuilt from it with the scripts in
this repository. That is what LGPL §6 requires.

### New dependencies at v1.0.0

Upstream v1.0.0 added three native dependencies the v0.5.1 build did not
carry: `dep_ffmpeg` gained `libxml2`, and `dep_libass` gained `fontconfig`
and `libunibreak`. None is copyleft, and none adds a shared object — the
`.so` count per ABI is unchanged at 10.

- **fontconfig 2.17.1** — MIT-style permissive license in the historical
  X11 / HPND form ("Permission to use, copy, modify, distribute, and sell
  this software and its documentation for any purpose is hereby granted
  without fee…"). Copyright Keith Packard, Patrick Lam, Red Hat Inc.,
  Google Inc. and other fontconfig contributors. Linked because libass
  links it; NostalgiaTV sets `sub-font-provider=none`, so it is present
  but unused at runtime.
- **libunibreak 6.1** — zlib License. Copyright Wu Yongwei, Tom Hacohen,
  Petr Filipsky, Andreas Röver.
- **libxml2 2.15.2** — MIT License. Copyright (C) 1998-2012 Daniel
  Veillard; Copyright (C) The Libxml2 Contributors.

## Rebuilding

Run the GitHub Actions workflow on this branch, or locally on Linux:

    cd buildscripts && ./download.sh && ./patch.sh && ./build.sh

`download.sh` fetches every dependency at the exact version pinned in
`include/depinfo.sh`, each from its canonical upstream.

## Source availability

Source archives for every (L)GPL-licensed component — plus the three new
permissive dependencies, attached for completeness rather than
obligation — at exactly the versions pinned by `depinfo.sh`, are attached
to the `v1.0.1-lgpl` GitHub Release (and, for the previous build, to
`v1.0.0-lgpl`) alongside the binary AAR they correspond to. The pinned
versions did not change between the two. This repository and that release together constitute the
LGPL §6 offer of corresponding source.
