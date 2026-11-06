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
