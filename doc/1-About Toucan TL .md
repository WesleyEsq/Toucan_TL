# Toucan Translation Lens

**CloudCanopy.net**
author: West36
date:   2026/11/07

---

## The project:

LLM based translations are starting to be a crucial niche in fanmade translations, following the technology of the traditional machine translation software options, we have a few hiccups that can be addressed with this project. These concerns that will be addressed should be:

- **OCR options are a last resort:** Other forms of text parsing are infinitely faster, more reliable and efficient on resource use, but are important for cases when we can't parse the text or get it through memory hooks, in these cases OCR needs a stronger tool for translations.

- **Translations lack context:** With gen AI, translations are a mixed bag, if we just feed them directly the ai will have trouble identifying cues about the tone of the scenes in media, and will make mistakes based on literal translations of words. LLMs can bridge this gap if they are fed the needed context, so they can help instead in scenes and light localization work.

- **Tools are behind paywalls:** Existing OCR software depend entirely on the cloud, sending screen captures of the device to cloud models to read, translate and bring back the translation, this is inefficient and expensive or demands ads due to the architecture.

- **Portability:** A big advantage of OCR is that it just immediately works without any kind of setup, due to how the only requirement it asks for is to get an image of the text. This is a point that can be taken advantage of with a tool that is focused on portability.

We may want to also do a document parser such as a pdf reader into the software, but for now the main focus is to read through OCR and do real time translations of media such as visual novels, RPGs and manga which are fields where the technology benefits the most of. 

---

# Things to consider

## What do we actually require

ToucanTL is meant to be a desktop program, written mostly in python, using PySide6 (Qt6) for a lightweight multi-window UI, and with Llama.cpp to run small GGUF models as well as PaddleOCR. **Importantly:**

+ I need to make sure that these core dependencies are not tightly coupled to the code.
+ The program needs to do screen sharing on Wayland via xdg-desktop-portal (ScreenCast) and PipeWire.
+ The program will get a stream from the window or screen selected, and will take snapshots in memory.
+ The user needs to either select a small section on the screen or set the whole area.
+ Using a debounce algorithm in the image, I will get if there are changes on the image
+ Then, use OCR and detect changes comparing the previous change to the new one
+ Finally pass the information to the LLM and then receive it back
+ Or optionally pass it directly to a traditional machine translation engine

The main fundamentals of what the program should do are:

1. Be efficient

2. Be as quick as you possibly can with network calls to avoid adding too much delay overhead.

3. Also implement options for using their own API Keys in the cloud (candidates are deepseek and gemini).

4. Be reliable, use TDD and good practices whenever possible.

---

## The problems of scene context + delay, and how to solve it

The information in a conversation could be passed by giving all the text to the flash model, but this would in turn cause significant costs in input tokens and delay in the response. 

+ We can solve this with a document database. We can store information about the conversations as well as the characters of the media and the tone of the scenes, so the AI can parse through those small bits of structured information and give answers that are adequate to the reader experience.

Also, as the second issue, we have that directly calling the API from a desktop application would be troublesome since the calls need to go through the network of the AI from python libraries for the API. 

+ This has been solved in a prototype, by setting up a small server that is always connected to the AI endpoint, we can just send requests more easily with a tiny gateway serving as a middleman. IDK Why but just having a server in the middle of the client and the API key is significantly much more faster by 400 milliseconds from my observations.

Finally, it's clear to say that we can simply use the fastest and less intelligent flash models, since the task of translation and localization can be trivialized with enough context to improve the quality. 

+ The use of traditional machine translation engines can also work without the hassle of context in exchange of losing it entirely, **this can actually be beneficial for a lot of uses.** so it should be an option for both local hosting as well as with the google translate API.


## Wait no, context can be horrible to do properly

The main thing that prevents a context engine to be made and work properly is simply how the user can just go back to previous texts inside the game or inside the manga and other pages instead of reading linearly, the ai might become confused and fail with either how the dialog repeats or scenes dramatically change. To alleviate this without tracking the internal game state we need to be more deliberate with what we store, by only keeping superficial information about the setting. This dramatically decreases the appeal of what we might gain from a real time AI translator. Solutions to fix this need to be evaluated carefully so that they aren't just hacks on top of a makeshift system. This would be the one flaw the project has.

---

## Accessibility and other concerns

The tool needs to follow strict accessibility concerns with the WCAG 2 for the client, leveraging PySide6 native AT-SPI2 support on Linux and Qt Style Sheets (QSS) for global UI zooming and high-contrast plain themes. Also there needs to be a setup wizard upon entering the program, to first set the language for the UI with a picker, then the language that you will read your translations in. Afterwards the color theme and the option for what AI or machine translation tool you will use, if you need to setup these engines and which ones.

This program will also be programmed first and foremost for linux environments with Wayland (see 4-Client and screen capture.md for how Wayland overlays and capture are handled). For software good practices the main one would be a very strict MVC architecture and Object oriented programming, I don't want any complaints about that. And additionally by the adoption of the traditional TDD methodology, avoiding too many changes without doing a proper commit, a stricter code writing style and documenting everything neatly with parameters, these last two concerns are on the second markdown titled "guidelines".

---

# Project specifications

## Main objective

Develop a desktop application based in Optical Character Recognition (OCR) by using a complete toolset for translating text and turn based videogames in real time.

## Secondary Objectives

1. Develop a comprehensive UI in PySide6 that is customizable with how much screen the tool takes (using a second monitor, or a small frameless translucent widget with optional click-through support for games where dialog is displayed)

2. Implement a successful hybrid pipeline for translating text, OCR → Query Context(if LLM) → Sending to the Translation model (if by API key) → translating → displaying on screen.

3. Design a successful UI mockup prototype with plain on a prototype directory, with a modular set of files, determine the right tool to do this and do so to guide the client design, can be a wireframe.

4. Adopt accessibility standards, not directly based on the WCAG 2 but inspired on it. Investigate and evaluate what we can reasonably bring to the project (focusing on visually impaired related issues). This means ability to zoom in the UI too.

5. Capture images inside Wayland via xdg-desktop-portal and PipeWire so that they can be used in the pipeline for screen translation.

6. The program can reasonably be used out of the box by an user that doesn't require technical skills, by using metaphors with icons and with translations. The Color theme needs to be a bit plain, without animations and gradient effects.

7. A configuration menu that is easy to use, to set the mode and values for translation.

## Firmly Out of Scope

1. End to end testing and integration tests, as well as configuring a CI/CD pipeline with the repository, this means tests need to be setup for changes manually by the developer to guide them.

2. Specific LLM compatibility for the AI router with API keys, we need to not adhere to specific tools.

3. Any other type of client than a desktop client for linux with Wayland. We will make an initial testing version for my own DE and for linux systems for the foreseeable future.

4. Text to speech. Would be great with the accessibility standard but we need to prioritize the pipeline. I will reuse another project to fulfill this niche within this software.

5. Overcomplicating the context engine. Should be a smaller piece of the pipeline.

## Acceptance Criteria

- The application can select a screen or a window
- It can handle the screen resizing
- Can host a small Language model as well as an OCR engine inside the same device.
- Can work entirely with cloud API instead of running the translations locally
- Counts the amount of tokens used in the UI for cloud usage or whatever the OpenAI API standard has.
- Can switch between LLM and machine translations
- The application can work without significantly obstructing the UI
- Translated text is displayed on screen in accordance to the selected style
- Unit tests are easily human readable (AAA and builder Pattern), and are focused on business logic cases.
- Every function and file is documented with a small header in accordance to the file 2-guidelines.md.
- Documentation is up to date within this parallel documentation folder.

## Nice to haves

1. The program can setup everything properly with the setup wizard and start working right away.
2. Support about 5 languages. These are necessarily English, Spanish, French, Japanese and Mandarin Chinese for the UI
3. Automatically detect the language in a screenshot, we can rely on the user manually inputting the language for now.
5. A flatpak, this will probably be the last thing that we make.
6. Customizable color theme options for the UI.