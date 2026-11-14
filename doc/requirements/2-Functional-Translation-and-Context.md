# Functional requirements: Translation and context

Status: approved. This document specifies the requirements for translation session ordering, provider adapters, local model lifecycle management, and session context. Related architectural constraints live in [3-Architecture.md](../3-Architecture.md).

---

## 1. Translation session and queue ordering (`TRN`)

- **TRN-1 (Session and sequence identifiers):** A single translation-session service must coordinate all translation requests using a source-agnostic text request model so future non-OCR text sources can feed the same pipeline. Every request must be assigned the current session ID and a monotonically increasing sequence ID when created.

- **TRN-2 (Newest-line priority and bounded queue):** In normal dialogue mode, the session must prioritize the newest stable text. At most one translation request may be in flight at a time, with at most one newer stable line waiting in the queue. If an even newer stable line arrives before the queued line starts, the older queued line must be dropped.

- **TRN-3 (Obsolete request cancellation):** When a new stable line supersedes an in-flight request, or when the user pauses capture or changes session settings, the session service must request cancellation from the active provider adapter if the adapter supports cancellation.

- **TRN-4 (Stale-response rejection):** When a translation response arrives, the session service must verify that its session ID matches the active session and its sequence ID is newer than the currently displayed translation. Any response from a previous session or with an older sequence ID must be discarded and must never overwrite a newer translation on screen.

- **TRN-5 (Configuration change invalidation):** Changing the active translation provider, source language, target language, crop region, or context state must immediately invalidate all pending and in-flight requests from the prior configuration.

- **TRN-6 (Bounded retries and failure states):** When a provider fails due to network timeout, offline state, malformed output, or rate limiting, retries must be strictly bounded. Repeated capture frames must not queue additional retries while the provider remains in a failing state, and the session must surface a clear error state to the UI.

---

## 2. Translation providers and model lifecycle (`PRV`)

- **PRV-1 (Capability-based provider interface):** Every translation engine must implement a provider adapter interface that explicitly reports its capabilities:
  - Supported source-to-target language pairs.
  - Support for session context and glossary prompts.
  - Support for streaming partial responses.
  - Support for in-flight request cancellation.
  - Support for usage reporting (tokens, characters, estimated cost/count, or unavailable).

- **PRV-2 (Supported provider categories):** The architecture must support four provider categories without coupling the core session pipeline to any single vendor:
  - Local small language models via `llama.cpp` (GGUF format).
  - Local traditional machine translation (with `Argos Translate` as the primary v1 candidate).
  - Cloud LLM providers using user-supplied API keys (such as Gemini or DeepSeek).
  - Cloud traditional machine translation using user-supplied API keys (such as Google Translate API).

- **PRV-3 (Provider isolation on switch):** Switching providers must start a clean provider state. Pending requests, API keys, usage counters, and provider-specific session handles must never leak from one provider to another.

- **PRV-4 (Network connection reuse):** Cloud provider adapters must use a persistent HTTP client that reuses connections across requests. Any local or remote gateway adapter remains optional and may only be enabled if benchmarked faster than direct connection reuse and configured with explicit access controls.

- **PRV-5 (Unified local model lifecycle owner):** A single lifecycle service must manage loading, unloading, downloads, checksum/origin verification, and memory tracking for local OCR and local translation models together.

- **PRV-6 (Memory release and CPU fallback):**
  - Cloud translation mode must operate without loading a local translation model into RAM or VRAM.
  - Switching from a local translation model to a cloud provider must unload the local translation model from memory.
  - If GPU memory is insufficient for a local model, the lifecycle service must fall back to CPU execution where feasible or stop cleanly with an explicit insufficient-memory message rather than crashing.

---

## 3. Session context and glossary (`CTX`)

- **CTX-1 (Optional and session-scoped by default):** Conversation context must be optional and scoped to the active translation session. Starting a new session must not silently carry over dialogue history from a previous session.

- **CTX-2 (Two-part bounded context structure):** When context is enabled for a provider that supports it, the context store must consist of two bounded parts:
  - **Rolling dialogue window:** A fixed maximum number of recent recognized lines and their completed translations from the current session.
  - **Durable user facts:** User-confirmed character names, pronouns, glossary terms, or brief setting notes with known provenance.

- **CTX-3 (Inspect, edit, clear, and disable controls):** The user must be able to inspect the current rolling dialogue window and durable facts, edit or delete durable facts, clear the rolling dialogue history independently of durable facts (such as after loading a save or changing scenes), clear all session context, or disable context entirely. Clearing or disabling context must take effect on the very next translation request.

- **CTX-4 (Protection against context poisoning):** Recognized OCR text and model outputs must be treated strictly as untrusted data. The translation model must never autonomously create, modify, or overwrite durable facts or application settings.

- **CTX-5 (Non-linear reading and repeated lines):** When the user views an in-game backlog, reloads an earlier save, or encounters a repeated line that already appears at the tail of the rolling dialogue window, the system must still display the translation without appending duplicate consecutive entries into the context window.

- **CTX-6 (Optional translation caching):** If a session translation cache is enabled, cache lookups must key on the full request signature: normalized source text, source language, target language, provider ID, model ID, prompt template version, and active context fingerprint.
