# Functional requirements: Capture and OCR

Status: approved. This document specifies the requirements for Wayland screen and window capture, crop region management, frame filtering, and local Optical Character Recognition (OCR). Architectural notes for the capture stack live in [4-Client and screen capture.md](../4-Client%20and%20screen%20capture.md).

---

## 1. Screen and window capture (`CAP`)

- **CAP-1 (Portal session acquisition):** The system must request screen or window capture exclusively through `xdg-desktop-portal` (`org.freedesktop.portal.ScreenCast`) over D-Bus and read the resulting video stream via PipeWire and GStreamer behind a capture adapter.

- **CAP-2 (Capture source types):** The system must support full-monitor capture on all supported Wayland desktops and single-window capture wherever the user's portal backend provides window selection.

- **CAP-3 (Portal cancel, denial, and retry):** If the user cancels or denies the portal permission dialog, or if the portal fails to start a stream, the system must remain responsive, display a clear explanation of the state, and allow the user to retry source selection without restarting the application.

- **CAP-4 (In-memory frame handling):** Captured frames and cropped snapshots must be held and processed in memory. The system must not write screenshots to disk during normal operation unless the user explicitly enables a temporary diagnostic capture mode.

- **CAP-5 (Frame-relative crop regions):** The user must be able to translate either the entire captured frame or a rectangular crop region. Crop coordinates must be stored relative to the captured stream dimensions and display scale rather than global desktop coordinates.

- **CAP-6 (Source resize and display scale changes):** When a captured window or monitor changes resolution or display scale, the capture adapter must keep the crop region aligned if the frame aspect ratio is preserved. If the aspect ratio changes or the crop bounds fall outside the new frame, the system must pause capture and prompt the user to confirm or re-select the region.

- **CAP-7 (Single active region in v1):** The initial release must support one active rectangular crop region per session. Selecting a new region replaces the previous region and invalidates any pending OCR or translation work from the old region.

- **CAP-8 (Self-OCR prevention):** The system must prevent feedback loops where the OCR engine reads ToucanTL's own output:
  - The region selector UI must hide before capture sampling resumes.
  - When using monitor capture with the translation lens on the same monitor, the system must warn the user where surface geometry overlap can be detected, suppress OCR results that match the text currently displayed in the lens (translation-echo suppression), and support placing the lens outside the crop region, on a second monitor, or using single-window capture so the lens is excluded from the stream.

- **CAP-9 (Pause, resume, and manual trigger):** The user must be able to pause and resume continuous capture at any time, as well as trigger an immediate single-frame capture and translation on demand. A manual single-frame trigger must bypass the image-difference gate (`OCR-1`), multi-frame stability window (`OCR-5`), and duplicate-line suppression (`OCR-6`).

- **CAP-10 (Stream interruption handling):** When a capture stream ends unexpectedly due to target window closure, monitor disconnection, portal permission revocation, or PipeWire disconnection, the system must immediately cancel pending OCR and translation tasks, transition the session to a paused/disconnected state, and notify the user with an option to reconnect.

---

## 2. Frame filtering and OCR (`OCR`)

- **OCR-1 (Image-difference gate):** Before invoking OCR, the pipeline must compare each cropped frame against the previously processed frame and drop unchanged frames immediately. Animated scenes outside the crop region must not trigger OCR evaluations.

- **OCR-2 (Decoupled OCR adapter):** Text recognition must run locally behind an OCR adapter interface that accepts an in-memory cropped image and source language setting, and returns recognized text along with confidence or layout metadata where available. `PaddleOCR` is the baseline adapter for v1.

- **OCR-3 (Source language and script support):** Each OCR adapter must advertise the source languages and text orientations it supports (including horizontal and vertical layouts for Japanese, Chinese, Korean, and Latin scripts). The UI must only allow selecting source languages supported by the active OCR engine.

- **OCR-4 (Empty and low-confidence frames):** Frames that yield empty text or fall below the adapter's minimum recognition threshold must be ignored without creating translation requests or clearing the currently displayed translation unless the user clears the lens.

- **OCR-5 (Text-stability window):** After OCR produces non-empty text during continuous capture, the pipeline must wait for the recognized text to remain stable across a short configurable stability interval (which may be set to zero for static manga or instant-text display) before emitting a translation request. Partially rendered typewriter dialogue must not trigger intermediate translations while characters are still appearing.

- **OCR-6 (Text deduplication and noise tolerance):** Once a stable line has been sent for translation, subsequent OCR results that match that line (after normalizing insignificant whitespace and minor OCR punctuation jitter) must not trigger duplicate translation requests while the same dialogue remains on screen.
