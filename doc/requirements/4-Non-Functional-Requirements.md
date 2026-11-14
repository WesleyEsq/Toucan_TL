# Non-functional requirements

Status: approved. This document specifies the non-functional requirements for accessibility, performance, resource bounds, security, privacy, reliability, and packaging. Verification methods and security analysis live in [6-Testing strategy.md](../6-Testing%20strategy.md) and [7-Threat model.md](../7-Threat%20model.md).

---

## 1. Accessibility (`NFR-ACC`)

ToucanTL uses WCAG 2 principles as practical guidance for a Linux desktop application without claiming formal web conformance.

- **NFR-ACC-1 (Keyboard-only operation):** Every action in the setup wizard, main control window, region selector, and focusable lens window must be operable via keyboard alone, with a logical tab order and visible focus indicators.

- **NFR-ACC-2 (Screen-reader integration via AT-SPI2):** All interactive Qt widgets, status indicators, and error banners must expose descriptive accessible names and roles through the Linux AT-SPI2 accessibility bus.

- **NFR-ACC-3 (Controlled translation announcements):** When a translation completes, screen-reader announcement behavior must be configurable (automatic on completed line, manual trigger, or muted). If a provider streams partial tokens visually, AT-SPI2 announcements must wait until the full line completes so the screen reader is not interrupted on every token chunk.

- **NFR-ACC-4 (UI zoom without clipping):** The UI and translation lens must support text and layout scaling (up to 200%) with proper layout reflow so controls and translated dialogue are not clipped or overlapping.

- **NFR-ACC-5 (Plain high-contrast themes):** All built-in themes must use flat, solid colors without decorative animations or gradients, and provide a high-contrast preset suitable for low-vision users reading over varying backgrounds.

---

## 2. Performance and resource usage (`NFR-PERF`)

- **NFR-PERF-1 (Reference benchmark budgets):** Concrete numeric budgets for startup time, idle CPU usage, combined RAM/VRAM usage (OCR plus local LLM), and p50/p95 latency (capture-to-OCR, OCR-to-translation, and end-to-end) must be recorded on the documented reference hardware during Phase 1 ([5-Development plan.md](../5-Development%20plan.md)) and enforced as release gates thereafter.

- **NFR-PERF-2 (Zero idle translation overhead):** When the captured screen region is static, image-difference filtering must prevent OCR and translation calls so background CPU/GPU usage stays minimal and zero cloud quota is consumed.

- **NFR-PERF-3 (Single-process UI footprint):** All windows (main window, lens, and region selector) must share a single Python/PySide6 process rather than spawning separate embedded browser processes.

- **NFR-PERF-4 (Bounded memory under load):** Rapidly advancing dialogue or slow provider responses must never cause unbounded growth in frame buffers, request queues, or session context memory.

---

## 3. Security and privacy (`NFR-SEC`)

These requirements enforce the mitigations defined in [7-Threat model.md](../7-Threat%20model.md).

- **NFR-SEC-1 (Local capture and OCR boundary):** Screenshots and video frames must never be transmitted over the network. Cloud providers may only receive recognized OCR text and enabled session context.

- **NFR-SEC-2 (Credential storage):** Cloud API keys must be stored in the OS-backed keyring (`SecretService` / `libsecret` via keyring adapter) where available. Keys must never appear in normal configuration files, source code, test fixtures, logs, or crash reports. If secure OS storage is unavailable, the system must warn the user explicitly before allowing any fallback.

- **NFR-SEC-3 (Privacy-safe logging):** Default application logs must not record screenshots, recognized OCR text, assembled prompts, session context, or translated output. Any diagnostic logging mode must be opt-in, clearly indicated in the UI, temporary, and stored in a user-controlled local directory.

- **NFR-SEC-4 (Untrusted input isolation):** Recognized OCR text and remote model responses must be handled strictly as untrusted data. They must never be executed as code, interpreted as system commands, or permitted to alter application configuration or persistent context facts.

- **NFR-SEC-5 (Controlled model and dependency origins):** Local model downloads must use verified URLs and checksum validation where available, store files in standard user data directories (`XDG_DATA_HOME`), and document third-party model and library licenses.

- **NFR-SEC-6 (Optional gateway boundary):** If an optional local gateway adapter is enabled, it must bind strictly to loopback (`127.0.0.1` / `::1`) and restrict access so unrelated local processes cannot use the user's API keys. Any remote gateway must be treated as a separate trusted boundary requiring explicit authentication, transport encryption (TLS), data-retention disclosure, and key-ownership controls before use.

---

## 4. Reliability, maintainability, and packaging (`NFR-REL`, `NFR-PKG`)

- **NFR-REL-1 (Soak stability):** The application must complete a two-hour continuous reference session without crashes, unhandled exceptions, deadlocked queues, or unbounded memory growth.

- **NFR-REL-2 (Fault isolation):** Failures in an external provider, portal session, or local model load must be caught at the adapter boundary and reported through the session service without crashing the UI event loop.

- **NFR-REL-3 (Modular codebase and testability):** Domain logic must remain decoupled from Qt widgets, D-Bus portals, OCR engines, and HTTP clients so unit and contract suites run deterministically offline with zero network calls, following [2-guidelines.md](../2-guidelines.md) and [6-Testing strategy.md](../6-Testing%20strategy.md).

- **NFR-PKG-1 (Clean installation and offline startup):** A packaged build must install cleanly on a target Linux Wayland system, complete first-run setup without terminal intervention, and start and operate completely offline when configured with local OCR and a local translation engine.

- **NFR-PKG-2 (Transparent uninstallation):** Documentation and packaging scripts must clearly state where configuration (`XDG_CONFIG_HOME`), downloaded models (`XDG_DATA_HOME`), and keyring entries are stored so the user can remove them cleanly upon uninstall.
