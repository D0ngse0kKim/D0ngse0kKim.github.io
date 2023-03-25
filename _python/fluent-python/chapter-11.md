---
layout: distill
title: Chapter 11
description: A Pythonic Object
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Object Representations
  - name: Vector Class Redux
  - name: An Alternative Constructor
  - name: classmethod Versus staticmethod
  - name: Formatted Displays
  - name: A Hashable Vector2d
  - name: Supporting Positional Pattern Matching
  - name: Complete Listing of Vector2d, Version 3
  - name: Private and “Protected” Attributes in Python
  - name: Saving Memory with __slots__
  - name: Overriding Class Attributes
  - name: Links
---

## Abstract

This is a summary of chapter 11 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/).
This chapter covers the basic special methods for sequences by implementing a class `Vector2d` which represents 2-dimentional vectors.

## Object Representations

`repr(object)`, `__repr__`
:   Return a string representing the object as the developer wants to see it.
    A string displayed in Python console or debugger for the object.


`str(object)`, `__str__`
:   Return a string representing the object as the user wants to see it.
    A string displayed by calling `print()` function with the object as arguments.

`bytes(object)`, `__bytes__`
:   Return a representation of the object in byte sequence.

`format(object, format_spec)`, `__format__`
:   Rturns a representation of the object in string.
    Special formatting specifier can be specified.
    Introduced in [PEP-3101](https://peps.python.org/pep-3101/#controlling-formatting-on-a-per-type-basis)

## Vector Class Redux

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='70132a37c2e2012fc6bb0622da7bce17fc79c9aa'
     path='11-pythonic-obj/vector2d_v0.py'
     language="python"
     line='6-25'
></div>

* `<2>` : `Vector2d` can be unpacked to a tuple of variables by defining a `__iter__` method.
* `<3>` : `repr()` displays a representation of `v1` in source code.
* `<5>` : `Vector2d` supports `==` operator by defining a `__eq__` method.
* `<6>` : `print()` displays a represenatation of `v1` in pair of the vector components.
* `<7>` : `Vector2d` supports `bytes()` by defining a `__bytes__` method.
* `<8>` : `Vector2d` supports `abs()` by defining a `__abs__` method.
* `<9>` : `Vector2d` supports `bool()` by defining a `__bool__` method. `bool()` returns` False` when a `Vector2d` has zero magnitude

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='70132a37c2e2012fc6bb0622da7bce17fc79c9aa'
     path='11-pythonic-obj/vector2d_v0.py'
     language="python"
     line='31-63'
></div>

* `<3>` : `__iter__` makes a `Vector2d` iterable and allows unpacking. In this function generator expression is used.
* `<4>` : Components of `Vector2d` is unpacked by `*self` and `repr()` of each components is printed by [`{!r}`](https://peps.python.org/pep-3101/#explicit-conversion-flag).
* `<7>` : The byte representation of components is achieved by assigning [`array.array()`](https://docs.python.org/3/library/array.html#array.array) to `bytes()` function.
* `<9>` : [`math.hypot`](https://docs.python.org/3/library/math.html#math.hypot) is used for calculating magnitude of `Vector2d`

## An Alternative Constructor

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='70132a37c2e2012fc6bb0622da7bce17fc79c9aa'
     path='11-pythonic-obj/vector2d_v1.py'
     language="python"
     line='71-75'
></div>

A function generating `Vector2d` from `bytes` is implemented by using `@classmethod`.

## classmethod Versus staticmethod

```python
class Test:
    @classmethod
    def class_method(cls, *args):
        ...

    @staticmethod
    def static_method(*args):
        ...
```

* classmethod receives its own class from the first argument.
* staticmethod receives only assigned arguments.

## Formatted Displays

```python
>>> brl = 1 / 4.82 # BRL to USD currency conversion rate
>>> brl
0.20746887966804978
>>> format(brl, '0.4f')  # <1>
'0.2075'
>>> '1 BRL = {rate:0.2f} USD'.format(rate=brl)  # <2>
'1 BRL = 0.21 USD'
>>> f'1 USD = {1 / brl:0.2f} BRL'  # <3>
'1 USD = 4.82 BRL'
```

Formatting specifier can be used in three ways.

* `<1>` : `format(instance, format_spec)` returns string representation specified by `format_spec`.
* `<2>` : `str.format(instance)` with `{:format_spec}` representation on `str`.
* `<3>` : `f'...'` with `{:format_spec}` representation on f-string `f'...'`.

For more information on formatting specifier, see [here](https://docs.python.org/3/library/string.html#formatspec).

### Implementation of `__format__`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='70132a37c2e2012fc6bb0622da7bce17fc79c9aa'
     path='11-pythonic-obj/vector2d_v2.py'
     language="python"
     line='104-116'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='70132a37c2e2012fc6bb0622da7bce17fc79c9aa'
     path='11-pythonic-obj/vector2d_v2.py'
     language="python"
     line='60-65'
></div>

* Each component of `Vector2d` can be represented according to the specified formatting specifier.
* formatting specifier which ends with `p` makes polar coordinates representation.

## A Hashable Vector2d

`Vector2d` can be hashable by
* forbidding each components to be written.
* defining a `__hash__` special method.


<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3_prophash.py'
     language="python"
     line='95-111'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3_prophash.py'
     language="python"
     line='131-132'
></div>

## Supporting Positional Pattern Matching

By default, It is mandatory to specify the attribute name in the `case` statement.

```python
    match v:
        case Vector2d(x=0, y=0): print(f'{v!r} is null')
```

A class attribute `__match_args__` lists the instance attributes in the order they will be used for positional pattern matching.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3.py'
     language="python"
     line='90-91'
></div>

This allows attributes in `__match_args__` to be used for positional pattern matching.

```python
    match v:
        case Vector2d(0, 0): print(f'{v!r} is null')
```

## Complete Listing of Vector2d, Version 3

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3.py'
     language="python"
></div>

## Private and “Protected” Attributes in Python

An instance attribute with two leading underscores and zero or at most one
trailing underscore `__attr` is stored in the `__dict__` in this form by
*name mangling*.

```
_classname__attr
```

An attribute name prefixed with a single underscore is also often treated as a private attribute.

## Saving Memory with __slots__

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3_slots.py'
     language="python"
     line='90-94'
></div>

* `__slots__` must be present when defining the class.
* Change of `__slots__` or adding `__slots__` after defining class has no effect.
* `tuple` or `list` can be used to define `__slots__`.
     * `tuple` is preferred because changing `__slots__` has no effect.
* Attribute names stored in `__slots__` are not stored in `__dict__` of the class.
* Trying set an attributes which is not listed in `__slots__` raises `AttributeError`.
* Attributes can be added to instances of subclasses of classes with __slots__.

### Simple Measure of __slots__ Savings

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='11-pythonic-obj/vector2d_v3_slots.py'
     language="python"
     line='90-94'
></div>

* `<1>` : `__match_args__` defines names of public attributes
* `<2>` : `__slots__` defines the names of private attributes.
* Instances of the class with `__slots__` doesn't have `__dict__` attributes. This reduces size of the instances.

### Summarizing the Issues with __slots__

* A Class with `__slots__` has no `__dict__` unless `__dict__` is defined in `__slots__`
* Subclass of a class with `__slots__` has `__dict__` unless `__slots__` are defined.
* Instances will only be able to have the attributes listed in `__slots__`, unless you include `__dict__` in `__slots__`.
* Classes using `__slots__` cannot use the `@cached_property` decorator, unless they explicitly name `__dict__` in `__slots__`.
* Instances cannot be targets of weak references, unless you add `__weakref__` in `__slots__`.

## Overriding Class Attributes

* Accessing an attribute using `self.attribute` refers to the class attribute.
* If an instance attribute with the same name as a class attribute is created, accessing `self.attribute` will return the instance attribute's value.
* Creating an instance attribute with the same name as a class attribute does not change the value of the class attribute.
* Class attributes can be accessed using either `cls.attribute` or `ClassName.attribute`.

## Links

* [The definitive guide on how to use static, class or abstract methods in Python](https://julien.danjou.info/guide-python-static-class-abstract-methods/)
* [Format Specification Mini-Language](https://docs.python.org/3/library/string.html#formatspec)
* [Data Model - The Python Language Reference](https://docs.python.org/3/reference/datamodel.html)
     * This document includes descriptions of data model in Python and some special methods and attributes.
