# Toucan Translation Lens

**CloudCanopy.net**
author: West36
date:   2026/11/07

---

## The project:

LLM based translations are starting to be a crucial niche in fanmade translations, following the technology of the traditional machine translation software options, we have a few hiccups that can be addressed with this project. These concerns that will be addressed should be:

- **OCR options are a last resort:** Other forms of text parsing are much faster, more reliable and efficient on resource use. OCR is still important for cases when we can't parse the text or get it through memory hooks, and it is the input method for the first version of ToucanTL. Other text sources can be added later through the same pipeline instead of making the whole application depend on OCR.

- **Translations lack context:** With gen AI, translations are a mixed bag, if we just feed them directly the ai will have trouble identifying cues about the tone of the scenes in media, and will make mistakes based on literal translations of words. LLMs can bridge this gap if they are fed the needed context, so they can help instead in scenes and light localization work.

- **Tools are behind paywalls:** Existing OCR software depend entirely on the cloud, sending screen captures of the device to cloud models to read, translate and bring back the translation, this is inefficient and expensive or demands ads due to the architecture.

- **Portability:** A big advantage of OCR is that it can work without game-specific hooks or modifications, since the main requirement is an image of the text. The application itself can still need language data, models, permissions or API keys, so portability should not be confused with having no setup at all.

We may want to also do a document parser such as a pdf reader into the software, but for now the main focus is to read through OCR and do real time translations of media such as visual novels, RPGs and manga which are fields where the technology benefits the most of. 

---

# Things to consider

## What do we actually require

ToucanTL is meant to be a desktop program, written mostly in python, using PySide6 (Qt6) for a lightweight multi-window UI, and with Llama.cpp to run small GGUF models as well as PaddleOCR. **Importantly:**

+ I need to make sure that these core dependencies are not tightly coupled to the code.
+ The program needs to do screen sharing on Wayland via xdg-desktop-portal (ScreenCast) and PipeWire.
+ The program will get a stream from the window or screen selected, and will take snapshots in memory.
+ The user needs to either select a small section on the screen or set the whole area.
+ Use image debouncing to avoid running OCR on unchanged frames.
+ Run OCR and compare the resulting text with the previous stable result, since an image can change without the dialogue changing.
+ Wait for text to remain stable briefly before translating it, so partially typed dialogue does not create unnecessary requests.
+ Pass the information to the LLM, or optionally pass it directly to a traditional machine translation engine.
+ Keep only a bounded amount of pending work and reject stale responses, so an old translation cannot replace a newer one.

The main fundamentals of what the program should do are:

1. Be efficient

2. Reuse network connections and measure capture-to-translation delay instead of assuming that a particular network setup is faster.

3. Also implement options for using their own API Keys in the cloud (candidates are deepseek and gemini).

4. Be reliable, use TDD and good practices whenever possible.

---

## The problems of scene context + delay, and how to solve it

The information in a conversation could be passed by giving all the text to the flash model, but this would in turn cause significant costs in input tokens and delay in the response. 

+ We can use a small embedded store for bounded conversation history and user-confirmed information about characters or the setting. The AI receives only the small relevant part, not the entire history. The storage format should follow the data we actually need; a document database is an option, not a requirement by itself.

Also, as the second issue, we have that directly calling the API from a desktop application would be troublesome since the calls need to go through the network of the AI from python libraries for the API. 

+ A prototype used a small server that stays connected to the AI endpoint, and it appeared to save around 400 milliseconds. That is a useful observation, not proof that every setup needs a gateway. First compare it with a direct client that reuses its HTTP connection. If the gateway stays, it must be an optional adapter with a defined local or remote deployment, key ownership and access control.

Fast flash models are good candidates, but model choice still needs a small quality and latency evaluation. Context can improve a translation, but it cannot guarantee that a weaker model will always be enough.

+ The use of traditional machine translation engines can also work without the hassle of context in exchange of losing it entirely, **this can actually be beneficial for a lot of uses.** so it should be an option for both local hosting as well as with the google translate API.


## Wait no, context can be horrible to do properly

The main thing that prevents a context engine from working properly is that the user can go back to previous text, open a backlog or load an old save instead of reading linearly. The AI might become confused by repeated dialogue or dramatic scene changes. Without tracking internal game state, I cannot reliably infer what every change means.

Context therefore starts as optional, bounded and session-scoped. The user can inspect, clear and disable it, and starting a new session does not silently reuse an old one. Durable character or setting facts should only be added when their source is known and the user can correct them. The model must not rewrite persistent context by itself, because OCR text is untrusted input and a bad inference could poison later translations. Context also needs to be compared against a no-context baseline before I add more complexity.

---

## Accessibility and other concerns

The tool should use WCAG 2 as inspiration for a practical desktop accessibility checklist rather than claim WCAG conformance. This means keyboard-only operation, visible focus, screen-reader labels through AT-SPI2, zoom without clipped content, high-contrast plain themes and control over when new translations are announced. Qt provides useful support here, but the widgets still need testing.

There needs to be a setup wizard on first run. It sets the UI language, source OCR language and target translation language, then the theme and translation provider. English is the initial UI language; Spanish, French, Japanese and Mandarin Chinese are later UI localization goals. OCR source languages and translation language pairs are separate capabilities and must be listed by the selected engines. The wizard is required, but it should only ask for what the chosen provider needs.

This program will be programmed first and foremost for Linux environments with Wayland (see [4-Client and screen capture.md](4-Client%20and%20screen%20capture.md) for how Wayland overlays and capture are handled). The main structure is MVC with object oriented programming, while allowing small services where they make ownership clearer. I also want traditional TDD, reasonably frequent commits, a consistent code style and useful documentation of public functions and parameters. These last concerns are in [2-guidelines.md](2-guidelines.md).

---

# Project specifications

## Main objective

Develop a desktop application based in Optical Character Recognition (OCR) for translating visual novels, RPGs and manga with low latency. Real time is the goal, but the actual delay depends on the selected OCR and translation engines and must be measured on reference hardware.

## Secondary Objectives

1. Develop a comprehensive UI in PySide6 that is customizable with how much screen the tool takes (using a second monitor, or a small frameless translucent widget with optional click-through support for games where dialog is displayed)

2. Implement a hybrid pipeline for translating text: capture → crop → stable text → optional context → local or cloud translation → display. The pipeline must have bounded queues, cancellation where supported and stale-response rejection.

3. Design a successful UI mockup prototype with plain on a prototype directory, with a modular set of files, determine the right tool to do this and do so to guide the client design, can be a wireframe.

4. Adopt accessibility standards, not directly based on the WCAG 2 but inspired on it. Investigate and evaluate what we can reasonably bring to the project (focusing on visually impaired related issues). This means ability to zoom in the UI too.

5. Capture images inside Wayland so that they can be used in the pipeline for screen translation.

6. The program can be used without a terminal or game-specific technical setup. The first-run wizard may still install or select language data and models, request capture permission or ask for an API key. The color theme needs to be plain, without animations and gradient effects.

7. A configuration menu that is easy to use, to set the mode and values for translation.

8. Unit tests and Integration tests.

9. Keep the [threat model](7-Threat%20model.md) short and cover captured text, API keys, logs, stored context, cloud providers, the optional gateway and untrusted OCR text. OWASP is useful for ideas, but the threats should be based on this desktop application's actual data flow.

## Firmly Out of Scope

1. Automated end-to-end testing across every desktop and deployment pipeline work. Unit, component and integration tests can run in CI, while real Wayland system tests remain manual or semi-scripted on selected environments.

2. Hard-coding the core pipeline to one LLM vendor. Provider-specific code belongs behind adapters, and providers only expose the capabilities they actually support.

3. Any other type of client than a desktop client for linux with Wayland. We will make an initial testing version for my own DE and for linux systems for the foreseeable future.

4. Text to speech. Would be great with the accessibility standard but we need to prioritize the pipeline. I will reuse another project to fulfill this niche within this software.

5. Overcomplicating the context engine. Should be a smaller piece of the pipeline.

## Acceptance Criteria

- The application can select a monitor, and a window where the desktop portal supports it.
- Resizing and display scaling keep the selected region aligned, or pause and ask the user to select it again.
- A normal translation window works on the reference desktop; overlay and click-through support depend on the compositor and have a normal-window fallback.
- A supported small language model and OCR engine can run on the documented reference hardware, with clear behavior for CPU fallback and insufficient memory.
- Cloud translation can be used without running a local translation model.
- The UI shows provider-reported usage when available, and clearly labels estimates or unavailable values instead of treating every provider as token-based.
- The user can switch between LLM and traditional machine translation without leaking requests or context between sessions.
- The user can pause capture, see the original OCR text and translation, and understand provider or permission failures.
- Context is optional, bounded, inspectable and clearable.
- Unit tests are human readable (AAA and builders for complex objects) and focus on business logic.
- Public functions and files have useful documentation in accordance with [2-guidelines.md](2-guidelines.md).
- Documentation is up to date within this parallel documentation folder.
- Performance and memory targets are recorded after the feasibility benchmarks instead of being invented before I have reference hardware.

## Nice to haves

1. Support about 5 UI languages: English, Spanish, French, Japanese and Mandarin Chinese. English comes first.
2. Automatically detect the language in a screenshot; I can rely on the user manually selecting it for now.
3. A Flatpak. This will probably be the last packaging target, although a small packaging check should happen early enough to catch portal and dependency problems.
4. Customizable color theme options for the UI.
