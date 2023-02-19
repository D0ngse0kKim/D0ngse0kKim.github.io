---
layout: distill
title: Chapter 8
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

## About Gradual Typing
A gradual type system:
* Is optional.
    * type checker assumes `Any` type when it cannot determine the type of an object.
* Does not catch type errors at runtime.
* Does not enhance performance.

## Gradual Typing in Practice
function definition without type hints for paramters and return value

```python
def show_count(count, word):
    if count == 1:
        return f"1 {word}"
    count_str = str(count) if count else "no"
    return f"{count_str} {word}s"
```

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

```python
def show_count(count: int, singular: str, plural: str = '') -> str:
    if count == 1:
        return f'1 {singular}'
    count_str = str(count) if count else 'no'
    if not plural:
        plural = singular + 's'
    return f'{count_str} {plural}'
```

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
* Duck typing (python adopts this typing system)
    * objects have types.
    * variables including parameters are untyped.
        * Only the operations that the variable actually supports are important.
    * checking type by duck typing is executed at runtime.
    * more flexible but more errors at runtime compared to nomial typing
* Nominal typing
    * objects and variables including parameters have types.
    * typing is enforced statically

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

```python
>>> shanghai = 31.2304, 121.4737
>>> geohash(shanghai)
'wtw3sjq6q'

from geolib import geohash as gh  # type: ignore

PRECISION = 9

def geohash(lat_lon: tuple[float, float]) -> str:
    return gh.encode(*lat_lon, PRECISION)
```

#### Tuples as records with named fields
* `typing.NamedTuple` is consistent-with `tuple`.

```python
from typing import NamedTuple
from geolib import geohash as gh # type: ignore

PRECISION = 9

class Coordinate(NamedTuple):
    lat: float
    lon: float

def geohash(lat_lon: Coordinate) -> str:
    return gh.encode(*lat_lon, PRECISION)

def display(lat_lon: tuple[float, float]) -> str:
    lat, lon = lat_lon
    ns='N' if lat >= 0 else 'S'
    ew='E' if lon >= 0 else 'W'
    return f'{abs(lat):0.1f}°{ns}, {abs(lon):0.1f}°{ew}'
```

#### Tuples as immutable sequences
* `tuple[int, ...]` is a tuple of unspecified length that are used as immutable lists with `int` items
* `stuff: tuple[Any, ...]` or `stuff: tuple` means that `stuff` is a tuple of unspecified length with objects of any type

```python
from collections.abc import Sequence

def columnize(
    sequence: Sequence[str], num_columns: int = 0
) -> list[tuple[str, ...]]:
    if num_columns == 0:
        num_columns = round(len(sequence) ** 0.5)
    num_rows, reminder = divmod(len(sequence), num_columns)
    num_rows += bool(reminder)
    return [tuple(sequence[i::num_rows]) for i in range(num_rows)]

# list[tuple[str, ...]] is list of tuple of unspecified length with str type items

>>> animals = 'drake fawn heron ibex koala lynx tahr xerus yak zapus'.split()
>>> table = columnize(animals)
>>> table
[('drake', 'koala', 'yak'), ('fawn', 'lynx', 'zapus'), ('heron', 'tahr'), ('ibex', 'xerus')]
>>> for row in table:
...     print(''.join(f'{word:10}' for word in row))
...
drake koala yak
fawn lynx zapus
heron tahr
ibex xerus
```

### Generic Mappings
* `MappingType[KeyType, ValueType]`

```python
import sys
import re
import unicodedata
from collections.abc import Iterator

RE_WORD = re.compile(r'\w+')
STOP_CODE = sys.maxunicode + 1

def tokenize(text: str) -> Iterator[str]:
    """return iterable of uppercased words"""
    for match in RE_WORD.finditer(text):
        yield match.group().upper()

def name_index(start: int = 32, end: int = STOP_CODE) -> dict[str, set[str]]:
    index: dict[str, set[str]] = {}
    for char in (chr(i) for i in range(start, end)):
        if name := unicodedata.name(char, ''):
            # unicodedata.name(char, '') is assigned to name and check name
            for word in tokenize(name):
                index.setdefault(word, set()).add(char)
    return index    # returns dict of str and set of str

>>> index = name_index(32, 65)
>>> index['SIGN']
{'$', '>', '=', '+', '<', '%', '#'}
>>> index['DIGIT']
{'8', '5', '6', '2', '3', '0', '1', '4', '7', '9'}
>>> index['DIGIT'] & index['EIGHT']
{'8'}
```

### Abstract Base Classes

#### *Postel's Law*

> Be conservative in what you send, be liberal in what you accept

* Set argument type hints as loosely as possible
* Set return value type hints as strictly as possible

```python
from collections.abc import Mapping

def name2hex(name: str, color_map: Mapping[str, int]) -> str:   # 1

def name2hex(name: str, color_map: dict[str, int]) -> str:      # 2
```

* `# 1` accepts `Mapping` type of color_map.
* `# 2` accepts `dict` type of color_map.
* `UserDict` is not subtype-of `dict`, so function in `# 2` cannot accept `UserDict` type variable as argument.
* `collections.abc.MutableMapping` or `collections.abc.Mapping` is appropriate because these types are superclass of `dict` and `UserDict`.
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
```python
from collections.abc import Iterable

FromTo = tuple[str, str]        # Type alias

def zip_replace(
    text: str,
    changes: Iterable[FromTo]   # changes accepts all iterable type used in for loop
) -> str:
    for from_, to in changes:
        text = text.replace(from_, to)
    return text

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
* A parameterized generic is a generic type, written as `list[T]`, where `T` is a type variable that will be bound to a specific type with **each usage**.
* This allows a parameter type to be reflected on the result type.

```python
from collections.abc import Sequence
from random import shuffle
from typing import TypeVar

T = TypeVar('T')

def sample(population: Sequence[T], size: int) -> list[T]:
    if size < 1:
        raise ValueError('size must be >= 1')
    result = list(population)
    shuffle(result)
    return result[:size]
```

* The items in the `list` of return value are same type as the items in the `population` of `Sequence` type in the first argument.
    * If `population` is `tuple[int, ...]` which is consistent-with `Sequence[int]`, return type is `list[int]`
    * If `population` is `str` which is consistent-with `Sequence[str]`, return type is `list[str]`

#### Restrict TypeVar

* `TypeVar('NameTypeVar', restrictType1, restrictType2, ...)`
    * Additional positional arguments on `TypeVar` restricts the type parameter.

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

```python
from collections import Counter
from collections.abc import Iterable, Hashable
from typing import TypeVar

HashableT = TypeVar('HashableT', bound=Hashable)

def mode(data: Iterable[HashableT]) -> HashableT:
    ...
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

```python
from collections.abc import Iterable
from typing import Any, Protocol, TypeVar  # 1.


class SupportsLessThan(Protocol):  # 2.
    def __lt__(self, other: Any) -> bool:
        ...


LT = TypeVar('LT', bound=SupportsLessThan)


def top(series: Iterable[LT], length: int) -> list[LT]:  # 3.
    ordered = sorted(series, reverse=True)
    return ordered[:length]

```

```python
from collections.abc import Iterator
from typing import TYPE_CHECKING

import pytest
from top import top


# several lines omitted
def test_top_tuples() -> None:
    fruit = 'mango pear apple kiwi banana'.split()
    series: Iterator[tuple[int, str]] = ((len(s), s) for s in fruit)
    # Pylance annotates type of `series` as Generator[tuple[int, LiteralString]].
    # This type hint provide type information of `series`
    #   as Iterator[tuple[int, str]] to Mypy
    length = 3
    expected = [(6, 'banana'), (5, 'mango'), (5, 'apple')]
    result = top(series, length)
    if TYPE_CHECKING:           # TYPE_CHECKING is True at static type checking
                                # TYPE_CHECKING is False at runtime
        reveal_type(series)     # reveal_type() is Mypy debugging facility
                                #   which prints a type of argument variable
        reveal_type(expected)
        reveal_type(result)
    assert result == expected


# intentional type error
def test_top_objects_error() -> None:
    series = [object() for _ in range(4)]
    if TYPE_CHECKING:
        reveal_type(series)
    with pytest.raises(TypeError) as excinfo:
        top(series, 3)      # Mypy flags this line as an error
    assert "'<' not supported" in str(excinfo.value)
```

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
* `Callable[..., ReturnType]` can be used for callable with arbitrary arguments type.

#### Variance in Callable types

```python
from collections.abc import Callable
def update(
    probe: Callable[[], float],
    display: Callable[[float], None]
) -> None:
    temperature = probe()
    display(temperature)

def probe_ok() -> int:
    return 42

def display_wrong(temperature: int) -> None:
    print(hex(temperature))

update(probe_ok, display_wrong) # type error
    # Argument 2 to "update" has incompatible type "Callable[[int], None]";
    # expected "Callable[[float], None]"  [arg-type]

def display_ok(temperature: complex) -> None:
    print(temperature)

update(probe_ok, display_ok) # OK
```

* `Callable[[], int]` is subtype-of `Callable[[], float]`
    * return value of type `int` have all method of type `float`
* `Callable[[int],None]` is not a subtype-of `Callable[[float], None]`
    * Because type `int` has more method compare to type `float`, \
        argument inside of `Callable[[int],None]` would call method \
        which does not support in type `float` (for example, `hex` method)
* `Callable[[Complex],None]` is subtype of `Callable[[float],None]`
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
