MILESTONE 38g v2 — "TRUE transparent guide (TiviMate-identical) + VOD 7-across + text density" (bundle edit; wrapper untouched)

CONTEXT — Neil's RC43 on-box verdict, CORRECTED diagnosis (photos Oct 7):

FACTS FROM THE RC43 ON-BOX PHOTO (disambiguated):
- Neil's verdict stands: "it's not transparent" — the guide reads as an opaque solid navy grid.
- The ONLY video artifacts visible in his photo: the ACCN/ESPN broadcast bug + clipped ticker at
  the BOTTOM SCREEN EDGE (below/outside the guide panel's bounds). The panel ITSELF — grid,
  cells, columns, everything inside the guide's rectangle — is opaque: zero video content shows
  through anywhere inside it.
- The bug/ticker visible below the panel = video showing in the region the guide does NOT cover
  (the native video is still playing and compositing at the screen's bottom edge OUTSIDE the
  guide's opaque surface). Video-behind-UI compositing WORKS on the device (proven — the video
  texture renders through/below where no app layer blocks it).
- Conclusion: the guide panel root and/or its layers are drawn OPAQUE on-box. Browser-based rgba
  tests pass only because their video is an HTML <video> — on TV the sink is a NATIVE TextureView,
  and opaque layer(s) INSIDE the guide panel root cover it. The guide's "rgba background" is
  being defeated on-device.
- TARGET = TiviMate (his photo-2 reference): full-bleed video visible THROUGH the guide — header
  strip, channel column, cells, info panel — under a light smoked-glass scrim. Video, captions,
  channel bug, ad logos read plainly through the whole guide rectangle.

== CHANGE 1 (THE FIX — root-cause the on-device opaque panel) ==
Find and remove what makes the guide rectangle opaque ON THE TV. Work through the stack
systematically and report what it was:
1. The guide panel root (.m38b-tguide / .tv-guide-layer / .tv-stage in tguide state): any
   solid/gradient background (linear-gradient indigo, #0d1126 hex, etc.) on the root or grid
   wrapper must go — replace with rgba(13,17,38,.55) or lighter.
2. ANCESTOR layers: #overlay, #app, body, html and shell wrappers still carrying opaque
   backgrounds in tguide state — set them transparent. On TV, TextureView composites BEHIND
   WebView; ANY opaque ancestor of the WebView's visible content, OR the WebView surface itself
   if sized as an opaque layer above the TextureView, hides the video. Two proven mechanisms
   exist in this app's OWN history: the M-keys deep-black toggle era and the OLD app showed
   native video behind translucent guide panels — locate how those cleared the panel
   ("setLayerType(View.LAYER_TYPE_HARDWARE)"? surface z-order? WebView background transparent
   via set setBackgroundColor(0x00000000)/setBackgroundColor(Color.TRANSPARENT)?). IF the WebView
   itself needs a wrapper change to composite natively (WebView.setBackgroundColor(transparent)
   + setLayerType(LAYER_TYPE_HARDWARE) so HTML alpha blends over the TextureView), that wrapper
   edit is AUTHORIZED — smallest possible change, keystore-safe build through Logan's pipeline;
   bundle + wrapper change coordinated in this milestone (bridge file stays byte-identical; only
   MainActivity WebView setup smali if strictly required).
3. Per-cell fills: current cells are light-ish opaque blue — on-box they fully block video.
   Target: cell fill rgba(255,255,255,.06-.10) so the video shows through cells themselves,
   white bold text on top; hairline borders rgba(255,255,255,.12).
4. Channel rail + header strip: same near-transparent treatment (video bug visible through the
   rail — target photo 2 shows the Paramount bug showing through the rail).
5. Expanded detail panel: translucent dark (video visible through it) EXCEPT the focus pill cell
   (≈0.9 light gray + dark text by design) and synopsis panel background ≈ rgba(13,17,38,.75).
6. The dim/scrim layer used elsewhere must NOT stack an opaque dark layer on top in tguide state —
   audit and remove/relax it.
ACCEPTANCE TEST ON-DEVICE: with the guide open on the TV, Neil must clearly see the playing
video (people/captions/watermark) through the ENTIRE guide rectangle — header, rail, cells,
info panel (same as his TiviMate photo-2). Strip line "TGUIDE native compositing: <status>".
Neil is the acceptance authority for this item.

== CHANGE 2 — VOD 7-across ==
Carousel viewport / 7 = card width for EVERY row in Movies AND Shows (posters keep aspect;
rating badge + readable title at the smaller size; row labels unchanged).

== CHANGE 3 — VOD text density ==
SIDEBAR: row text ~half current size; every label fits its row in ONE line — no mid-word
clipping ever (long names ellipsize with "…"); counts right-aligned with a real gap ("All movies
115" never "All movies115"); focus box wraps the full item; ~10-14 rows visible at 1080p.
HERO: cast = first 4-5 names + … (ONE line); director one line; synopsis max 3 lines ellipsized;
DETAILS state keeps full cast/synopsis (internally scrollable).

== UNCHANGED / DO NOT REGRESS ==
All VOD wiring (routes, counts, provider order, no-fabrication, play funnel + banner), drawer
slide-geometry (28vw beside the grid), expanded-row behavior on scroll, banner-on-tune/zap 5s
rules, OK info overlay + recent tiles, zap, last-channel toggle, short-Back stairs, long-Back
exit, History/Previous lists, IME/login, 15s funnel, alt-cycling, diagnostics default-off.
Strip: "TGUIDE native compositing: <status>", "VOD density applied"; keep prior lines.

== ACCEPTANCE ==
A1 (DEVICE-CRITICAL): guide rectangle shows the playing video through header/rail/cells/info
    panel on TV (TiviMate-equivalent, photo-2); NO opaque region inside the guide rectangle.
    Root cause + fix documented in CHANGE-REPORT (name the exact layer/mechanism defeated).
A2: per-cell/video-through-cells style values per recipe (computed-style asserts + screenshots
    on a native-sink-like background; on-box pass = Neil).
A3: all carousel rows (movies + shows) = exactly 7 cards across at 1920; scroll continues.
A4: sidebar fits/count-gap/one-line items (~10-14 visible); no mid-word clip.
A5: hero: cast ≤5 names one line, synopsis ≤3 lines; details full.
A6: Everything else byte-identical where untouched (integrity tests: VOD wiring, drawer slide,
    expanded row, banner, OK overlay, zap, toggle, stairs, exit, lists, IME, funnel, diagnostics).
A7: node --check all blocks; 7000-channel + carousel windowing + banner timing tests pass.
A8: If a wrapper smali change was required (WebView transparency), document exactly what changed
    in CHANGE-REPORT + attach the smali diff. Bundle-only delivery otherwise.

DELIVERY: zip with index.html (+ smali diff if strictly required) + CHANGE-REPORT.md (name the
defeated opaque layer; wrap/HTML compositing mechanism; density math) + jsdom tests A1-A8 +
TEST-RESULTS.txt + 1080p screenshots (7-across rows x2, compact sidebar, capped hero,
guide-over-video fills). Logan builds RC44 (incl. smali if required) and runs the full gate chain.