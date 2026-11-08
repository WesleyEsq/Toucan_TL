# Testing strategy

Status: planning. This document defines how ToucanTL is verified. It follows the TDD expectations in [2-guidelines.md](2-guidelines.md), the scope in [1-About Toucan TL .md](1-About%20Toucan%20TL%20.md), and the phases in [5-Development plan.md](5-Development%20plan.md).

---

## Purpose and principles

- Tests guide design and protect business logic.
- Unit tests are human readable, use arrange-act-assert, and use builders for complex objects.
- The default suite is fast, deterministic, and offline.
- Network, GPU, portal, and compositor behavior are isolated behind adapters so they can be faked.
- A fake proves the contract, not the platform. Wayland compatibility is only proven on a real desktop session.
- Tests must not encode personal data, API keys, or captured text from real users.

## Test levels

### Unit

Cover pure logic: context caps and deduplication, queue policy, stale-response rejection, prompt assembly, configuration validation, usage reporting, and language and provider selection. No I/O, no network, and no Qt event loop.

### Component and contract

Cover each adapter against its interface: OCR, translation providers, capture source, repository, and key storage. Contract tests use fakes and recorded fixtures and assert shapes, errors, timeouts, and cancellation. They do not assert exact model prose.

### Integration

Cover two or more real components together with fakes at the edges: pipeline ordering, cancellation, CPU fallback, repository persistence, and provider switching. Qt tests may run offscreen, but offscreen tests do not replace desktop tests.

### System

Cover the packaged application on a real Wayland session with the reference hardware. These are manual or semi-scripted and are listed below.

### Accessibility and exploratory

Cover keyboard-only use, screen-reader labels, zoom without clipping, high contrast over changing content, focus order, controlled announcement of new translations and first-run comprehension. Test the normal focusable translation window as well as click-through mode. Exploratory sessions look for confusing states, stuck queues and unclear errors.

### Packaging

Cover install, first run, upgrade, uninstall, missing dependencies, and offline start from a clean machine or container.

## Initial system tests

1. First-run setup: choose translation language, theme, and provider; confirm the UI language (English in the MVP); reach a working state without a terminal.
2. Portal cancel, deny, and retry: cancelling or denying the picker leaves the app usable and explains what happened; retry works.
3. Monitor, window and region capture: select each supported target, crop a region and confirm only that region is read.
4. Resize and rescale: resize or move the source and change display scale; capture and crop stay correct or pause for a new selection.
5. Unchanged-frame and stable-text suppression: a static screen produces no OCR calls, and animation without new stable text produces no translation calls.
6. Ordered translation: rapid dialogue changes never show an older line after a newer one, and pending work remains bounded.
7. Timeout, offline and rate limit: each provider failure shows a clear state, retries within its limit and does not grow the queue.
8. Context clear: clearing or disabling context changes the next prompt and is reflected in the UI.
9. Provider switching: switch between traditional MT and LLM without leaking requests, keys, usage or context to the wrong provider.
10. Overlay and second monitor: test the normal-window baseline everywhere supported; overlay and click-through work only where documented, and failure is graceful.
11. Stream interruption: target closure, monitor removal, permission loss and PipeWire disconnection stop or recover the session cleanly.
12. Lens feedback: the application does not repeatedly OCR and translate its own lens or region selector.
13. Privacy: cloud mode sends only documented OCR text and enabled context; logs and temporary files do not contain keys or captured text by default.
14. Clean packaged install: install on a clean system, complete setup, translate one line, test offline local mode where configured, and uninstall.

## Test matrix

Do not run every possible combination manually. Use the author's desktop and reference hardware for the release smoke test, then choose additional cases based on the feature that changed and known platform risks.

The reference release set covers:

- The author's compositor at 100% and one fractional scale, with monitor capture and window capture where supported.
- One normal online provider run and deterministic offline, timeout and rate-limit tests.
- Local OCR with the supported local translation path, plus CPU fallback or insufficient-memory behavior.
- Normal-window display and any overlay mode claimed for that compositor.

Additional scheduled or pre-release checks cover GNOME, KDE and one wlroots compositor; 125%, 150% and 200% scaling; traditional MT, cloud LLM and local LLM; GPU, no GPU and GPU out of memory; and online, offline, timeout and rate-limit behavior. Use pairwise or risk-based selections rather than a full Cartesian product.

## AI-specific testing rules

- Do not assert exact LLM prose in ordinary automated tests. Assert structure, ordering, non-empty output, language, and invariants instead.
- Use recorded responses for deterministic tests and scrub them of personal data.
- Test prompts as data: snapshot the assembled prompt, not the model's answer.
- Test context behavior with the scenarios in [5-Development plan.md](5-Development%20plan.md): linear dialogue, repeated lines, backlog viewing, save and load, scene changes, and character-name consistency.
- Treat model quality as an evaluation, not a unit test. Keep a small licensed or synthetic evaluation set covering the intended OCR languages and content layouts. Record OCR errors, translation meaning, names, reading order, latency, model and prompt version, and compare context against no context.
- Always test timeout, cancellation, malformed response, and rate-limit paths.
- Never let a test depend on a live paid API by default.

## Fixtures and privacy

- Use synthetic, licensed, or self-made screenshots.
- Keep fixtures small and store them with the tests.
- Scrub API keys, prompts, and captured text from logs and recordings.
- Do not commit real user media or profiles.
- Document the origin and license of every fixture.

## CI and release gates

CI is in scope for unit, component/contract and suitable integration tests, lint and import smoke checks. Real Wayland system tests and the selected compositor, GPU and cloud-provider checks are manual or scheduled. There is no requirement to automate every end-to-end desktop case or build a deployment pipeline.

Before merge:

- Unit and component/contract suites pass.
- Lint and import smoke checks pass.
- No new network calls in the default suite.

Before release:

- Integration suite passes.
- The system-test list above is completed on the reference hardware.
- Packaging smoke test passes on a clean install.
- Accessibility checklist is reviewed.
- Benchmark results are recorded and compared with the previous release.

## Quality metrics

- Business-logic coverage, not total line coverage.
- Flaky-test rate, tracked and kept near zero.
- p50 and p95 latency for capture-to-OCR, OCR-to-translation and total, with the reference hardware and provider recorded.
- Process and model memory, startup time, dropped frames and bounded queue behavior under load.
- Crash-free sessions of a defined length; start with a two-hour reference soak test and adjust it from real use.
- OCR and translation evaluation results against the fixed evaluation set, including a no-context baseline.
- Number of unresolved accessibility and threat-model findings.
- Provider failure recovery in the selected system tests.

The feasibility phase records the first measurements and turns them into release budgets. Until then, these are metrics to collect, not invented performance promises.
