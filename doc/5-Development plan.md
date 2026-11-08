# Development plan

Status: planning. This is a short order of work, not a promise that every feature must be finished before the basic application can be used.

## 1. Check the risky assumptions first

- Pick the author's desktop and one machine as the first reference environment. Record its CPU, GPU, RAM, compositor and display scale.
- Test portal monitor and window capture, PipeWire frames, cropping, resizing and scaling.
- Test normal translation windows, overlays and click-through separately. Write down which compositors support each one and keep a normal-window fallback.
- Measure PaddleOCR and a candidate GGUF model together, including memory, startup time and CPU fallback. Do not set final latency or memory limits before these measurements.
- Compare a direct reusable HTTP connection with the prototype gateway before making the gateway part of the design.
- Keep the threat model and a small packaging check in this phase so permissions, keys and native dependencies do not become surprises later.

## 2. Make one complete translation path

- Build the first-run wizard and a simple configuration window in English, with keyboard navigation, accessible labels and zoom considered from the start.
- Capture a monitor or supported window, crop a region and run OCR on stable changes.
- Translate through one provider and show both the OCR text and translation in a normal accessible window.
- Add pause, clear errors, bounded pending work, timeouts and stale-response rejection.
- Use one session owner for the pipeline. The UI should display its state rather than coordinate OCR and provider calls itself.
- Add the unit, contract and integration tests around this path as it is written.

## 3. Add the provider choices

- Add traditional local MT, cloud LLM and local LLM adapters without making them pretend to have identical features.
- Add provider switching, secure key storage, model download or selection, and usage reporting where the provider supports it.
- Keep cloud use explicit: OCR and capture stay local, while the selected cloud translation provider receives the OCR text and any enabled context. Screenshots are not sent by this pipeline.
- Test offline, timeout, malformed response, rate limit, cancellation and insufficient-memory behavior.

## 4. Add context carefully

- Start with optional, bounded, session-scoped context that the user can inspect, clear and disable.
- Do not let the model rewrite persistent facts by itself. Add durable character or setting information only with provenance and a way to correct it.
- Compare translations with and without context before adding retrieval or a more complicated database.
- Evaluate these scenarios: linear dialogue, legitimately repeated lines, backlog viewing, save and load, scene changes and character-name consistency.
- A repeated line is not automatically a new scene, and a scene change is not proof that old facts are false. When the application cannot tell, it should avoid inventing a durable conclusion.

## 5. Finish the desktop experience

- Add supported overlay modes and second-monitor placement, then finish high contrast and the full keyboard and screen-reader checks.
- Add the remaining UI translations independently from OCR and translation language support.
- Run the reference desktop tests and a small set of additional compositor and scaling tests based on risk.
- Finish packaging, including model and language-data installation, upgrades, uninstall and offline startup.

## What counts as progress

A phase is useful when it leaves a working application and tests behind, not just disconnected components. Performance, memory and translation-quality targets should be written down after the first benchmarks and then used as release checks. The later phases do not remove features from the project; they stop uncertain features from blocking the first complete pipeline.
