---
layout: distill
title: Chapter 9
description: Decotors and Closures
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Decorators 101
  - name: When Python Executes Decorators
  - name: Variable Scope Rules
  - name: Closures
  - name: Implementing a Simple Decorator
  - name: Decorators in the Standard Library
  - name: Parameterized Decorators
  - name: Links
---

## Decorators 101

### *Decorators with no arguments*

A decorator with no arguments is a function like this:
1. has only one argument
2. receives a function from that argument
3. returns a function

Below is examples of decorators.

```python
def my_decorator1(func):
    def decorated_function(*args, **kwargs):
        # do something
        ret = func(*args, **kwargs)
        # do something
        return ret
    return decorated_function

def my_decorator2(func):
    def decorated_function(*args, **kwargs):
        # do something
        ret = func(*args, **kwargs)
        # do something
        return ret*2
    return decorated_function
```

Decorators can be used by prefixing a function with a decorator marked with `@`.
The following two expressions are equivalent.

```python
@my_decorator1
@my_decorator2
def my_function():
    pass
```

```python
def my_function():
    pass

my_function = my_decorator1(my_decorator2(my_function))
```

When a function is decorated by a decorator with no argument,
1. call the decorator with decorated function. (`my_decorator1(func)`)
2. returned function from decorator is assigned to same name of function with decorated function.

### *Decorators with arguments(Parameterized Decorators)*

A decorator with arguments is a function like this:
1. has same number of arguments with decorator.
2. returns a **decorator** defined by the assigned arguments

Decorator with arguments can be defined like this:

```python
def my_decorator(arg1, arg2):
    def generated_decorator(func):
        def decorated_function(*args, **kwargs):
            print("Decorator arguments:", arg1, arg2)
            return func(*args, **kwargs)
        return decorated_function
    return generated_decorator
```

The following two expressions are equivalent.

```python
@my_decorator(arg1='hello', arg2='world')
def my_function():
    print("Inside my_function")
```

```python
my_function = my_decorator(arg1='hello', arg2='world')(my_function)
```
When a function is decorated by a decorator with arguments,
1. `my_decorator(arg1, arg2)` is called with the arguments.
2. `generated_decorator(func)` is returned from `my_decorator(arg1, arg2)`.
3. decorated function `my_function()` is assigned to the first argument of returned decorator `generated_decorator(func)`.
4. new function `decorated_function()` is returned from `generated_decorator(func)`.

## When Python Executes Decorators

Decorators run right after the decorated function is defined.

When a module with decorated function is imported, decorators are executed as soon as the module is imported.

In other words, decorators are executed at *import time*.

## Variable Scope Rules

Variable scope are categorized to two scopes:
1. Module Global Scope
2. Function Local Scope

Inside of function local scope, `global <variable>` statement is needed to assign a value to a variable defined in the module global scope.

```python
>>> var_global = 10
>>> def function(a):
...     global var_global
...     print(a)
...     print(b)
...     b=5
...
>>> function(1)
1
10
>>> b
5
```

When a value is assigned to module global scope without `global` statement, An exception `UnboundLocalError` is occurred.

## Closures

Closures are functions `f()` that reference a variable that:

1. does not reside in the scope of global variables or local variables of `f()`.
2. resides in the scope that encompasses the function `f()`.

Such variables must come from the local scope of an outer function that encompasses the function `f()`


### The nonlocal Declaration

When assignment a value to outer variable is needed, `nonlocal` declaration is needed for the assignment.

```
def generator_counter():
    count = 0

    def counting_function():
        nonlocal count
        count += 1
        print(count)

    return counting_function

counter = generator_counter()
counter() # prints 1
counter() # prints 2
counter() # prints 3
```

### Variable Lookup Logic
1. If there is a `global x` declaration, `x` comes from and is assigned to the `x` global variable module.
2. If there is a `nonlocal x` declaration, `x` comes from and is assigned to the `x` local variable of the *nearest* surrounding function where `x` is defined.
3. If `x` is a parameter or is assigned a value in the function body, then `x` is the local variable.
4. If `x` is referenced but is not assigned and is not a parameter:
    * `x` will be looked up in the local scopes of the surrounding function bodies (nonlocal scopes).
    * If not found in surrounding scopes, it will be read from the module global scope.
    * If not found in the global scope, it will be read from `__builtins__.__dict__`[^builtinsdict].

[^builtinsdict]: `__builtins__.__dict__` is a dictionary that contains the names of all the built-in functions, exceptions, and types. For example, if you want to access the `int()` function, you can use `__builtins__.__dict__['int']`

## Implementing a Simple Decorator

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Elapsed time: {end_time - start_time:.5f} seconds")
        return result
    return wrapper

@timer
def slow_function(n):
    time.sleep(n)
    return n

>>> slow_function(2)
Elapsed time: 2.00100 seconds
```

### How It Works

A property `__name__` of decorated function is replaced to `wrapper` defined in the decorator function `timer()`.

```python
>>> slow_function.__name__
'wrapper'
```

`functools.wraps(func)` decorator copies accompanying properties of function `func` to decorated function.
For example, `__name__` and `__doc__` properties are copied to decorated function.

```python
import time
import functools

def timer(func):
                                # functools.wraps(func) copies properties of
    @functools.wraps(func)      # func to wrapper(*args, **kwargs) function.
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Elapsed time: {end_time - start_time:.5f} seconds")
        return result
    return wrapper

@timer
def slow_function(n):
    time.sleep(n)
    return n

>>> slow_function.__name__
'slow_function'
```

## Decorators in the Standard Library

### Memoization with functools.cache

```python
import functools

@timer
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

@functools.cache
@timer
def fibonacci_cached(n):
    if n < 2:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)

print(fibonacci(10))  # prints 55
print(fibonacci_cached(10))  # prints 55
```

### Using lru_cache

Python >= 3.8, `functools.lru_cache` can be applied like this:

```python
@functools.lru_cache
def costly_function(a, b):
    ...
```

Python >= 3.2, `functools.lru_cache` can be applied like this:

```python
@functools.lru_cache()
def costly_function(a, b):
    ...
```

`functools.lru_cache` decorator takes several optional arguments to control its behavior.

#### `maxsize`
* The maximum number of function call results that will be stored in the cache.
* If more than `maxsize` function calls are made, the least recently used result will be discarded from the cache.
* If `maxsize` is set to `None`, the cache can grow without bound.

#### `typed`
* If set to `True`, arguments of different types will be cached separately.
* For example, `my_func(1)` and `my_func(1.0)` will be cached separately.
* If set to `False` (the default), arguments will be treated as equivalent if they are equal according to the `==` operator.

### Single Dispatch Generic Functions

`functools.singledispatch` is a decorator allowing to define a function that can handle different types of arguments with differently defined functions.

`functools.singledispatch` can be useful in situations where a function that needs to handle different types of arguments differently.

`functools.singledispatch` acts as the entry point for a *generic function*, which does same operation over different object.

#### Function singledispatch
```python
import functools
import collections
import fractions
import numbers
import decimal

@functools.singledispatch
def my_func(arg):
    print("I don't know how to handle", type(arg))

@my_func.register(str)
def _(arg):
    print(f'Handling str:{arg:s}')

@my_func.register
def _(arg: collections.abc.Sequence):
    print('Handling abc.Sequence:{}'.format(' '.join(repr(item) for item in arg)))

@my_func.register
def _(n: numbers.Integral) -> str:
    return f'Handling number.Integral:{n}'

@my_func.register
def _(n: bool) -> str:
    return f'Handling bool:{n}'

@my_func.register(decimal.Decimal)
@my_func.register(float)
def _(x) -> str:
    frac = fractions.Fraction(x).limit_denominator()
    return f'Handling Decimal or float{x} ({frac.numerator}/{frac.denominator})'

>>> my_func(1)
'Handling number.Integral:1'

>>> my_func("abc")
Handling str:abc

>>> my_func([1, 3, 2.5, fractions.Fraction(2,3)])
Handling abc.Sequence:1 3 2.5 Fraction(2, 3)

>>> my_func(True)
'Handling bool:True'

>>> my_func(2.5)
'Handling Decimal or float2.5 (5/2)'
```

* `my_func.register(type)` can be used to specify the type of object.
* The `singledispatch` logic seeks the implementation with the most specific matching type, regardless of the order they appear in the code.
* Assigning ABCs(abstract classes) type such as `collections.abc.Sequence`, `numbers.Integral` makes function more versatile instead of assigning concreat type like `int` and `list`.

## Parameterized Decorators

Parameterized decorators return decorator, which is then applied to the function to be decorated.

Refer [this](#decorators-with-argumentsparameterized-decorators)

### The Parameterized Clock Decorator

```python:Example9-24
import time
DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

def clock(fmt=DEFAULT_FMT):
    def decorator(func):
        def function_decorated(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)  # actual returned value is saved in _result
            elapsed = time.perf_counter() - t0

            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)

            # **locals() makes local variables to be assigned to format string fmt
            print(fmt.format(**locals()))

            return _result
        return function_decorated
    return decorator

@clock()
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)

@clock('{name}: {elapsed}s')    # format string can be modified as needed.
def snooze_simple(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)
```

### A Class-Based Clock Decorator

```python:Example9-27
import time
DEFAULT_FMT = '[{elapsed:0.8f}s] {name}({args}) -> {result}'

class clock:
    def __init__(self, fmt=DEFAULT_FMT):
        self.fmt = fmt

    def __call__(self, func):   # this method makes instances of class clock callable
        def clocked(*_args):
            t0 = time.perf_counter()
            _result = func(*_args)
            elapsed = time.perf_counter() - t0

            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)

            print(self.fmt.format(**locals()))
            return _result
        return clocked

@clock()
def snooze(seconds):
    time.sleep(seconds)

for i in range(3):
    snooze(.123)
```

## Links

* [PEP 443 - Single-dispatch generic functions](https://peps.python.org/pep-0443/)
* [@functools.singledispatch](https://docs.python.org/3/library/functools.html#functools.singledispatch)
* [Blog posts by GrahamDumpleton](https://github.com/GrahamDumpleton/wrapt/blob/develop/blog/README.md)
* [decorator package](https://pypi.org/project/decorator/)
* [Examples of usage of decorator](https://wiki.python.org/moin/PythonDecoratorLibrary)
* [PEP 3104 - Accessto Names in Outer Scopes](https://peps.python.org/pep-3104/)
* [PEP 227 – Statically Nested Scopes](https://peps.python.org/pep-0227/)
* [implementation of generic functions](https://www.artima.com/weblogs/viewpost.jsp?thread=101605)
