# PyCExtension

![Learn Python C Extensions](images/logo.png)


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

The repo also serves as a template for starting Python C extension packages. It has the entire setup
to build and develop a C extension you can publish to the Python package index.

### Why make a C Extension?

C extesnions are fast, performant python libraries that can serve several purposes. Those include:

#### `High Performance`: 

C extensions can perform hundreds of times faster than equivalent code written in Python. 
This is because c functions are natively compiled, and just a thin layer over assembly code. Additionally, some
tasks can be slower to perform in Python, such as string processing. Python has no concept of a character, just strings of different lengths.
While C, has a very raw and effecient string composed purely of a block of memory terminated with a `\0` character. Overall,
C extensions provide a way to gain a powerhouse of performance in Python.

#### `Wrapping`:

Lots of widely used software libraries are written in C. However, many application level systems, like web development
framewroks, or mobile development frameworks, are written in languages like Java or Python. C functions can't be
called directly from Python, because Python does not understand C types without converting them to Python types. 
However, extensions can be used to wrap C code to make it callable from Python. The building and parsing of Python
types will be explained later.

#### `Low Level Tools`:

In Python, the degree to which one can utilize low level and operating system level utilities is 
quite limited. Python uses a Global Interpreter Lock (GIL), that allows only one thread at a time to execute
Python bytecode. This means that although some I/O bound tasks like file writes or network requests can
happen concurrently, access to Python objects and functions cannot.

With C, a program has complete and unrestricted freedom to any resources it can load and use. In a C extension,
the `GIL` can be released, allowing for multi-threaded python work flows.

## The Python C API

The Python language provides an extensive C API that allows you to compile and build C functions that
can accept and process Python typed objects. This is done through writing a special form of a C library,
that is not only linked with the Python libraries, but creates a *module object* the Python interpreter imports
like a regular Python module.

Before we get into the building steps, lets understand how a C function can process Python objects as input
and return Python objects as output. Let's look at the function below:

```c
#include <Python.h>

static PyObject* print_message(PyObject* self, PyObject* args)
{
    const char* str_arg;
    if(!PyArg_ParseTuple(args, "s", &str_arg)) {
        puts("Could not parse the python arg!");
        return NULL;
    }

    printf("msg %s\n", str_arg);
    // This can also be done with Py_RETURN_NONE
    Py_INCREF(Py_None);
    return Py_None;
}
```

The type, `PyObject*`, is the *dynamic* type that represents any Python object. You can think of it like a
base class, where every other Python object, like `PyBool` or `PyTuple` inherits from `PyObject`. The C
language has no *true* concept of classes. Yet, there are some tricks to implement an inheritance, polymorphic like system.
The details of this are beyond the scope of this guide, but one way to think about it is this:

```c

#define TYPE_INFO int type; \
                  size_t size

struct a_t {
    TYPE_INFO;
};

struct b_t {
    TYPE_INFO;
    char buf[20];
};

struct b_t foo;
// Fields are always ordered, this will work
(struct a_t*)(&foo)->type
```

In the above example, both `a_t` and `b_t` share the same fields at the beginning of their definitions.
