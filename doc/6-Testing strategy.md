# Testing strategy

Status: planning. This document defines how ToucanTL is verified, following the TDD expectations in [2-guidelines.md](2-guidelines.md), the scope in [1-About.md](1-About.md), and the build order in [5-Development plan.md](5-Development%20plan.md).

---

## Purpose and principles

Tests guide design, protect business logic, and keep the default automated suite fast, deterministic, and offline. Unit tests follow Arrange-Act-Assert with builders for complex objects. Because network endpoints, GPUs, portals, and compositors cannot be tested directly in normal unit runs, they sit behind adapters so automated tests can use fakes while real Wayland compatibility is checked on an actual desktop session. Tests and fixtures never store personal data, real API keys, or captured text from real users.

---

## Test levels

- **Unit:** Pure business logic with no I/O, network, or Qt event loop—context caps and deduplication, queue policies, stale-response rejection, prompt assembly, configuration validation, usage reporting, and language/provider selection.
- **Component and contract:** Each adapter (OCR, translation providers, capture source, repository, key storage) tested against its interface using fakes and recorded fixtures to verify response shapes, error mapping, timeouts, and cancellation without asserting exact model prose.
- **Integration:** Two or more real components running together with fakes at the boundaries—pipeline ordering, cancellation, CPU fallback, repository persistence, and provider switching. Offscreen Qt tests verify UI wiring, while desktop sessions remain necessary to prove compositor behavior.
- **System:** Manual or semi-scripted verification of the packaged application on a real Wayland session using reference hardware.
- **Accessibility and exploratory:** Keyboard-only operation, screen-reader labels, zoom without clipping, high contrast over changing backgrounds, focus order, and controlled announcement of new translations across both normal focusable windows and click-through mode, plus exploratory checks for stuck queues and unclear errors.
- **Packaging:** Clean install, first run, upgrade, uninstall, missing dependencies, and offline startup in a clean machine or container.

---

## Initial system tests

1. **First-run setup:** Choose translation language, theme, and provider; confirm the UI language (English in v1); reach a working state without a terminal.
2. **Portal cancel, deny, and retry:** Cancelling or denying the screen picker leaves the application usable, explains what happened, and allows retry.
3. **Monitor, window, and region capture:** Select each supported source, crop a region, and confirm only that region is read.
4. **Resize and rescale:** Resize or move the source window and change display scale; confirm the crop stays aligned or pauses for a new selection.
5. **Unchanged-frame and stable-text suppression:** Static screens trigger no OCR calls, and animation without new stable text triggers no translation calls.
6. **Ordered translation:** Rapid dialogue changes never show an older line after a newer one, and pending work stays bounded.
7. **Timeout, offline, and rate limit:** Each provider failure reports a clear state, retries within its bound, and does not grow the queue.
8. **Context clear:** Clearing or disabling context changes the next prompt and updates the UI.
9. **Provider switching:** Switching between traditional MT and LLM providers never leaks requests, keys, usage stats, or context across services.
10. **Overlay and second monitor:** Confirm the normal-window baseline works everywhere; overlays and click-through work where documented and fall back cleanly otherwise.
11. **Stream interruption:** Target closure, monitor removal, permission loss, and PipeWire disconnection stop or recover the session cleanly.
12. **Lens feedback:** The application does not repeatedly OCR and translate its own lens or region selector.
13. **Privacy:** Cloud mode sends only OCR text and enabled context; logs and temporary files stay free of keys and captured text by default.
14. **Clean packaged install:** Install on a clean system, complete setup, translate a line, test offline local mode where configured, and uninstall cleanly.

---

## Test matrix

Coverage uses risk-based selection rather than trying every possible combination manually. Every release runs the reference smoke set on the author's desktop:

- The reference compositor at 100% and one fractional scale, testing monitor capture and window capture where supported.
- One normal online provider run alongside deterministic offline, timeout, and rate-limit checks.
- Local OCR with the supported local translation path, plus CPU fallback and insufficient-memory handling.
- Normal-window display and any overlay mode supported for that compositor.

Additional scheduled or pre-release checks sample across GNOME, KDE, and a wlroots compositor; 125%, 150%, and 200% scaling; traditional MT, cloud LLM, and local LLM; GPU, CPU-only, and GPU out-of-memory; and failure paths.

---

## AI testing rules and fixtures

- Automated tests verify structure, ordering, non-empty output, target language, and invariants rather than exact LLM prose.
- Snapshot assembled prompts as data, use scrubbed recorded responses for deterministic runs, and keep live paid APIs out of the default suite.
- Treat OCR and translation quality as a separate evaluation using a small licensed or synthetic evaluation set across target languages and layouts. Record OCR errors, translation accuracy, character names, reading order, latency, model and prompt versions, and context compared against a no-context baseline.
- Always cover timeout, cancellation, malformed response, and rate-limit paths.
- Keep fixtures small, synthetic or properly licensed, scrubbed of personal data, stored with tests, and documented with their origin and license.

---

## CI gates and tracked metrics

CI runs unit, component/contract, and CI-suitable integration tests along with lint and import smoke checks; real Wayland system, GPU, and cloud-provider checks remain manual or scheduled.

Before merge:

- Unit and component/contract suites pass, with zero network calls in the default suite.
- Lint and import smoke checks pass.

Before release:

- Full integration suite and the 14 reference system checks pass.
- Clean-install packaging check and accessibility checklist are reviewed.
- Benchmarks are recorded and compared with the previous release.

Tracked metrics (turned into release budgets after Phase 1 benchmarks):

- Business-logic coverage and low test flakiness.
- p50 and p95 latency for capture-to-OCR, OCR-to-translation, and end-to-end (with hardware and provider noted).
- Process and model memory, startup time, dropped frames, and bounded queue behavior under load.
- Crash-free stability across a two-hour reference soak test.
- OCR and translation evaluation scores against the fixed set, including a no-context baseline.
- Open accessibility and threat-model findings, plus failure-recovery results in system tests.
