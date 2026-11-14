# Software requirements overview and scope

Status: approved. This directory contains the Software Requirements Specification (SRS) for Toucan Translation Lens (ToucanTL). It defines what the application must do and the constraints it must meet, serving as the single source of truth for functional and non-functional requirements.

For project background and motivation, see [1-About.md](../1-About.md). For implementation structure and verification, see [3-Architecture.md](../3-Architecture.md), [4-Client and screen capture.md](../4-Client%20and%20screen%20capture.md), [5-Development plan.md](../5-Development%20plan.md), [6-Testing strategy.md](../6-Testing%20strategy.md), and [7-Threat model.md](../7-Threat%20model.md).

---

## 1. Purpose

ToucanTL is a Linux desktop application that captures on-screen text from visual novels, RPGs, and manga, recognizes the text locally using Optical Character Recognition (OCR), translates it through local or cloud translation engines, and displays the translated result with low latency.

The system is designed around three core principles:

- Local-first privacy and efficiency: screen capture, cropping, frame filtering, and OCR always run locally in memory.

- Pluggable engines: OCR, traditional machine translation (MT), local small language models (LLMs), and cloud APIs sit behind capability-based adapters rather than hard-coding the pipeline to a single vendor.

- Predictable session behavior: translation requests stay ordered and bounded, stale responses are rejected, and conversation context remains optional, session-scoped, and under user control.

---

## 2. Operating environment and actors

ToucanTL targets Linux desktop environments running Wayland.

- **Primary actor (User):** Configures languages and providers, selects a monitor or window and crop region, reads translated dialogue or manga panels, manages optional session context, and controls capture state.

- **Wayland compositor and desktop portal:** Provides screen and window streams through `xdg-desktop-portal` (ScreenCast) over D-Bus and delivers video frames via PipeWire. Window stacking and overlay behavior depend on the active compositor (`xdg-shell`, `layer-shell-qt`, or XWayland fallback).

- **Accessibility bus (AT-SPI2):** Receives widget labels, focus state, and completed translation announcements for screen readers.

- **Local engines:** Local OCR (`PaddleOCR` in v1) and local translation engines (`llama.cpp` for GGUF models and `Argos Translate` as the local MT candidate) running on the user's CPU or GPU.

- **Cloud translation endpoints:** Optional remote APIs (such as Gemini, DeepSeek, or cloud MT services) accessed using the user's own API keys stored in the OS keyring.

---

## 3. Scope boundaries

### In scope for v1

- Desktop client for Linux on Wayland built with Python and PySide6 (Qt6).

- First-run setup wizard and fast configuration interface in English, structured so additional UI translations can be added independently.

- Screen and window capture via `xdg-desktop-portal` and PipeWire, with in-memory cropping, image-difference filtering, local OCR, and text-stability filtering.

- Ordered translation sessions supporting local LLM, local traditional MT, and cloud providers using user-supplied API keys.

- Bounded, inspectable, session-scoped context and user-confirmed glossary/character notes for context-capable providers.

- Normal translation window and second-monitor placement as the universal baseline, plus translucent overlay and click-through modes on compositors that support them.

- Desktop accessibility features inspired by WCAG 2 (keyboard navigation, AT-SPI2 screen-reader support, UI zoom, and plain high-contrast themes).

### Post-v1 extensions and nice-to-haves

- UI localization into Spanish, French, Japanese, and Mandarin Chinese.

- Automatic source-language detection from captured frames.

- Multi-region capture per frame (such as selecting multiple manga speech bubbles simultaneously) and alternative text input sources (such as document/PDF parsing or text hooks).

- Flatpak packaging and user-defined custom color palettes beyond the built-in plain themes.

### Explicitly out of scope

- Clients for non-Linux platforms or legacy X11-only capture pipelines.

- Cloud-based OCR that uploads user screenshots to remote servers.

- Built-in text-to-speech synthesis (handled by external software or screen readers via AT-SPI2).

- Unbounded background queues or autonomous model modifications to persistent context.

---

## 4. Domain glossary

- **Capture source:** The monitor or application window authorized by the user through `xdg-desktop-portal` and streamed via PipeWire.

- **Crop region:** A rectangle defined in normalized coordinates relative to the captured video frame that restricts which area is passed to OCR.

- **Capture moment:** An in-memory cropped image frame paired with its capture timestamp and frame metadata.

- **Stable text:** Recognized OCR text that has differed from the previous translated text and remained unchanged across the stability window, ready for translation.

- **Translation session:** The active stateful coordination boundary that assigns sequence IDs, tracks bounded pending requests, manages optional session context, and isolates provider state.

- **Context store:** The bounded set of recent session dialogue lines and user-confirmed durable facts (such as character names or setting notes) supplied to context-capable providers.

- **Translation lens:** The minimal window or translucent overlay widget that displays translated text to the user during reading or gameplay.

---

## 5. Specification map

- [1-Functional-Capture-and-OCR.md](1-Functional-Capture-and-OCR.md): Screen and window acquisition, crop regions, self-OCR prevention, image diffing, OCR execution, and text stability (`CAP-*`, `OCR-*`).

- [2-Functional-Translation-and-Context.md](2-Functional-Translation-and-Context.md): Session ordering, queue bounds, provider capability adapters, local model lifecycle, and context management (`TRN-*`, `PRV-*`, `CTX-*`).

- [3-Functional-UI-and-Workspaces.md](3-Functional-UI-and-Workspaces.md): First-run wizard, main control window, translation lens modes, region selector, and status/usage reporting (`UI-*`).

- [4-Non-Functional-Requirements.md](4-Non-Functional-Requirements.md): Accessibility, latency and memory budgets, security and privacy, reliability, and packaging (`NFR-*`).
