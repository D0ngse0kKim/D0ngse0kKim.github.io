---
layout: distill
title: Chapter 17
description: Iterators, Generators, and Classic Coroutines
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstarct
  - name: A Sequence of Words
  - name: Why Sequences Are Iterable The iter Function
  - name: Iterables Versus Iterators
  - name: Sentence Classes with __iter__
  - name: Lazy Sentences
  - name: When to Use Generator Expressions
  - name: An Arithmetic Progression Generator
  - name: Generator Functions in the Standard Library
  - name: Iterable Reducing Functions
  - name: Subgenerators with yield from
  - name: Generic Iterable Types
  - name: Classic Coroutines
  - name: Chapter Summary
  - name: Links
---

## Abstarct

This is a summary of the chapter 17 of the book "Fluent Python" by Luciano
Ramalho.

This chapter describes the following topics:
* Iterables, Iterators
* Generator Functions
* Generator Expressions
* Generator Functions in the Standard Library
* Classic Coroutines


## A Sequence of Words

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='17-it-generator/sentence.py'
     language='python'
     line='16-35'
></div>

```python
>>> s = Sentence('"The time has come," the Walrus said,')
>>> s
<Sentence('"The time ha... Walrus said,')>
>>> for word in s:  # <1>
...     print(word)
...
The
time
has
come
the
Walrus
said
>>> list(s)  # <2>
['The', 'time', 'has', 'come', 'the', 'Walrus', 'said']
>>> s[0]  # <3>
'The'
>>> s[5]  # <4>
'Walrus'
```

The example above shows that an object of the `Sentence` class is iterable.

* `<1>` : Because the `Sentence` object `s` is iterable, a `for` loop can be
    used to iterate over the  `s` object and produce the words in the text.
* `<2>` : Because the `Sentence` object `s` is iterable, it can be used as
    input to the `list` constructor to build a list of words.
* `<3>`, `<4>` : The `s` object implements the `__getitem__` method, which
    means that the `s` object supports indexing.


## Why Sequences Are Iterable: The iter Function

The `iter(object)` built-in function[^pythonorg__builtin_iter] does the
following processes:

1. If the `object` implements the `__iter__` method,
    * Call the `__iter__` method to obtain an iterator and returns the iterator.
2. If the `object` does not implement the `__iter__` method, but does
    implement the `__getitem__`,
    * Creates an iterator that attempts to fetch items in order, starting from
        index 0.
    * If the `__getitem__` method raises an `IndexError` exception, the
        iterator terminates.
    * Otherwise, the iterator increments the index and tries again.
3. If the `object` does not implement the `__iter__` method or the
    `__getitem__` method, the `TypeError` exception is raised.

### *How to check if an object is iterable?*

#### *Using Duck Typing*

Using duck typing, it is possible to check whether an object is iterable or not.

The following function is an example of using duck typing to check if an object
is iterable.

```python
>>> def is_iterable(obj):
...     try:
...         iter(obj)
...         return True
...     except TypeError:  # not iterable
...         return False
...
```

Simple examples of using the `is_iterable` function are shown below.

```python
>>> is_iterable('a string')
True
>>> is_iterable([1, 2, 3])
True
>>> is_iterable(100)
False
>>> is_iterable(3.14)
False
```

An object is iterable if the object implements the `__iter__` method which
returns an iterator or the `__getitem__` method.

```python
>>> class ImplementGetItem:
...     def __getitem__(self, index):
...         print(f'index = {index}')
...         if index >= 10:
...             raise IndexError
...         return index
...
>>> class ImplementIter:
...     def __iter__(self):
...         return iter(ImplementGetItem())  # returns an iterator
...
>>> is_iterable(ImplementIter())
True
>>> is_iterable(ImplementGetItem())
True
```

#### *Using Goose Typing*

Using goose typing, it is difficult to check whether an object is iterable or
not.

The following function is an example of using goose typing to check if an object
is iterable.

```python
>>> from collections.abc import Iterable
>>> def is_iterable_goose(obj):
...     return isinstance(obj, Iterable)
...
```

An object of the `ImplementIter` class pass the goose typing test, but an
object of the `ImplementGetItem` class does not pass the goose typing test
because the `ImplementGetItem` class does not implement the `__iter__` method.

```python
>>> is_iterable_goose(ImplementIter())
True
>>> is_iterable_goose(ImplementGetItem())
False
```

An object of the `ImplementInvalidIter` class passes the goose typing test,
but the object is not iterable.

```python
>>> class ImplementInvalidIter:
...     def __iter__(self):
...         pass
...
>>> is_iterable_goose(ImplementInvalidIter())
True
```

To sum up, the duck typing is more accurate than the goose typing in checking
if an object is iterable or not.

### Using iter with a Callable

The official documentation of the `iter()` built-in function
[^pythonorg__builtin_iter] states that the `iter()` built-in function can be
called with two arguments, like `iter(callable, sentinel)`.

The `iter(callable, sentinel)` built-in function returns an iterator that:
1. Repeatedly calls the `callable` argument with no arguments.
2. If the value returned by the `callable` argument is equal to the `sentinel`
    argument, the iterator terminates.
3. Otherwise, the iterator returns the value returned by the `callable`
    argument.

The following example shows how to use the `iter(callable, sentinel)` built-in
function.

```python
>>> def generate_random_number():
        import random
        return_value = random.randint(0, 5)
        print(f'generate_random_number() returns {return_value}')
        return return_value
...
>>> for random_number in iter(generate_random_number, 3):
...     print(random_number)
...
generate_random_number() returns 1
1
generate_random_number() returns 2
2
generate_random_number() returns 5
5
generate_random_number() returns 4
2
generate_random_number() returns 3
```

The following example is a more practical example of using the
`iter(callable, sentinel)` built-in function, which is described in the
official documentation of the `iter()` built-in function
[^pythonorg__builtin_iter].

```python
>>> from functools import partial
>>> with open('mydata.db', 'rb') as f:
...     read_64bytes = partial(f.read, 64)
...     for chunk in iter(read_64bytes, b''):
...         process_data(chunk)
...
```

## Iterables Versus Iterators

### *Iterable*

An object is **iterable** if the object returns an *iterator* when the
`iter(object)` built-in function is called.

There are two type of iterable objects:
1. Objects that implement the `__iter__` method returning an *iterator*.
2. Objects that implement the `__getitem__` method.

### *Iterator*

An object is an **iterator** if the object implements the following methods:
1. `__next__` method, which is an abstract method that:
    * Returns the next available item.
    * Raising `StopIteration` exception when there are no more items.
2. `__iter__` method, which is a mixin method that:
    * Returns `self`.

### *UML Diagram*

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-17/iterable_iterator.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="UML Diagram of the <code>iterable</code> and <code>iterator</code> ABCs.<br>Red : Abstract methods, Blue : Mixin methods."
        %}
    </div>
</div>

### *For Loop, `iter()`, and `next()` Built-in Functions*

The `for` loop, `iter()` and `next()` built-in functions are used to iterate
over an iterable object.

The following example shows a simple `for` loop on an iterable object.

```python
>>> s = 'ABC'
>>> for char in s:
...     print(char)
...
A
B
C
```

This `for` loop is equivalent to the following code.

```python
>>> s = 'ABC'
>>> it = iter(s)  # <1>
>>> while True:
...     try:
...         print(next(it))  # <2>
...     except StopIteration:  # <3>
...         del it  # <4>
...         break  # <5>
...
A
B
C
```

* `<1>` : The `iter()` built-in function is called to obtain an iterator from
    the iterable object `s`.
* `<2>` : The `next()` built-in function extracts the next item from the `it`
    iterator.
* `<3>` : If there are no further items in the `it` iterator, `next(it)` raises
    the `StopIteration` exception to signal the end of iteration.
* `<4>` : The `del` statement deletes the `it` iterator.
* `<5>` : The `break` statement terminates the `while` loop.

Therefore, `for` loop generates an iterator from an iterable object by using
`iter()` built-in function, and then repeatedly calls `next()` built-in
function over the iterator until the `StopIteration` exception is raised.

### *Implementation of iterable and iterator ABCs in Standard Library*

The following table shows the implementation of iterable and iterator ABCs in
the standard library.

<div class='embed-github-src'
     repo='python/cpython'
     branch='b1930bf75f276cd7ca08c4455298128d89adf7d1'
     path='Lib/_collections_abc.py'
     language='python'
     line='253-287'
></div>

`__subclasshook__` makes the `Iterable` and `Iterator` ABCs supports structural
type checking.

### *Advice to check whether an object is iterator or not*

According to the following comments in the source code of the standard library,
iterator in Python is a protocol, not a type.

<div class='embed-github-src'
     repo='python/cpython'
     branch='ed4a978449c856372d1a7cd389f91cafe2581c87'
     path='Lib/types.py'
     language='python'
     line='6-9'
></div>

Therefore, it is recommended to check whether an object is iterator or not
using `isinstance(obj, collections.abc.Iterator)`.

### *Properties of Iterators*

The protocol of iterator defines that:
1. An iterator must implement the `__next__` method.
2. An iterator must implement the `__iter__` method that returns the iterator
    itself.

Therefore, iterators have the following properties:
1. There is no way to check whether there are remaining items in an iterator
    without calling the `next()` built-in function and catching
    the `StopIteration` exception.
2. It is not possible to reset an iterator. If you need to start over, you must
    create a new iterator object by calling `iter()` built-in function on
    an iterable.
3. Calling `iter()` on an iterator returns the iterator itself. Therefore,
    calling `iter()` on an iterator does not build a new iterator.

## Sentence Classes with `__iter__`

### Sentence Take #2: A Classic Iterator

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='17-it-generator/sentence_iter.py'
     language='python'
     line='9-43'
></div>

* `<1>` : The `__iter__` method is implemented to return an iterator.
* `<2>` : Because the `SentenceIterator` is an iterator class,
    the `SentenceIterator(self.words)` is an iterator.
* `<3>` : `SentenceIterator` holds a reference to the list of words in the
    `Sentence` object.
* `<4>` : `SentenceIterator` holds index of the next word to fetch.
* `<5>` : Get the word at `self.index`.
* `<6>` : If `IndexError` is raised because `self.index` is out of range,
    raise `StopIteration` exception.
* `<7>`, `<8>` : Otherwise, increment `self.index` and return the word.
* `<9>` : `__iter__` methods implements the protocol of iterator by returning
    the iterator itself.

### Don't Make the Iterable an Iterator for Itself

Iterable
: Iterables have an `__iter__` method that instantiates a new iterator every
    time.
    It is not recommended to make an iterable an iterator for itself by
    implementing `__next__` method in the iterable class.

Iterator
: Iterators have a `__next__` method that returns individual items, and an
    `__iter__` method that returns `self`.

#### *Iterator Pattern*

The "Applicability" section of the "Iterator" design pattern[^gof__iterator]
states that:

> Use the Iterator pattern
> * to access an aggregate object's contents without exposing its internal
>   representation.
> * to support multiple traversals of aggregate objects.
> * to provide a uniform interface for traversing different aggregate
>   structures (that is, to support polymorphic iteration).

*Aggregate objects* means that the objects that have internal collections of
other objects, such as lists, tuples, sets, and dictionaries.

To support *multiple traversals* of aggregate objects, the iterator pattern
generates a new independant iterator from same iterable object every time the
`__iter__` method is called.

Therefore, it is not recommended to make an iterable an iterator for itself by
implementing `__next__` method in the iterable class.

### Sentence Take #3: A Generator Function

Using a generator function makes it possible to implement the `Sentence` class
more concisely without having to implement another iterator class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/sentence_gen.py'
     language='python'
     line='6-27'
></div>

* `<1>` : Iterate over `self.words`.
* `<2>` : `yield` keyword is used to return the word.
* `<3>` : Explicit `return` keyword is not used, and explicit `StopIteration`
    exception is not raised.
* `<4>` : No need to implement a separate iterator class.

In the following example, the `__iter__` method is a generator function, which
returns an iterator object (a generator object).

### How a Generator Works

A generator function is a function that:
1. contains one or more `yield` keywords in its body.
2. returns a generator object when it is called.

The following example shows how a generator function works.

```python
>>> def simple_generator_function():
...     yield 1
...     yield 2
...     yield 3
...
>>> simple_generator_function
<function simple_generator_function at 0x7f8b1c0b7d30>
>>> simple_generator_function()
<generator object simple_generator_function at 0x7f8b1c0b7e40>
>>> for value in simple_generator_function():
...     print(value)
...
1
2
3
>>> gen = simple_generator_function()
>>> next(gen)
1
>>> next(gen)
2
>>> next(gen)
3
>>> next(gen)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

* Generator functions return a generator object when they are called.
* Generator objects implement the `Iterator` protocol, therefore they can be
    used in a `for` loop.
* The `next()` built-in function on a generator object returns the next item
    in the generator object.
* The next item in the generator object is the value `yield`ed from
    its generator function.
* When there are no more items in the generator object, the `next()` built-in
    function raises the `StopIteration` exception.

The following example shows how the generator function works:

```python
>>> def generator_function():
...     print('before yield 1')
...     yield 1
...     print('before yield 2')
...     yield 2
...     print('end of generator function')
...
>>> generator_function()
<generator object generator_function at 0x7f8b1c0b7e40>
>>> gen = generator_function()
>>> next(gen)
before yield 1
1
>>> next(gen)
before yield 2
2
>>> next(gen)
end of generator function
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

* When a generator function is called, the body of the generator function is
    not executed.
* The body of the generator function is executed when the `next()` built-in
    function is called on the generator object.
* The body of the generator function is executed until the `yield` keyword is
    reached.
* The value after the `yield` keyword is returned by the `next()` built-in
    function.
* The body of the generator function is executed until the end of the generator
    function is reached.
* The `StopIteration` exception is raised when the end of the generator
    function is reached.

## Lazy Sentences

The above the `Sentence` class is not lazy because the whole list of words is
built when the `__init__` method is called.

In this chapter, lazy version of the `Sentence` class will be implemented.

In modern programmming, laziness is considered a good trait because it can
reduce memory usage, and may avoid waisting CPU cycles in case the program
does not need to use the whole list of words.

### Sentence Take #4: Lazy Generator

In the following example, `re.finditer` function is used to find all matches
of a regular expression `RE_WORD` in a `self.text`. The difference of
`re.findall()` and `re.finditer()` can be described as follows:

* `re.findall()` : returns a list of all matches.
* `re.finditer()` : returns a generator that yields `re.Match` one by one.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='17-it-generator/sentence_gen2.py'
     language='python'
     line='6-23'
></div>

* `<2>` : `RE_WORD.finditer(self.text)` builds an iterator over the matches
    of the regular expression `RE_WORD` in `self.text`. This contributes to
    saving memory because the whole list of words is not built.
* `<3>` : `match.group()` returns the word matched by the regular expression
    `RE_WORD`.

### Sentence Take #5: Lazy Generator Expression

By using a generator expression, the `Sentence` class can be implemented more
concisely. Before describing the implementation of the `Sentence` class using
a generator expression, a simple example of using a generator expression is
shown below.

```python
>>> def generator_function():
...     print('before yield 1')
...     yield 1
...     print('before yield 2')
...     yield 2
...     print('end of generator function')
...
>>> ret_val = [x for x in generator_function()]  # <1>
before yield 1
before yield 2
end of generator function
>>> ret_val
[1, 2]
>>> ret_val_gen = (x for x in generator_function())  # <2>
>>> ret_val_gen
<generator object <genexpr> at 0x7f8b1c0b7e40>
>>> for x in ret_val_gen:
...     print(x)
...
before yield 1
1
before yield 2
2
end of generator function
```

* `<1>` : When building a list comprehension from the generator function,
    the body of the generator function is executed immediately.
* `<2>` : When building a generator expression from the generator function,
    execution of the body of the generator function is delayed until the
    generator expression is iterated over.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='17-it-generator/sentence_genexp.py'
     language='python'
     line='6-21'
></div>

## When to Use Generator Expressions

Generator expressions are a syntatic sugar for creating generator objects,
replacing a generator function.
The rules for using generator expressions are as follows:
* If the generator expression spans more than a couple of lines, it is
    better to use a generator function.

### *Syntax Tip*

When a generator expression is passed as a single argument to a function, there
is no need to enclose the generator expression in parentheses.

```python
>>> sum(x * x for x in range(10))  # <1>
285
>>> sum((x * x for x in range(10)))  # <2>
285
>>> sum(x * x for x in range(10)) == sum((x * x for x in range(10)))  # <3>
True
```

However, if there are more than one arguments, the generator expression must be
enclosed in parentheses.

### *Comparing iterators and generators*

The following table compares iterators and generators.

*iterator*
: Any object implementing a `__next__` method is an iterator. Iterator objects
    returns individual items when `next()` built-in function is called on the
    iterator object or when the iterator object is used in a `for` loop.

*generator*
: An iterator built by the Python compiler from a generator function or
    generator expression. Therefore, there is no need to implement the
    `__next__` method for a generator object. Instead, `yield` keyword is used
    to make a generator function, which is a factory of generator objects.
    A generator expression is a syntatic sugar for creating generator objects,
    replacing a generator function.

## An Arithmetic Progression Generator

In this section, an arithmetic progression generator is implemented for not
only integers but also floating point numbers.

The following examples shows how to use a `ArithmeticProgression` class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='17-it-generator/aritprog_v1.py'
     language='python'
     line='6-22'
></div>

The `ArithmeticProgression` class is implemented as follows:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='17-it-generator/aritprog_v1.py'
     language='python'
     line='29-44'
></div>

Generator function version of the `ArithmeticProgression` class is implemented
as follows:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='17-it-generator/aritprog_v2.py'
     language='python'
     line='23-30'
></div>

### Arithmetic Progression with itertools

The `itertools` module[^pythonorg__itertools] provides various generator
functions for creating iterators.
By using generator functions in the `itertools` module, it is possible to build
the `aritprog_gen()` generator function more concisely.

`itertools.takewhile(predicate, it)` yields items from `it` until `predicate`
function returns `false`. The following example shows how to use
`itertools.takewhile()`.

```python
>>> import itertools
>>> list(itertools.takewhile(lambda n: n < 3, itertools.count(start=0, step=0.5)))
[0, 0.5, 1.0, 1.5, 2.0, 2.5]
```

By using the above `itertools.takewhile()` function, the `aritprog_gen()` can be
implemented more concisely. The following example shows the implementation of
the `aritprog_gen()` generator function using the `itertools.takewhile()`
function.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/aritprog_v3.py'
     language='python'
     line='2-10'
></div>

## Generator Functions in the Standard Library

### *Filtering generator functions*

| Module      | Function                          | Description                                                                                                                                                                 |
| ----------- | --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `itertools` | `compress(it, selector_it)`       | Consume two iterators in parallel; yield items from `it` whenever the corresponding item in `selector_it` is truthy.                                                        |
| `itertools` | `dropwhile(predicate, it)`        | Consumes `it`, skipping items while `predicate(item)` computes truthy, then yields every remaining item.                                                                    |
| `itertools` | `takewhile(predicate, it)`        | Consumes `it`, yielding items while `predicate(item)` computes truthy, then stops;                                                                                          |
| (built-in)  | `filter(predicate, it)`           | Applies `predicate()` function to each item of `it`, yielding the item if `predicate(item)` is truthy.; if `predicate` is `None`, only truthy items are yielded.            |
| `itertools` | `filterfalse(predicate, it)`      | Applies `predicate()` function to each item of `it`, yielding the item if `predicate(item)` is falsy.; if `predicate` is `None`, only falsy items are yielded.              |
| `itertools` | `islice(it, stop)`                | Returns a new iterator from `it` that returns first `stop` items, then raises `StopIteration`. `it` can be any iterable and the yielding operation is lazy.                 |
| `itertools` | `islice(it, start, stop, step=1)` | Returns a new iterator from `it` that returns items from `start` index until `stop` index, stepping by `step`. `it` can be any iterable and the yielding operation is lazy. |

The following code shows how to use the above filtering generator functions.

```python
>>> import itertools
>>> list(itertools.compress('ABCDEF', [1, 0, 1, 0, 1, 1]))
['A', 'C', 'E', 'F']
>>> list(itertools.compress('ABCDEF', [1, 0, 1, 0, 1, 1, 0]))  # Longer selector
['A', 'C', 'E', 'F']
>>> list(itertools.compress('ABCDEF', [1, 0, 1]))  # Shorter selector
['A', 'C']
>>> list(itertools.dropwhile(lambda x: x < 5, [1, 4, 6, 4, 1]))
[6, 4, 1]
>>> list(itertools.takewhile(lambda x: x < 5, [1, 4, 6, 4, 1]))
[1, 4]
>>> list(filter(lambda x: x < 5, [1, 4, 6, 4, 1]))
[1, 4, 4, 1]
>>> list(filter(None, [1, 0, 2, 0, 3, 0, 4]))  # Truthy values
[1, 2, 3, 4]
>>> list(itertools.filterfalse(lambda x: x < 5, [1, 4, 6, 4, 1]))
[6]
>>> list(itertools.filterfalse(None, [1, 0, 2, 0, 3, 0, 4]))  # Falsy values
[0, 0, 0]
>>> list(itertools.islice('ABCDEFG', 2))  # Stop is specified
['A', 'B']
>>> list(itertools.islice('ABCDEFG', 2, 4))  # Start and stop is specified
['C', 'D']
>>> list(itertools.islice('ABCDEFG', 2, None))  # Start and stop is specified
['C', 'D', 'E', 'F', 'G']
>>> list(itertools.islice('ABCDEFG', 0, None, 2))  # Start, stop, and step is specified
['A', 'C', 'E', 'G']
```

### *Mapping generator functions*

| Module      | Function                          | Description                                                                                                                                                                           |
| ----------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `itertools` | `accumulate(it, [func])`          | Yields accumulated sums; if `func` is provided, yields the result of applying it to the first pair of items, then to the first result and next item, and so on.                       |
| (built-in)  | `enumerate(it, start=0)`          | Yields 2-tuples of the form `(index, item)`, where `index` is counted from `start`, and `item` is taken from `it`.                                                                    |
| `itertools` | `map(func, it1, [it2, ..., itN])` | Applies `func` to each item of `it`, yielding the result. If `it2`, ..., `itN` is given, `func` must be able to take `N` arguments and the arguments are taken from each iterable.    |
| `itertools` | `starmap(func, it)`               | Applies `func` to each item of `it`, yielding the result. If `it` is a sequence of `N`-tuples, `func` must be able to take `N` arguments and the arguments are taken from each tuple. |

The following code shows how to use the above mapping generator functions.

```python
>>> import itertools
>>> list(itertools.accumulate([1, 2, 3, 4, 5]))
[1, 3, 6, 10, 15]
>>> import operator
>>> list(itertools.accumulate([1, 2, 3, 4, 5], operator.mul))
[1, 2, 6, 24, 120]
>>> list(itertools.accumulate([5, 3, 7, 5, 9, 2, 0], min))
[5, 3, 3, 3, 3, 2, 0]
>>> list(itertools.accumulate([5, 3, 7, 5, 9, 2, 0], max))
[5, 5, 7, 7, 9, 9, 9]
>>> list(enumerate('ABC'))
[(0, 'A'), (1, 'B'), (2, 'C')]
>>> list(enumerate('ABC', 1))
[(1, 'A'), (2, 'B'), (3, 'C')]
>>> list(map(lambda x: x * x, range(5)))
[0, 1, 4, 9, 16]
>>> list(map(lambda x, y: x * y, range(5), range(5, 0, -1)))
[0, 4, 6, 6, 4]
>>> list(map(lambda x, y: x * y, range(5), range(5, 3, -1)))  # Different length
[0, 4]
>>> list(itertools.starmap(lambda x, y: x * y, [(0, 5), (1, 4), (2, 3), (3, 2), (4, 1)]))
[0, 4, 6, 6, 4]
```

### *Merging generator functions*

| Module      | Function                                     | Description                                                                                                                                                                                                                |
| ----------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `itertools` | `chain(it1, ..., itN)`                       | Yields items from each iterable until it is exhausted, then proceeds to the next iterable, until all of the iterables are exhausted.                                                                                       |
| `itertools` | `chain.from_iterable(it)`                    | Yields items from each iterable produced by `it`, one after the other, until all of the iterables are exhausted.                                                                                                           |
| `itertools` | `product(it1, ..., itN, repeat=1)`           | Yields tuples containing the Cartesian product of the items from all iterables. `repeat` specifies the number of times that each item can be repeated in the tuples.                                                       |
| (built-in)  | `zip(it1, ..., itN, strict=False)`           | Yields N-tuples built from items taken from the iterables in parallel, silently stopping when the first iterable is exhausted. If `strict` is `True` and the length of iterable is different, `zip()` raises `ValueError`. |
| `itertools` | `zip_longest(it1, ..., itN, fillvalue=None)` | Yields N-tuples built from items taken from the iterables in parallel, stopping only when the last iterable is exhausted, filling blanks with `fillvalue`.                                                                 |

The following code shows how to use the above merging generator functions.

```python
>>> import itertools
>>> list(itertools.chain('ABCD', [0,1,2,3]))
['A', 'B', 'C', 'D', 0, 1, 2, 3]
>>> list(itertools.chain(enumerate('ABCD')))  # chain with one iterable is useless
[(0, 'A'), (1, 'B'), (2, 'C'), (3, 'D')]
>>> list(itertools.chain.from_iterable(enumerate('ABCD')))
[0, 'A', 1, 'B', 2, 'C', 3, 'D']
>>> list(itertools.product('AB', range(2)))
[('A', 0), ('A', 1), ('B', 0), ('B', 1)]
>>> from pprint import pprint
>>> pprint(list(itertools.product('AB', range(2), repeat=2)))
[('A', 0, 'A', 0),
 ('A', 0, 'A', 1),
 ('A', 0, 'B', 0),
 ('A', 0, 'B', 1),
 ('A', 1, 'A', 0),
 ('A', 1, 'A', 1),
 ('A', 1, 'B', 0),
 ('A', 1, 'B', 1),
 ('B', 0, 'A', 0),
 ('B', 0, 'A', 1),
 ('B', 0, 'B', 0),
 ('B', 0, 'B', 1),
 ('B', 1, 'A', 0),
 ('B', 1, 'A', 1),
 ('B', 1, 'B', 0),
 ('B', 1, 'B', 1)]
>>> list(zip('ABCD', 'xy'))
[('A', 'x'), ('B', 'y')]
>>> list(zip('ABCD', 'xy', range(5)))
[('A', 'x', 0), ('B', 'y', 1)]
>>> list(zip('ABCD', 'xy', range(5), strict=True))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: zip() argument 2 is shorter than argument 1
>>> list(itertools.zip_longest('ABCD', 'xy', range(5)))
[('A', 'x', 0), ('B', 'y', 1), ('C', None, 2), ('D', None, 3), (None, None, 4)]
>>> list(itertools.zip_longest('ABCD', 'xy', range(5), fillvalue='*'))
[('A', 'x', 0), ('B', 'y', 1), ('C', '*', 2), ('D', '*', 3), ('*', '*', 4)]
```

### *Expanding generator functions*

| Module      | Function                                     | Description                                                                                                                                |
| ----------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `itertools` | `combinations(it, out_len)`                  | Yields combinations of `out_len` items from `it`.                                                                                          |
| `itertools` | `combinations_with_replacement(it, out_len)` | Yields combinations of `out_len` items from `it`, allowing individual items to be repeated more than once.                                 |
| `itertools` | `count(start=0, step=1)`                     | Yields numbers starting at `start`, incremented by `step`, indefinitely.                                                                   |
| `itertools` | `cycle(it)`                                  | Yields items from `it`, saving a copy of each. When `it` is exhausted, yields items from the saved copy.                                   |
| `itertools` | `pairwise(it)`                               | Yields successive overlapping pairs from `it`; for example, `pairwise('ABC')` yields `('A', 'B')`, `('B', 'C')`.                           |
| `itertools` | `permutations(it, out_len=None)`             | Yields permutations of `out_len` items from `it`. `out_len` means the length of each permutation; by default, `out_len` is `len(list(it))` |
| `itertools` | `repeat(item, [times])`                      | Yields `item` over and over again, indefinitely unless `times` is specified.                                                               |

The following code shows how to use the above expanding generator functions.

```python
>>> import itertools
>>> list(itertools.combinations('ABC', 2))
[('A', 'B'), ('A', 'C'), ('B', 'C')]
>>> list(itertools.combinations_with_replacement('ABC', 2))
[('A', 'A'), ('A', 'B'), ('A', 'C'), ('B', 'B'), ('B', 'C'), ('C', 'C')]
>>> count_iterator = itertools.count(1, 0.5)
>>> next(count_iterator), next(count_iterator), next(count_iterator)
(1, 1.5, 2.0)
>>> list(itertools.islice(count_iterator, 3, 6))
[4.0, 4.5, 5.0]
>>> list(itertools.islice(itertools.cycle('ABC'), 5))
['A', 'B', 'C', 'A', 'B']
>>> list(itertools.pairwise('ABCDEF'))
[('A', 'B'), ('B', 'C'), ('C', 'D'), ('D', 'E'), ('E', 'F')]
>>> list(itertools.permutations('ABC', 2))
[('A', 'B'), ('A', 'C'), ('B', 'A'), ('B', 'C'), ('C', 'A'), ('C', 'B')]
>>> list(itertools.permutations('ABC'))  #  by default, out_len is len(list(it))
[('A', 'B', 'C'), ('A', 'C', 'B'), ('B', 'A', 'C'), ('B', 'C', 'A'), ('C', 'A', 'B'), ('C', 'B', 'A')]
>>> list(itertools.repeat('ABC', 3))
['ABC', 'ABC', 'ABC']
>>> list(itertools.repeat('ABC', 0))
[]
>>> list(map(pow, range(10), itertools.repeat(2)))
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

### *Rearranging generator functions*

| Module      | Function                 | Description                                                                                                                                                                                                           |
| ----------- | ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `itertools` | `groupby(it, [keyfunc])` | Yields 2-tuples of the form `(key, group)`, where `keyfunc` is the grouping criterion and `group` is a generator yielding the items in the group. `keyfunc` is used to extract the grouping criterion from each item. |
| (built-in)  | `reversed(seq)`          | Yields items from `seq` in reverse order. `seq` must be a sequence or implement the `__reversed__` special method.                                                                                                    |
| `itertools` | `tee(it, n=2)`           | Yields `n` independent iterators from a single iterable.                                                                                                                                                              |

The following code shows how to use the above rearranging generator functions.

```python
>>> import itertools
>>> from pprint import pprint
>>> pprint(list(itertools.groupby('LLLLAAGGG')))
[('L', <itertools._grouper object at 0x104c25de0>),
 ('A', <itertools._grouper object at 0x1052a3850>),
 ('G', <itertools._grouper object at 0x1052a37c0>)]
>>> for char, group in itertools.groupby('LLLLAAGGG'):
...     print(char, '->', list(group))
...
L -> ['L', 'L', 'L', 'L']
A -> ['A', 'A']
G -> ['G', 'G', 'G']
>>> animals = ['duck', 'eagle', 'rat', 'giraffe', 'bear', 'bat', 'dolphin', 'shark', 'lion']
>>> animals.sort(key=len)
>>> animals
['rat', 'bat', 'duck', 'bear', 'lion', 'eagle', 'shark', 'giraffe', 'dolphin']
>>> for length, group in itertools.groupby(animals, len):
...     print(length, '->', list(group))
...
3 -> ['rat', 'bat']
4 -> ['duck', 'bear', 'lion']
5 -> ['eagle', 'shark']
7 -> ['giraffe', 'dolphin']
>>> for length, group in itertools.groupby(reversed(animals), len):
...     print(length, '->', list(group))
...
7 -> ['dolphin', 'giraffe']
5 -> ['shark', 'eagle']
4 -> ['lion', 'bear', 'duck']
3 -> ['bat', 'rat']
>>>
```

```python
>>> import itertools
>>> r1, r2 = itertools.tee('ABC')
>>> r1
<itertools._tee object at 0x104c25f40>
>>> r2
<itertools._tee object at 0x104c25f40>
>>> list(itertools.islice(r1, 2))
['A', 'B']
>>> list(r2)
['A', 'B', 'C']
>>> list(r1)
['C']
>>> list(zip(*itertools.tee('ABC')))
[('A', 'A'), ('B', 'B'), ('C', 'C')]
>>> list(zip(*itertools.tee('ABC', 3)))
[('A', 'A', 'A'), ('B', 'B', 'B'), ('C', 'C', 'C')]
```

## Iterable Reducing Functions

The following functions are used to reduce iterables to single values.

| Module      | Function                       | Description                                                                                                                                                                                  |
| ----------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| (built-in)  | `all(it)`                      | Returns `True` if all items in `it` are truthy, otherwise returns `False`. Applying `all` to an iterator without item returns `True`, for example, `all([]) == True`.                        |
| (built-in)  | `any(it)`                      | Returns `True` if any item in `it` is truthy, otherwise returns `False`. Applying `any` to an iterator without item returns `False`, for example, `any([]) == False`.                        |
| (built-in)  | `max(it, [key=], [default=])`  | Returns the maximum item in `it`. `key` is an ordering function. `default` is returned if `it` is empty.                                                                                     |
| (built-in)  | `max(arg1, arg2, ..., [key=])` | Returns the maximum item in `arg1`, `arg2`, `...`. `key` is an ordering function. `default` is returned if `it` is empty.                                                                    |
| (built-in)  | `min(it, [key=], [default=])`  | Returns the minimum item in `it`. `key` is an ordering function. `default` is returned if `it` is empty.                                                                                     |
| (built-in)  | `min(arg1, arg2, ..., [key=])` | Returns the minimum item in `arg1`, `arg2`, `...`. `key` is an ordering function. `default` is returned if `it` is empty.                                                                    |
| `functools` | `reduce(func, it, [initial])`  | Returns the result of applying `func(x, y)` to the first pair of items in `it`, then to the result and the next item, and so on. If `initial` is given, `initial` treated as the first pair. |
| (built-in)  | `sum(it, [start=0])`           | Returns the sum of the items in `it`, plus `start` (which defaults to 0).                                                                                                                    |
| `math`      | `fsum(it)`                     | Returns an accurate floating-point sum of the items in `it`.                                                                                                                                 |

The following code shows how to use the above iterable reducing functions.

```python
>>> all([1, 2, 3])
True
>>> all([1, 2, 0])
False
>>> all([])
True
>>> all(itertools.islice(range(5), 0))
True
>>> any([1, 2, 3])
True
>>> any([1, 2, 0])
True
>>> any([0, 0, 0])
False
>>> any([])
False
>>> any(itertools.islice(range(5), 0))
False
>>> it = (item for item in [0, 0, 1, 2])
>>> any(it)  # yields item until it reaches the first truthy value. 2 is not yielded at this point.
True
>>> next(it)  # 2 will be yielded.
2
```

```python
>>> max([1, 2, 3])
3
>>> max([1, 2, 3], key=lambda x: -x)
1
>>> max([1, 2, 3], default=0)
3
>>> max([], default=0)
0
>>> max(1, 2, 3)
3
>>> max(1, 2, 3, key=lambda x: -x)
1
>>> min([1, 2, 3])
1
>>> min([1, 2, 3], key=lambda x: -x)
3
>>> min([1, 2, 3], default=0)
1
>>> min([], default=0)
0
>>> min(1, 2, 3)
1
>>> min(1, 2, 3, key=lambda x: -x)
3
```

```python
>>> import functools
>>> functools.reduce(lambda x, y: x * y, [1, 2, 3, 4, 5])
120
>>> functools.reduce(lambda x, y: x * y, [1, 2, 3, 4, 5], 10)
1200
>>> functools.reduce(lambda x, y: x * y, [1, 2, 3, 4, 5], 0)
0
>>> functools.reduce(lambda x, y: x * y, [], 0)
0
>>> functools.reduce(lambda x, y: x * y, [], 1)
1
>>> sum([1, 2, 3, 4, 5])
15
>>> sum([1, 2, 3, 4, 5], 10)
25
>>> sum([1, 2, 3, 4, 5], -10)
5
>>> sum([1, 2, 3, 4, 5], 0.0)
15.0
>>> import math
>>> math.fsum([1, 2, 3, 4, 5])
```

## Subgenerators with yield from

`yield from` is a new syntax introduced in Python 3.3. The `yield from`
statement is used to delegate to a subgenerator. The following example shows
how to use `yield from` statement.

The following example shows how to use `yield from` statement to delegate to
a subgenerator.

```python
>>> def gen():
...     for c in 'AB':
...         yield c
...     for i in range(1, 3):
...         yield i
...
>>> list(gen())
['A', 'B', 1, 2]
>>> def sub_gen_ab():
...     for c in 'AB':
...         yield c
...
>>> def sub_gen_12():
...     for i in range(1, 3):
...         yield i
...
>>> def delegating_gen():
...     yield 'a'
...     yield 'b'
...     yield from sub_gen_ab()
...     yield from sub_gen_12()
...
>>> list(delegating_gen())
['a', 'b', 'A', 'B', 1, 2]
```

In the above example, the following are the terminologies used in the
explanation of `yield from` statement.

1. `delegating_gen()` is the *delegating generator*.
2. `sub_gen_ab()` and `sub_gen_12()` are the *subgenerators*.
3. `yield from sub_gen_ab()` and `yield from sub_gen_12()` are the *yield from
    expressions*.

The following is the execution flow of the above example.

1. When `delegating_gen()` is called, it returns a generator object.
2. When the `next()` built-in function is called on the generator object
    returned by `delegating_gen()`, the execution of `delegating_gen()` starts.
3. When `yield 'a'` and `yield 'b'` is executed, delegating generator object
    yields `'a'` and `'b'`.
4. When `yield from sub_gen_ab()` is executed, the control is transferred to
    the subgenerator `sub_gen_ab()`.
5. When `sub_gen_ab()` is exhausted, the control is transferred back to the
    delegating generator.

When the subgenerator executes a `return` statement, the delegating generator
can capture the returned value from the subgenerator by using `yield from` as
follows:

```python
>>> def sub_gen():
...     yield 1
...     return 'return value'
...
>>> def delegating_gen():
...     yield from sub_gen()
...
>>> list(delegating_gen())
[1]
>>> def delegating_gen():
...     ret = yield from sub_gen()
...     print('sub_gen() returned', ret)
...
>>> list(delegating_gen())
sub_gen() returned return value
[1]
```

The following next subsections explain real-world examples of using
the `yield from` statement.

### Reinventing chain

The following example shows how to use the `yield from` statement to implement
`chain()` function.

```python
>>> def chain_without_yield_from(*iterables):
...     for it in iterables:
...         for item in it:
...             yield item
...
>>> list(chain_without_yield_from('ABC', range(3)))
['A', 'B', 'C', 0, 1, 2]
>>> def chain_with_yield_from(*iterables):
...     for it in iterables:
...         yield from it
...
>>> list(chain_with_yield_from('ABC', range(3)))
['A', 'B', 'C', 0, 1, 2]
```

### Traversing a Tree

In this subsection, an example of traversing a tree structure of subclasses
using the `yield from` statement is shown.

Before explaining the example, the following is properties contained in classes
in the example.

1. `class.__name__`
    : The name of the class.
2. `class.__subclasses__()`
    : A list of immediate subclasses of the class.

#### *Step0*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='8a330d822b997c7992d0b7675c82ad75832300c6'
     path='17-it-generator/tree/step0/tree.py'
     language='python'
     line='1-11'
></div>

The output of this step is as follows:

```
BaseException
```

#### *Step1*

In this example, the exception hierarchy levels 0 and 1 are traversed and
printed.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='17-it-generator/tree/step1/tree.py'
     language='python'
     line='1-14'
></div>

* `<1>` : `yield` statement can return two values. In this example, the second
    return value represents the depth in the tree structure.
* `<2>` : In this `for` loop, `cls.__subclasses__()` captures the subclasses of
    the class `cls`.
* `<3>` : The subclasses from the `cls.__subclasses__()` and the depth are
    yielded.
* `<4>` : This statement builds indentation string for the output.

The output of this step is as follows:

```
BaseException
    BaseExceptionGroup
    Exception
    GeneratorExit
    KeyboardInterrupt
    SystemExit
```

#### *Step2*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='17-it-generator/tree/step2/tree.py'
     language='python'
     line='1-18'
></div>

* `<1>` : The `for` loop with `yield` statement is replaced into a function
    `sub_tree(cls)` and the function is called by `yield from` statement.
* `<2>` : The `sub_tree(cls)` function yields the subclasses of the class `cls`
    and the depth `1`.

The output of this step is as follows:

```
BaseException
    BaseExceptionGroup
    Exception
    GeneratorExit
    KeyboardInterrupt
    SystemExit
```

### *Step3*

In this step, the exception hierarchy levels 0, 1, and 2 are traversed and
printed.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='17-it-generator/tree/step3/tree.py'
     language='python'
     line='1-20'
></div>

In this step, `sub_tree()` is extended to traverse the subclasses of the class
`cls` up to level 2 of the exception hierarchy.

The output of this step is as follows:

```
BaseException
    BaseExceptionGroup
        ExceptionGroup
    Exception
        ArithmeticError
        AssertionError
        AttributeError
        BufferError
        EOFError
        ImportError
        LookupError
        MemoryError
        NameError
        OSError
        ReferenceError
        RuntimeError
        StopAsyncIteration
        StopIteration
        SyntaxError
        SystemError
        TypeError
        ValueError
        Warning
        ExceptionGroup
        _OptionError
        error
        TokenError
        StopTokenizing
        ClassFoundException
        EndOfBlock
    GeneratorExit
    KeyboardInterrupt
    SystemExit
```

Step 4 will be skipped and step 5 will be explained directly.

### *Step5*

In this step, the `sub_tree()` function is extended to traverse the subclasses
of the class `cls` recursively.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='17-it-generator/tree/step5/tree.py'
     language='python'
     line='1-19'
></div>

### *Step6*

In this step, the `tree(cls)` function incorporates the implementation of
the `sub_tree()` function, and the `tree(cls, level)` function is recursively
called by the `yield from` statement.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='17-it-generator/tree/step6/tree.py'
     language='python'
     line='1-14'
></div>

The output of this step is as follows:

```
BaseException
    BaseExceptionGroup
        ExceptionGroup
    Exception
        ArithmeticError
            FloatingPointError
            OverflowError
            ZeroDivisionError
        AssertionError
        AttributeError
        BufferError
        EOFError
        ImportError
            ModuleNotFoundError
            ZipImportError
        LookupError
            IndexError
            KeyError
            CodecRegistryError
        MemoryError
        NameError
            UnboundLocalError
        OSError
            BlockingIOError
            ChildProcessError
            ConnectionError
                BrokenPipeError
                ConnectionAbortedError
                ConnectionRefusedError
                ConnectionResetError
            FileExistsError
            FileNotFoundError
            InterruptedError
            IsADirectoryError
            NotADirectoryError
            PermissionError
            ProcessLookupError
            TimeoutError
            UnsupportedOperation
            itimer_error
        ReferenceError
        RuntimeError
            NotImplementedError
            RecursionError
            _DeadlockError
        StopAsyncIteration
        StopIteration
        SyntaxError
            IndentationError
                TabError
        SystemError
            CodecRegistryError
        TypeError
        ValueError
            UnicodeError
                UnicodeDecodeError
                UnicodeEncodeError
                UnicodeTranslateError
            UnsupportedOperation
        Warning
            BytesWarning
            DeprecationWarning
            EncodingWarning
            FutureWarning
            ImportWarning
            PendingDeprecationWarning
            ResourceWarning
            RuntimeWarning
            SyntaxWarning
            UnicodeWarning
            UserWarning
        ExceptionGroup
        _OptionError
        error
        TokenError
        StopTokenizing
        ClassFoundException
        EndOfBlock
    GeneratorExit
    KeyboardInterrupt
    SystemExit
```

## Generic Iterable Types

A *generic iterable type* is an iterable type that yields items of the same
type. This chapter shows how to annotate generic iterable types.

### *Annotating a generic iterable*

To annotate a generic iterable, use the `collections.abc.Iterable` abc.
The following example shows how to annotate a generic iterable.

```python
from collections.abc import Iterable

def replace_text(text: str, replace_rules: Iterable[tuple[str, str]]) -> str:
    for from_, to in replace_rules:
        text = text.replace(from_, to)
    return text

replace_rules = [('a', 'A'), ('b', 'B'), ('c', 'C')]
replace_text('abc', replace_rules)  # This prints 'ABC'
```

### *Annotating a generic iterator*

A generic iterator types don`t appear as often as generic iterable types.
To annotate a generic iterator, use the `collections.abc.Iterator` abc.
The following example shows how to annotate a generic iterator.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/fibo_gen.py'
     language='python'
     line='1-7'
></div>

Annotating a return type of a function as `collections.abc.Iterator` is
used to annotate:
1. a generator function with `yield`, `yield from` statements
2. an iterator object generated by a class with handwritten `__next__` method.

### *Annotating a function that returns an iterator*

The `collections.abc.Generator` can be used to annotate a generator function,
but it is verbose for a generator used as an iterator.

The following example shows that the `collection.abc.Iterator` type is
a simplified version of the `collection.abc.Generator` type.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='17-it-generator/iter_gen_type.py'
     language='python'
     line='1-13'
></div>

* `<1>` : `short_kw` is a generator expression that yields a Python keyword
    with less than 5 characters.
* `<2>` : `mypy` prints the type of `short_kw` as
    `typing.Generator[builtins.str, None, None]`.
* `<3>` : `long_kw` is a generator expression that yields a Python keyword
    with more than 4 characters. `mypy` does not raise on this type annotation,
    `Iterator[str]`.
* `<4>` : `mypy` prints the type of `long_kw` as
    `typing.Iterator[builtins.str]`.

From the above example, it can be seen that `collections.abc.Iterator[str]` is
*consistent-with* `collections.abc.Generator[str, None, None]`.

* `collections.abc.Iterator[T]` : An iterator that yields items of type `T`.
* `collections.abc.Generator[T, None, None]` : A generator that yields items of
    type `T`, but that does not consume or return values.

## Classic Coroutines

PEP 342 [^pythonorg__pep342] introduced the `send()` and other features to
make generators act as classic coroutine. This section explains the concept of
classic coroutines.

### *Coroutines*

In Python, there are two coroutines concepts:
1. *Classic coroutines* : Extended generators that can be used to produce and
    consume values. `send()` is used to send values to a coroutine.
2. *Native coroutines* : Native coroutine will be explained in the later
    section.

### *Type Annotation for Classic Coroutines*

The classic coroutines are annotated as follows:

```python
collections.abc.Generator[YieldType, SendType, ReturnType]
```

The `SendType` is relevant only for classic coroutines. The `SendType` is the
type of the value `x` sent to the coroutine with `send(x)`.

### *Tips for Generators, Iterators, and Coroutines*

David Beazley explains the following tips for generators, iterators, and
coroutines [^pycon2009__david_beazley]:

1. Generators produce data for iteration.
2. Coroutines are consumers of data.
3. To keep your brain from exploding, don't mix the two concepts together.
4. **Coroutines are not related to iteration.**
5. Note: There is a use of having `yield` produce a value in a coroutine, but
    it's not tied to iteration.

### Example: Coroutine to Compute a Running Average

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager.py'
     language='python'
     line='32-42'
></div>

* `<1>` : The `averager` coroutine is annotated as
    `collections.abc.Generator[float, float, None]`.
    This means that the `averager` coroutine
    * yields `float` values,
    * receives `float` values, and
    * returns `None` value,
* `<2>` : Infinite loop to receive values from the caller.
* `<3>` : The `yield average` statement yields the average of the received
    values and then waits for the next value from the caller.
* `<3>` : `term = yield average` assigns the value sent by `.send()` to the
    `term` variable.

One attractive point in the above implementation is that `total`, `count`, and
`average` can be local variables in the `averager` coroutine. There is no need
to use classes, instance attributes or closures to store the state of the
coroutine.

The following example shows how to use the `averager` coroutine.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager.py'
     language='python'
     line='5-13'
></div>

* `<1>` : Creates a `coro_avg` coroutine object by invoking the `averager()`
    function.
* `<2>` : Starts the `coro_avg` coroutine by calling `next()`.
* `<3>` : Each call to `coro_avg.send()` sends a value to the `coro_avg`
    coroutine and returns the average of the received values.

In the above example, `next(coro_avg)` is used to advance the coroutine to the
first `yield` statement. The `next(coro_avg)` is equivalent to
`coro_avg.send(None)`. It is not possible to send a non-`None` value to a
coroutine before it is started. This will raise a `TypeError` as follows:

```python
>>> coro_avg = averager()
>>> coro_avg.send(10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't send non-None value to a just-started generator
```

After the coroutine is started, the coroutine is suspended at the `yield`
statement, and waits for a value to be sent to it. The `coro_avg.send(10)`
resumes the coroutine and receives the value `10` from `yield` statement and
assigns it to the `term` variable.

To terminate the coroutine, `.close()` method is used as follows:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager.py'
     language='python'
     line='18-25'
></div>

* `<2>` : The `coro_avg.close()` method terminates the coroutine.
* `<3>` : Calling `.close()` on a previously terminated coroutine has no
    effect.
* `<4>` : Calling `.send()` on a previously terminated coroutine raises a
    `StopIteration` exception.

### Returning a Value from a Coroutine

The previous version of the `averager()` classic coroutine yields the average
of the received values every time the `.send()` method is invoked.
In this chapter, the `averager()` classic coroutine is modified to return the
average of the received values when the coroutine is terminated, instead of
yielding the partial average of the received values.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager2.py'
     language='python'
     line='65-78'
></div>

* `<1>` : The class `Result` contains the average of the received values and
    the number of received values.
* `<2>` : To ignore raising error when replacing the `.count()` method of
    the `NamedTuple` into `count` attribute, the `# type: ignore` comment is
    used.
* `<3>` : The class `Sentinel` is used to make a sentinel object which is used
    to terminate the coroutine.
* `<4>` : The `STOP` is a singleton sentinel object.
* `<5>` : The `SendType` is a type alias for the type of the value sent to the
    coroutine. The `SendType` is `Union[float, Sentinel]`.

`SendType = Union[float, Sentinel]` can be rewritten as follows using
`typing.TypeAlias` [^pythonorg__typing_typealias]:

```python
import typing.TypeAlias
SendType: TypeAlias = float, Sentinel
```

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager2.py'
     language='python'
     line='81-94'
></div>

* `<1>` : The type of return value of the `averager2()` coroutine is
    `collections.abc.Generator[None, SendType, Result]`.
* `<2>` : `term = yield` statement does not yield a value. The `term` variable
    is assigned with the value sent by `.send()` method.
* `<3>` : The `if isinstance(term, Sentinel)` statement checks if the
    `term` variable is a sentinel object. If it is a sentinel object, the
    infinite loop is stopped.
* `<4>` : The above `isinstance` check prevents raising an error by
    the type checker on the `total += term` statement.
* `<5>` : The `Result` object is returned when the coroutine is terminated.

The following example demonstrates how to use the `averager2()` coroutine.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager2.py'
     language='python'
     line='8-13'
></div>

* `<1>` : The `coro_avg.send()` does not return the average of the received
    values. Instead, the `coro_avg.send()` returns `None`. When the `None` is
    returned, Python's console does not print the returned value.
* `<2>` : The `coro_avg.close()` terminates the coroutine but does not return
    the average value. When the `coro_avg.close()` is called,
    the `GeneratorExit` exception is raised at the line of the `term = yield`
    statement, so the `return` statement will not be executed.

To get the return value of the coroutine, sending the `STOP` sentinel object
is needed. The following example shows how to get the return value of the
`averager2()` coroutine.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager2.py'
     language='python'
     line='22-33'
></div>

* `<1>` : The `coro_avg.send()` returns `None` when the `STOP` sentinel object
    is sent to the coroutine. This statement is written in `try:` block to
    catch the `StopIteration` exception.
* `<2>` : When the `StopIteration` exception is raised, the `Result` object
    is returned as the `StopIteration` exception's value. To get the value of
    the `StopIteration` exception, the `exc.value` attribute is used.
* `<3>` : The `result` variable is assigned with the `Result` object returned
    by the `coro_avg.send(STOP)` method.


A delegating generator can get the return value of a subgenerator by using
assigning the return value of `yield from` statement. The following example
shows how to get the return value of the `averager2()` coroutine by using
a delegating generator.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='6527037ae7319ba370a1ee2d9fe79214d0ed9452'
     path='17-it-generator/coroaverager2.py'
     language='python'
     line='42-59'
></div>

* `<1>` : The `res = yield from averager2(True)` statement catches the return
    value from the `averager2()` coroutine.
* `<3>` : The returned value from the `return res` statement can be catched by
    the `StopIteration` exception's value.
* `<4>` : Creating a delegating coroutine object.
* `<5>` : To initialize the coroutine, iterator containing the `None` value on
    the first item is used for `for` loop.
* `<7>`, `<8>` : This statement catches the return value of the `comp` coroutine
    object.

### Generic Type Hints for Classic Coroutines

<div class='embed-github-src'
     repo='python/cpython/'
     branch='6f743e7a4da904f61dfa84cc7d7385e4dcc79ac5'
     path='Lib/typing.py'
     language='python'
     line='543-546'
></div>

<div class='embed-github-src'
     repo='python/cpython/'
     branch='6f743e7a4da904f61dfa84cc7d7385e4dcc79ac5'
     path='Lib/typing.py'
     language='python'
     line='2060-2061'
></div>

By the above code of Python 3.6, the `typing.Generator` is
1. covariant on the `YieldType` and `ReturnType` type parameters, and
2. contravariant on the `SendType` type parameter.

Therefore, the following relation holds:

```
float :> int
Generator[float, Any, float] :> Generator[int, Any, int]
Generator[Any, float, Any] :> Generator[Any, int, Any]
```

## Chapter Summary

* An *iterator* is an object that implements the `__next__()` method and the
    `__iter__()` method. The `__next__()` method returns the next item in the
    sequence. The `__iter__()` method returns the iterator object itself.
* An *iterable* is an object that generates an iterator when the `iter()`
    function is called on it. Two type of iterables are:
    * Object that implements the `__iter__()` method.
    * Object that implements the `__getitem__()` method.
* It is not recommended to make an iterable object implement the `__next__()`
    method. The `__next__()` method should be implemented in an iterator object.
* Using a generator function on `__iter__()` method is a convenient way to
    implement an iterable object.
* The `yield` statement is used to create a generator function.
* The `yield from` statement is used to delegate to a subgenerator.
* The `itertools` module contains functions that return iterators for
    efficient looping. If possible, use the `itertools` module instead of
    handwritten iterators.
* To annotate a generic iterable, use the `collections.abc.Iterable` abc.
    * `Iterable[T]` : An iterable that yields items of type `T`.
* To annotate a generic iterator, use the `collections.abc.Iterator` abc.
    * `Iterator[T]` : An iterator that yields items of type `T`.
    * A type annotation of the return value of generator functions with `yield`
        statement is `Iterator[T]`.
* `collections.abc.Iterator[T]` is *consistent-with*
    `collections.abc.Generator[T, None, None]`.
* A *classic coroutine* is an extended generator that can be used to produce
    and consume values.
    * The `next()` built-in function or `.send(None)` method is used to advance
        a coroutine to the first `yield` statement.
    * The `send()` method is used to send values to a coroutine.
    * The `close()` method is used to terminate a coroutine.
    * The `collections.abc.Generator[YieldType, SendType, ReturnType]` is used
        to annotate a classic coroutine.
    * `try: ... except StopIteration as exc: ...` is used to get the return
        value of a coroutine.
    * A delegating generator can get the return value of a subgenerator by
        using assigning the return value of `yield from` statement.

## Links

[^pythonorg__builtin_iter]:[Built-in Functions : **`iter`**`(object)`](https://docs.python.org/3/library/functions.html#iter)
[^gof__iterator]:[Design Patterns : **Iterator**](https://www.oodesign.com/iterator-pattern.html)
[^pythonorg__itertools]:[`itertools` — Functions creating iterators for efficient looping](https://docs.python.org/3/library/itertools.html)
[^pythonorg__pep342]:[PEP 342 – Coroutines via Enhanced Generators](https://peps.python.org/pep-0342/)
[^pycon2009__david_beazley]:[A Curious Course on Coroutines and Concurrency by *David Beazley* at PyCon'2009](http://www.dabeaz.com/coroutines/Coroutines.pdf)
[^pythonorg__typing_typealias]:[typing — Support for type hints : **`typing.TypeAlias`**](https://docs.python.org/3/library/typing.html#typing.TypeAlias)
