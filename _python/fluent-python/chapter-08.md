---
layout: distill
title: Chapter 08
description: Type Hints in Functions
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: About Gradual Typing
  - name: Gradual Typing in Practice
  - name: Types Are Defined by Supported Operations
  - name: Types Usable in Annotations
  - name: Annotating Positional Only and Variadic Parameters
  - name: Imperfect Typing and Strong Testing
  - name: Summary
  - name: Links
---

## Abstract

This is summary of chapter 8 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/).
Chapter 8 covers the topic of type hints in Python function, which allows developers to provide information about the expected data types for the function's parameters and return values.

## About Gradual Typing
A gradual type system:
* Is optional.
    * type checker assumes `Any` type when it cannot determine the type of an object.
* Does not catch type errors at runtime.
* Does not enhance performance.

## Gradual Typing in Practice
The following function `show_count()` is defined using a function definition *without* type hints for paramters and return value.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/messages/no_hints/messages.py'
     line='14-18'
     language='python'
></div>

### Starting with Mypy
* installation in VSCode is described in [here](../../development-environment/vscode/)

### Making Mypy More Strict
* CLI option `--disallow-untyped-defs` makes Mypy flag any function definition with no type hints for parameters and return value
* CLI option can be added in VSCode using option `"python.linting.mypyArgs":[]`

### A Default Parameter Value
* type hint for paramter
    * `<name-parameter>: <type-parameter> = <default-value>`
* type hint for return value
    * `def <name-function>(<arguments>) -> <type-return-value>:`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/messages/hints_2/messages.py'
     line='17-23'
     language='python'
></div>

### Using None as a Default
* `Optional` can be used for using `None` as a default value.
* Using `None` as a default value is good choice because `None` is immutable.

```python
from typing import Optional

def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
    if count == 1:
        return f'1 {singular}'
    count_str = str(count) if count else 'no'
    if not plural:
        plural = singular + 's'
    return f'{count_str} {plural}'
```

## Types Are Defined by Supported Operations

### Duck typing

* Python adopts this typing system.
* Objects have types.
* Variables including parameters are untyped.
    * Only the operations that the variable actually supports are important.
* Checking type by duck typing is executed at runtime.
* More flexible but more errors at runtime compared to nomial typing.

### Nominal typing
* Objects and variables including parameters have types.
* Typing is enforced statically.

## Types Usable in Annotations

### The Any Type
* `Any` is assumed to support every possible operation.

#### Subtype-of versus consistent-with
```python
class T1:
    ...

class T2(T1):
    ...

def f1(p: T1) -> None:
    ...

o2 = T2()

f1(o2) # OK
```

* `T2` is subtype-of `T1` means, `T2` is a subclass of `T1`
* If an operation `f1` can be applied to `T1`, `f1` can be applied to `T2` which is subtype-of `T1`.
    * -> Liskov Substitution Principle
    * if an object of type `T2` substitutes an object of type `T1` and the program still behaves correctly, then `T2` is subtype-of `T1`

##### rule for consistent-with
1. Given `T1` and a subtype `T2`, then `T2` is consistent-with `T1` (Liskov substitution).
2. Every type is consistent-with `Any`: you can pass objects of every type to an argument declared of type `Any`.
3. `Any` is consistent-with every type: you can always pass an object of type `Any` where an argument of another type is expected.

```python
class T1:
    ...

class T2(T1):
    ...

def f1(p: T1) -> None:
    ...

def f2(p: T2) -> None:
    ...

def f3(p: Any) -> None:
    ...

o0 = object()
o1 = T1()
o2 = T2()

f3(o0) #
f3(o1) # all OK: rule #2
f3(o2) #

def f4(): # implicit return type: `Any`
    ...

o4 = f4() # inferred type: `Any`

f1(o4) #
f2(o4) # all OK: rule #3
f3(o4) #
```

### Simple Types and Classes
* `int`, `float`, `str`, `bytes`

### Optional and Union Types
* `Optional[type]` is shortcut for `Union[type, None]`

    ```python
    from typing import Optional

    def show_count(count: int, singular: str, plural: Optional[str] = None) -> str:
        ...

    from typing import Union

    def show_count(count: int, singular: str, plural: Union[str, None] = None) -> str:
        ...

    # only available in Python 3.10 or above
    def show_count(count: int, singular: str, plural: str | None = None) -> str:
        ...
    ```

* `Union[typeA, typeB]` can be used in type of parameters or return value

    ```python
    from typing import Union
    def parse_token(token: str) -> Union[str, float]:
        try:
            return float(token)
        except ValueError:
            return token
    ```

* Nested `Union` is equivalant with unnested `Union`. The following two examples are equivalant.

    ```python
    Union[A, B, Union[C, D, E]]

    Union[A, B, C, D, E]
    ```

### Generic Collections
* Python collections like type `list` can hold different types of items.
* A type annotation can be used to constrain the collections to hold only items of the same type.
* The example below defines a function that returns a list that holds only members of type `str`.

    ```python
    # Available in Python >= 3.9
    def tokenize(text: str) -> list[str]:
        return text.upper().split()

    from typing import List
    def tokenize(text: str) -> List[str]:
        return text.upper().split()
    ```

* `stuff: list` and `stuff: list[Any]` is same meaning: list `stuff` can hold members of any type.

#### List of types which can be used in generic collections type hint

| collections type      | Python >= 3.9                   | Python >= 3.5            |
| :-------------------- | :------------------------------ | :----------------------- |
| `list`                | `list[itemType]`                | `typing.List`            |
| `collections.deque`   | `collections.deque[itemType]`   | `typing.Deque`           |
| `abc.Sequence`        | `abc.Sequence[itemType]`        | `typing.Sequence`        |
| `abc.MutableSequence` | `abc.MutableSequence[itemType]` | `typing.MutableSequence` |
| `set`                 | `set[itemType]`                 | `typing.Set`             |
| `abc.Container`       | `abc.Container[itemType]`       | `typing.Container`       |
| `abc.Set`             | `abc.Set[itemType]`             | `typing.Set`             |
| `abc.MutableSet`      | `abc.MutableSet[itemType]`      | `typing.MutableSet`      |
| `frozenset`           | `frozenset[itemType]`           | `typing.FrozenSet`       |
| `abc.Collection`      | `abc.Collection[itemType]`      | `typing.Collection`      |

* In Python 3.7 and 3.8, `from __future__ import annotations` is needed to use notations using built-in collections like `list[]`.
* details of `typing` can be found in [here](https://docs.python.org/3/library/typing.html)

### Tuple Types
* Tuples as records
* Tuples as records with named fields
* Tuples as immutable sequences

#### Tuples as records
* `tuple[str, float, str]` can accept a tuple like `('Shanghai', 24.28, 'China')`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/coordinates/coordinates.py'
     language="python"
     line='5-7'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/coordinates/coordinates.py'
     language="python"
     line='11-16'
></div>

#### Tuples as records with named fields
* `typing.NamedTuple` is consistent-with `tuple`.


<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/coordinates/coordinates_named.py'
     language="python"
     line='12-23'
></div>


<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/coordinates/coordinates_named.py'
     language="python"
     line='27-31'
></div>

#### Tuples as immutable sequences
* `tuple[int, ...]` is a tuple of unspecified length that are used as immutable lists with `int` items
* `stuff: tuple[Any, ...]` or `stuff: tuple` means that `stuff` is a tuple of unspecified length with objects of any type

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='08-def-type-hints/columnize.py'
     language="python"
     line='2-11'
></div>

* `list[tuple[str, ...]]` is list of tuple of unspecified length with str type items

```python
>>> animals = 'drake fawn heron ibex koala lynx tahr xerus yak zapus'.split()
>>> table = columnize(animals)
>>> table
[('drake', 'koala', 'yak'), ('fawn', 'lynx', 'zapus'), ('heron', 'tahr'), ('ibex', 'xerus')]
>>> for row in table:
...     print(''.join(f'{word:10}' for word in row))
...
drake     koala     yak
fawn      lynx      zapus
heron     tah
ibex      xerus
```

### Generic Mappings
* `MappingType[KeyType, ValueType]`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/charindex.py'
     language="python"
     line='15-34'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/charindex.py'
     language="python"
     line='5-11'
></div>

### Abstract Base Classes

#### *Postel's Law*

> Be conservative in what you send, be liberal in what you accept

* Set argument type hints as loosely as possible.
* Set return value type hints as strictly as possible.

```python
from collections.abc import Mapping

def name2hex(name: str, color_map: Mapping[str, int]) -> str:   # 1

def name2hex(name: str, color_map: dict[str, int]) -> str:      # 2
```

* `# 1` accepts `Mapping` type of color_map.
* `# 2` accepts `dict` type of color_map.
* `UserDict` is not subtype-of `dict`, so function in `# 2` cannot accept `UserDict` type variable as argument.
* `collections.abc.MutableMapping` or `collections.abc.Mapping` are appropriate because these types are superclass of `dict` and `UserDict`.
* In general, it is desirable to give arguments as loose type hints as possible.
    * [`typing.List` in Python document](https://docs.python.org/3/library/typing.html#typing.List) says that it is preferred to use abc type for annotating arguments.

#### The fall of the numeric tower
* `numbers.Number`
* `numbers.Complex` is subclass of `numbers.Number`
* `numbers.Real` is subclass of `numbers.Complex`
* `numbers.Rational` is subclass of `numbers.Real`
* `numbers.Integral` is subclass of `numbers.Rational`

* [PEP 484 - Type Hints](https://peps.python.org/pep-0484/) treats built-in types `int`, `float`, `complex` as special cases.
* For annoatating numeric arguments for static type checking, these options are possible.
    1. Use one of the concrete types int, float, or complex—as recommended by PEP 484.
    2. Declare a union type like Union[float, Decimal, Fraction].
    3. If you want to avoid hardcoding concrete types, use numeric protocols like `SupportsFloat`.

### Iterable

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/replacer.py'
     language="python"
     line='16-23'
></div>

* `# <1>` : defines type alias.
* `# <2>` : Argument `changes` accepts all iterable type variable which is used in for loop.

```python
>>> l33t = [('a', '4'), ('e', '3'), ('i', '1'), ('o', '0')]
>>> text = 'mad skilled noob powned leet'
>>> from replacer import zip_replace
>>> zip_replace(text, l33t)
'm4d sk1ll3d n00b p0wn3d l33t'
```

#### abc.Iterable versus abc.Sequence
* An argument type annotation `abc.Iterable` can accept endless iterable such as `itertools.cycle`.
    * A function that accepts `abc.Iterable` as argument can consume all memory and crash Python process.
* An argument type annotation `abc.Sequence` only accepts variable which have `__len__` method.
* Specifying `abc.Iterable` or `abc.Sequence` as return value type annotation is too vague.

### Parameterized Generics and TypeVar
* A parameterized generic is a generic type, written as `list[T]`, where `T` is a type variable that will be bound to a specific type with each usage.
* This allows a parameter type to be reflected in the result type.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/sample.py'
     language="python"
     line='2-13'
></div>

* The items in the `list` of return value are same type as the items in the `population` of `Sequence` type in the first argument.
    * If `population` is `tuple[int, ...]` which is consistent-with `Sequence[int]`, return type is `list[int]`
    * If `population` is `str` which is consistent-with `Sequence[str]`, return type is `list[str]`

#### Restrict TypeVar

```python
TypeVar('NameTypeVar', restrictType1, restrictType2, ...)
```

Additional positional arguments on `TypeVar` restricts the type parameter.

```python
from collections import Counter
from collections.abc import Iterable
from typing import TypeVar
from decimal import Decimal
from fractions import Fraction

NumberT = TypeVar('NumberT', float, Decimal, Fraction)

def mode(data: Iterable[NumberT]) -> NumberT:
    ...
```

#### Bounded TypeVar
* A keyword argument `bound` of `TypeVar` restrict the type parameter to subclass of the keyword argument `bound`.
* In the example below, any subclasses of `hashable` can be accepted as the type of elements of the `data`.
    * `hashable` have only the `__hash__` method.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/mode/mode_hashable.py'
     language="python"
     line='2-12'
></div>

```python
>>> mode(["red", "blue", "blue", "red", "green", "red", "red"]) 'red'
'red'
```

* A restricted type variable will be set to one of the types named in the `TypeVar` declaration.
* A bounded type variable will be set to the inferred type of the expression - as long as the inferred type is consistent-with the boundary declared in the `bound=` keyword argument of `TypeVar`.

#### The AnyStr predefined type variable
* `AnyStr` is a predefined type variable which accepts `bytes` or `str`
```python
AnyStr = TypeVar('AnyStr', bytes, str)
```

### Static Protocols

* A `typing.Protocol` type is defined by specifying one or more methods
* the type checker verifies that those methods are implemented where that protocol type is required.

#### *Usage*
1. import `typing.Protocol`
2. define a subclass of `typing.Protocol` which specifies methods for the protocol type.
3. use the subclass as the keyword argument `bound` of `TypeVar`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='2f8bf06270f58670cdddf07715d1d1526cc87df2'
     path='08-def-type-hints/comparable/comparable.py'
     language="python"
     line='1-4'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/comparable/top.py'
     language="python"
     line='23-32'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/comparable/top_test.py'
     language="python"
     line='2-7'
></div>

* `# <1>` : `typing.TYPE_CHECKING` is always `False` at runtime and always `True` at static type checking.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='08-def-type-hints/comparable/top_test.py'
     language="python"
     line='24-44'
></div>

* `# <2>` : Explicit type declaration for the `series` variable.
    * Mypy and Pylance annotates type of `series` as `Generator[tuple[int, LiteralString]]` if this type annotation does not exist.
* `# <3>` : This prevents execution of `reveal_type()` at runtime.
* `# <4>` : `reveal_type(variable)` reveals type of `variable`. This function is debug facility of Mypy and cannot be executed at runtime.
* `# <5>` : Mypy flags this line as an error

### Callable

* `Callable[[ParamType1, ParamType2], ReturnType]`
    * `collections.abc.Callable` is available for Python >= 3.9.
    * `typing.Callable` is available for Python < 3.9.
    * `[ParamType1, ParamType2]` can have 0 or 1, 2, 3, ... types.

#### *usage*
```python
from collection
def repl(input_fn: Callable[[Any], str] = input) -> None:

def input(__prompt: Any = ...) -> str: ...
    # input is consistent-with Callable[[Any], str]
```
* `Callable[..., ReturnType]` can be used for callable with arbitrary arguments type and number.
* A plain `Callable` is equivalant with `Callable[..., Any]`

#### Variance in Callable types

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='434f8d26b6a11e6f59aba288933192af92f211ad'
     path='08-def-type-hints/callable/variance.py'
     language="python"
     line='1-22'
></div>

* `# <6>` : Mypy flags this line because `Callable[[int], None]` is not *consistent-with* `Callable[[float], None]`.
* `# <8>` : `Callable[[complex], None]` is *consistent-with* `Callable[[float], None]`.

* `Callable[[], int]` is subtype-of `Callable[[], float]`.
    * Return value of type `int` have all method of type `float`.
* `Callable[[int],None]` is not a subtype-of `Callable[[float], None]`
    * Because type `int` has more method compare to type `float`, \
        argument inside of `Callable[[int],None]` would call method \
        which does not support in type `float` (for example, `hex` method).
* `Callable[[Complex],None]` is subtype of `Callable[[float],None]`.
* `Callable` is *covariant* on the return type.
* `Callable` is *contravariant* on the declared parameter types.

### NoReturn
* `NoReturn` is used only to annotate the return type of functions that never return.
    * `def exit(__status: object = ...) -> NoReturn: ...`

## Annotating Positional Only and Variadic Parameters

```python
# for Python >= 3.8
from typing import Optional
def tag( name: str,
        /,
        *content: str,
        class_: Optional[str] = None,
        **attrs: str,
) -> str:

# for Python <= 3.7
from typing import Optional
def tag(
    __name: str, # positional-only parameter must have name with two underscores
    *content: str,
    class_: Optional[str] = None,
    **attrs: str
) -> str:
```
* `content` is type of `tuple[str,...]` inside of function `tag`.
* `attrs` is type of `dict[str, str]` inside of function `tag`.
* `**attrs: float` is type of `dict[str, float]` inside of function.
    * keyword parameter have key which is type `str`

## Imperfect Typing and Strong Testing

### *Limitation of static type check in Python*
* Some handy features cannot be statically checked.
    * Argument unpacking like `config(**settings)` cannot be checked.
* Advanced features like properties, descriptors, metaclasses, and \
    metaprogramming in general are poorly supported.
* Type checkers lag behind Python releases, rejecting or even crashing while \
    analyzing code with new language features.

## *Summary*

### Simple Types and Classes

* `int`, `float`, `str`, `bytes`

### Union

* `typing.Union[typeA, typeB, typeC]`
* `typeA | typeB | typeC` (Python >= 3.10)

### Optional

* `Optional[type]` is shortcut for `Union[type, None]`

### Generic Collections

* `CollectionType[ItemType]`

| collections type      | Python >= 3.9                   | Python >= 3.5            |
| :-------------------- | :------------------------------ | :----------------------- |
| `list`                | `list[itemType]`                | `typing.List`            |
| `collections.deque`   | `collections.deque[itemType]`   | `typing.Deque`           |
| `abc.Sequence`        | `abc.Sequence[itemType]`        | `typing.Sequence`        |
| `abc.MutableSequence` | `abc.MutableSequence[itemType]` | `typing.MutableSequence` |
| `set`                 | `set[itemType]`                 | `typing.Set`             |
| `abc.Container`       | `abc.Container[itemType]`       | `typing.Container`       |
| `abc.Set`             | `abc.Set[itemType]`             | `typing.Set`             |
| `abc.MutableSet`      | `abc.MutableSet[itemType]`      | `typing.MutableSet`      |
| `frozenset`           | `frozenset[itemType]`           | `typing.FrozenSet`       |
| `abc.Collection`      | `abc.Collection[itemType]`      | `typing.Collection`      |

* In Python 3.7 and 3.8, `from __future__ import annotations` is needed to use notations using built-in collections like `list[]`.

### Tuple

* `tuple` is available on Python >= 3.9.
* `typing.Tuple` is available on Python >= 3.5.

#### Tuples as records

* `tuple[str, float, str]` have only 3 items which are `str`, `float` and `str` type

#### Tuples as records with named fields

* `tuple` can be used to type hints for `typing.NamedTuple`
    * `typing.NamedTuple` is consistent-with `tuple`.

#### Tuples as immutable sequences

* `tuple[int, ...]` is tuple of unspecified length which contains item of type `int`.

### Generic Mappings

* `MappingType[KeyType, ValueType]`

### Abstract Base Classes

* Set argument type hints as loosely as possible
    * `collections.abc.Mapping[str, int]` is better compare to `dict[str, int]`
* Set return value type hints as strictly as possible

### Sequence and Iterable

* Use `typing.Sequence` for function argument which is used as iterable with *limited* length.
    * For example if you need to know the length using the len function.
* Use `typing.Iterable` for function argument which is used as iterable with *unlimited* length.

### Parameterized Generics and TypeVar

* `TypeVar('T')` can be used when generic item type of argument and return type are same.

#### Restrict TypeVar

* `TypeVar('NameTypeVar', restrictType1, restrictType2, ...)`

#### Bounded TypeVar

* `HashableT = TypeVar('HashableT', bound=Hashable)`

### Static Protocols

* subclass of `typing.Protocol` can be used as `bound` argument of `TypeVar()`

```py
class SupportsLessThan(Protocol):
    def __lt__(self, other: Any) -> bool:
        ...

LT = TypeVar('LT', bound=SupportsLessThan)
```

### Callable

* `Callable[[ParamType1, ParamType2], ReturnType]`
* `collections.abc.Callable` is available for Python >= 3.9.
    * `typing.Callable` is available for Python < 3.9.

#### Variance in Callable types

* `Callable` is *covariant* on the return type.
* `Callable` is *contravariant* on the declared parameter types.

### NoReturn

* `NoReturn` is used only to annotate the return type of functions that never return.

### Annotating Positional Only and Variadic Parameters

* Positional argument `*args : str` is type of `tuple[str,...]` inside of function.
* Keyword argument `**kwargs : float` is type of `dict[str, float]` inside of function.

## Links

* [MyPy](https://mypy.readthedocs.io/en/stable/)
* [PEP 484 - Type Hints](https://peps.python.org/pep-0484/)
* [PEP 544 - Protocols: Structural subtyping(static duck typing)](https://peps.python.org/pep-0544/)
* [`typing` - Support for type hints](https://docs.python.org/3/library/typing.html)
* [Generic Alias Type](https://docs.python.org/3/library/stdtypes.html#types-genericalias)
* [`collections.abc` - Abstract Base Classes for Containers](https://docs.python.org/3/library/collections.abc.html)
* [Python Type Checking(Guide)](https://realpython.com/python-type-checking/#adding-stubs)
* [Hypermodern Python Chapter 4: Typing](https://cjolowicz.github.io/posts/hypermodern-python-04-typing/)
