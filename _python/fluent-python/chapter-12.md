---
layout: distill
title: Chapter 12
description: Special Methods for Sequences
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstract
  - name: Vector A User-Defined Sequence Type
  - name: Vector Take 1 Vector2d Compatible
  - name: Protocols and Duck Typing
  - name: Vector Take 2 A Sliceable Sequence
  - name: Vector Take 3 Dynamic Attribute Access
  - name: Vector Take 4 Hashing and a Faster __eq__
  - name: Vector Take 5 Formatting
  - name: Chapter Summary
  - name: Links
---

## Abstract

This is a summary of chapter 12 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/).
In this chapter, class `Vector2d` is extended to `Vector` which can represent arbitrary dimentional vector.
By extending `Vector2d` to `Vector` class, this chapter covers the following concepts:

* Basic sequence protocol:`__len__` and `__getitem__`
* `repr()` for instances with many items
* Slicing support and creating new `Vector` instance by slicing
* Hashing for instance with many items
* Custom formatting language extension

## Vector: A User-Defined Sequence Type

We will use [composition](../../../softwaredesign/uml-diagram/relationships/) to implement `Vector` class.
This `Vector` class will behave like an immutable flat sequence and be compatible with `Vector2d` class that we implemented [before](../../fluent-python/chapter-11).

## Vector Take #1: Vector2d Compatible

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='12-seq-hacking/vector_v1.py'
     language="python"
     line='86-125'
></div>

* `<1>` : The new `Vector` class uses [`array.array()`](https://docs.python.org/3/library/array.html) to store values of its components.
    * The `Vector` has a composition relationship with the `array.array` class.
    * `array.array(typecode[, initializer])` can accept an optional initializer value,
    which must be a list, a bytes-like object, or iterable over elements of the appropriate type.
    * Below are type annotations of the `initializer` argument.
        * `bytes | bytearray | Iterable[int]`
        * `bytes | bytearray | Iterable[float]`
        * `bytes | bytearray | Iterable[str]`
* `<2>` : Create an iterator using `iter(self._components)`.
* `<3>` : The [`reprlib.repr()`](https://docs.python.org/3/library/reprlib.html#reprlib.repr) returns a string like `array('d', [0.0, 1.0, 2.0, 3.0, 4.0, ...])`.
* `<4>` : Remove the `array('d', ` and `)` parts from the beginning and end of the string.
* `<5>` : Create `bytes` instance using the `__bytes__` method of `self._components`.
* `<6>` : This is equivalent to `math.sqrt(sum(x * x for x in self))`.
* `<7>` : The [`memoryview.cast`](https://docs.python.org/3/library/stdtypes.html#memoryview.cast) method returns a sequence `memv` that can be used as input to the `__init__` method of the `Vector` class.

## Protocols and Duck Typing

Protocol
: An informal interface which is defined in documentation and not in code.

* It is possible to implement a fully functional sequence type by implementing the special methods that satisfy the sequence protocol.
    * For example, the sequence protocol requires `__len__`and `__getitem__` methods.
* A class behaves like a protocol-defined type by implementing the required methods of protocol.
* It is possible to implement only some of the methods required by protocols because protocols are informal and not enforced.
    * For example, to support iteration, only `__getitem__` is required.
* These properties are known as *duck typing*.

## Vector Take #2: A Sliceable Sequence

### *Class `slice`*

Instances of [class `slice`](https://docs.python.org/3/library/functions.html#slice) is generated from slice operation.

```python
>>> class TestSlice:
...     def __getitem__(self, index):
...         return index
...
>>> ins = TestSlice()
>>> ins[0]
0
>>> ins[2:4]            # <1>
slice(2, 4, None)
>>> ins[1:4:2]          # <2>
slice(1, 4, 2)
>>> ins[1:4:2, 6]       # <3>
(slice(1, 4, 2), 6)
>>> ins[1:4:2, 6:10]    # <4>
(slice(1, 4, 2), slice(6, 10, None))
```

* `<1>` : `2:4` becomes `slice(2, 4, None)`.
* `<2>` : `1:4:2` becomes `slice(1, 4, 2)`.
* `<3>` : Commas inside the `[]` generates tuple.
* `<4>` : The tuple can hold several `slice` objects.

### *`indices` method of class `slice`*

`help(slice.indices)` prints the following description.

```
indices(...) method of builtins.slice instance
    S.indices(len) -> (start, stop, stride)

    Assuming a sequence of length len, calculate the start and stop
    indices, and the stride length of the extended slice described by
    S. Out of bounds indices are clipped in a manner consistent with the
    handling of normal slices.
```

`indices(len)` adjusts `start`, `stop`, and `stride` values by using the `len` argument.

```
>>> slice(None, 15, 3).indices(9)
(0, 9, 3)
>>> slice(None, 15, 3).indices(10)
(0, 10, 3)
>>> slice(-3, None, None).indices(7)
(4, 7, 1)
```

### A Slice-Aware `__getitem__`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='12-seq-hacking/vector_v2.py'
     language="python"
     line='149-157'
></div>

* `<1>` : If key is a `slice`,
* `<2>` : get a class by using `type(self)` and
* `<3>` : returns a new instance by `cls(self._components[key])`
* `<4>` : If an `int` type of index can be extracted from `key` by using [`operator.index`](https://docs.python.org/3/library/operator.html#operator.index)(refer to [PEP-357](https://peps.python.org/pep-0357/)),
* `<5>` : returns `self._components[index]`.

https://docs.python.org/3/library/operator.html#operator.index

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='12-seq-hacking/vector_v2.py'
     language="python"
     line='96-106'
></div>

* `<1>` : An `int` index can extract a component from the instance `v7`.
* `<2>` : Slice can generate a new vector instance with the components specified by the slice index.
* `<3>` : Slice of length == 1 can generate a new vector.
* `<4>` : `Vector` does not support multidimensional indexing, so indexing with comma raises an error.

## Vector Take #3: Dynamic Attribute Access

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='12-seq-hacking/vector_v3.py'
     language="python"
     line='202-213'
></div>

* `<1>` : The original purpose of `__match_args__`  is to enable the `Vector` class to support positional pattern matching. However, in this implementation, `__match_args__` is also utilized to specify attribute names for dynamic attribute access.
* `<2>` : Get the `Vector` class.
* `<3>` : Try to find an attribute `name` and get the position of the `name` in `__match_args__`.
* `<5>` : Check whether it is possible to return a component with `pos` index in `self._components` or not.
* `<4>`, `<6>`: Implementation for handling failure in getting the position of the `name` in `__match_args__`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='12-seq-hacking/vector_v3.py'
     language="python"
     line='217-229'
></div>

* `<1>` : Handle only attribute names consisting of a single character.
* `<2>` : Raise an error if attempting to set an attribute in `__match_args__`.
* `<3>` : Raise an error if attempting to set a single character attribute name.
* `<4>` : Otherwise, attribute can be set.
* `<5>` : Raise an `AttributeError` when `error` is not an empty string.
* `<6>` : Set an attribute with `__setattr__` of base class of the `Vector` class.

### *Examples of dynamic attribute access*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='12-seq-hacking/vector_v3.py'
     language="python"
     line='107-151'
></div>

## Vector Take #4: Hashing and a Faster __eq__

The class `Vector` can accepts arbitrary number of components. Therefore, considering the aspect of memory consumption, calculations using `tuple` of whole components are not efficient. In this example, the hash value is computed using the generator to reduce memory consumption.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='12-seq-hacking/vector_v4.py'
     language="python"
     line='178-184'
></div>

* `__eq__` method checks the following properties.
     * length of `self` and `other`
     * equility of each components by using built-in function [`zip`](https://docs.python.org/3/library/functions.html#zip) and [`all`](https://docs.python.org/3/library/functions.html#zip)
* `__hash__` method calculates the hash value by the following steps.
     * Builds a generator which yields the hash value of each components (`hashes`).
     * Calculates the XOR value of all components using [`functools.reduce`](https://docs.python.org/3/library/functools.html#functools.reduce) and [`operator.xor`](https://docs.python.org/3/library/operator.html#operator.xor).

## Vector Take #5: Formatting

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='12-seq-hacking/vector_v5.py'
     language="python"
     line='2-285'
></div>

* `<2>` : Calculate one of the angular coordinates
* `<3>` : Returns generator which yields all angular coordinates of instance.
* `<4>` : Concatenates two iterables by using [`itertools.chain`](https://docs.python.org/3/library/itertools.html#itertools.chain).
* `<5>` : Specify format string for hyperspherical coordinates.
* `<6>` : Specify format string for cartesian coordinates
* `<7>` : Build a generator that produces strings containing coordinates in a specified format.
* `<8>` : Concatenate the strings containing coordinates with `, `.

## Chapter Summary

* `__len__`, `__getitem__` : required by sequence protocol
     * `__getitem__` must support the `slice` instance to handle sequence slicing.
     * Sequence slicing produces a slice instance of [class `slice`](https://docs.python.org/3/library/functions.html#slice).
* `__getattr__`, `__setattr__` : Dynamic attribute access, which allows attribute access with specific attribute names, can be achieved by implementing these methods.
* `__hash__` : makes instances to be hashable. [`functools.reduce`](https://docs.python.org/3/library/functools.html#functools.reduce) and genexp is used for efficient calculation of the hash value.
* `__format__` : makes instances to be used with the [`format`](https://docs.python.org/3/library/functions.html#format) function and f-strings.

## Links

* [`array.array()`](https://docs.python.org/3/library/array.html)
* [`reprlib.repr()`](https://docs.python.org/3/library/reprlib.html#reprlib.repr)
* [`memoryview.cast`](https://docs.python.org/3/library/stdtypes.html#memoryview.cast)
* [class `slice`](https://docs.python.org/3/library/functions.html#slice)
* [`functools.reduce`](https://docs.python.org/3/library/functools.html#functools.reduce)
* [`itertools.chain`](https://docs.python.org/3/library/itertools.html#itertools.chain)
* [`functools.reduce`](https://docs.python.org/3/library/functools.html#functools.reduce)
* [`format`](https://docs.python.org/3/library/functions.html#format)
* [`zip`](https://docs.python.org/3/library/functions.html#zip)
