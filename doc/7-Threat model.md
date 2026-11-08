# Threat model

Status: initial planning. This is a small practical threat model for the desktop application, not a claim that a desktop app has no security concerns.

## What we are protecting

- Screen captures and OCR text, which may contain private conversations or account information.
- Translations, prompts and stored context.
- Cloud API keys and the user's provider quota.
- Local models, language data, configuration and application files.
- The availability of the capture and translation session.

## Data flow and trust boundaries

Capture, cropping and OCR happen locally. Screenshots stay in memory for the normal pipeline and are not written to disk unless a separate explicit debug feature asks for it.

With local translation, OCR text and enabled context stay on the device. With cloud translation, the selected provider receives the OCR text and any enabled context needed for the request, but not the screenshot. The UI must make this clear before the first cloud request and when switching providers.

The main trust boundaries are the desktop portal and PipeWire stream, local model and OCR dependencies, persistent storage, cloud providers and the optional gateway. OCR text and model output are untrusted data, even when they came from a game the user trusts.

## Main threats and what we do about them

- **API key exposure:** Use OS-backed key storage where available. Do not put keys in ordinary configuration, source, fixtures, logs or crash reports. If secure storage is unavailable, explain the fallback and its risk instead of silently writing a plaintext key.
- **Sensitive logs:** Do not log captures, OCR text, prompts, context or responses by default. Debug capture or prompt logging must be explicit, temporary and clearly marked, with a safe storage location and redaction where possible.
- **Unexpected cloud disclosure:** Show the active provider and what it receives. A provider switch must not silently send old context or queued text to a different service.
- **Prompt injection and context poisoning:** Treat recognized text as content to translate, never as application instructions. The model cannot change configuration, call tools or rewrite persistent context by itself. Stored facts need provenance and user correction.
- **Gateway access:** Prefer a direct reusable HTTP client. If a local gateway is used, bind it to loopback, avoid exposing it to the network and prevent unrelated local processes from using the user's keys. A remote gateway is a separate trusted service and needs its own authentication, transport security, retention and key-ownership decisions before use.
- **Untrusted files and dependencies:** Validate configuration and imported data, keep model downloads in controlled locations, verify their origin or checksum where available, and record dependency and model licenses. Do not execute model output or downloaded content.
- **Local data retention:** Keep context bounded and session-scoped by default. Provide inspect, clear and disable controls. Define what uninstall removes and what user data remains.
- **Resource exhaustion:** Bound frame and request queues, limit retries, reject stale work and handle provider failures without repeatedly spending quota. Model downloads and local inference need explicit cancellation and out-of-memory handling.

## Things to check before release

- Test cloud and local modes to confirm that only the documented data crosses each boundary.
- Check configuration, logs, crash output and temporary files for keys and captured text.
- Test key-storage failure, provider switching, context clearing and gateway access where applicable.
- Review third-party dependency, model and fixture origins and licenses.
- Update this document when a new provider, remote gateway, persistent context or debug export changes the data flow.
