# PyCExtension

![Learn Python C Extensions](images/logo.png)

A template repo to learn about and develop Python C Extensions


## Intro

Python is one of the world's most popular programming languages. C is a super fast, older language
that inspired the syntax and protocols of many languages in use today. They are both powerful languages,
but using them together brings unparalleled performance and simplicity to the playing field.

This is a guide to learn how to write, build, and ship Python C Extensions. It assumes some programming experinece
with both Python and `C`. At the end of this guide, you will be able to make a small extension module that can process
small values as input like this:

```py
>>> import DemoPackage
>>> DemoPackage.print_message("hello!")
printer hello!
```

The repo also serves as a template for starting Python C extension packages.

### Why make a C Extension?