---
layout: distill
title: Chapter 7
description: Functions as First-Class Objects
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Treating a Function Like an Object
  - name: Higher-Order Functions
  - name: Anonymous Functions
  - name: The Nine Flavors of Callable Objects
  - name: User-Defined Callable Types
  - name: From Positional to Keyword-Only Parameters
  - name: Packages for Functional Programming
  - name: Links
---

## Treating a Function Like an Object

* Functions have `__doc__` attribute, which is a [Docstring](https://peps.python.org/pep-0257/) of the function.
    * For consistency, always use `"""triple double quotes"""` around docstrings.
* `help(func)` in python console session displays `__doc__` attribute of `func()`.
* `type(func)` displays `<class 'function'>` which means `func` is an instance of `function` class.
* Instances of `function` class such as `func` can be assigned to other variables.
* Instances of `function` class such as `func` can be assigned to arguments of other functions.

## Higher-Order Functions

A function that takes a function as an argument or returns a function calls a *higher-order* function.

For example, `def sorted(items: List[T], key: Optional[Callable[[T], any]]) -> List[T]:` takes a function by second argument.

### Modern Replacements for map, filter, and reduce

`map` can be replaced with a list comprehension.

```python
>>> list(map(factorial, range(6)))
[1, 1, 2, 6, 24, 120]
>>> [factorial(n) for n in range(6)]
[1, 1, 2, 6, 24, 120]
```

`map` with `filter` can be replaced with a list comprehension with `if` statement.

```python
>>> list(map(factorial, filter(lambda n: n % 2, range(6))))
[1, 6, 120]
>>> [factorial(n) for n in range(6) if n % 2]
[1, 6, 120]
```

`reduce` with `add` can be replaced with a `sum` function.

```python
>>> from functools import reduce
>>> from operator import add
>>> reduce(add, range(100))
4950
>>> sum(range(100))
4950
```

`all(iterable)` returns `True` if there are no falsy elements in the `iterable`. `all([])` returns `True`.

`any(iterable)` returns `True` if any element of the `iterable` is truthy. `any([])` returns `False`.

## Anonymous Functions

`lambda <argument>, ...: <expression of return value>`

It is not recommended to use a `lambda` function if the `lambda` function is hard to read.

## The Nine Flavors of Callable Objects

*User-defined functions*
: can be created using `def` statements or `lambda` expressions.

*Built-in functions*
: are implemented in C (for CPython), such as `len` or `time.strftime`.

*Built-in methods*
: are also implemented in C, such as `dict.get`.

*Methods*
: are functions defined in the body of a class.

*Classes*
: When invoked, a class runs its `__new__` method to create an instance,
followed by `__init__` to initialize it.
The instance initialized by `__init__` is returned to the caller.
In Python, calling a class is similar to calling a function because there is no `new` operator.

*Class instances*
: If a class has a `__call__` method, then its instances can be invoked as functions.

*Generator functions*
: use the `yield` keyword in their body and return a generator object when called.

*Native coroutine functions*
: are defined using `async def` and return a coroutine object. Added in Python 3.5.

*Asynchronous generator functions*
: are defined using `async def` that have `yield` in their body.
When called, they return an asynchronous generator for use with `async for`.
Added in Python 3.6.

## User-Defined Callable Types

Objects can behave like functions by implementing a `__call__` instance method.

```python
class Multiplier:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, number):
        return self.factor * number
```

Instances of class `Multiplier` behave like functions by using `()` statement.

```python
>>> doubler = Multiplier(2)
>>> tripler = Multiplier(3)
>>> print(doubler(5))
10
>>> print(tripler(5))
15
>>> callable(tripler)
True
```

## From Positional to Keyword-Only Parameters

```python:Example7-9
def tag(name, *content, class_=None, **attrs):
    """Generate one or more HTML tags"""
    if class_ is not None:
        attrs['class'] = class_
    attr_pairs = (f' {attr}="{value}"' for attr, value in sorted(attrs.items()))
    attr_str = ''.join(attr_pairs)
    if content:
        elements = (f'<{name}{attr_str}>{c}</{name}>' for c in content)
        return '\n'.join(elements)
    else:
        return f'<{name}{attr_str} />'
```

```python:Example7-10
>>> tag('br')
'<br />'
>>> tag('p', 'hello')
'<p>hello</p>'
>>> print(tag('p', 'hello', 'world'))
<p>hello</p>
<p>world</p>
>>> tag('p', 'hello', id=33)
'<p id="33">hello</p>'
>>> print(tag('p', 'hello', 'world', class_='sidebar'))
<p class="sidebar">hello</p>
<p class="sidebar">world</p>
>>> tag(content='testing', name="img")
'<img content="testing" />'
>>> my_tag = {'name': 'img', 'title': 'Sunset Boulevard',
... 'src': 'sunset.jpg', 'class': 'framed'}
>>> tag(**my_tag)
'<img class="framed" src="sunset.jpg" title="Sunset Boulevard" />'
```

### *Specifying Keyword Argument*

When defining a function, keyword argument can be specified by using prefixed asterisk `*`.
This feature added in Python 3.

```python
def f(a, *, b):     # b is keyword argument
    return a, b

f(1, b=2)           # OK
f(1, 2)             # NG TypeError occurs
```
### Positional-Only Parameters

When defining a function, positional-only parameter can be specified by using postfixed slash `/`.
This feature added in Python 3.8.

```python
def divmod(a, b, /):
    return(a // b, a % b)
```

## Packages for Functional Programming

### The operator Module

#### *Simple Operation*

```python
from functools import reduce
def factorial(n):
    return reduce(lambda a, b: a*b, range(1, n+1))

from operator import mul        # multiplication defined in operator module
def factorial_with_operator(n):
    return reduce(mul, range(1, n+1))
```

#### *Retrive items with operator.itemgetter*

```python
>>> metro_data = [
...     ('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
...     ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
...     ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
...     ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
...     ('São Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
... ]
>>>
>>> from operator import itemgetter
>>> for city in sorted(metro_data, key=itemgetter(1)):
...     print(city)
...
('São Paulo', 'BR', 19.649, (-23.547778, -46.635833))       # items are sorted by second item of each data
('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
('Mexico City', 'MX', 20.142, (19.433333, -99.133333))
('New York-Newark', 'US', 20.104, (40.808611, -74.020386))

>>> cc_name = itemgetter(1, 0)  # second and first item are get by using cc_name.
>>> for city in metro_data:
...     print(cc_name(city))
...
('JP', 'Tokyo')
('IN', 'Delhi NCR')
('MX', 'Mexico City')
('US', 'New York-Newark')
('BR', 'São Paulo')
```

`operator.itemgetter` supports class that defines the  `__getitem__` method, because `itemgetter` uses the `[]` operator internally.

#### *Retrive attributes with operator.attrgetter*

```python
>>> from collections import namedtuple
>>> LatLon = namedtuple('LatLon', 'lat lon')
>>> Metropolis = namedtuple('Metropolis', 'name cc pop coord')
>>> metro_areas = [Metropolis(name, cc, pop, LatLon(lat, lon)) for name, cc, pop, (lat, lon) in metro_data]
>>> metro_areas[0]
Metropolis(name='Tokyo', cc='JP', pop=36.933, coord=LatLon(lat=35.689722, lon=139.691667))
>>> metro_areas[0].coord.lat
35.689722
>>> from operator import attrgetter
>>> name_lat = attrgetter('name', 'coord.lat')
>>>
>>> for city in sorted(metro_areas, key=attrgetter('coord.lat')):
...     print(name_lat(city))
...
('São Paulo', -23.547778)
('Mexico City', 19.433333)
('Delhi NCR', 28.613889)
('Tokyo', 35.689722)
('New York-Newark', 40.808611)
```

#### *Calling a instance method with operator.methodcaller*

Instance methods can be called by using `operator.methodcaller`.

```python
>>> from operator import methodcaller
>>> s = 'The time has come'
>>> upcase = methodcaller('upper')
>>> upcase(s)
'THE TIME HAS COME'
>>> hyphenate = methodcaller('replace', ' ', '-')
>>> hyphenate(s)
'The-time-has-come'
```

### Freezing Arguments with functools.partial

Arguments of a function can be reduced by using `functools.partial`.

```python
>>> from operator import mul
>>> from functools import partial
>>> triple = partial(mul, 3)
>>> triple(7)
21
>>> list(map(triple, range(1, 10)))
[3, 6, 9, 12, 15, 18, 21, 24, 27]
```

## Links

* [Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html)
* [PEP 3102 - Keyword-Only Argument](https://peps.python.org/pep-3102/)
* [Why is functools.partial necessary?](https://stackoverflow.com/questions/3252228/python-why-is-functools-partial-necessary)
