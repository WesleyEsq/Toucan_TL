# The desktop client and screen capture

Building a real-time translation lens on Linux with Wayland presents two main client-side problems: keeping transparent, accessible windows light on memory, and capturing frames from games or manga readers within Wayland's security model.

---

## Why PySide6

PySide6 (Qt6) fits the view layer for several practical reasons:

- Multiple small `QWidget` windows share a single Python process instead of requiring separate browser processes, though combined memory use still needs measurement alongside OCR and local models.
- Qt supports per-pixel transparency through `FramelessWindowHint` and `WA_TranslucentBackground`, depending on compositor behavior.
- Click-through behavior can use `WA_TransparentForMouseEvents`, which varies across Wayland compositors and must be tested on each supported environment with a normal focusable window fallback.
- Qt Style Sheets (QSS) handle flat themes and font scaling, paired with layout reflow and clipping checks when zooming.
- Qt6 integrates with the Linux AT-SPI2 accessibility bus, though custom widgets still need explicit accessible names, focus order, keyboard navigation, and screen-reader testing.

Tkinter and CustomTkinter were ruled out due to weak Wayland scaling and whole-window alpha transparency that fades text along with the background, while DearPyGui lacks native accessibility support.

---

## Window layout and Wayland always-on-top behavior

Following the modular guidelines, windows and genuinely reusable widgets live in small classes under `view/` without mixing in pipeline logic. The client uses three window roles:

1. **Main control window:** Houses the first-run setup wizard, translation mode toggles, language pickers, theme and zoom settings, and provider usage information when available.
2. **Translation lens widget:** A minimal, resizable, frameless translucent box that shows translated text either over the game or docked on a second monitor.
3. **Region selector:** Allows the user to crop capture to a specific dialog box or manga speech area instead of scanning the full screen.

On Wayland, normal native windows (`xdg-shell`) cannot force global screen coordinates or always-on-top stacking. Support is therefore organized by what each compositor allows:

- **Normal and second-monitor windows:** The baseline mode. Standard windows avoid most overlay limitations, though placement and fullscreen interaction still depend on the compositor.
- **Layer-shell overlays:** Supported wlroots compositors and KDE versions can expose overlay surfaces through `layer-shell-qt`. Stacking, fullscreen games, resizing, and input pass-through must be tested on each environment before listing it as supported. GTK4 with `gtk4-layer-shell` remains a backup investigation only if Qt cannot meet a required case.
- **XWayland fallback:** The Qt `xcb` backend can be tested as an alternative, though X11 always-on-top hints cannot reliably stack above native Wayland or fullscreen surfaces and do not provide a general GNOME solution.
- **Graceful fallback:** When overlay or click-through support is unavailable, translations stay in a normal window and the limitation is explained clearly to the user.

---

## Screen and window capture on Wayland

Wayland isolates client windows from one another, so legacy X11 screenshot libraries cannot read pixels from other programs. Instead of writing compositor-specific workarounds, `capture_controller` wraps the standard Linux desktop capture stack behind a clean adapter.

First, the adapter communicates with `xdg-desktop-portal` (ScreenCast) over D-Bus so the desktop displays its native permission picker. Monitor capture is the baseline everywhere, while window capture and session restore depend on the portal frontend. Cancelling, denying, or retrying must leave the application usable.

Once approved, the portal returns a PipeWire video stream node, and GStreamer reads frame snapshots directly in memory without disk writes. Before running OCR, the controller crops each frame to the selected region and performs an image-difference check so static frames are dropped immediately. Text stability and deduplication run separately afterward so animated backgrounds do not trigger repeated translations when dialogue stays the same.

Keeping the capture stream behind an adapter allows fakes and recorded fixtures in tests. Command-line utilities such as `grim` and `slurp` remain optional compositor-specific development aids rather than part of the portable portal path.

---

## Regions, interruptions, and self-OCR prevention

Because Wayland does not provide reliable global window coordinates, crop regions are stored relative to the captured video frame while accounting for frame dimensions and display scale. When the source window resizes, moves, rotates, or changes scale, the adapter transforms the crop region accurately or pauses and asks the user to select it again.

The stream owner handles target closure, monitor removal, permission revocation, PipeWire disconnection, and retry. Whenever a stream stops, pending OCR and translation work stops with it so background jobs do not keep running.

In monitor capture, the translation lens and region selector can appear inside the captured feed. Unless the capture path excludes them, the selected region must exclude the lens (or the lens must sit on a separate monitor or normal window), and the selector UI must hide before taking frames for OCR. Otherwise the application can read its own translated text and create a feedback loop.
