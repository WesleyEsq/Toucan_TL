# The desktop client and screen capture

Building a real-time translation lens on Linux with Wayland introduces two major client-side challenges: rendering lightweight, accessible, translucent windows without eating system memory, and capturing frames from games or manga readers inside Wayland's strict security model.

---

## Why PySide6 instead of web wrappers

Web technologies make layout and styling convenient, which makes tools like pywebview (backed by WebKitGTK on Linux) tempting when avoiding Electron. However, web wrappers fall short when building tiny desktop overlay widgets for three main reasons:

- **Multi-window RAM overhead:** pywebview windows can require separate WebKit web processes. Opening a main configuration window, a translation widget and a region selector can multiply this cost next to a running game, PaddleOCR and a local GGUF model. The earlier rough figures are only observations; compare actual process memory on the reference system.

- **Wayland transparency glitches:** Hardware-accelerated rendering (DMABUF) in WebKitGTK frequently breaks alpha channels under Wayland, turning transparent backgrounds solid black or white unless software rendering fallbacks are forced.

- **No input pass-through or surface control:** Web wrappers only expose standard desktop windows. They cannot easily let mouse clicks pass straight through a floating dialog box into the game beneath it.

Because of these constraints, **PySide6 (Qt6)** is the ideal choice for our View layer:

+ Several small QWidget windows can share one Python process instead of requiring separate browser processes. Their actual memory use still needs to be measured together with OCR and models.
+ Qt supports per-pixel window transparency using FramelessWindowHint and WA_TranslucentBackground, subject to compositor behavior.
+ WA_TransparentForMouseEvents is a candidate for click-through behavior, but it is not a universal Wayland guarantee. It must be tested per supported compositor, with a normal focusable window as fallback.
+ Qt Style Sheets (QSS) can provide flat themes and font scaling. Zoom also needs layout reflow and clipping checks; changing font size alone is not enough.
+ Qt6 integrates with the Linux AT-SPI2 accessibility bus. Custom widgets still need accessible names, focus order, keyboard operation and real screen-reader testing.

*(Note: Tkinter and CustomTkinter were ruled out due to poor Wayland scaling and whole-window alpha that fades text along with the background, while DearPyGui lacks native WCAG accessibility support).*

---

## Window layout and Wayland Always-on-Top behavior

Following our modular guidelines, windows and genuinely reusable widgets live in small classes inside the view folder without mixing in pipeline logic. The client revolves around three window roles:

1. **Main control window:** Houses the first-time setup wizard, translation mode toggles, language pickers, theme and zoom settings, and provider usage information where available.

2. **Translation lens widget:** A minimal, resizable, frameless translucent box that displays translated text either over the game or docked cleanly on a second monitor.

3. **Region selector:** Allows the user to crop capture down to a specific dialog box or manga speech area instead of scanning the entire screen.

On Wayland, normal native windows (xdg-shell) cannot freely force global coordinates or stacking. Support therefore needs to be described by capability rather than promising one universal overlay:

- **Normal and second-monitor windows:** This is the baseline path. Standard windows avoid most overlay problems, although placement and fullscreen interaction still depend on the compositor.

- **Layer-shell overlays:** Supported wlroots compositors and KDE versions can expose layer-shell through layer-shell-qt. Test stacking, fullscreen games, resizing and input pass-through on each environment before listing it as supported. GTK4 with gtk4-layer-shell remains an investigation only if Qt cannot meet a required case.

- **XWayland experiment:** The Qt xcb backend can be tested as a fallback, but X11 always-on-top hints do not guarantee stacking above native Wayland or fullscreen surfaces. It is not a universal GNOME solution.

- **Graceful fallback:** If overlay or click-through support is unavailable, keep the translation in a normal window and explain the limitation. Do not silently claim that the overlay is working.

---

## Screen and window capture on Wayland

Wayland isolates client windows from one another, meaning legacy X11 screenshot libraries cannot read pixels from other programs. Instead of writing compositor-specific hacks from scratch, we rely on the standard Linux desktop capture stack behind our capture_controller:

- **xdg-desktop-portal (ScreenCast):** We communicate with the desktop portal over D-Bus to request sharing. The desktop provides its picker, but available target types and restore behavior depend on the portal frontend. Monitor capture is the baseline; window capture is supported where the portal provides it. Cancel, deny and retry must leave the application usable.

- **PipeWire and GStreamer:** Once the user approves the target window or screen, the portal returns a PipeWire video stream node. Reading frames from PipeWire via GStreamer lets us pull snapshots directly in memory without slow disk writes.

- **In-memory cropping and debouncing:** Before touching OCR, the controller crops the PipeWire frame to the user's selected region and runs a fast image-difference check. Static frames are dropped, while OCR text stability and deduplication are handled separately so animated backgrounds do not cause repeated translations.

- **Decoupled capture backend:** Keeping the capture stream behind a clean adapter allows fakes and fixtures in tests. Tools such as grim and slurp are optional compositor-specific development aids, not replacements for the portable portal path.

## Regions, interruptions and the lens itself

A region is stored relative to the captured video frame, not as a guessed global Wayland coordinate. The capture adapter must account for the actual frame dimensions and display scale. When the source is resized, moved, rotated or rescaled, the application either transforms the region correctly or pauses and asks the user to select it again.

The stream owner handles target closure, monitor removal, permission revocation, PipeWire disconnection and retry. A stopped stream must not leave OCR or translation work running indefinitely.

The translation lens and region selector can also appear inside a monitor capture. If the capture path cannot exclude them, the selected region must exclude them or the lens must use another monitor or normal-window placement. Otherwise the application could OCR its own translation and create a feedback loop. Selector UI should be hidden or excluded before taking the frame used for OCR.
