# How should we implement this? In brief terms

This is a brief summary. More detailed documents can be added under `general architecture` when a component actually needs them. The code should be organized around MVC at the UI boundary, with small application services where that makes ownership clearer. The existing directories are the following:

- **Assets:** For storing images, SVG and other UI elements that are used in the software.

- **Controller:** Coordinates UI actions and application services, named like "[thing]_controller". It should not become a place for every rule in the application.
    + A single `Toucan_Controller` may compose the application at startup, but it must not become a god object that owns every implementation detail.

- **View:** This is where the pages and the UI code should be declared, and nothing else.

- **Model:** Contains the data and domain rules for captures, text, sessions, languages and configuration.

- **util:** A folder for general helper functions made by us that are reused over and over.

- **router:** Place where all external communications with API endpoints are done at, should be renamed.

- **database:** To contain embedded persistence if the context or configuration design needs it.

---

## What classes will be declared? (To do)

Probably not a lot of them, the core problem is kinda simple. Capture, OCR and translation can have their own adapters and controllers, but one translation-session service owns the order of work. It gives each request a session and sequence ID, keeps pending work bounded, cancels obsolete work where possible and rejects stale responses. The UI observes this session instead of coordinating providers itself.

For the model we need a few classes too, such as capture_moment, language, configuration and translation_request. Domain options such as supported languages can be enums. Classes should be grouped by cohesion rather than forcing every tiny class into its own file.

For the view, we will use PySide6 so we can have a main window for controlling and configuring the tool alongside small frameless QWidget windows (such as the translucent translation box and region selector). For the util I haven't really thought much about it; it could hold genuinely shared image cropping or file management, but it should not become a dumping ground.

## Pipeline and provider rules

- Image changes trigger OCR, but OCR text must remain stable briefly before translation. Image deduplication and text deduplication solve different problems.
- A normal dialogue session prioritizes the newest stable line. Older pending frames can be dropped, and an older response can never replace a newer displayed line. If a later history feature needs every line, it must use an explicit separate policy rather than an unbounded queue.
- Changing provider, language, capture region or context invalidates affected pending work. Retries are bounded and must not keep growing the queue while offline or rate limited.
- OCR, capture, translation, key storage and persistence sit behind adapters. Provider adapters expose capabilities such as language pairs, context support, streaming, cancellation and usage reporting instead of forcing an OpenAI-shaped interface on everything.
- Usage can be provider-reported, estimated or unavailable. Traditional MT may report characters or nothing, while local runtimes may only provide their own token counts.
- Model lifecycle has an explicit owner for loading, unloading, downloads, CPU fallback and out-of-memory errors. OCR and LLM memory use must be measured together.
- Cached translations, if added, must account for source text, target language, provider, model, prompt version and relevant context.

---

## What technologies should we use?

For an initial release paddleOCR seems like a good idea for the OCR engine. We can also relegate the local hosting of ai in the application to llama.cpp.

For the front end library we will use PySide6 (Qt6). We want to avoid Electron and web wrappers because browser processes add memory beside OCR and local models, and Wayland transparency and overlay control can be troublesome. PySide6 gives us shared-process Qt windows, translucent frameless widgets, QSS theming and AT-SPI2 integration. These features still need measurement and accessibility testing. GTK4 with gtk4-layer-shell is only a fallback investigation for overlays, not a second client to maintain in parallel.

For Wayland screen and window capture, we will use xdg-desktop-portal (ScreenCast) paired with PipeWire and GStreamer behind a decoupled capture adapter. Please read [4-Client and screen capture.md](4-Client%20and%20screen%20capture.md) for a complete breakdown of the UI client and Wayland capture behavior.

For persistence, start with the smallest embedded store that fits configuration and bounded session context. A document database remains an option if the data actually benefits from it. Context is optional, inspectable, clearable and session-scoped by default. Durable facts need provenance and user correction; the AI cannot silently rewrite them. OCR text and model output are data, never instructions for changing application behavior.

As for traditional machine translation we also can't afford to make a makeshift library and will instead reuse the work of others. Argos Translate is a candidate for local MT, subject to language-pair, quality, packaging and license checks.

The network gateway is optional. A direct provider adapter should reuse HTTP connections first; the gateway must show a measured benefit and have a defined trust boundary before it becomes necessary. API keys belong in OS-backed key storage where available, not ordinary configuration files or logs.