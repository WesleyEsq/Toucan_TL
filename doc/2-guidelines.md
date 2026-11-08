# How to actually write code here!

If you are reading this, you should be tasked with writing code for this project. There are some punctual things that I want to enforce in improving code quality, maintainability and good practices.

## Commit frequently

If you have changed more than a few files, stop to ask to commit and push your work on the dev branch. Commit messages need to describe what was done in a brief manner, with the title containing the verb enclosed in brackets[], words are add, Mod, fix, del and then the thing that you did.

## Commenting and internal docs.

We need to keep the public functions documented using docstring to describe the parameters (if any) as well as the return value (if any), above the docstring should be a bit of text detailing what the function even is and what does it help with, this needs to be human readable above all else for maintainability.

The header for each python file should follow this pattern:

``` python
#!/usr/bin/env python3

''' 
- file:   name_of_file.py
- date:   yyyy/mm/dd 
- by:     the name of the user
- desc:   one line description of the purpose of this file 
'''

import... #here would be imports and then the prod code...
```

We should also avoid having comments that obscure the production code, specially line comments with # are frivolously placed anywhere barely explaining the code, we can make use of them to the side of confusing sections of code, instead of making them completely cut the code time and time again. Example:

``` python
# This is great:
def foo ():
    '''function that helps in logging how many screen captures have happened'''
    val = do_something_super_duper_complicated() # We call a function that can give us the number of
    val ++                                       # times the screen captures have happened
    print("val")                                

# This is a practice I don't really like:
def foo ():
    '''function that helps in logging how many screen captures have happened'''
    # We call a function that can give us the number of times the screen captures have happened
    val = do_something_super_duper_complicated() 
    val ++                                       
    print("val")    
```

The internal documentation of code should form a part of purposefully explaining it too, we don't need a comment indicating why an addition happened or such, we need them to clarify confusing lines of code within the functions and the classes.

## Avoid large python files at all costs!!

Why are you writing a file that's long? That is where modularity can save us by using good practices. We should avoid files that are too large, if a file has more than 300 lines we should seriously consider refactoring them, the size of the files can be checked by a bash script or something cheap like that.

The work across the codebase can be restructured into an object oriented manner. By implementing encapsulation and classes we can reuse code more effectively, do mind that inheritance is weird and isn't a good idea for reusing code, in accordance to good practices in OOP, alongside others. We also need to abstract them so we can have simpler code that doesn't require as many fields, in our case it's images.

## Be careful with outdated dependencies.

Dependencies are the most important part of python, the package manager allows developers to import and use several packages to reduce their own code. Since we can check over the internet, we can avoid using tools that have fallen into disrepair or have changed their names with a newer release, python is python.

## Use helper and auxiliary functions to avoid code repetition

We can reuse code a lot, and that can help the maintainability of our software here, by putting a heavy use on auxiliary and helper functions we can also increase code understandability. When it comes to backend development we can make classes that separate the initialization of a class or file in an init file, which is composed also of smaller functions, instead of having long codeblocks that decrease understandability.

The use of components when coding the UI is also important, we can reuse front end components not only on this project but on more projects as well in the future, so an important stepping stone in the UI is that everything has to be implemented with reusable components. With PySide6, each UI component should be a small, self-contained QWidget class styled through shared QSS stylesheets without mixing controller logic into it.

## How to keep and maintain external documentation

I do believe that AI can handle this job, this is ok, but we need to mind that the documentation is readable not only for ourselves but for other people, and be presentable. There are common mistakes the AI makes when writing markdown files, here are the most easy to solve:

1. ***Too `many` random uses of markdown!*** Markdown is meant to both be readable when compiled and when it's on its source code, reading the file from notepad or similar tools. 
    
    + The AI prefers to write for the compiled state, with extremely long tables that are hard to understand from the file. 
    
    + It also abuses a lot of features that markdown has `such as highlighting technical words like this for no reason at all`, **excessive use of bold** *or italic* and specially the preference of using tables.

    + I do prefer the ai to just write a long list of points *, that is honestly more understandable, but the existence of **small** paragraphs is important for helping the reader go through the document.

2. **Markdownlint based issues** It does annoy me that the AI prefers to bundle together sentences and lists of items for the sake of making the file smaller, it decreases its readability a lot. By this, I mean that the ai tends to write text immediately after a # header instead of giving it separation.

3. **Large markdown files:** A thing that I'm not fond of. If a file has too many topics on itself we should split it so each file is about a specific topic, we cannot have a file that's 200 lines long explaining the entire architecture of the software, that needs to be refactored into sequential pieces.

4. **Where should the docs live?** A common practice is to keep them all enclosed in a doc folder, just like this one, where we either make a parallel directory to every module in the application to explain each one. I don't really like this approach, but I do know that this is how it should be done usually, and internal markdown documentation nowadays is for helping agents understand our code better.
