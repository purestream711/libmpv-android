# LGPL Compliance — NostalgiaTV build of libmpv-android

This branch (`lgpl`) is a fork of jarnedemeulemeester/libmpv-android at
tag v0.5.1, modified ONLY to produce an LGPL-licensed native stack (plus
a CI trigger line for this branch):

- FFmpeg 7.1.1 configured WITHOUT `--enable-gpl` (and never
  `--enable-nonfree`); `--enable-version3` retained (Mbed TLS is
  Apache-2.0), so FFmpeg is licensed LGPL v3. libpostproc is not built.
- mpv 0.40.0 built with `-Dgpl=false` → LGPL v2.1+.
- All libraries are built as separate shared objects
  (`--disable-static --enable-shared`) and are loaded dynamically at
  runtime — each `.so` in the APK can be replaced and relinked by end
  users.

The complete diff vs upstream v0.5.1 is `changes.diff`. Exact dependency
versions are pinned in `buildscripts/include/depinfo.sh`. To rebuild:
run the GitHub Actions workflow on this branch, or locally on Linux:
`cd buildscripts && ./download.sh && ./patch.sh && ./build.sh`.

Source tarballs for every (L)GPL-licensed component, exactly the
versions pinned by `depinfo.sh`, are attached to the `v0.5.1-lgpl`
GitHub Release alongside the binary AAR they correspond to.
