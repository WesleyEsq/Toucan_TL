# Development plan

Status: planning. This is the build order for ToucanTL, ordered so risky platform and hardware assumptions are checked before uncertain features block getting a working translator running.

---

## 1. Check the risky assumptions first

- Pick the author's desktop and one machine as the initial reference environment, recording its CPU, GPU, RAM, compositor, and display scale.
- Test portal monitor and window capture, PipeWire frames, cropping, resizing, and display scaling.
- Test normal translation windows, overlays, and click-through separately; document which compositors support each mode and keep a normal-window fallback.
- Measure `PaddleOCR` and a candidate GGUF model running together, including memory use, startup time, and CPU fallback, before setting final latency or memory targets.
- Compare a direct HTTP client reusing connections against the prototype gateway before deciding whether the gateway belongs in the architecture.
- Keep [7-Threat model.md](7-Threat%20model.md) and a small packaging check in this phase so permissions, key storage, and native dependencies do not create late surprises.

---

## 2. Make one complete translation path

- Build the first-run setup wizard and a simple English configuration window with keyboard navigation, accessible labels, and working zoom.
- Capture a monitor or supported window, crop a region, and run OCR when stable changes appear.
- Translate through one provider and display both the OCR text and translation in a normal accessible window.
- Use one translation-session service to handle ordering, pause, clear errors, bounded pending work, timeouts, and stale-response rejection so the UI only observes state.
- Add unit, contract, and integration tests around this path as it is written.

---

## 3. Add the provider choices

- Add traditional local MT, cloud LLM, and local LLM adapters behind capability interfaces so each exposes only what it supports.
- Add provider switching, secure key storage, model selection or downloads, and usage reporting where supported.
- Keep cloud boundaries clear: capture and OCR remain local, while cloud translation providers receive only recognized OCR text and enabled context (never screenshots).
- Test offline mode, timeouts, malformed responses, rate limits, cancellation, and insufficient-memory handling.

---

## 4. Add context carefully

- Start with optional, bounded, session-scoped context that the user can inspect, clear, and disable.
- Add durable character or setting facts only with known provenance and user correction; the model cannot rewrite persistent facts by itself.
- Compare translations with and without context before adding retrieval or a more complex store.
- Evaluate these scenarios: linear dialogue, legitimately repeated lines, backlog viewing, save and load, scene changes, and character-name consistency. A repeated line does not automatically indicate a new scene, and a scene change does not mean prior facts are false; when uncertain, the application avoids inventing durable conclusions.

---

## 5. Finish the desktop experience

- Add supported overlay modes and second-monitor placement, then complete high contrast and full keyboard and screen-reader checks.
- Add the remaining UI translations independently from OCR and translation language support.
- Run the reference desktop tests along with a targeted set of compositor and scaling checks based on risk.
- Complete packaging, covering model and language-data installation, upgrades, uninstall, and offline startup.

---

## What counts as progress

Each phase should leave a working application and passing tests behind rather than disconnected pieces. Performance, memory, and translation-quality targets are written down after the Phase 1 benchmarks and then used as release checks.
