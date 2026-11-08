# The tvOS AVFoundation audio build (branch `tvos-avfoundation-audio`)

This branch is `0.40.0-everything` (mpv v0.40.0, FFmpeg n8.0, LGPL) with one change: its Libmpv is rebuilt for tvOS
with mpv's `avfoundation` audio output (AVSampleBufferAudioRenderer), next to `audiounit`.

**Why:** on an Apple TV with Continuous Audio Connection on (tvOS 26.4 and later, on by default when the Apple TV is
connected to a Dolby-capable receiver, soundbar or TV), mpv's AudioUnit output opens and plays nothing. Video plays
and there is no sound ([mpvkit/MPVKit#67](https://github.com/mpvkit/MPVKit/issues/67)). The patch that lets the
avfoundation output build for iOS and tvOS is upstream's
[mpvkit/MPVKit#73](https://github.com/mpvkit/MPVKit/pull/73), here as
`Sources/BuildScripts/patch/libmpv/0004-enable-avfoundation-ao-tvos.patch`. It applies cleanly to v0.40.0. An app
picks the output with `ao=avfoundation,audiounit`.

**The audio buffer leak (release `0.40.0-tvos-avfoundation-2`, 2026-11-08):** mpv 0.40.0's avfoundation output
allocates a buffer each time the renderer asks for samples, and when mpv has none ready (an underrun) it returned without
freeing it. Upstream fixed that after 0.40.0 (mpv 59d5f774a); `0005-avfoundation-feed-no-leak.patch` carries the fix and
also gives the renderer nothing, instead of copying into a null pointer, if the allocation fails. An Apple TV crashed
there during Live TV (EXC_BAD_ACCESS at 0x0 in `read_buffer` under the output's feed block), most likely once the leak had
used up the allocator. The first release, `0.40.0-tvos-avfoundation`, has the leak.

**What changed against 0.40.0-everything:**
- `Package.swift`: only `Libmpv` points somewhere new, at this repo's release `0.40.0-tvos-avfoundation`. Every other
  binary is 0.40.0-everything's.
- In that release's `Libmpv.xcframework`, only the two tvOS slices (`tvos-arm64_arm64e`,
  `tvos-arm64_x86_64-simulator`) are new. The iOS, macOS, Mac Catalyst and visionOS slices are 0.40.0-everything's,
  unchanged, so they still have no avfoundation output on iOS.
- mpv's enabled features are exactly 0.40.0-everything's plus `avfoundation`.
- The build script (`main.swift`) unpacks 0.40.0-everything's own `FFmpeg-all.zip` instead of building FFmpeg. Libmpv
  is a static archive, so it compiles against exactly those headers and links against exactly those binaries in the
  app.

**Rebuilding:**
1. `make build platform=tvos,tvsimulator`. It needs meson 1.4.2, ninja and pkg-config.
2. On a Mac without Homebrew, put shims for `brew` (a no-op), `wget` (curl) and `pkg-config` (pkgconf) first on
   `BaseBuild.defaultPath`.
3. Give pkg-config `.pc` files for the SDK's zlib, bzip2, expat, libxml-2.0 and iconv, which Homebrew's pkg-config
   would supply.
4. Put the new `dist/release/xcframework/Libmpv.xcframework` tvOS slices into 0.40.0-everything's
   `Libmpv.xcframework`.
5. Zip it with `zip -qry`, then take the checksum with `swift package compute-checksum`.
