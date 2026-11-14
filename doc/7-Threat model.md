# Threat model

Status: initial planning. This is a concise threat model based on ToucanTL's actual desktop data flow.

---

## What we are protecting

- Screen captures and OCR text, which may contain private conversations or account information.
- Translations, prompts, and stored context.
- Cloud API keys and the user's provider quota.
- Local models, language data, configuration, and application files.
- Session availability and responsiveness.

---

## Data flow and trust boundaries

Capture, cropping, and OCR happen locally, and screenshots stay in memory during normal operation rather than being written to disk unless an explicit debug option is enabled.

With local translation, OCR text and enabled context remain on the device. With cloud translation, the active provider receives only the recognized OCR text and enabled context (never the screenshot), and the UI makes this clear before the first cloud request and whenever switching providers.

The main trust boundaries are the desktop portal and PipeWire stream, local OCR and model dependencies, persistent storage, cloud providers, and the optional gateway. OCR text and model output are always handled as untrusted data, even when captured from a game the user trusts.

---

## Main threats and mitigations

- **API key exposure:** Use OS-backed key storage where available, and keep keys out of normal configuration files, source, fixtures, logs, and crash reports. When secure storage is unavailable, warn the user about the fallback and its risk instead of silently saving plaintext keys.
- **Sensitive logs:** Do not log captures, OCR text, prompts, context, or responses by default. Explicit debug capture or prompt logging must be temporary, clearly marked, safely stored, and redacted where possible.
- **Unexpected cloud disclosure:** Show the active provider and what it receives. Switching providers must isolate sessions so queued text or older context cannot leak to a different service.
- **Prompt injection and context poisoning:** Treat recognized text strictly as content to translate, never as application instructions. Models cannot alter configuration, invoke tools, or rewrite persistent context by themselves; stored facts require provenance and user correction.
- **Gateway access:** Prefer a direct reusable HTTP client. If a local gateway runs, bind it to loopback and prevent unrelated local processes from using the user's keys. Any remote gateway is a separate trusted service and requires its own authentication, transport security, retention rules, and key-ownership design before use.
- **Untrusted files and dependencies:** Validate configuration and imported data, store model downloads in controlled locations, verify origins or checksums where available, record dependency and model licenses, and never execute model output or downloaded files.
- **Local data retention:** Keep context bounded and session-scoped by default with inspect, clear, and disable controls, and document clearly what uninstall removes and what remains.
- **Resource exhaustion:** Bound frame and request queues, limit retries, reject stale work, and handle provider failures without repeatedly spending quota. Model downloads and local inference require explicit cancellation and out-of-memory handling.

---

## Pre-release checks

- Test both local and cloud modes to verify only documented data crosses each boundary.
- Inspect configuration files, logs, crash output, and temporary files for leaked keys or captured text.
- Test key-storage failure, provider switching, context clearing, and gateway access restrictions where applicable.
- Review third-party dependency, model, and fixture origins and licenses, and update this document whenever a new provider, remote gateway, persistent store, or debug export alters the data flow.
