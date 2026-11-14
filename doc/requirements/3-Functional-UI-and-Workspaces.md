# Functional requirements: UI and workspaces

Status: approved. This document specifies the user interface requirements across ToucanTL's three PySide6 windows: the main control window, the translation lens widget, and the region selector. Wayland windowing details are covered in [4-Client and screen capture.md](../4-Client%20and%20screen%20capture.md).

---

## 1. First-run setup wizard (`UI-WIZ`)

- **UI-WIZ-1 (Terminal-free onboarding):** On first launch (or when configuration is missing), the application must present a setup wizard that brings the user to a working state without requiring command-line flags, configuration file edits, or game-specific hooks.

- **UI-WIZ-2 (Wizard configuration steps):** The setup wizard must guide the user through selecting:
  - UI display language (English in v1, with architecture ready for Spanish, French, Japanese, and Mandarin Chinese).
  - Source OCR language and target translation language (filtered by engine capabilities).
  - Visual theme preset (plain light, dark, or high-contrast theme).
  - Translation provider and only the setup inputs required by that provider (such as selecting/downloading a local model or entering a cloud API key).

- **UI-WIZ-3 (Safe cancellation and re-entry):** The user must be able to go back between wizard steps or re-run the setup flow later from the main configuration window.

---

## 2. Main control window (`UI-MAIN`)

- **UI-MAIN-1 (Quick configuration access):** The main window must let the user change capture source, crop region, source/target languages, active translation provider, and theme within a few clicks or keystrokes without restarting the application.

- **UI-MAIN-2 (Session and context controls):** The main window must provide controls to start or stop capture, pause or resume translation, trigger a single manual capture, and inspect, edit, clear (rolling dialogue only or all context), or disable session context.

- **UI-MAIN-3 (Source and translation inspection):** The user must be able to view both the recognized source OCR text and the resulting translation for the current line so OCR misreads can be distinguished from translation errors.

- **UI-MAIN-4 (Clear status and error reporting):** The UI must clearly indicate the current session state (idle, waiting for portal permission, capturing, translating, paused, or error) and explain permission denials, stream disconnects, offline failures, rate limits, or insufficient memory in plain language.

---

## 3. Translation lens widget (`UI-LENS`)

- **UI-LENS-1 (Resizable translucent display):** The translation lens must be a minimal, resizable PySide6 window that displays translated text with configurable font size, background opacity, and plain high-contrast text styling. The user may optionally toggle showing the recognized source OCR text above the translation inside the lens.

- **UI-LENS-2 (Universal normal-window baseline):** The lens must support running as a standard focusable desktop window on the primary monitor or docked on a second monitor on every Wayland compositor.

- **UI-LENS-3 (Compositor overlay and click-through modes):** On Wayland compositors that support `layer-shell-qt` and input region pass-through (`WA_TransparentForMouseEvents`), the user may enable always-on-top overlay mode and click-through mode so mouse clicks reach the underlying game or reader.

- **UI-LENS-4 (Graceful fallback and shortcut recovery):**
  - If the active compositor does not support always-on-top overlays or click-through, the lens must fall back to a normal window and inform the user of the compositor limitation.
  - When click-through is active, the user must be able to toggle click-through off or reposition/close the lens from the main control window so the lens cannot become permanently unreachable.

---

## 4. Region selector (`UI-REG`)

- **UI-REG-1 (Frame-aligned region selection):** The region selector must allow the user to draw or adjust a rectangular crop area mapped accurately to the captured PipeWire stream for both single-window and full-monitor capture, or select the full frame with one action.

- **UI-REG-2 (Keyboard and pointer operation):** The user must be able to confirm, cancel (via `Escape`), or adjust the crop selection using either the pointer or keyboard controls.

---

## 5. Cloud disclosures and usage display (`UI-PRV`)

- **UI-PRV-1 (Cloud boundary disclosure):** Before sending the first request to a cloud provider, and whenever switching to a different cloud provider, the UI must clearly inform the user which provider is active and that recognized OCR text and enabled context (never screenshots) will be sent to that service.

- **UI-PRV-2 (Honest usage reporting):** The UI must display provider usage based on the active adapter's capability: exact token or character counts when reported by the provider, clearly marked estimates when calculated locally, or an explicit "usage unavailable" label when the engine does not report usage.
