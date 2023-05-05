---
layout: distill
title: Chapter 13
description: Interfaces, Protocols, and ABCs
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstract
  - name: The Typing Map
  - name: Two Kinds of Protocols
  - name: Programming Ducks
  - name: Goose Typing
  - name: Static Protocols
  - name: Chapter Summary
  - name: References
---

## Abstract

This chapter provides an overview of typing in Python, covering nominal and structural typing, runtime and static protocols, duck typing, defensive programming, and abstract base classes (ABCs). The chapter explains how to define and use ABCs, subclass an ABC, and implement `__subclasshook__` for your own ABCs.
The chapter also covers best practices for protocol design and provides a summary of the content.

## The Typing Map

### *Nominal Typing vs Structural Typing*

Nominal Typing[^wikipedia__nominal_type_system]
: Nominal typing means that two variables are type-compatible
if and only if their declarations name the same type.
For example, in C, two struct types with different names
in the same translation unit are never considered compatible,
even if they have identical field declarations.

Structural Typing[^wikipedia__structural_type_system]
: In structural typing, an element is considered to be compatible with another if,
for each feature within the second element's type,
a corresponding and identical feature exists in the first element's type.
Two types are considered to be identical if each is compatible with the other.

### *Typing System in Python*

| Kinds                  | Version       | Type systems | Timing  | Remarks                                                                                            |
| ---------------------- | ------------- | ------------ | ------- | -------------------------------------------------------------------------------------------------- |
| **Duck typing**        | Any Python    | Structural   | Runtime | avoiding `isinstance` checks                                                                       |
| **Goose typing**       | Python >= 2.6 | Nominal      | Runtime | using `isinstance` checks against ABCs                                                             |
| **Static typing**      | Python >= 3.5 | Nominal      | Static  | PEP 484[^pythonorg__PEP_484] <br> type hints and <br> external type checker                        |
| **Static duck typing** | Python >= 3.8 | Structural   | Static  | PEP 544[^pythonorg__PEP_544] <br> `typing.Protocol` <br> type hints and <br> external type checker |

Duck typing
: Default typing system in Python.

Goose typing
: Typing system which is supported by abstract base classes (ABCs).
This typing system executes runtime checks of objects against ABCs.
This typing system can be used in Python >= 2.6.
Refer PEP 3119[^pythonorg__PEP_3119] and
official documents of ABC[^pythonorg__abc].

Static typing
: Traditional typing systems used in C and Java.
Static typing system is available in Python >= 3.5 by the
`typing` module [^pythonorg__typing] and it requires external type checker
compatible with PEP 484[^pythonorg__PEP_484].

Static duck typing
: Typing system used in the Go language.
Static duck typing is available in Python >= 3.8, which can be used by using
`typing.Protocol`[^pythonorg__typing_protocol]
and it requires external type checker.
Static duck typing system is added in PEP 544[^pythonorg__PEP_544].

## Two Kinds of Protocols

In this book, the following terms are adopted.

Dynamic Protocol
: The informal protocols of Python.
Dynamic protocols are implicit, defined by convention, described in the documentation.
These protocols are supported by the Python interpreter.
Documentation can be found in the Data Model Chapter of the Python Language Reference[^pythonorg__data_model]

Static Protocol
: A protocol defined in PEP 544[^pythonorg__PEP_544], since Python 3.8.
Static protocol has an explicit definition which is defined by subclassing `typing.Protocol`[^pythonorg__typing_protocol]

The major differences between these two protocols are described below.

1. The dynamic protocol does not require fulfilling all the protocol's requirements.
In contrast, the static protocol requires that the class with static protocol must implement every method declared in the protocol class.
2. The static protocols can be verified by static type checkers. In constrast, dynamic protocols cannot be checked statically.

## Programming Ducks

This subchapter covers the sequence and iterable dynamic protocols.

### Python Digs Sequences

`collections.abc.Sequence` is an ABC of sequence container type.
Details of `collections.abc.Sequence` can be found in official documentation.[^pythonorg__collections_abc_collections]

The following UML diagram describes the relationship between classes related to the `collections.abc.Sequence` class.
The red-colored methods are abstract methods that must be implemented in concrete method that inherits the ABC.
The blue-colored methods are mixin methods that are provided by the ABC or can be overridden by a subclass.

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-13/collections_abc_sequence.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML class diagram for the classes related with collections.abc.Sequence. <br>Red : Abstract methods, Blue : Mixin methods."%}
    </div>
</div>

According to this UML diagram, a correct subclass of `collections.abc.Sequence` have these properties.

* A subclass of `collections.abc.Sequence` must implement the following abstract methods:
    1. `__getitem__`
    2. `__len__`
* A subclass of `collections.abc.Sequence` can inherit the following mixin methods, which can be overridden by the subclasses:
    1. `__contains__`
    2. `__iter__`
    3. `__reversed__`
    4. `index`
    5. `count`

#### *Sequence like classes*

A class that
1. does not inherit from `collections.abc.Sequence` and
2. does not implement all abstract methods of `collections.abc.Sequence`

may still support some of the functions that `collections.abc.Sequence` does.

The following is a partial implement of a `collection.abc.Sequence`-like class.

```python
>>> class MyList:
...     def __init__(self, items):
...         self.items = items
...     def __getitem__(self, index):
...         return self.items[index]
>>> mylist = MyList([1, 2, 3, 4, 5])
>>> mylist[0]
1
>>> mylist[-1]
5
>>> for num in mylist: print(num)
...
1
2
3
4
5
>>> 5 in mylist
True
>>> 6 in mylist
False
```

Despite the absence of several methods, instances of `MyList` are still usable as sequences.

1. Despite the absence of `__iter__` method, iteration using `for` over the instances of the `MyList` class is available.
2. Despite the absence of `__contains__` method, the `in` operator over the instances of the `MyList` class is available.

Python manages to make iteration and `in` operator work by using `__getitem__` when `__iter__` and `__contains__` are unavailable.

### Monkey Patching: Implementing a Protocol at Runtime

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='104b2c9d2ddd1aaed07a5971d754f5251aa732b1'
     path='01-data-model/frenchdeck.py'
     language="python"
     line='1-17'
></div>

The `FrenchDeck` class does not have `__setitem__` method.
Therefore, `random.shuffle`[^pythonorg__random_shuffle]
on the instances of the `FrenchDeck` class raises an `TypeError`.

```python
>>> import random
>>> deck = FrenchDeck()
>>> random.shuffle(deck)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/user/.pyenv/versions/3.11.1/lib/python3.11/random.py", line 380, in shuffle
    x[i], x[j] = x[j], x[i]
    ~^^^
TypeError: 'FrenchDeck' object does not support item assignment
```

In this situation, it is possible to make the `FrenchDeck` class to support
`random.shuffle` by adding `__setitem__` method outside of the implementation of the `FrenchDeck` class.

```python
>>> def setitem(instance, position, item):
...     instance._cards[position] = item
...
>>> FrenchDeck.__setitem__ = setitem
>>> deck = FrenchDeck()
>>> random.shuffle(deck)
>>> deck[:5]
[Card(rank='8', suit='hearts'), Card(rank='10', suit='clubs'), Card(rank='5', suit='diamonds'), Card(rank='8', suit='spades'), Card(rank='3', suit='clubs')]
```

The `random.shuffle` method only requires the argument's class to implement the methods from the mutable sequence protocol.
It does not mandate that the argument's class belongs to a specific type or class.

### Defensive Programming and "Fail Fast"

#### *An Example of `__init__` Method*

```python
    def __init__(self, iterable):
        self.data = list(data)
```

In this example, the `__init__` function invokes `list()` on the `iterable`
argument instead of restricting the argument to the `list` type.
By doing this, it offers the following advantages.

1. The `__init__` function can accepts any iterable.
2. If the `__init__` function accepts a non-iterable argument, it raises a `TypeError` exception during the object initialization phase.

However, there are also some potential disadvantages to consider:

1. Data that should not be copied cannnot be used.
2. Infinite generator as argument can consume all available memory.
    * This disadvantage can be resolved by calling the `len()` function on the `iterable` argument.

#### *An Example of Error Handling with Duck Typing*

```python
try:
    field_names = field_names.replace(',', ' ').split()  # <1>
except AttributeError:  # <2>
    pass  # <3>
field_names = tuple(field_names)  # <4>
if not all(s.isidentifier() for s in field_names):  # <5>
    raise ValueError('field_names must all be valid identifiers')
```

* `<1>` : Assume `field_names` is a `str` and split with `,` or ` ` with `str.split`[^pythonorg__built_in_types_str_split].
* `<2>` : If `field_names` is not a `str`, assume it as sequence of `str`.
* `<3>` : In case of sequence of `str`, there is no need to split string.
* `<4>` : To make sure that `field_names` is sequence of `str`, make `tuple` of `str` with `tuple()`
* `<5>` : Check all strings in `field_names` to ensure that each string is a valid name for an attributes of the class.

## Goose Typing

Python does not have an `interface` keyword.
Instead of using functions related the `interface` keyword, it is necessary to
use abstract base classes in Python for explicit type checking at runtime.

abstract base class[^pythonorg__glossary_abstract_base_class]
in Python glossary explains the advantage of introducing ABC into duck typing:

> Abstract base classes complement
> duck-typing[^pythonorg__glossary_duck_typing] by
> providing a way to define interfaces when other techniques like
> `hasattr()`[^pythonorg__built_in_functions_hasattr] would
> be clumsy or subtly wrong (for example with
> magic methods[^pythonorg__data_model_special_lookup]
> ).
> ABCs introduce virtual subclasses, which are classes that don't inherit from
> a class but are still recognized by
> `isinstance()`[^pythonorg__built_in_functions_isinstance]
> and `issubclass()`[^pythonorg__built_in_functions_issubclass];
> see the `abc` module documentation[^pythonorg__abc].

### *Summary of Waterfowl and ABCs and comments of author*

* Subclassing from ABCs to explicitly indicate that you are implementing a previously defined interface.
* Use ABCs for runtime type checking instead of concrete classes as the second argument for `isinstance` and `issubclass`.
    * Runtime type checking with ABCs is more acceptable and does not limit polymorphism.
    * Enforcing an API contract is possible using runtime type checking with ABCs.
* Replace runtime type checking with concrete classes using `if`/`elif`/`else` with polymorphism.
* Avoid defining custom ABCs or metaclasses; instead, use the predefined ABCs provided.
    * ABCs are intended to encapsulate very general concepts and abtractions introduced by a framework, such as sequences and exact numbers.
    * In most cases, it is sufficient to use an existing ABCs instead of creating a new one.

### Subclassing an ABC

#### `collections.abc.MutableSequence`

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-13/collections_abc_mutablesequence.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML class diagram for the classes related with collections.abc.MutableSequence. <br>Red : Abstract methods, Blue : Mixin methods."%}
    </div>
</div>

The above figure is a UML class diagram of classes related with `collections.abc.MutableSequence`. According to the above figure, `collections.abc.MutableSequence` must implement the following abstract method.

* `__getitem__`
* `__setitem__`
* `__delitem__`
* `__len__`
* `insert`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/frenchdeck2.py'
     language="python"
     line='1-26'
></div>

In order to satisfy the requirement from abstract methods of `collections.abc.MutableSequence`, The new `FrenchDeck2` have to implement `__setitem__`, `__delitem__`, and `insert` methods.

### ABCs in the Standard Library

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-13/collections_abc.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML class diagram for the classes in collections.abc module."%}
    </div>
</div>

`Iterable`, `Container`, `Sized`
: Basic classes for ABCs of collection. `Iterable` supports iteration with `__iter__`. `Containter` supports the `in` operator with `__contains__`. `Sized` supports built-in `len()` function with `__len__`.

`Collection`
: Subclass of `Iterable`, `Container` and `Sized`. It was added in Python 3.6 to improve the efficiency of subclassing.

`Sequence`, `Mapping`, `Set`
: Main immutable collection types.

`MutableSequence`, `MutableMapping`, `MutableSet`
: Main mutable collection types.

`KeysView`, `ItemsView`, `ValuesView`
: Classes for objects returned from `dict.keys()`, `dict.values()` and `dict.items()`. `KeysView` and `ItemsView` support methods from the `Set` class.

`Iterator`
: `Iterator` must implement a `__next__` method that returns succesive items in the stream and raises a `StopIteration` exception after the final item has been returned. Any further invocation of the `__next__` method raises the `StopIteration` exception indicating that the iterator object is exhausted. `Iterator` also must implement `__iter__` method that returns the iterator object itself. For more information, Refer to the Glossary page [^pythonorg__glossary_iterator] and Built-in Types[^pythonorg__built_in_types_iterator] of iterator in official documentation.

`Callable`, `Hashable`
: This supports type checking objects that must be callable or hashable.

### Defining and Using an ABC

Defining Interfaces and providing type hints to support static typing can be achieved by using abstract base classes in Python.

The following example creates a `Tombola` ABC that provides a non-repeating random-picking process framework.
The `Tombola` ABC has the following four methods:
* `.load(...)` : Abstract method
    * Put items into the container.
* `.pick()` : Abstract method
    * Remove and return a random item from the container.
* `loaded()` : Concrete method
    * Return `True` if there is at lease one item in the containter.
* `inspect()` : Concrete method
    * Return a `tuple` that holds the items currently in the container, without changing its contents.
    * The internal ordering of the items in the container is not preserved.

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-13/tombola.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML class diagram for the <code>Tombola</code> ABC.<br>Red : Abstract methods, Blue : Mixin methods. "%}
    </div>
</div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='13-protocol-abc/tombola.py'
     language="python"
     line='3-31'
></div>

* `<1>`: To define an ABC, subclass `abc.ABC`
* `<2>`, `<3>`: Abstract methods can be defined using `abc.abstractmethod` decorator.
* `<4>` : ABC can define concrete methods.
* `<5>`, `<6>`, `<7>` : Concrete method must rely only on the interface defined by the ABC.
* `.pick()` `.inspect()` method uses `LookupError` for checking whether the container is empty
    * `LookupError` covers `IndexError` for sequences and `KeyError` for mappings.
    * For details of exception hierarchy, refer an official document of exceptions[^pythonorg__exceptions_hierarchy].

### ABC Syntax Details

```python
import abc

class NewABC(abc.ABC):
    @classmethod
    @abc.abstractmethod
    def new_classmethod(cls, ...):
        pass

    @staticmethod
    @abc.abstractmethod
    def new_staticmethod(...):
        pass
```

To define an abstract class method and an abstract static method, you can use the `abc.abstractmethod` decorator in combination with the `classmethod` or `staticmethod` decorator.
It's important to note that the `abc.abstractmethod` decorators should be applied as the innermost decorator.

### Subclassing an ABC

The following examples present the definition of concrete classes subclassing the `Tombola` ABC.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/bingo.py'
     language="python"
     line='3-26'
></div>

* `<1>` : Subclass `Tombola` ABC.
* `<2>` : `random.SystemRandom` implements the random API using the `os.urandom()` function, which provides random bytes suitable for cryptographic use.
* `<3>` : Load initial items using the `.load()` method in `Tombola` ABC.
* `<4>` : Shuffles items in the container using the `random.SystemRandom` instance.
* `<5>` : Implement the abstract method `.pick()` in `Tombola` ABC.
* `<6>` : Make instances of class `BingoCage` callable.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='13-protocol-abc/lotto.py'
     language="python"
     line='3-27'
></div>

* `<1>` : The argument `iterable` accepts any iterable which can be used to build a `list`.
* `<2>` : `random.randrange()` raises `ValueError` if the range is empty.
* `<3>` : The randomly selected item is popped.
* `<4>` : Override the `.loaded()` method in the `Tombola` ABC using `bool()`. This makes `.loaded()` method faster and more memory-efficient compared to the original implementation in the `Tombola` ABC.
* `<5>` : Override the `.inspect()` method in the `Tombola` ABC using `tuple()`.

### A Virtual Subclass of an ABC

An important characteristic of goose typing is the ability to register a class as a *virtual subclass* of an ABC, even if it does not inherit from it.
By doing this, It is possible to ensure that the class implements the interface defined in the ABC.


<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-13/tombolist.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML class diagram for the <code>TomboList</code> ABC.<br>Red : Abstract methods, Blue : Mixin methods. "%}
    </div>
</div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='13-protocol-abc/tombolist.py'
     language="python"
     line='1-23'
></div>

* `<1>` : Registering a class as a virtual subclass of an ABC can be done by calling the `register` class method of the ABC.
* `<2>` : `TomboList` extents `list` instead of `Tombola` ABC.
* `<3>`, `<4>`, `<6>` : Use the boolean behavior and `.pop()` method, inherited from `list` class.
* `<5>` : Implement the `load` abstract method using the `extent` method from the `list` class.
* `<7>` : It is possible to register a class as a virtual subclass after its implementation.
    * This is useful when registering a class that you do not maintain, but which satisfies the interface.

By doing this, the registered `TomboList` class will be recognized as subclass by the following calling methods.

* `issubclass(TomboList, Tombola)`
* `isinstance(TomboList(...), Tombola)`

### Usage of register in Practice

The following codes from CPython use the `.register()` class method of the `Sequence` ABC to register the `tuple`, `str`, `range` and `memoryview` classes as virtual subclasses.

<div class='embed-github-src'
     repo='python/cpython'
     branch='0bbf30e2b910bc9c5899134ae9d73a8df968da35'
     path='Lib/_collections_abc.py'
     language="python"
     line='1078-1081'
></div>

### Structural Typing with ABCs

Nominal Typing Aspect of ABC
: Subclassing ABC or registering as a virtual subclasses of ABC makes `issubclass(SubClass, ABCClass)` returns `True`.

Structural Typing Aspect of ABC
: Some ABCs support structural typing, which enable the usage of `issubclass()` and `isinstance()` without requiring subclassing or virtual subclass registeration.\
The structural typing aspect of an ABC can be supported by implementing a special class method `__subclasshook__()`. For example, the `__subclasshook__()` method of `Sized` class checks whether the class argument has an attribute named `__len__` and returns `True`, which means the class in the argument is subclass of the `Sized` class.\
The following example shows the structural typing aspect of the `collections.abc.Sized` class. The `__subclasshook__()` method searches for `__len__()` method in the `__dict__` of the class.

```python
class HaveLenClass:
    def __len__(self): return 10

from collections import abc
print(isinstance(HaveLenClass(), abc.Sized)) # True
print(issubclass(HaveLenClass, abc.Sized)) # True
```

<div class='embed-github-src'
     repo='python/cpython'
     branch='0fbddb14dc03f61738af01af88e7d8aa8df07336'
     path='Lib/_collections_abc.py'
     language="python"
     line='369-381'
></div>

<div class='embed-github-src'
     repo='python/cpython'
     branch='0fbddb14dc03f61738af01af88e7d8aa8df07336'
     path='Lib/_collections_abc.py'
     language="python"
     line='74-84'
></div>

#### *Implementing `__subclasshook__` for your own ABCs*

It is not a good idea to implement `__subclasshook__` for your own ABCs, as it can be risky to make assumptions based on the existence of method implementation in the interface provided by the ABC.

The `abc.Sequence` class does not implement `__subclasshook__`. It is impossible to check whether an class is a subclass of `abc.Sequence` or not based on the existence of methods `__len__`, `__getitem__` and `__iter__` because subclasses of `Mapping` also have these method.

In most use cases, it is more effective and comprehensive to use subclassing or virtual subclass registering with ABCs compared to using the structural typing aspect of ABCs.

## Static Protocols

A protocol defined in PEP 544[^pythonorg__PEP_544], since Python 3.8.
Static protocol has an explicit definition which is defined by subclassing `typing.Protocol`[^pythonorg__typing_protocol]

The static protocols can be verified by static type checkers. In constrast, dynamic protocols cannot be checked statically.

### The Typed double Function

In this example, the following simple `double` function will be typed statically by subclassing `typing.Protocol`[^pythonorg__typing_protocol]

```python
>>> def double(x):
...     return x * 2
...
>>> double(1.5)
3.0
>>> double('A')
'AA'
>>> double([10, 20, 30])
[10, 20, 30, 10, 20, 30]
>>> from fractions import Fraction
>>> double(Fraction(2, 5))
Fraction(4, 5)
```

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/double/double_protocol.py'
     language="python"
></div>

* `<1>`, `<2>` : The `__mul__` method in the `Repeatable` protocol defines a static protocol that can annotate the type of the argument for the `double` function. The type annotation `T` on the argument `self` and return value represents that the result type of the `double` method is the same as the type of `self`.
* `<3>` : The `RT` type variable is bounded by the `Repeatable` protocol.
* `<4>` : The type checker can ensure that the type of the argument `x` must support multiplication, or the `__mul__` method. Furthermore, the type checker verifies that the type of the argument `x` is the same as the type of return value of the `double` function.

### Runtime Checkable Static Protocols

The above example illustrates the usage of `typing.Protocol` to describe a static type checking feature of a subclass of the `typing.Protocol`.
This subclass of `typing.Protocol` allows for type checking during compile time, but by using the `runtime_checkable` decorator[^pythonorg__typing_runtime_checkable], it can also be used for runtime type checking.

There are several predefined protocols that are available to use for runtime type-checking:
1. `class typing.SupportsComplex`[^pythonorg__typing_supportscomplex] : An ABC with one abstract method, `__complex__`.
2. `class typing.SupportsFloat`[^pythonorg__typing_supportsfloat] : An ABC with one abstract method, `__float__`.

The source code below defines `class typing.SupportsComplex` in CPython:

<div class='embed-github-src'
     repo='python/cpython'
     branch='3635388f52b42e5280229104747962117104c453'
     path='Lib/typing.py'
     language="python"
     line='1751-1758'
></div>

It includes an abstract method `__complex__` that takes only a `self` argument and returns a `complex`.
This method defines an ABC that support compatibility to convert to `complex`.

The `runtime_checkable` decorator is applied to the `typing.SupportsComplex` to provice a runtime type-checking facility.

The class uses an empty `__slots__` for optimizing memory usage.

```python
>>> from typing import SupportsComplex
>>> import numpy as np
>>> c64 = np.complex64(3+4j)
>>> isinstance(c64, complex)
False
>>> isinstance(c64, SupportsComplex)
True
>>> c = complex(c64)
>>> c
(3+4j)
>>> isinstance(c, SupportsComplex)
False
>>> complex(c)
(3+4j)
>>> isinstance(c, (complex, SupportsComplex))
True
>>> import numbers
>>> isinstance(c, numbers.Complex)
True
>>> isinstance(c64, numbers.Complex)
True
```

`isinstance(val, SupportsComplex)` works well for `val` that is compatible with conversion to `Complex`, but does not work for `val` that is an instance of `Complex`.
To handle this problem, you can use the following methods to call the `isinstance` method:
1. `isinstance(val, (complex, typing.SupportsComplex))`
2. `isinstance(val, numbers.Complex)` : Note that this convention is not recommended because it is not recognized by static type checkers.

### Duck Typing Is Your Friend

It is possible to simplify a type-checking process by using duck typing.

#### Runtime Type-checking with Static Protocol(`typing.Protocol`)

```python
if isinstance(o, (complex, SupportsComplex)):
    # do something that requires `o` to be convertible to complex
else:
    raise TypeError('o must be convertible to complex')
```

#### Goose Typing (`isinstance` with ABC)

```python
if isinstance(o, numbers.Complex):
    # do something with `o`, an instance of `Complex`
else:
    raise TypeError('o must be an instance of Complex')
```

#### Duck Typing with a Custom Message

```python
try:
    c = complex(o)
except TypeError as exc:
    raise TypeError('o must be convertible to complex') from exc
```

#### Duck Typing with a Predefined Message

```python
c = complex(o)
```

### Limitations of Runtime Protocol Checks

`isinstance`/`issubclass` checks only examine the presence or absence of methods, without verifying their signatures, or type annotations.

In Python 3.9, there are `__float__` method on class `Complex` for purpose of raising `TypeError` exception.
This causes instances of `Complex` to pass `isinstance`/`issubclass` checks on `typing.SupportsFloat`:

```python
>>> import sys
>>> sys.version
'3.9.5 (v3.9.5:0a7dcbdb13, May 3 2021, 13:17:02) \n[Clang 6.0 (clang-600.0.57)]'
>>> c = 3+4j
>>> c.__float__
<method-wrapper '__float__' of complex object at 0x10a16c590>
>>> c.__float__()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't convert complex to float
>>> from typing import SupportsFloat
>>> c = 3+4j
>>> isinstance(c, SupportsFloat)
True
>>> issubclass(complex, SupportsFloat)
True
```

### Supporting a Static Protocol

#### *Supporting a Static Protocol without Static Type-Checking Coverage*

It is possible to check type in runtime by implementing `__complex__` method on class `Vector2d`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/typing/vector2d_v4.py'
     language="python"
     line='166-171'
></div>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/typing/vector2d_v4.py'
     language="python"
     line='89-97'
></div>

#### *Supporting a Static Protocol with Static Type-Checking Coverage*

Static type-checking is available by providing type annotation to the `__abs__`, `__complex__` and `fromcomplex` method.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/typing/vector2d_v5.py'
     language="python"
     line='164-173'
></div>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/typing/vector2d_v5.py'
     language="python"
     line='91-98'
></div>

### Designing a Static Protocol

A simple static protocol with only one method is more useful and flexible compared to a static protocol with multiple methods.

The ABC `Tombola` specifies two method: `load` and `pick`.
Based in the previous statement regarding static protocol definition, it is recommended to define a static protocol with only the `pick` method for implementing a random picking process.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/typing/randompick.py'
     language="python"
></div>

Usages of the static protocol `RandomPicker` is described in the following example codes:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='13-protocol-abc/typing/randompick_test.py'
     language="python"
></div>

* `<5>` : This type hint is available because `SimplePicker` is consistent-with `RandomPicker`.
* `<6>` : This test will be passed because `isinstance(popper, RandomPicker)` will returns `True`.
* `<8>` : Mypy will display that `Revealed type is 'Any'`.

### Best Practices for Protocol Design

#### *Designing Protocols*

Define a narrow protocol
: It is clear that a narrow protocol is more useful.
Such protocols often have only a single method, rarely more than a couple of methods.
This resolves tight coupling between the methods used for defining the protocol.
This also satisfies Interface segregation principle[^wikipedia__interface_segregation_principle].

Define a protocol near the function that uses it
: It is recommended to define a static protocol in "client code" instead of the library code.
This resolves tight coupling between the "client code" and library code.

Martin Fowler wrote a post defining role interface[^external__martin_fowler_roleinterface],
a useful idea to keep in mind when designing protocols.

#### *Naming Conventions for Protocols* [^github__python_typeshed_contributing]

Use plain names for protocols that represent a clear concept
: `Iterator`, `Container`

Use `SupportsX` for protocols that provide callable methods
: `SupportsInt`, `SupportsRead`, `SupportsReadSeek`

Use `HasX` for protocols that have readable and/or writable attributes or getter/setter methods
: `HasItems`, `HasFileno`

### Extending a Protocol

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/typing/randompick.py'
     language="python"
></div>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='13-protocol-abc/typing/randompickload.py'
     language="python"
></div>

* `<1>` : If you want to make an extended protocol runtime-checkable, you must apply the `runtime_checkable` decorator to it.
* `<2>` : In order to create an extended protocol, it is necessary to explicitly include `typing.Protocol` as one of its base classes in addition to the protocol being extended. Unlike regular inheritance in Python, this is a requirement. [^pythonorg__PEP_544_merging_and_extending_protocols]
* `<3>` : We only need to define the additional method required to define the new extended protocol.

### The numbers ABCs and Numeric Protocols

* Using `isinstance` with classes in `numbers`[^pythonorg__numbers] : Runtime-checkable but impossible to static-type-check.
* Using `Union` and `TypeVar` on numeric types in the standard library : impossible to use on numeric types outside of the standard library.
    * `_Number = Union[float, Decimal, Fraction]`
    * `_NumberT = TypeVar('_NumberT', float, Decimal, Fraction)`
* Using the numeric protocols provided by the `typing` module. [^pythonorg__typing_protocols] : Recommended, but some of use cases, such as `SupportsFloat` on `complex` in Python 3.9, are problematic.
    * `SupportsComplex`, `SupportsFloat`

## Chapter Summary

### The Typing Map

| Kinds                  | Version       | Type systems | Timing  | Remarks                                                                                            |
| ---------------------- | ------------- | ------------ | ------- | -------------------------------------------------------------------------------------------------- |
| **Duck typing**        | Any Python    | Structural   | Runtime | avoiding `isinstance` checks                                                                       |
| **Goose typing**       | Python >= 2.6 | Nominal      | Runtime | using `isinstance` checks against ABCs                                                             |
| **Static typing**      | Python >= 3.5 | Nominal      | Static  | PEP 484[^pythonorg__PEP_484] <br> type hints and <br> external type checker                        |
| **Static duck typing** | Python >= 3.8 | Structural   | Static  | PEP 544[^pythonorg__PEP_544] <br> `typing.Protocol` <br> type hints and <br> external type checker |

### Two Kinds of Protocol

Dynamic Protocol
: The informal protocols of Python.
Dynamic protocols are implicit, defined by convention, described in the documentation.
These protocols are supported by the Python interpreter.
Documentation can be found in the Data Model Chapter of the Python Language Reference[^pythonorg__data_model]

Static Protocol
: A protocol defined in PEP 544[^pythonorg__PEP_544], since Python 3.8.
Static protocol has an explicit definition which is defined by subclassing `typing.Protocol`[^pythonorg__typing_protocol]

### Programming Ducks

* An abstract of `collections.abc.Sequence`[^pythonorg__collections_abc_collections]
* Monkey patching for satisfying dynamic protocol at runtime

### Goose Typing

* Subclassing from library defined ABCs
* Implementation of an user defined ABC
* Subclassing from the user defined ABC
* Registering a class as a virtual subclass of an ABC using `register` method.
* `isinstance` against ABCs
    * Nominal Typing Aspect of ABC by using `issubclass(SubClass, ABCClass)`
    * Structural Typing Aspect of ABC and `__subclasshook__()` method

### Static Protocols

* Definition of a static protocol using subclass of `typing.Protocol`
* Type annotation using a static protocol
* Runtime type check with a static protocol
* How to make the existing class support a static protocol
* How to design a static protocol and best practices for protocol design
* How to extend an existing protocol
* Usage of `numbers` ABCs and numeric protocols


## References

[^wikipedia__nominal_type_system]:[Wikipedia : Nominal Type System](https://en.wikipedia.org/wiki/Nominal_type_system)
[^wikipedia__structural_type_system]:[Wikipedia : Structural Type System](https://en.wikipedia.org/wiki/Structural_type_system)
[^wikipedia__interface_segregation_principle]:[Wikipedia : Interface segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle)
[^pythonorg__PEP_484]:[PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
[^pythonorg__PEP_544]:[PEP 544 – Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-0544/)
[^pythonorg__PEP_544_merging_and_extending_protocols]:[PEP 544 – Protocols: Structural subtyping (static duck typing) : Merging and extending protocols](https://peps.python.org/pep-0544/#merging-and-extending-protocols)
[^pythonorg__PEP_3119]:[PEP 3119 – Introducing Abstract Base Classes](https://peps.python.org/pep-3119/)
[^pythonorg__abc]:[abc — Abstract Base Classes](https://docs.python.org/3/library/abc.html)
[^pythonorg__typing]:[typing — Support for type hints](https://docs.python.org/3/library/typing.html)
[^pythonorg__typing_protocol]:[typing — Support for type hints : `typing.Protocol`](https://docs.python.org/3/library/typing.html#typing.Protocol)
[^pythonorg__typing_protocols]:[typing — Support for type hints : Protocols](https://docs.python.org/3/library/typing.html#protocols)
[^pythonorg__numbers]:[numbers — Numeric abstract base classes](https://docs.python.org/3/library/numbers.html)
[^pythonorg__data_model]:[The Python Language Reference » 3. Data model](https://docs.python.org/3/reference/datamodel.html)
[^pythonorg__glossary_abstract_base_class]:[Glossary : abstract base class](https://docs.python.org/3/glossary.html#term-abstract-base-class)
[^pythonorg__collections_abc_collections]:[collections.abc — Abstract Base Classes for Containers : Collections Abstract Base Classes](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes)
[^pythonorg__random_shuffle]:[random — Generate pseudo-random numbers : random.shuffle(x)](https://docs.python.org/3/library/random.html#random.shuffle)
[^pythonorg__built_in_types_str_split]:[Built-in Types : `str.split`](https://docs.python.org/3/library/stdtypes.html#str.split)
[^pythonorg__glossary_duck_typing]:[Glossary : duck typing](https://docs.python.org/3/glossary.html#term-duck-typing)
[^pythonorg__built_in_functions_hasattr]:[Built-in Functions : `hasattr`](https://docs.python.org/3/library/functions.html#hasattr)
[^pythonorg__data_model_special_lookup]:[The Python Language Reference » 3. Data model : 3.3.11. Special method lookup](https://docs.python.org/3/reference/datamodel.html#special-lookup)
[^pythonorg__built_in_functions_isinstance]:[Built-in Functions : `isinstance`](https://docs.python.org/3/library/functions.html#isinstance)
[^pythonorg__built_in_functions_issubclass]:[Built-in Functions : `issubclass`](https://docs.python.org/3/library/functions.html#issubclass)
[^pythonorg__glossary_iterator]:[Glossary : `iterator`](https://docs.python.org/3/glossary.html#term-iterator)
[^pythonorg__built_in_types_iterator]:[Built-in Types : Iterator Types](https://docs.python.org/3/library/stdtypes.html#iterator-types)
[^pythonorg__exceptions_hierarchy]:[Built-in Exceptions : Exception hierarchy](https://docs.python.org/3/library/exceptions.html#exception-hierarchy)
[^pythonorg__typing_runtime_checkable]:[typing — Support for type hints : `typing.runtime_checkable`](https://docs.python.org/3/library/typing.html#typing.runtime_checkable)
[^pythonorg__typing_supportscomplex]:[typing — Support for type hints : `typing.SupportsComplex`](https://docs.python.org/3/library/typing.html#typing.SupportsComplex)
[^pythonorg__typing_supportsfloat]:[typing — Support for type hints : `typing.SupportsFloat`](https://docs.python.org/3/library/typing.html#typing.SupportsFloat)
[^github__python_typeshed_contributing]:[Python Repository : Contributing to typeshed](https://github.com/python/typeshed/blob/main/CONTRIBUTING.md#conventions)
[^external__martin_fowler_roleinterface]:[Martin Fowler : RoleInterface](https://martinfowler.com/bliki/RoleInterface.html)
