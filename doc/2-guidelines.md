# How to actually write code here!

If you are reading this, you should be tasked with writing code for this project. There are some punctual things that I want to enforce in improving code quality, maintainability and good practices.

## Commit frequently

If you have changed more than a few files, stop to ask to commit and push your work on the dev branch. Commit messages need to describe what was done in a brief manner, with the title containing the verb enclosed in brackets[], words are add, Mod, fix, del and then the thing that you did.

## Commenting and internal docs.

We need to keep public functions documented when their purpose, parameters or return value are not obvious from clear names and types. The docstring should explain what the function helps with in human readable terms. Avoid repeating the signature or adding documentation just to satisfy a count.

Python modules should have a short module docstring describing their purpose. The shebang is only needed for executable scripts, and author or date headers are optional because version control already tracks them. A file header can follow this pattern when it is useful:

```python
"""One line description of the purpose of this module."""

# Imports and production code follow.
```

We should also avoid having comments that obscure the production code, specially line comments with # are frivolously placed anywhere barely explaining the code, we can make use of them to the side of confusing sections of code, instead of making them completely cut the code time and time again. Example:

```python
# A short comment can clarify a confusing expression when the code cannot make it clear.
capture_count = get_complicated_capture_count()  # Includes captures restored after reconnecting.

# Do not add a comment that merely repeats an already clear function call.
capture_count = get_capture_count()
```

The internal documentation of code should form a part of purposefully explaining it too, we don't need a comment indicating why an addition happened or such, we need them to clarify confusing lines of code within the functions and the classes.

## Avoid large python files at all costs!!

Why are you writing a file that's long? That is where modularity can save us by using good practices. If a file has more than 300 lines, seriously consider whether it has too many responsibilities. This is a review signal, not a hard limit: do not split a cohesive class or algorithm into fragments just to satisfy a line count.

The work across the codebase can be structured in an object oriented manner where encapsulation helps. Prefer composition over inheritance for reuse, but do not turn every small value or helper into a class. Abstractions should reduce coupling and clarify ownership, especially around images, OCR and providers, rather than just reduce the number of visible fields.

## Be careful with outdated dependencies.

Dependencies let us reuse work instead of maintaining our own OCR, translation and capture implementations. Pin and record the versions we support, check maintenance and licenses, and review native-library, model and packaging requirements before adopting them. Updates should be deliberate and tested rather than based on whichever package name appears newest.

## Use helper and auxiliary functions to avoid code repetition

We can reuse code where the same behavior genuinely appears more than once. Helper functions can improve understandability, but too many tiny helpers can also hide the flow. Keep initialization readable and close to the object it initializes; do not move it into an `__init__.py` file just to make another file shorter.

The use of components when coding the UI is also important. Extract a QWidget component when it has a clear independent role or is actually reused; do not design every widget for unknown future projects. Shared QSS can handle common styling, while controller and pipeline logic stays outside the view.

## How to keep and maintain external documentation

I do believe that AI can handle this job, this is ok, but we need to mind that the documentation is readable not only for ourselves but for other people, and be presentable. There are common mistakes the AI makes when writing markdown files, here are the most easy to solve:

1. ***Too `many` random uses of markdown!*** Markdown is meant to both be readable when compiled and when it's on its source code, reading the file from notepad or similar tools. 
    
    + The AI prefers to write for the compiled state, with extremely long tables that are hard to understand from the file. 
    
    + It also abuses a lot of features that markdown has `such as highlighting technical words like this for no reason at all`, **excessive use of bold** *or italic* and specially the preference of using tables.

    + I do prefer the ai to just write a long list of points *, that is honestly more understandable, but the existence of **small** paragraphs is important for helping the reader go through the document.

2. **Markdownlint based issues** It does annoy me that the AI prefers to bundle together sentences and lists of items for the sake of making the file smaller, it decreases its readability a lot. By this, I mean that the ai tends to write text immediately after a # header instead of giving it separation.

3. **Large markdown files:** A thing that I'm not fond of. If a file has too many topics, split it so each file is about a specific topic. Around 200 lines is a reason to review the structure, not an automatic reason to fragment one coherent explanation.

4. **Where should the docs live?** A common practice is to keep them all enclosed in a doc folder, just like this one, where we either make a parallel directory to every module in the application to explain each one. I don't really like this approach, but I do know that this is how it should be done usually, and internal markdown documentation nowadays is for helping agents understand our code better.
