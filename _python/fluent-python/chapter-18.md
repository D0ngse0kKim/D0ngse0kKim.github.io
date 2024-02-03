---
layout: distill
title: Chapter 18
description: with, match, and else Blocks
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstarct
  - name: Context Managers and with Blocks
  - name: Pattern Matching in lis.py A Case Study
  - name: Do This Then That else Blocks Beyond if
  - name: Chapter Summary
  - name: Links
---

## Abstarct

This is a summary of the chapter 18 of the book "Fluent Python" by Luciano
Ramalho.

This chapter covers the following topics:

1. The `with` statement and context managers
2. The `match`/`case` statement for pattern matching
3. The `else` block in `for`, `while`, and `try` statements

## Context Managers and with Blocks

The `with` statement was introduced in Python 2.5, and it is used to simplify
the `try/finally` pattern [^pythonorg__try_finally].

The `try/finally` pattern is used to ensure that some operation is performed
after a block of code, even if the block is terminated by

1. `return` statement,
2. raising an exception, or
3. a `sys.exit()` call.

A code block that starts with `finally` clause usually performs cleanup actions
such as releasing a critical resource or restoring some previous state that was
temporarily changed.

The `with` statement simplifies the `try/finally` pattern by encapsulating the
cleanup logic in so-called *context managers*. There is some examples from
the standard library:

1. "How to use the connection context manager" in `sqlite3` module documents
    [^pythonorg__sqlite3_context_manager]
2. "Using locks, conditions, and semaphores in the with statement" in
    the `threading` module documents [^pythonorg__threading_example_with]
3. Usage of `decimal.localcontext()` for temporary changes to the precision
    and rounding rules for the `Decimal` class
    [^pythonorg__decimal_example_with]
4. Patching objetc for testing with `unittest.mock.patch()`
    [^pythonorg__unittestmock_example_with]

### *The context manager interface*

The context manager interface consists of the following methods:

`__enter__()`
: <!---->

* This method is called with no arguments when the execution flow enters
    the `with` block.
* return value: the object to be bound to the target variable in the `with`
    statement

`__exit__(exc_type, exc_value, traceback)`
: <!---->

* This method is called when the execution flow exits the `with` block for
    any reason.
* `exc_type` : exception class (e.g. `ZeroDivisionError`)
* `exc_value` : exception instance (e.g.
    `ZeroDivisionError("integer division or modulo by zero")`)
* `traceback` : traceback object
* return value: `True` if the exception was handled, `False` or `None` if
    it was not handled

### *Simple example of a context manager*

```python
>>> with open('hello.txt', 'w') as fp:  # <1>
...     fp.write('hello world!')  # <2>
...
>>> fp  # <3>
<_io.TextIOWrapper name='hello.txt' mode='w' encoding='UTF-8'>
>>> fp.closed, fp.encoding  # <4>
(True, 'UTF-8')
>>> fp.write('hello again!')  # <5>
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: I/O operation on closed file.
```

* `<1>` : `open()` returns a `TextIOWrapper` object, and a return value from
    the `__enter__()` method of `TextIOWrapper` is bound to the variable `fp`.
    * The `TextIOWrapper` object is a context manager that implements
        the `__enter__()` and `__exit__()` methods.
* `<2>` : `fp.write()` is called to write a string to the file.
* `<3>` : `fp` is still bound to the `TextIOWrapper` object.
    * **Note**: `with` block does not define a new scope.
* `<4>` : `fp` is closed and the encoding is set to `'UTF-8'`.
    * `__exit__()` method of `TextIOWrapper` is called and `fp.closed` is set
        to `True` when the execution flow exits the `with` block.
* `<5>` : `fp.write()` raises a `ValueError` because `fp` is closed.

Important points are as follows:

1. The context manager is a result of evaluating the expression after the
    `with` keyword.
2. The context manager exists until the execution flow exits the `with` block
    and the `__exit__()` method of the context manager is called.
3. The bounded variable is assigned the return value of the `__enter__()`
    method of the context manager.
4. `with` block does not define a new scope.
    * The context manager object is bound to the variable in the `with`
        statement.

### *Example of a context manager class*

In this subsection, a class that implements the context manager interface is
implemented. The following code demonstrates how to use the context manager
class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='18-with-match/mirror.py'
     language='python'
     line='9-19'
></div>

* `<1>` : The context manager is an instance of the `LookingGlass` class.
    Python invokes the `__enter__()` method of the `LookingGlass` class and
    binds the return value of the `__enter__()` method to the variable `what`.
* `<2>` : The context manager monkey-patches the `sys.stdout.write` object.
    Therefore, the output of each `print()` will be reversed.
* `<3>`, `<4>` : The context manager is exited and the `__exit__()` method of
    the context manager object is called.
    * The `__exit__()` method restores the original `sys.stdout.write` object.
    * The output of program is restored to normal.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='18-with-match/mirror.py'
     language='python'
     line='72-89'
></div>

* `<1>`, `<4>` : The `__enter__()` method with no arguments is called when the
    execution flow enters the `with` block. The return value `JABBERWOCKY` of
    the `__enter__()` method will be bound to the variable in the `with`
    clause.
* `<2>`, `<3>`, `<5>` : The `__enter__` method monkey-patches
    the `sys.stdout.write` method by the `self.reverse_write(text)` method.
    * The `self.reverse_write(text)` method reverses the text and calls the
        original `sys.stdout.write` method.
* `<6>` : The `__exit__()` method is called when the execution flow exits the
    `with` block.
    * When the execution flow exits the `with` block without an exception,
        `exc_type`, `exc_value`, and `traceback` are `None`.
    * If an exception is raised in the execution of the `with` block,
        `exc_type`, `exc_value`, and `traceback` are the exception class,
        exception instance, and traceback object, respectively.
* `<7>` :  The `__exit__()` method restores the original `sys.stdout.write`
    method.
* `<8>`, `<9>` : When the execution flow exits the `with` block with
    a `ZeroDivisionError` exception, the `__exit__()` method prints an error
    message and returns `True` to indicate that the exception was handled.
* `<10>` : If the `__exit__()` method returns `False` or `None`, the exception
    is re-raised after the `__exit__()` method returns.

In the following example, `ZeroDivisionError` is raised in the `with` block.
Because `True` is returned from the `__exit__()` method, the exception is not
delegated to the caller.

```python
>>> with LookingGlass() as what:
...     print("abcd")
...     print(1/0)
...
dcba
Please DO NOT divide by zero!
```

For a detailed explanation of the `__enter__()` and `__exit__()` methods, see
the following codes which does the same thing as the `with` statement.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='18-with-match/mirror.py'
     language='python'
     line='28-41'
></div>

* `<1>` : The variable `manager` holds the context manager object returned by
    the `LookingGlass()` constructor.
* `<2>` : The `__enter__()` method of the context manager object is called and
    the return value `JABBERWOCKY` is bound to the variable `monster`.
* `<4>` : Call `manager.__exit__(None, None, None)` to restore the original
    `sys.stdout.write` method.

### *Parenthesized context managers in Python 3.10*

In Python 3.10[^pythonorg__contextlib_parenthesized_context_managers],
the `with` statement allows multiple context managers to be specified in
a single statement. The following code is an example of using multiple
context managers in a single `with` statement.

```python
with (
    CtxManager1() as var1,
    CtxManager2() as var2,
    CtxManager3() as var3,
):
    # do something
```

### The contextlib Utilities

There are some utilities in the `contextlib` module [^pythonorg__contextlib]
that help to create context managers.

#### *Functions for creating context managers*

`contextlib.closing(thing)` [^pythonorg__contextlib_closing]
: This utility creates a context manager that closes the argument object
    using the `thing.close()` method.

`contextlib.suppress(*exceptions)` [^pythonorg__contextlib_suppress]
: This utility creates a context manager that suppresses the specified
    exceptions temporarily.

`contextlib.nullcontext(enter_result=None)` [^pythonorg__contextlib_nullcontext]
: This utility creates a context manager that returns the `enter_result` when
    the `__enter__()` method is called and does nothing when the `__exit__()`
    method is called.

#### *Decorators, ABC for creating context managers*

`@contextlib.contextmanager` [^pythonorg__contextlib_contextmanager]
: This decorator converts a generator function into a context manager. The
    usage of this decorator will be explained in the next subsection.

`class contextlib.AbstractContextManager` [^pythonorg__contextlib_abstractcontextmanager]
: An abstract base class for creating a class for context managers.
The following methods are provided by this class:
* `__enter__()` : This method is a concrete method which returns `self` by
    default. It is possible to override this method to return a different
    object.
* `__exit__()` : This method is an abstract method which raises
    `NotImplementedError` by default. It is necessary to override this method
    to implement a context manager.

`class contextlib.ContextDecorator` [^pythonorg__contextlib_contextdecorator]
: A base class that enables a context manager to also be used as a decorator.

`class contextlib.ExitStack` [^pythonorg__contextlib_exitstack]
: This class is a context manager that can be used to combine multiple context
managers. The `ExitStack` class is useful when the number of context
managers is not known in advance. For example, the `ExitStack` class can be
used to close multiple files.

To use the `ExitStack` class, instantiate the `ExitStack` class and use the
object of `ExitStack` class as a context manager with `with` clause. The
return value of `__enter__()` method of the `ExitStack` object has
the `enter_context()` method. This method is used to add a context manager
to the `ExitStack` object. The `enter_context()` method returns the context
manager object that is added to the `ExitStack` object.

When the execution flow exits the `with` block, the `__exit__()` method of
each context manager is called in the reverse order of the `__enter__()`
method calls.

```python
with ExitStack() as stack:
    files = [stack.enter_context(open(fname)) for fname in filenames]
    # All opened files will automatically be closed at the end of
    # the with statement, even if attempts to open files later
    # in the list raise an exception
```

### Using @contextmanager

The `@contextmanager` decorator is used to convert a generator function into
a context manager. By using this decorator, it is possible to create a context
manager without defining a class that implements the context manager interface.

In a generator decorated with `@contextmanager`, `yield` is used to split the
body of the function into two parts:

* The code before the `yield` statement is executed in the `__enter__()` method
    of the context manager.
* The code after the `yield` statement is executed in the `__exit__()` method
    of the context manager.
* The yielded value is bound to the variable in the `with` statement.

The following code is an example of using the `@contextmanager` decorator.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='2f2f87d4fb297869cb5763ec9b2ffcad3e841d20'
     path='18-with-match/mirror_gen.py'
     language='python'
     line='66-78'
></div>

* `<1>` : The `@contextlib.contextmanager` decorator convert
    the `looking_glass()` generator function into a context manager.
* `<2>` : `original_write` holds the original `sys.stdout.write` method.
* `<3>` : Defines the reversed version of `sys.stdout.write` method as
    a `reverse_write()` function.
* `<4>` : Monkey-patches the `sys.stdout.write` method into
    the `reverse_write()` function.
* `<5>` : The yielded value is bound to the variable in the `with` statement.
* `<6>` : Restores the original `sys.stdout.write` method.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='2f2f87d4fb297869cb5763ec9b2ffcad3e841d20'
     path='18-with-match/mirror_gen.py'
     language='python'
     line='9-19'
></div>

* `<1>` : Invoking the `looking_glass()` function returns a context manager
    object. By applying the `with` statement to the context manager object,
    the `__enter__()` method of the context manager object is called and the
    return value `JABBERWOCKY` is bound to the variable `what`.

If an exception is raised in the body of the block started by the `with`
statement, the execution flow is executed the following steps:

1. The Python interpreter catches the exception when the exception is raised.
2. The execution flow is transferred into the `yield` statement in
    the `looking_glass()`, and the Python interpreter raises the exception
    again.
3. After the exception is raised, the `looking_glass()` generator is terminated
    and the remaining code after the `yield` statement in the `looking_glass()`
    is not executed.

The following code is an example that raises an exception in the body of the
block started by the `with` statement.

```python
>>> with looking_glass() as what:
...      print('Alice, Kitty and Snowdrop')
...      print(f'{1/0}')  # <1>
...
pordwonS dna yttiK ,ecilA
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
ZeroDivisionError: division by zero
>>> what  # <2>
'YKCOWREBBAJ'
>>> print('stdout is not restored!')  # <3>
!derotser ton si tuodts
```

* `<1>` : `ZeroDivisionError` is raised in the body of the block started by
    the `with` statement. The remaining code after the `yield` statement in
    the `looking_glass()` is not executed.
* `<2>`, `<3>` : Therefore, the original `sys.stdout.write` method is not
    restored and the output of `print()` is reversed.

To handle this flaw on the original code of the `looking_glass()` function,
the `try:`, `except:`, and `finally:` pattern on the `yield` statement can be
used. The following code is an example of using the `try:`, `except:`, and
`finally:` pattern on the `yield` statement.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='18-with-match/mirror_gen_exc.py'
     language='python'
     line='78-97'
></div>

* `<1>` : The `msg` variable holds the exception message. By default, the
    `msg` variable is an empty `str` object.
* `<2>` : The `try:`, `except:`, and `finally:` pattern is used on the `yield`
    statement. In this pattern, `ZeroDivisionError` is caught and the
    exception message is assigned to the `msg` variable.
* `<3>`, `<4>` : The `finally:` block implements the `__exit__()` method parts
    of a context manager. In this block of code, the original `sys.stdout.write`
    method is restored and prints the exception message.

```python
>>> with looking_glass() as what:
...      print('Alice, Kitty and Snowdrop')
...      print(f'{1/0}')  # <1>
...
pordwonS dna yttiK ,ecilA
Please DO NOT divide by zero!  # <2>
>>> what  # <3>
'JABBERWOCKY'
>>> print('back to normal')  # <4>
back to normal
>>> with looking_glass() as what:
...      print('Alice, Kitty and Snowdrop')
...      print(f'{1 + [1, 2]}')
...
pordwonS dna yttiK ,ecilA
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
TypeError: unsupported operand type(s) for +: 'int' and 'list'
>>> print('back to normal again...')  # <5>
```

* `<1>` : `ZeroDivisionError` is raised in the body of the block started by
    the `with` statement.
* `<2>` : The exception message is printed.
* `<3>`, `<4>` : The original `sys.stdout.write` method is restored.
* `<5>` : The original `sts.stdout.write` method is restored. In case of
    `TypeError`, the exception is not caught by the `except ZeroDivisionError:`.
    Therefore, any error message is not printed.

The decorated function can be used as a decorator. The following code is an
example of using the decorated function as a decorator.

```python
>>> @looking_glass()
... def prints():
...     print('Alice, Kitty and Snowdrop')
...     print(f'{1/0}')
...
>>> prints()  # <1>
pordwonS dna yttiK ,ecilA
Please DO NOT divide by zero!
>>> print('back to normal')
back to normal
```

### *Real-world example of a contextmanager decorator*

The following code is an example of using the `@contextlib.contextmanager`
decorator which is described in this link [^MartijnPieters_easyinplacefilerewriting].

In this example, the following `inplace()` function is used to replace
the content of a file with new content.

```python
import csv

with inplace(csvfilename, 'r', newline='') as (infh, outfh):
    reader = csv.reader(infh)
    writer = csv.writer(outfh)

    for row in reader:
        row += ['new', 'columns']
        writer.writerow(row)
```

The implementation of the `inplace()` function is as follows:

```python
from contextlib import contextmanager
import io
import os


@contextmanager
def inplace(filename, mode='r', buffering=-1, encoding=None, errors=None,
            newline=None, backup_extension=None):
    """Allow for a file to be replaced with new content.

    yields a tuple of (readable, writable) file objects, where writable
    replaces readable.

    If an exception occurs, the old file is restored, removing the
    written data.

    mode should *not* use 'w', 'a' or '+'; only read-only-modes are supported.

    """

    # move existing file to backup, create new file with same permissions
    # borrowed extensively from the fileinput module
    if set(mode).intersection('wa+'):
        raise ValueError('Only read-only file modes can be used')

    backupfilename = filename + (backup_extension or os.extsep + 'bak')
    try:
        os.unlink(backupfilename)
    except os.error:
        pass
    os.rename(filename, backupfilename)
    readable = io.open(backupfilename, mode, buffering=buffering,
                       encoding=encoding, errors=errors, newline=newline)
    try:
        perm = os.fstat(readable.fileno()).st_mode
    except OSError:
        writable = open(filename, 'w' + mode.replace('r', ''),
                        buffering=buffering, encoding=encoding, errors=errors,
                        newline=newline)
    else:
        os_mode = os.O_CREAT | os.O_WRONLY | os.O_TRUNC
        if hasattr(os, 'O_BINARY'):
            os_mode |= os.O_BINARY
        fd = os.open(filename, os_mode, perm)
        writable = io.open(fd, "w" + mode.replace('r', ''), buffering=buffering,
                           encoding=encoding, errors=errors, newline=newline)
        try:
            if hasattr(os, 'chmod'):
                os.chmod(filename, perm)
        except OSError:
            pass
    try:
        yield readable, writable
    except Exception:
        # move backup back
        try:
            os.unlink(filename)
        except os.error:
            pass
        os.rename(backupfilename, filename)
        raise
    finally:
        readable.close()
        writable.close()
        try:
            os.unlink(backupfilename)
        except os.error:
            pass
```

In this implementation, the `@contextlib.contextmanager` decorator is used to
convert the `inplace()` function into a context manager. The `inplace()`
function returns a context manager object.

The file specified with the parameter `filename` is renamed to
`filename + '.bak'` in the `backupfilename = ...` statement. The `backupfilename`
is used to restore the original file when an exception is raised in the body
of the block started by the `with` statement.

The `yield` statement in the `inplace()` function returns a tuple of
`(readable, writable)` file objects. The `readable` file object is a file
object that is opened in the `inplace()` function. The `writable` file object
is a file object that is opened in the `inplace()` function. The `writable`
file object replaces the `readable` file object when the execution flow exits
the `with` block.

### *Implementation of contextmanager decorator*

The following code is an implementation of the `@contextlib.contextmanager`
decorator.

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='272-302'
></div>

The `@contextlib.contextmanager` decorator has one argument, `func`, which is
a generator function.
The `@contextlib.contextmanager` decorator returns an `helper()` function
which is a closure function that returns
`_GeneratorContextManager(func, args, kwds)` object.
Note that the `@wraps(func)` decorator is used to copy the `__name__` and
`__doc__` attributes of the `func` function to the `helper()` function.

The `_GeneratorContextManager` class implements the context manager interface.
In the implementation of the `_GeneratorContextManager` class, there is no
implementation of `__init__()` method. Therefore, it is needed to investigate
the parent class of the `_GeneratorContextManager` class to understand how the
`_GeneratorContextManager` class is initialized.

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='125-130'
></div>

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='101-122'
></div>

In the `_GeneratorContextManagerBase` class, there is an implementation of
`__init__()` method. In this method, `self.gen` is initialized by
`func(*args, **kwds)`. In addition, `self.func`, `self.args`, `self.kwds` and
`self.__doc__` is initialized.

The property `self.gen` is a generator object that is created by calling
`func(*args, **kwds)`. This is used in the `__enter__()` method of the
`_GeneratorContextManager` class. `self.args`, `self.kwds`, and `self.func` is
deleted in the `del` statement.

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='125-139'
></div>

In the `__enter__()` method of the `_GeneratorContextManager` class,
the decorated generator function is initialized by calling `next(self.gen)`.
The `next(self.gen)` call executes the code before the `yield` statement in
the decorated generator function. The yielded value is bound to the variable
in the `with` statement.

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='141-190'
></div>

The execution flow is divided into two parts by the `typ` is None or not.

If `typ` is `None`, any exception is not raised in the body of the block
started by the `with` statement.
In this case, the `__exit__()` method of the `_GeneratorContextManager` class
is called with `None`, `None`, and `None` as arguments.
The `__exit__()` method of the `_GeneratorContextManager` class calls
`next(self.gen)` to execute the code after the `yield` statement in
the decorated generator function.


If `typ` is not `None`, an exception is raised in the body of the block started
by the `with` statement.
In this case, the `__exit__()` method of the `_GeneratorContextManager` class
is called with `typ`, `val`, and `traceback` as arguments.
The `__exit__()` method of the `_GeneratorContextManager` class calls
`self.gen.throw(typ, val, traceback)` to raise the exception in the decorated
generator function. The details of the `throw()` method of the generator object
can be found in the this link[^pythonorg__pep0342_throw].

A generator function decorated with `@contextlib.contextmanager` can be used
as a decorator. This functionality is implemented by the `ContextDecorator`
class that is a parent class of the `_GeneratorContextManager` class.

<div class='embed-github-src'
     repo='python/cpython/'
     branch='3.11'
     path='Lib/contextlib.py'
     language='python'
     line='62-82'
></div>

By the `__call__` method in the `ContextDecorator` class, the decorated
generator function is called with `*args` and `**kwds` as arguments in the
`with self._recreate_cm()` statement. The `self._recreate_cm()` method returns
`self`, therefore the context manager described in the implementation of
the `_GeneratorContextManager` class will be used.

## Pattern Matching in lis.py: A Case Study

In this section, real-world usage of `match`, `case` statement will be
explained by the `lis.py` module which is a Lisp interpreter written by
Peter Norvig [^lispy].

### Scheme Syntax

Scheme is a dialect of Lisp. The following code is an example of Scheme syntax.

```
(define (mod m n)
    (- m (* n (quotient m n))))

(define (gcd m n)
    (if (= n 0)
        m
        (gcd n (mod m n))))

(display (gcd 18 45))
```

In the Scheme syntax, there is no distinction between statements and
expressions. Also, there is no infix operator. All operators are prefix
operators, for example, `(+ 1 2)` is an expression that is equivalant to `1+2`
in Python.
The parentheses are used to group expressions.

The following Python code is equivalant to the Scheme code above.

```python
def mod(m, n):
    return m - (m // n * n)

def gcd(m, n):
    if n == 0:
        return m
    else:
        return gcd(n, mod(m, n))

print(gcd(18, 45))
```

The details of the Scheme expressions can be found in the following links
[^scheme_expressions] [^lispy].

The following subsections explain details of the `lis.py` module. In the Fluent
Python book, minor revised version of the `lis.py` module is used.

### Imports and Types

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='13-21'
></div>

The type aliases `Symbol`, `Atom` and `Expression` are defined in the
`lis.py` module.

`Symbol`
: A type alias of `str`. `Symbol` is used to represent identifiers in Scheme.
    In the sample code, assume that there is no string data type in Scheme for
    simplicity. Therefore, `Symbol` is used to represent only identifiers.

`Atom`
: A type alias of `Union[int, float, Symbol]`. `Atom` is used to represent
    integers, floating point numbers and identifiers in Scheme.

`Expression`
: A type alias of `Union[Atom, List]`. `Expression` is used to represent
    expressions in Scheme. `Expression` is a recursive data type.

### The Parser

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='27-59'
></div>

The entry point of the parser of the Scheme syntax in `lis.py` is the `parse()`
function.

`def parse(program: str) -> Expression:`
: This function accepts a string that represents a Scheme program and returns
    an `Expression` object that represents the Scheme program.

`def tokenize(s: str) -> list[str]:`
: This function accepts a string that represents a Scheme program and returns
    a list of tokens. The tokens are separated by whitespace characters.
    By using this property, adding whitespace characters before and after each
    parenthesis and using `split()` on the string, the list of tokens can be
    obtained.

`def read_from_tokens(tokens: list[str]) -> Expression:`
: This function accepts a list of tokens and returns an `Expression` object
    that represents the Scheme program. This function is a recursive function.
    The `read_from_tokens()` function calls the `read_from_tokens()` function
    recursively to parse the nested expressions.
1. When an open parenthesis is found, the `read_from_tokens()` function
    generates an empty list for the nested expression.
2. And then, the `read_from_tokens()` function calls the `read_from_tokens()`
    function recursively to parse the nested expression.
3. When a close parenthesis is found, the `read_from_tokens()` function
    returns the list of tokens that represents the nested expression.
4. When an atom is found, the `read_from_tokens()` function returns the atom by
    invoking the `parse_atom()` function.

`def parse_atom(token: str) -> Atom:`
: This function accepts a token and returns an `Atom` object that represents
    the token.
1. If the token is an integer, the `int()` function is used to convert
    the token into an integer.
2. If the token is a floating point number, the `float()` function is used to
    convert the token into a floating point number.
3. Otherwise, the `Symbol()` function is used to convert the token into a
    `Symbol` object.

The following code is an example of using the `parse()` function.

```python
>>> from lis import parse
>>> parse('1.5')
1.5
>>> parse('(gcd 15 6)')
['gcd', 15, 6]
>>> parse('''
... (define (gcd m n)
...     (if (= n 0)
...         m
...         (gcd n (mod m n))))
... ''')
['define', ['gcd', 'm', 'n'], ['if', ['=', 'n', 0], 'm', ['gcd', 'n', ['mod', 'm', 'n']]]]
```

### The Environment

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='65-74'
></div>

The `Environment` class is used to represent the environment of the Scheme
interpreter. The `Environment` class extends the `collections.ChainMap` class
[^pythonorg__collections_chainmap] by adding the `change()` method.

The `change()` method of the `Environment` class is a method that is used to
update existing keys. If the key is not found in the `Environment` object,
the `change()` method raises a `KeyError` exception.

The following code shows how to use the `Environment` class.

```python
>>> from lis import Environment
>>> env_first = {'a': 1, 'b': 2}
>>> env_second = {'b': 3, 'c': 4}
>>> env = Environment(env_first, env_second)
>>> env['a']
1
>>> env['b']  # <1>
2
>>> env['c']
4
>>> env['a'] = 10  # <2>
>>> env['d'] = 5  # <3>
>>> env
Environment({'a': 10, 'b': 2, 'd': 5}, {'b': 3, 'c': 4})
>>> env.change('c', 40)  # <4>
>>> env
Environment({'a': 10, 'b': 2, 'd': 5}, {'b': 3, 'c': 40})
>>> env.change('b', 20)  # <5>
>>> env
Environment({'a': 10, 'b': 20, 'd': 5}, {'b': 3, 'c': 40})
```

* `<1>` : The `b` key is found in the `env_first` object. Therefore, the value
    of the `b` key in the `env_first` object is returned.
* `<2>`, `<3>` : Assigning a value to the `Environment` object with `[]` always
    overwrites or add a key-value pair in the first mapping object, `env_first`.
* `<4>` : The `change()` method is used to update the value of the `c` key
    in-place in the `env_second` object.
* `<5>` : The `change()` method is used to update the value of the `b` key
    in-place in the `env_first` object.

By using this `Environment` class, the `lis.py` module implements the
`standard_env()` function which returns the global environment.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='77-116'
></div>

To sum up the above `standard_env()` function, the following functions and
constants are defined in the global environment:

* All functions from the `math` module.
* Some functions from the `operator` module.
* Simple functions built with `lambda` expressions and built-in functions in
    Python.

### The REPL

REPL stands for read-eval-print-loop. The following code is the implementation
of the REPL by the Norvig's `lis.py` module.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='122-136'
></div>

`def repl(prompt: str = 'lis.py> ') -> NoReturn:`
: The `repl()` function is an infinite loop that reads a Scheme program from
the standard input, evaluates the Scheme program, and prints the result.
`NoReturn` represents that the function does not return and loops forever.
This function processes the following steps:

1. set `global_env` by using `Environment` class and `standard_env()`.
2. jump into the infinite loop.
3. get the input from the standard input by using the `input()` function.
4. parse the input by using the `parse()` function.
5. evaluate the parsed input by using the `evaluate()` with `global_env`.
6. if the result is not `None`, print the result with `lispstr()`

`def lispstr(exp: object) -> str:`
: Convert a python object into a string that is readable by the Scheme
interpreter. For example, the `lispstr()` function converts `['+', 1, 2]` into
`"(+ 1 2)"`.

### The Evaluator

The evaluator in the `lis.py` uses the `match`, `case` statement to evaluate
the Scheme program. The following code is the implementation of the evaluator
in the `lis.py` module:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='143-172'
></div>

The `evaluate()` function consists of only one `match` statement. Details of the
`case` statement in the `match` statement are as follows.

#### Evaluating numbers

```python
case int(x) | float(x):
    return x
```

Subject
: Instance of `int` or `float` class.

Action
: Return value as is.

Description
: This statement matches the `int` or `float` object. The `x` variable is bound
to the matched whole `int` or `float` object rather than an attribute of the
`int` or `float` object[^pythonorg__compoundstatement_classpattern].

Example
: <!---->

```python
>>> evaluate(1, {})
1
>>> evaluate(1.5, {})
1.5
```

#### Evaluating symbols

```python
case Symbol(var):
    return env[var]
```

Subject
: Instance of `Symbol` class which means `str` object.
(`Symbol: TypeAlias = str`)

Action
: Return value of the result of lookup in the environment `env`.

Description
: The `Symbol` class is a type alias of `str`. This statement matches the
`str` object. The `var` variable is bound to the matched whole `str` object
rather than an attribute of the `str` object
[^pythonorg__compoundstatement_classpattern].

Example
: <!---->

```python
>>> evaluate('a', {'a': 1})
1
>>> evaluate('+', standard_env())
<built-in function add>
```

#### `(quote ...)`

```python
case ['quote', x]:
    return x
```

Subject
: List object that has the `quote` as the first element and the length of the
list is 2.

Action
: Return the second element as is.

Description
: This statement matches the list that has the `quote` as the first element,
the `x` as the second element and the length of the list is 2.

Example
: <!---->

```python
>>> evaluate(['quote', 1], {})
1
>>> evaluate(['quote', ['+', 1, 2]], {})
['+', 1, 2]
>>> evaluate(parse('(quote (+ 1 2))'), {})
['+', 1, 2]
```

#### `(if ...)`

```python
case ["if", test, consequence, alternative]:
    if evaluate(test, env):
        return evaluate(consequence, env)
    else:
        return evaluate(alternative, env)
```

Subject
: List object that has the `if` as the first element and the length of the list
is 4.

Action
: Evaluate the `test` variable. If the result is `True`, evaluate the
`consequence` variable. Otherwise, evaluate the `alternative` variable.

Description
: This statement matches the list that has the `if` as the first element and
the length of the list is 3. The `test`, `consequence`, and `alternative`
variables are bound to the second, third, and fourth elements of the list
respectively.

The `test` variable is evaluated by using the `evaluate()` function recursively.
If the result of the `test` variable is `True`, the `consequence` variable is
evaluated by using the `evaluate()` function recursively and the result is
returned. Otherwise, the `alternative` variable is evaluated by using the
`evaluate()` function recursively and the result is returned.

Example
: <!---->

```python
>>> evaluate(['if', 1, 2, 3], {})
2
>>> evaluate(['if', 0, 2, 3], {})
3
>>> evaluate(parse('(if 1 2 3)'), standard_env())
2
>>> evaluate(parse('(if 0 2 3)'), standard_env())
3
```

#### `(lambda ...)`

```python
case ['lambda', [*parms], *body] if body:
    return Procedure(parms, body, env)
```

Subject
: List object that satisfies the following conditions:

1. List object that has the `lambda` as the first element,
2. List object that has a list object as the second element,
3. and the length of the list is 3 or more.

Action
: Return a `Procedure` object that represents the lambda expression.

Description
: This statement matches the list that has the `lambda` as the first element,
a list object as the second element, and the length of the list is 3 or more.
This statement catches the items in the list and bound to the `parms` and
`body` variables.

* `parms` : The `parms` variable is bound to the second element of the list.
    * The type of `parms` variable is `list`.
    * The second element of the list can be an empty list or a list that has
        one or more elements.
* `body` : The `body` variable is bound to the rest of the elements of the list.
    * The type of `body` variable is `list` which has one or more elements.

The `Procedure` class is used to represent the lambda expression. The details
of `Procedure` will be discussed in the next section.

Example
: <!---->

```python
>>> evaluate(['lambda', ['a', 'b'], ['+', 'a', 'b']], standard_env())(1,2)
3
>>> evaluate(['lambda', ['a', 'b'], '+', 'a', 'b'], standard_env())(1,2)
3
```

```python
>>> expr = '(lambda (a b) (* (/ a b) 100))'
>>> f = evaluate(parse(expr), standard_env())
>>> f
<lis.Procedure object at 0xffff89666120>
>>> f(15, 20)
75.0
```

#### `(define ...)`

```python
case ['define', Symbol(name), value_exp]:
    env[name] = evaluate(value_exp, env)
```

Subject
: List object that has the `define` as the first element, `Symbol` object as
the second element, and the length of the list is 3.

Action
: Evaluate the third expression and bind the result to the second expression.

Description
: This statement matches the list that has the `define` as the first element,
`Symbol` object as the second element, and the length of the list is 3.

The `name` variable is bound to the second element of the list. The `value_exp`
variable is evaluated by using the `evaluate()` function recursively and the
result is bound to the `name` variable in the `env` environment.

Example
: <!---->

```python
>>> env = standard_env()
>>> env['a']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python3.12/collections/__init__.py", line 1014, in __getitem__
    return self.__missing__(key)            # support subclasses that define __missing__
           ^^^^^^^^^^^^^^^^^^^^^
  File "/usr/local/lib/python3.12/collections/__init__.py", line 1006, in __missing__
    raise KeyError(key)
KeyError: 'a'
>>> evaluate(['define', 'a', 1], env)
>>> env['a']
1
>>> evaluate(['define', 'b', ['+', 1, 2]], env)
>>> env['b']
3
```

#### `(define ...)`

```python
case ['define', [Symbol(name), *parms], *body] if body:
    env[name] = Procedure(parms, body, env)
```

Subject
: List object that has the `define` as the first element, list object that has
`Symbol` object as the first element and the length of the whole list is 3 or
more.

Action
: Bind the `Procedure` object to the `name` variable in the `env` environment.

Description
: This statement matches the list that has the `define` as the first element,
list object that has `Symbol` object as the first element and the length of the
whole list is 3 or more.

The `name` variable is bound to the first element of the list. The `parms`
variable is bound to the second element of the list. The `body` variable is
bound to the rest of the elements of the list.

The `Procedure` class is used to represent the lambda expression. The details
of `Procedure` will be discussed in the next section.

Example
: <!---->

```python
>>> env = standard_env()
>>> evaluate(parse('(define (add a b) (+ a b))'), env)
>>> env['add']
<lis.Procedure object at 0xffff89666120>
>>> env['add'](1, 2)
3
```

#### `(set! ...)`

```python
case ['set!', Symbol(name), value_exp]:
    env.change(name, evaluate(value_exp, env))
```

Subject
: List object that has the `set!` as the first element, `Symbol` object as
the second element, and the length of the list is 3.

Action
: Evaluate the third expression and update the second expression.

Description
: This statement matches the list that has the `set!` as the first element,
`Symbol` object as the second element, and the length of the list is 3.

Example
: <!---->

```python
>>> env = standard_env()
>>> env['a'] = 1
>>> evaluate(['set!', 'a', 2], env)
>>> env['a']
2
```

#### Function call

```python
KEYWORDS = ["quote", "if", "lambda", "define", "set!"]

case [func_exp, *args] if func_exp not in KEYWORDS:
    proc = evaluate(func_exp, env)
    values = [evaluate(arg, env) for arg in args]
    return proc(*values)
```

Subject
: List object that has the first element that is not in the `KEYWORDS` and the
length of the list is 2 or more.

Action
: Evaluate the first expression and the rest of the expressions and call the
result of the first expression with the result of the rest of the expressions.

Description
: This statement matches the list that has the first element that is not in
the `KEYWORDS` and the length of the list is 2 or more.

The `func_exp` variable is evaluated by using the `evaluate()` function
recursively and the result is bound to the `proc` variable. The `args` variable
is also evaluated by using the `evaluate()` function recursively and
the results are bound to the `values` variable.

The `proc` variable is called with the `values` variable and the result is
returned.

Example
: <!---->

```python
>>> env = standard_env()
>>> evaluate(['+', 1, 2], env)
3
>>> evaluate(['define', ['add', 'a', 'b'], ['+', 'a', 'b']], env)
>>> evaluate(['add', 1, 2], env)
3
>>> evaluate(['add', 1, ['+', 1, 2]], env)
4
```

#### Default case

```python
case _:
    raise SyntaxError(lispstr(exp))
```

This case statement is used to raise a `SyntaxError` exception when the
`exp` variable does not match any of the patterns.

### Procedures: A Class Implementing a Closure

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='18-with-match/lispy/py3.10/lis.py'
     language='python'
     line='176-191'
></div>

* `<1>` : The constructor is called when `define` or `lambda` is evaluated.
* `<2>` : `self.parms` holds the names of the parameters, `self.body` holds
    the body of the function and `self.env` holds the environment in which the
    function was defined.
* `<3>` : The `Procedure` object can be called with the `args` variable.
* `<4>` : Build a new environment `local_env` that has the `self.parms` and
    `args` as the keys and values respectively.
* `<5>` : Build a combined environment by using `Environment` class. The
    `local_env` is the first mapping object and `self.env` is the second
    mapping object.
* `<6>` : Evaluate the `self.body` with the `env` environment.
* `<7>` : Return the result of the last expression in the `self.body`.

### Using OR-patterns

Details of OR-patterns in the `case int(x) | float(x):` statement can be found
in the following link[^pythonorg__compoundstatement_orpatterns].

## Do This, Then That: else Blocks Beyond if

The `else` can be used with the `for`, `while` and `try` clause. The usage and
meaning of the `else` clause with the `for`, `while` and `try` are different
from `if/else` statement.

The details of the `else` clause is as follows:

### `for`

The `else` clause in the `for` statement is executed when the loop is
terminated normally. The `else` clause is not executed when the loop is
terminated by a `break` statement.

```python
for item in items:
    if item == target:
        break
else:
    raise ValueError("Target not found")
```

### `while`

The `else` clause in the `while` statement is executed when the loop is
terminated by a `False` condition. The `else` clause is not executed when the
loop is terminated by a `break` statement.

```python
while condition:
    if item == target:
        break
else:
    raise ValueError("Target not found")
```

### `try`

The `else` clause in the `try` statement is executed when no exception is
raised in the `try` block. The `else` clause is not executed when an exception
is raised in the `try` block.

Exception occurred in the `else` clause is not caught by the preceding `except`
clause.

```python
try:
    do_something()
except ValueError:
    handle_value_error()
else:
    after_do_something()
```

### *EAFP and LBYL*

The `else` clause in the `try` statement is used to implement the EAFP
(Easier to Ask for Forgiveness than Permission) principle. The EAFP principle
is a programming style that is used to catch exceptions rather than check
conditions before the execution of the code.

This principle makes the code more readable and maintainable because the
normal path of the code is written first and the exceptional path of the code
is written later.

In contrast, the LBYL (Look Before You Leap) principle is a programming style
that is used to check conditions before the execution of the code.
This coding style explicitly checks the conditions before the execution of the
code.

Compared to the EAFP principle, the LBYL principle makes the code more
unreadable because the normal path of the code is written later and the
exceptional path of the code is written first.

## Chapter Summary

* The `with` statement is used to manage resources by using the context
    managers. The context managers are used to set up and tear down resources
    automatically.
* The context manager interface is implemented by the `__enter__()` and
    `__exit__()` methods.
* The `contextlib` module provides the utilities for the context managers.
* The `contextlib.contextmanager` decorator is used to create a context manager
    by using a generator function.
* The `contextlib.ExitStack` class is used to manage multiple context managers
    at the same time.
* The `match`, `case` statement is used to implement the pattern matching in
    Python.
* The `lis.py` module is a Lisp interpreter written by Peter Norvig. The
    `lis.py` module uses the `match`, `case` statement to implement the pattern
    matching.
* The `else` clause can be used with the `for`, `while` and `try` statement.
    The `else` clause is executed when the loop is terminated normally or no
    exception is raised in the `try` block.

## Links

[^pythonorg__try_finally]:[8. Errors and Exceptions :: 8.7. Defining Clean-up Actions](https://docs.python.org/3/tutorial/errors.html#defining-clean-up-actions)
[^pythonorg__sqlite3_context_manager]:[`sqlite3` — DB-API 2.0 interface for SQLite databases :: How to use the connection context manager](https://docs.python.org/3/library/sqlite3.html#how-to-use-the-connection-context-manager)
[^pythonorg__sqlite3_connection_commit]:[`sqlite3` — DB-API 2.0 interface for SQLite databases :: `Connection.commit()`](https://docs.python.org/3/library/sqlite3.html#sqlite3.Connection.commit)
[^pythonorg__threading_example_with]:[`threading` — Thread-based parallelism :: Using locks, conditions, and semaphores in the with statement](https://docs.python.org/3/library/threading.html#using-locks-conditions-and-semaphores-in-the-with-statement)
[^pythonorg__decimal_example_with]:[decimal — Decimal fixed point and floating point arithmetic :: `decimal.localcontext()`](https://docs.python.org/3/library/decimal.html#decimal.localcontext)
[^pythonorg__unittestmock_example_with]:[unittest.mock — mock object library :: `unittest.mock.patch()`](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch)
[^pythonorg__contextlib]:[`contextlib` — Utilities for `with`-statement contexts](https://docs.python.org/3/library/contextlib.html)
[^pythonorg__contextlib_closing]:[`contextlib` — Utilities for `with`-statement contexts :: `contextlib.closing(thing)`](https://docs.python.org/3/library/contextlib.html#contextlib.closing)
[^pythonorg__contextlib_suppress]:[`contextlib` — Utilities for `with`-statement contexts :: `contextlib.suppress(*exceptions)`](https://docs.python.org/3/library/contextlib.html#contextlib.suppress)
[^pythonorg__contextlib_nullcontext]:[`contextlib` — Utilities for `with`-statement contexts :: `contextlib.nullcontext(enter_result=None)`](https://docs.python.org/3/library/contextlib.html#contextlib.nullcontext)
[^pythonorg__contextlib_contextmanager]:[`contextlib` — Utilities for `with`-statement contexts :: `@contextlib.contextmanager`](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager)
[^pythonorg__contextlib_abstractcontextmanager]:[`contextlib` — Utilities for `with`-statement contexts :: `class contextlib.AbstractContextManager`](https://docs.python.org/3/library/contextlib.html#contextlib.AbstractContextManager)
[^pythonorg__contextlib_contextdecorator]:[`contextlib` — Utilities for `with`-statement contexts :: `class contextlib.ContextDecorator`](https://docs.python.org/3/library/contextlib.html#contextlib.ContextDecorator)
[^pythonorg__contextlib_exitstack]:[`contextlib` — Utilities for `with`-statement contexts :: `class contextlib.ExitStack`](https://docs.python.org/3/library/contextlib.html#contextlib.ExitStack)
[^pythonorg__contextlib_parenthesized_context_managers]:[Parenthesized context managers](https://docs.python.org/3/whatsnew/3.10.html#parenthesized-context-managers)
[^pythonorg__compoundstatement_classpattern]:[8. Compound statements :: 8.6.4.10. Class Patterns](https://docs.python.org/3/reference/compound_stmts.html#class-patterns)
[^pythonorg__compoundstatement_orpatterns]:[8. Compound statements :: 8.6.4.1. OR Patterns](https://docs.python.org/3/reference/compound_stmts.html#or-patterns)
[^pythonorg__pep0342_throw]:[New generator method: `throw(type, value=None, traceback=None)`](https://peps.python.org/pep-0342/#new-generator-method-throw-type-value-none-traceback-none)
[^MartijnPieters_easyinplacefilerewriting]:[Easy in-place file rewriting](https://www.zopatista.com/python/2013/11/26/inplace-file-rewriting/)
[^lispy]:[How to Write a (Lisp) Interpreter (in Python)](https://norvig.com/lispy.html)
[^scheme_expressions]:[Revised Report on the Algorithmic Language Scheme :: Chapter 4. Expressions](https://conservatory.scheme.org/schemers/Documents/Standards/R5RS/HTML/r5rs-Z-H-7.html)
[^pythonorg__collections_chainmap]:[`collections` — Container datatypes :: `class collections.ChainMap`](https://docs.python.org/3/library/collections.html#collections.ChainMap)
