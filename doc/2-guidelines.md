# How to write code and docs here

If you are reading this, you are likely working on this codebase. These guidelines keep the code modular, consistent, and straightforward to maintain.

---

## Commit frequently

If you have changed more than a few files, stop and ask before committing and pushing your work to the `dev` branch. Commit titles should briefly describe what was done, starting with the verb in brackets (`[add]`, `[Mod]`, `[fix]`, or `[del]`), followed by the change.

---

## Commenting and internal docs

Document public functions when their purpose, parameters, or return value are not already clear from well-chosen names and type hints. Write docstrings in plain English explaining what the function helps do, instead of repeating the signature or adding boilerplate to satisfy a check.

Each Python module should start with a short module docstring describing its purpose. Only executable scripts need a shebang, and author or date headers are unnecessary because version control tracks them:

```python
"""One-line description of the purpose of this module."""

# Imports and production code follow.
```

Avoid line comments (`#`) that break up readable code or repeat what the code already says. Use short side comments only when a line or expression has non-obvious behavior:

```python
# Useful: clarifies a non-obvious detail.
capture_count = get_complicated_capture_count()  # Includes captures restored after reconnecting.

# Redundant: repeats an already clear call.
capture_count = get_capture_count()
```

---

## Keep Python files focused and small

If a file grows past roughly 300 lines, check whether it has taken on too many responsibilities. Treat that length as a review signal rather than a rigid limit; keep a cohesive class or algorithm together instead of breaking it apart just to reduce a line count.

Use object-oriented structure where encapsulation clarifies ownership, particularly around images, OCR, and translation providers. Prefer composition over inheritance for reuse, while avoiding unnecessary classes for small values or simple helpers. Abstractions should reduce coupling and clarify ownership rather than just hiding fields.

---

## Dependencies

Dependencies let us reuse proven OCR, translation, and capture libraries instead of maintaining our own. Pin and record the versions we support, check maintenance activity and licenses, and inspect native-library, model, and packaging requirements before adding a package. Updates should be deliberate and tested afterward.

---

## Helpers and UI components

Extract helper functions when the same behavior appears in multiple places, without splitting clear control flow into excessive tiny wrappers. Keep initialization readable and close to the object it sets up; avoid moving setup code into `__init__.py` merely to shorten another file.

In the UI layer, extract a `QWidget` component when it has an independent role or is genuinely reused. Use shared QSS stylesheets for common styling, and keep controller and pipeline logic outside the views.

---

## Keeping external documentation readable

External documentation inside `doc/` helps both people and coding agents understand the system, so it needs to stay clear both when rendered and when read as raw markdown. Follow these habits when writing or updating docs:

1. **Keep the tone plain and direct:** Write straightforward, matter-of-fact technical prose where sentences connect cleanly from one point to the next. Avoid chatty filler on one hand, and stiff, repetitive disclaimers (*"this is an option, not a requirement"*) on the other.
2. **Avoid noisy formatting:** Do not scatter inline backticks on ordinary terms, overuse **bold** or *italics*, or rely on wide markdown tables that are hard to read in a plain text editor.
3. **Give structure breathing room:** Use short paragraphs alongside lists instead of cramming headings, prose, and lists together without blank lines.
4. **Say each rule once:** Put a design decision in the document where it belongs and link to it elsewhere rather than repeating the same constraints across multiple files.
5. **Keep documents focused:** Around 200 lines is a good cue to check whether a file covers too many topics and should be split, without fragmenting one coherent explanation unnecessarily.
6. **Keep docs alongside code:** Store module and architecture notes inside `doc/` matching the application layout so the structure stays predictable.
