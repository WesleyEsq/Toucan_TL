# Architecture overview

This document summarizes how ToucanTL is structured. More detailed design notes can be added under `doc/general architecture/` when an individual component grows enough to need them.

---

## Directory layout and MVC split

The codebase uses an MVC split at the UI boundary, supported by small application services where ownership is clearer. The repository folders map out as follows:

- `assets/`: Images, SVGs, and static UI resources.
- `controller/`: Coordinates UI actions and application services, named like `[thing]_controller`. A top-level `Toucan_Controller` can wire components together at startup, while domain rules stay elsewhere so it does not become a god object.
- `view/`: Pages, windows, and PySide6 UI components only.
- `model/`: Data structures and domain rules for captures, recognized text, sessions, languages, and configuration.
- `util/`: General helper functions reused across modules (such as shared image cropping or file utilities), kept focused so unrelated code does not collect there.
- `router/`: Currently the folder for external API communication; this should be renamed to a clearer name such as `providers/` or `adapters/`.
- `database/`: Embedded persistence if configuration or session context requires it.

---

## Core classes and ownership

The core domain stays relatively small. Capture, OCR, and translation use their own adapters and controllers, while one translation-session service owns request ordering. That service assigns each request a session and sequence ID, bounds pending work, cancels obsolete requests where supported, and rejects stale responses. The UI observes this session instead of coordinating providers directly.

The model layer defines cohesive types such as `capture_moment`, `language`, `configuration`, and `translation_request`, alongside enums for fixed domain options such as supported languages. Group related types together logically instead of placing every small class in its own file.

In the view layer, PySide6 provides the main window for controlling and configuring the tool alongside small frameless `QWidget` windows for the translucent translation box and region selector.

---

## Pipeline and provider rules

- Image differences skip unchanged frames before OCR runs, while a short text-stability check waits for OCR output to settle before calling a translation engine. Separating these checks prevents animated backgrounds or partially typed dialogue from triggering unnecessary translations.
- Normal dialogue mode prioritizes the newest stable line. If requests fall behind, older queued frames can be dropped, and an older response can never overwrite a newer line on screen. Any future full-history mode must define its own explicit policy rather than using an unbounded queue.
- Changing the provider, language, capture region, or context invalidates affected pending work. Retries remain bounded so offline states or rate limits cannot keep growing the queue.
- OCR, capture, translation, key storage, and persistence sit behind adapters. Provider adapters advertise the capabilities they actually support—language pairs, context support, streaming, cancellation, and usage reporting—instead of forcing every local model or MT engine behind an OpenAI-shaped interface. Usage displays whatever a provider supplies (tokens, characters, estimates, or unavailable).
- One explicit lifecycle owner manages loading, unloading, downloads, CPU fallback, and out-of-memory errors for local models, measuring OCR and LLM memory usage together.
- If a translation cache is added later, cache keys must include source text, target language, provider, model, prompt version, and relevant context.

---

## Technology stack

For the initial release, PaddleOCR handles local text recognition and `llama.cpp` runs small local GGUF models. Traditional machine translation remains an equal option alongside LLMs, with Argos Translate as the main local candidate pending checks on language pairs, translation quality, packaging, and licenses.

For the UI, PySide6 (Qt6) avoids running separate browser processes alongside local OCR and language models, while providing shared-process translucent windows, QSS theming, and AT-SPI2 accessibility integration. Wayland capture uses `xdg-desktop-portal` (ScreenCast), PipeWire, and GStreamer behind a decoupled capture adapter. See [4-Client and screen capture.md](4-Client%20and%20screen%20capture.md) for how Wayland capture and overlays are handled, noting that GTK4 with `gtk4-layer-shell` is only a fallback investigation for overlays and not a parallel client.

Persistence starts with the smallest embedded store that fits configuration and bounded session context, keeping a document database as an option if the stored structure benefits from it. Context rules are defined in [1-About.md](1-About.md), and data-protection rules are covered in [7-Threat model.md](7-Threat%20model.md). For networking, a direct HTTP client that reuses connections is tested first; the optional gateway only stays if benchmarks show a clear benefit and its trust boundary is defined. API keys belong in OS-backed key storage where available rather than configuration files or logs.
