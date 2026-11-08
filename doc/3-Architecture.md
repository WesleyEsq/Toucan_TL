# How should we implement this? In brief terms

This is a brief summary, for a more in depth explanation please go to the folder called general architecture. The code here should be organized in an MVC pattern and this is reflected in the existing directories, they are the following:

- **Assets:** For storing images, SVG and other UI elements that are used in the software.

- **Controller:** Sole place where business logic happens, named up like "[thing]_controller"
    + The controllers can be structured with a final singular "Toucan_Controller" class.

- **View:** This is where the pages and the UI code should be declared, and nothing else.

- **Model:** Contains classes to encapsulate and abstract the information in controller.

- **util:** A folder for general helper functions made by us that are reused over and over.

- **router:** Place where all external communications with API endpoints are done at, should be renamed.

- **database:** To contain an embedded database if needed.

---

## What classes will be declared? (To do)

Probably not a lot of them, the core problem is kinda simple. For the controller we can declare four main ones in the form of capture_controller, LLM_controller, translation_controller and OCR_controller. For the model we need to declare few classes too, such as capture_moment, language, configuration... and so on. Domain based options such as what languages are supported can be declared in an enum on this same folder. Each class should have its own py file!!

For the view, we will use PySide6 so we can have a main window for controlling and configuring the tool alongside small frameless QWidget windows (such as the translucent translation box and region selector) on screen at once without multiplying memory usage. For the util I haven't really thought much about it, could be useful maybe, like utilities for image cropping or file management that are used across files, but we may not even use it at all.

---

## What technologies should we use?

For an initial release paddleOCR seems like a good idea for the OCR engine. We can also relegate the local hosting of ai in the application to llama.cpp.

For the Front end library we will use PySide6 (Qt6). We need to avoid Electron and even lighter web wrappers like pywebview (WebKitGTK) because spawning a browser engine per window eats heavy RAM alongside our OCR and LLM models, suffers from Wayland alpha rendering bugs, and lacks click-through overlay support. PySide6 gives us tiny multi-window memory usage, per-pixel translucent frameless widgets, CSS-like theming via QSS, easy UI zooming, and native Linux accessibility (AT-SPI2). GTK4 with gtk4-layer-shell remains a secondary alternative only if native Wayland layer-shell overlays are strictly needed.

For Wayland screen and window capture, we will use xdg-desktop-portal (ScreenCast) paired with PipeWire and GStreamer behind a decoupled capture adapter. Please read 4-Client and screen capture.md for a complete breakdown of the UI client and Wayland capture behavior.

For the database, we should definitely use one for storing and getting context for media, and for the sake of flexibility we can use a document based database. This database should have a **small** schema to follow as a base, and we need to set an option for this, a boolean field on the configuration, if the ai is smart enough we can send it the document, and it can rewrite and tweak it for itself, that would not be complicated and would be a good use of document DBs. It obviously needs to be an embedded database.

As for traditional machine translation we also can't afford to make a makeshift library and will instead reuse the work of others, python style with importing a package for that. Argos translate is probably the best idea for accomplishing this task.