# The desktop client and screen capture

Building a real-time translation lens on Linux with Wayland introduces two major client-side challenges: rendering lightweight, accessible, translucent windows without eating system memory, and capturing frames from games or manga readers inside Wayland's strict security model.

---

## Why PySide6 instead of web wrappers

Web technologies make layout and styling convenient, which makes tools like pywebview (backed by WebKitGTK on Linux) tempting when avoiding Electron. However, web wrappers fall short when building tiny desktop overlay widgets for three main reasons:

- **Multi-window RAM overhead:** Every pywebview window spawns a separate WebKitWebProcess that consumes roughly 80 to 180 MB of memory. Opening a main configuration window, a tiny translation widget, and a region selector multiplies this cost right next to a running game, PaddleOCR, and a local GGUF model.

- **Wayland transparency glitches:** Hardware-accelerated rendering (DMABUF) in WebKitGTK frequently breaks alpha channels under Wayland, turning transparent backgrounds solid black or white unless software rendering fallbacks are forced.

- **No input pass-through or surface control:** Web wrappers only expose standard desktop windows. They cannot easily let mouse clicks pass straight through a floating dialog box into the game beneath it.

Because of these constraints, **PySide6 (Qt6)** is the ideal choice for our View layer:

+ Spawning three or four small QWidget windows inside a single Python process adds almost zero extra RAM, keeping the entire UI footprint around 35 to 55 MB.
+ Qt supports true per-pixel window transparency (using FramelessWindowHint and WA_TranslucentBackground), allowing an invisible window frame with a semi-transparent dialog box and crisp, 100% opaque text on top.
+ Adding the WA_TransparentForMouseEvents attribute enables click-through overlays so visual novel players can click directly through the translated text without losing focus on the game.
+ Qt Style Sheets (QSS) work just like plain CSS, making it trivial to build flat, distraction-free color themes without animations and implement global UI zooming by scaling root font sizes.
+ Qt6 integrates natively with the Linux AT-SPI2 accessibility bus for keyboard navigation, screen readers, and clean CJK (Japanese and Chinese) font fallbacks.

*(Note: Tkinter and CustomTkinter were ruled out due to poor Wayland scaling and whole-window alpha that fades text along with the background, while DearPyGui lacks native WCAG accessibility support).*

---

## Window layout and Wayland Always-on-Top behavior

Following our modular guidelines, each window and reusable UI widget lives in its own small class file inside the view folder. The client revolves around three lightweight window roles:

1. **Main control window:** Houses the first-time setup wizard, translation mode toggles, language pickers, theme and zoom settings, and API token usage counters.

2. **Translation lens widget:** A minimal, resizable, frameless translucent box that displays translated text either over the game or docked cleanly on a second monitor.

3. **Region selector:** Allows the user to crop capture down to a specific dialog box or manga speech area instead of scanning the entire screen.

On Wayland, normal native windows (xdg-shell) are intentionally forbidden by the compositor from forcing themselves Always on Top or moving to arbitrary global coordinates. We handle this cleanly depending on how the user runs the tool:

- **Second monitor mode:** Standard native Wayland windows work out of the box with zero workarounds.

- **Overlay on wlroots and KDE Plasma:** Compositors such as Sway, Hyprland, and KDE KWin support the Wayland layer-shell protocol, which allows true overlay surfaces that stay above fullscreen games (accessible via layer-shell-qt, or GTK4 with gtk4-layer-shell as a secondary toolkit option).

- **Universal XWayland fallback:** On strict compositors like GNOME Mutter that reject the layer-shell protocol, launching the overlay under XWayland (setting the Qt platform backend to xcb) restores traditional Always on Top window hints across every Linux desktop environment.

---

## Screen and window capture on Wayland

Wayland isolates client windows from one another, meaning legacy X11 screenshot libraries cannot read pixels from other programs. Instead of writing compositor-specific hacks from scratch, we rely on the standard Linux desktop capture stack behind our capture_controller:

- **xdg-desktop-portal (ScreenCast):** We communicate with the desktop portal over D-Bus to request screen or window sharing. As a bonus, the operating system provides a native, secure window and monitor picker dialog automatically across GNOME, KDE, and wlroots desktops.

- **PipeWire and GStreamer:** Once the user approves the target window or screen, the portal returns a PipeWire video stream node. Reading frames from PipeWire via GStreamer lets us pull snapshots directly in memory without slow disk writes.

- **In-memory cropping and debouncing:** Before touching OCR, the controller crops the PipeWire frame to the user's selected region and runs a fast image-difference debounce check. If the dialog box on screen has not changed, the frame is dropped immediately to save CPU, GPU, and cloud tokens.

- **Decoupled capture backend:** Keeping the capture stream behind a clean adapter class allows us to swap in lightweight snapshot tools (such as grim and slurp on minimal Wayland setups) for testing without changing a single line of OCR or UI code.
