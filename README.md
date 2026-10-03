# UtooTV — Your TV. Your Way.

IPTV media player for Android TV / Google TV. **Personal build** for Neil's
household — a media player only. Provides **no content**: bring your own
playlist (Xtream / M3U) from your own provider.

- **Web core:** single-file HTML app (all assets/fonts/hls.js/mpegts.js
  embedded, zero external app-host dependencies). First-run setup asks for
  your own Xtream portal URL + credentials; credentials are encrypted at
  rest (AES-GCM, IndexedDB) on-device and never bundled or exported.
- **APK:** thin WebView wrapper (original code) embedding the HTML bundle;
  native D-pad → keyboard mapping (arrows/OK/Back=Escape/Info=i), leanback
  launcher, immersive fullscreen. Min SDK 21, target 34.
- **Feature set:** EPG guide (translucent, inline focused row), favorites,
  search, settings + config export/import (JSON), Multi-view 2×2 (4
  simultaneous HLS streams, single-audio), bottom quick strip (Guide /
  History / recent channels).

## Releases
| Version | APK | SHA256 |
|---|---|---|
| 1.0.0 | UtooTV-1.0.0.apk | `93ef8aa9ad480b1c283b88db47303eb4c070aaced051d2e07e4751bed4deb51c` |

## Install (Android TV)
Settings → Network & Internet → enable *Network debugging* →
`adb install UtooTV-1.0.0.apk`, or sideload via Downloader using the
release asset URL.

## Compliance line
UtooTV is a media player only and provides no content. It is not affiliated
with any content provider and does not support unauthorized streaming. Any
paid upgrade unlocks app features only.

Owner: Neil (neilestate). Keystore never leaves the build machine.