---
layout: distill
title: Chapter 15
description: More About Type Hints
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstract
  - name: Overloaded Signatures
  - name: Max Overload
  - name: TypedDict
  - name: Type Casting
  - name: Reading Type Hints at Runtime
  - name: Implementing a Generic Class
  - name: Variance
  - name: Implementing a Generic Static Protocol
  - name: Chapter Summary
  - name: Further Reading
  - name: Links
---

## Abstract

This is a summary of chapter 15 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/)[^bible].
This chapter provides tips for type annotation in Python.

## Overloaded Signatures

It is possible to allow for accepting different combination of arguments by
using `@typing.overload`[^pythonorg__typing_overload] decorator.

### *Example of `sum()` built-in function*

The following result from the Python interpreter is the execution result of
`help(sum)`[^pythonorg__builtinfunctions_sum].

```python
>>> help(sum)
sum(iterable, /, start=0)
    Return the sum of a 'start' value (default: 0) plus an iterable of numbers

    When the iterable is empty, return the start value.
    This function is intended specifically for use with numeric values and may
    reject non-numeric types.
```

This indicates the following aspect of the argument `iterable`:
1. The argument `iterable` is positional-only parameter because `iterable` is
    positioned before the `/` syntax.\
    (Details of `/` syntax can be found in PEP 570[^pythonorg__pep570].)
2.  The argument `iterable` has no default value, therefore it is mandatory
    to specify the `iterable` when calling `sum` function.
3. The argument `iterable` has to be of `typing.Iterable` type.

This indicates the following aspect of the argument `start`:
1. The argument `start` is positional or keyword parameter because `start` is
    positioned after the `/` syntax and there is no `*` syntax.\
    (Details of `/` and `*` syntax can be found in PEP 570[^pythonorg__pep570].)
2. The argument `start` has default value `0`, therefore it is optional
    to specify the `start` argument.
3. When the argument `start` is not specified, `start` is automatically set to
    `0`, which is of type `int`.

Therefore, the following cases are possible combination of arguments types and
return values types.
1. In the case where `start` is not specified:
    * `iterable: typing.Iterable[_T]` is an arbitrary type `_T = TypeVar('_T')`
        of `typing.Iterable`.
    * `start` is not specified.
    * The return type is `_T` or `int` which is type of default value of
        `start`.
2. In the case where `start` is specified:
    * `iterable: typing.Iterable[_T]` is an arbitrary type
        `_T = typing.TypeVar('_T')` of `typing.Iterable`,\
    * `start` is an arbitrary type `_S = typing.TypeVar(_S)`,\
    * The return type is `_T` or `_S`.

This two cases can be found in the following python typeshed source codes.

<div class='embed-github-src'
     repo='python/typeshed'
     branch='a8834fcd46339e17fc8add82b5803a1ce53d3d60'
     path='stdlib/2and3/builtins.pyi'
     language="python"
     line='25-29'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='a8834fcd46339e17fc8add82b5803a1ce53d3d60'
     path='stdlib/2and3/builtins.pyi'
     language="python"
     line='1434-1437'
></div>

The same implementation can be found in the Fluent Python book.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/mysum.py'
     language="python"
></div>

* `<1>` : `T = TypeVar('T')` is a type for the argument `it`, and
    `S = TypeVar('S')` is a type for the argument `start`.
* `<2>` : Type annotations are provided in the case where `start` is not
    specified.
    * The return type can be `T` or the type of the default value of `start`,
        which is `int`.
* `<3>` : Type annotations are provided in the case where `start` is specified.
    * The return type can be `T` or `S`.
* `<4>` : This is the implementation of the `sum()` function.

It is possible to find many examples that use `@typing.overload` decorator in
the stub file for built-in types in the Python standard library
[^github__python_typeshed_stdlib_2and3_builtins_pyi].

## Max Overload

In this sub chapter, the following `max()` function will be type-annotated by
six `@typing.overload` decorator syntax.

### *Implementation of `max()` function*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='12-13'
></div>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='37-60'
></div>

The logic of `max()` function can be described as follows:
1. If `args` are not empty which means there are 2 or more positional parameter
    except for the `key` and `default` arguments, assign the following data to
    each variables:
    * `candidate` : Assign the first positional parameter (argument `first`).
    * `series` : The iterator starting from the second, third fourth, and so on
        positional parameters (argument `args`).
2. If `args` is empty, which means there is only one positional parameter
    except for the `key` and `default` arguments, assign the following data to
    each variables:
    * `series` : Generate an iterator using `iter(first)` and assign it to
        `series`. This process assumes that `first` must be iterable.
        If `first` is not iterable, an exception will be raised.
    * `candidate` : If the iterator generation `iter(first)` is successful,
        the first item from the iterator `series` will be assigned to
        `candidate` and the iterator will consume the first item of it by
        `next(series)`.
        * If it is impossible to get item from the iterator by using
            `next(series)` because there are no further items produced by
            the iterator `series`, try to return the default value from
            the argument `default`.
            If the argument `default` is not specified, raise
            `ValueError(EMPTY_MSG)`.
3. If the argument `key` is not specified (in the case of `key == None`),
    find the maximum value using inequality operator `<` in the items from
    the variable `candidate` and the iterator `series`.
4. If the argument `key` is specified with function, items from the variable
    `candidate` and the iterator `series` is compared by `key(item)` and
    the inequality operator `<`.
5. Return the maximum value stored in `candidate`.

### *Type hint by using `@typing.overload` decorator*

According to the logic of `max()` function, the eight combinations of arguments
can be provided.
Before explaining the implementation of type hint using `@typing.overload`
decorator, import and define some types and variables.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='2-13'
></div>

| Symbol             | Description                                                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| `SupportsLessThan` | Static protocol which is used to type hint an object that can accepts the `<` operator               |
| `LT`               | Type variable which is used to type hint an object that implements `SupportsLessThan` protocol       |
| `T`                | Type variable which is used to type hint an object that is used as an argument of the `key` function |
| `DT`               | Type variable which is used to type hint an object that is used to the argument `default`            |

Case 1 : In the case where multiple arguments are provided and `key`, `default`
is not provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                   |
| -------------------- | ---------------------------------------- |
| positional arguments | multiple arguments implementing `__lt__` |
| `key`                | not provided                             |
| `default`            | not provided                             |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='15-17'
></div>

To type hint that the multiple positional argument must support
the `<` operator, the type variable `LT` is used.

Because the variable `__arg1` and `__arg2` are used, two or more positional
arguments must be provided. Note that double underscores `__` are used to avoid
potential naming conflicts.

`key: None = ...` indicates that type of parameter `key` is `None` and
the ellipsis `...` indicates that the implementation of the default value is
omitted in this type hint. However, an actual default value is provided by
the implementation of the `max()` function.

Case 2 : In the case where single iterable arguments is provided and `key`,
`default` is not provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                          |
| -------------------- | --------------------------------------------------------------- |
| positional arguments | single iterable argument yielding objects implementing `__lt__` |
| `key`                | not provided                                                    |
| `default`            | not provided                                                    |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='21-23'
></div>

There is only one parameter before `*`. This indicates that single argument
must be provided. This argument is annotated with `__iterable: Iterable[LT]`,
which means that  `__iterable` is iterable which yields objects supporting `<`
operator.

`*` on the parameter list indicates that `key` and `default` are keyword only
parameters[^pythonorg__pep570_syntax_and_semantics].


Case 3 : In the case where multiple arguments are provided, `key` is provided
but `default` is not provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                                              |
| -------------------- | ----------------------------------------------------------------------------------- |
| `key`                | provided, function that returns objects implementing `__lt__`                       |
| `default`            | not provided                                                                        |
| positional arguments | multiple arguments with the same type that can be an argument of the `key` function |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='18-20'
></div>

The part of positional arguments is same as in case 1. Note that
the return type is `T`, which is the same as the type of `__arg1`, `__arg2` and
`args`.


Case 4 : In the case where single iterable arguments is provided, `key` is
provided but `default` is not provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                                         |
| -------------------- | ------------------------------------------------------------------------------ |
| `key`                | provided, function that returns objects implementing `__lt__`                  |
| `default`            | not provided                                                                   |
| positional arguments | single iterable yielding objects that can be an argument of the `key` function |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='24-26'
></div>


Case 5 : In the case where multiple arguments are provided, `key` is not
provided and `default` is provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                   |
| -------------------- | ---------------------------------------- |
| `key`                | not provided                             |
| `default`            | provided                                 |
| positional arguments | multiple arguments implementing `__lt__` |

```python
@overload
def max(__arg1: LT, __arg2: LT, *args: LT, key: None = ...,
        default: DT) -> Union[LT, DT]:
    ...
```

The part of positional arguments is the same as in case 1 and case 3. Note that
the return type is `Union[LT, DT]`, which implies that the return value can be
either the specified argument or the default value specified by the `default`
parameter.


Case 6 : In the case where single iterable arguments is provided, `key` is not
provided and `default` is provided
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                 |
| -------------------- | ------------------------------------------------------ |
| `key`                | not provided                                           |
| `default`            | provided                                               |
| positional arguments | single argument yielding objects implementing `__lt__` |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='28-30'
></div>


Case 7 : In the case where multiple arguments are provided and `key` and
`default` is provided.
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                                              |
| -------------------- | ----------------------------------------------------------------------------------- |
| `key`                | provided, function that returns objects implementing `__lt__`                       |
| `default`            | provided                                                                            |
| positional arguments | multiple arguments with the same type that can be an argument of the `key` function |

```python
@overload
def max(__arg1: T, __arg2: T, *args: T, key: Callable[[T], LT],
        default: DT) -> Union[LT, DT]:
    ...
```


Case 8 : In the case where single iterable arguments is provided and `key` and
`default` is provided.
: In this case, the following combination of arguments is provided.

| Argument             | Status                                                                         |
| -------------------- | ------------------------------------------------------------------------------ |
| `key`                | provided, function that returns objects implementing `__lt__`                  |
| `default`            | provided                                                                       |
| positional arguments | single iterable yielding objects that can be an argument of the `key` function |

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/protocol/mymax/mymax.py'
     language='python'
     line='32-34'
></div>

## TypedDict

`typing.TypedDict`[^pythonorg__typing_typeddict] can be used to provide type
hints for a `dict` object with predefined types for each values of keys.
For example, suppose adding a type annotation to the following JSON object.

```json
{
    "isbn": "0134757599",
    "title": "Refactoring, 2e",
    "authors": ["Martin Fowler", "Kent Beck"],
    "pagecount": 478
}
```

It is possible to attempt to provide type hint for this JSON object by using
the following conventional type annotations.

1. `dict[str, Any]` : The values may be of any type.
2. `dict[str, typing.Union[str, list[str], int]]` : Although difficult to read,
    it is possible to narrow the range of possible types of values compared to
    the type hint above.

Adopting the second type annotation cannot prevent the key of `"isbn"` from
becoming an object of `list[str]`. This problem can be solved by using
`typing.TypedDict`.

`typing.TypedDict` was introduced by PEP 589[^pythonorg__pep589].
`typing.TypedDict` provides type hints for dictionaries with a fixed set
of keys. The following code is an example of type hints for the above JSON
object.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/books.py'
     language='python'
     line='3-9'
></div>

`typing.TypedDict` provides the following functionality.

1. Provide type hints for each "field" of `dict` to the type checker.
    * Assigning an incompatible type for a field raises error by
        the type checker.
    * Assigning to a key that is not defined in the class definition raises
        an error by the type checker.
    * Deleting a key that is defined in the class definition raises an error by
        the type checker.
2. Class-like syntax to provide type hints for each field.
    * It is possible to generate a constructor by subclassing
        `typing.TypedDict` class, as shown in the example above with `BookDict`.
3. There is no runtime type checking facility. Therefore, the above type
    checking facilities are not provided during runtime.
4. A generated constructor such as `BookDict` has the same effect as calling
    the `dict` constructor with the same arguments.
    * The defined fields, like `isbn: str`, does not create instance attributes.
    * Setting default values for fields and method definitions is not allowed.
    * It is possible to set undefined fields on subclass definition.

### *Several Demos on the book*

#### *Example 15-5. Using a `BookDict`, but not quite as intended*

```python
>>> from books import BookDict
>>> pp = BookDict(title='Programming Pearls',  # <1>
...               authors='Jon Bentley',  # <2>
...               isbn='0201657880',
...               pagecount=256)
>>> pp  # <3>
{'title': 'Programming Pearls', 'authors': 'Jon Bentley', 'isbn': '0201657880',
'pagecount': 256}
>>> type(pp)  # <4>
<class 'dict'>
>>> pp.title  # <5>
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
AttributeError: 'dict' object has no attribute 'title'
>>> pp['title']
'Programming Pearls'
>>> pp.__annotations__  # <6>
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'dict' object has no attribute '__annotations__'. Did you mean: '__contains__'?
>>> BookDict.__annotations__  # <7>
{'isbn': <class 'str'>, 'title': <class 'str'>, 'authors': typing.List[str],
 'pagecount': <class 'int'>}
```

* `<1>` : The `BookDict`, which is a subclass of `TypedDict`, can be
    instantiated by using keyword arguments or by passing `dict` argument
    including a `dict` literal.
* `<2>` : Although the `authors` field was intentionally set to an object of
    `str` instead of `list[str]`, it can be instantiated without raising
    an exception.
* `<3>` : The result of `repr` of `pp` is the same as `dict`
* `<4>` : The type of `pp` is `dict` instead of `BookDict` or `TypedDict`
* `<5>` : It is not possible to access `title` property on the definition of
    `BookDict`, because `pp` is an object of `dict` instead of `BookDict`.
* `<6>` : The `pp` object also does not have the `__annotations__` property,
    which holds the information of type hints.
* `<7>` : The `BookDict` class has the `__annotations__` property.

#### *Example 15-6. `demo_books.py` : legal and illegal operations on a `BookDict`*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/demo_books.py'
     language='python'
     line='1-20'
></div>

* `<1>` : Annotated a return type to check the `demo()` function by `mypy`.
* `<2>` : Generate a valid object of `BookDict` with vaild type and complete
    number of fields.
* `<3>` : The `authors` should have the same type with `book['authors']`, which
    is `list[str]`.
* `<4>` : `if TYPE_CHECKING` becomes `True` only when the program is checked
    by the type checker. At runtime, `if TYPE_CHECKING` becomes `False`.
* `<5>` : The `reveal_type()` function prints the type of the argument to
    stdout during type checking.
* `<6>` : Although the `authors` is of type `list[str]`, this assignment can
    be done without raising an exception on runtime.

```bash
(.venv) ~/fluent-python % python demo_books.py
(.venv) ~/fluent-python % mypy --version
mypy 1.4.1 (compiled: yes)
(.venv) ~/fluent-python % mypy demo_books.py
demo_books.py:13: note: Revealed type is "builtins.list[builtins.str]"
demo_books.py:14: error: Incompatible types in assignment
    (expression has type "str", variable has type "list[str]")  [assignment]
demo_books.py:15: error: TypedDict "BookDict" has no key "weight"  [typeddict-unknown-key]
demo_books.py:16: error: Key "title" of TypedDict "BookDict" cannot be deleted  [misc]
Found 3 errors in 1 file (checked 1 source file)
```

* `python demo_books.py` can be executed without raising an exception.
* Note at `demo_books.py:13` : This indicates that the variable `authors` is of
    type `list[str]`
* Error at `demo_books.py:14` : This indicates that assigning an object of type
    `str` to `authors` is illegal.
* Error at `demo_books.py:15` : This indicates that assigning to a key that is
    not part of the `BookDict` definition is not allowed.
* Error at `demo_books.py:16` : This indicates that deleting a key that is part
    of the `BookDict` definition is not allowed.

#### *Example 15-7,8. `books.py` : `to_xml` function*

This is an example of the `BookDict` application. The `to_xml` function
converts a `BookDict` object into an XML string, as shown in
the following XML code.

```xml
<BOOK>
    <ISBN>0134757599</ISBN>
    <TITLE>Refactoring, 2e</TITLE>
    <AUTHOR>Martin Fowler</AUTHOR>
    <AUTHOR>Kent Beck</AUTHOR>
    <PAGECOUNT>478</PAGECOUNT>
</BOOK>
```

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/books.py'
     language='python'
     line='13-25'
></div>

* `<1>` : Annotate the type of the parameter `book` as `BookDict`.
* `<2>` : The contents of the XML code are stored temporarily in `elements`,
    which is of type `list[str]`. By doing this, the elements of `elements`
    must be `str` objects and this is checked by `mypy`.
* `<3>`, `<4>` : This codes generates `<AUTHOR>...</AUTHOR>` parts of XML code.
    Because `mypy` understands `isinstance()` checks, the type of `value` must
    be `list` in this block guarded by the `if` clause.


#### *Example 15-9. `books_any.py` : `from_json` function*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/books_any.py'
     language='python'
     line='29-31'
></div>

* `<1>` : The return type of `json.loads(data)` is `Any`.
* `<2>` : `Any` is *consistent-with* every type, therefore `whatever` can be returned.

The `<2>` of the above example reveals that using the `Any` type prevents
`mypy` from flagging any problem of the code. By adding `--disallow-any-expr`,
`mypy` will flag the two lines of the sample code.

```bash
(.venv) ~/fluent-python % mypy --version
mypy 1.4.1 (compiled: yes)
(.venv) ~/fluent-python % mypy books_any.py --disallow-any-expr
books_any.py:30: error: Expression has type "Any"
books_any.py:31: error: Expression has type "Any"
Found 2 errors in 1 file (checked 1 source file)
```

#### *Example 15-10. `books.py` : `from_json` function with variable annotation*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/books.py'
     language='python'
     line='29-31'
></div>

* `<1>` : By adding a type annotation as shown at `whatever: BookDict`,
    `--disallow-any-expr` will not flag any error on this line.
* `<2>` : Therefore, this line will return the declared return type,
    `BookDict`.

#### *Example 15-11, 12, 13. `demo_not_book.py` : `from_json` returns an invalid `BookDict`, and `to_xml` accepts it*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='dd535abcf76739d451c2baaf7813b8390a7dfe88'
     path='15-more-types/typeddict/demo_not_book.py'
     language='python'
     line='1-23'
></div>

* `<1>` : `not_book` is not a valid object of `BookDict`.
* `<2>` : In this block guarded by the `if TYPE_CHECKING` clause, the type of
    `not_book` and `not_book['authors']` will be revealed.
* `<3>` : This line is valid because `print()` function can handle every object.
* `<4>` : This line is invalid because an object of `BookDict` does not have
    a `flavor` field. But, this line does not raise an exception because
    runtime checking of `BookDict` is unavailable.
* `<5>` : Although the `to_xml()` function accepts only an object of `BookDict`,
    this line also does not raise an exception because runtime checking of
    `BookDict` is unavailable.
* `<6>` : The XML output of the `to_xml()` function will be printed to stdout
    without raising an exception.

```bash
(.venv) ~/fluent-python % mypy demo_not_book.py
../demo_books.py:12: note: Revealed type is "TypedDict('demo_books.BookDict',
    {'isbn': builtins.str, 'title': builtins.str,
     'authors': builtins.list[builtins.str], 'pagecount': builtins.int})"
../demo_books.py:13: note: Revealed type is "builtins.list[builtins.str]"
../demo_books.py:16: error: TypedDict "BookDict" has no key "flavor"  [typeddict-item]
```

Type checker raise an error when referencing `'flavor'` field of the `BookDict`
object.

```bash
(.venv) ~/fluent-python % python demo_not_book.py
{'title': 'Andromeda Strain', 'flavor': 'pistachio', 'authors': True}
pistachio
<BOOK>
        <TITLE>Andromeda Strain</TITLE>
        <FLAVOR>pistachio</FLAVOR>
        <AUTHORS>True</AUTHORS>
</BOOK>
```

The above stdout is the results of `python demo_not_book.py`. Although
the variable `not_book` and `not_book['flavor']` are invalid, all of
the `print` statements are executed normally.

Considering the above results, using `typing.TypedDict` while handling dynamic
data structures like JSON and XML lacks the ability to perform type-checking
during runtime. For performing type-checking while handling dynamic data
structures, use `pydantic`[^pypiorg__pydantic] library for type-checking.

The features of `typing.TypedDict`, including optional keys, inheritance
functionality, are decribed in the official website documentation[^pythonorg__typing_typeddict].

## Type Casting

`typing.cast()` is a function that is used to supress incorrect errors or
warnings generated by `mypy`. The official documentation for `typing.cast()`
can be found in mypy documentation[^mypy__documentation_casts].

<div class='embed-github-src'
     repo='python/cpython/'
     branch='bee66d3cb98e740f9d8057eb7f503122052ca5d8'
     path='Lib/typing.py'
     language='python'
     line='1340-1348'
></div>

`typing.cast(type, value)` returns the second argument `value` as the type
specified by the first argument `type`.
As shown in the following code, `typing.cast()` does nothing during runtime.

### *Example of `typing.cast()`*

```python
def find_first_str(a: list[object]) -> str:
    index = next(i for i, x in enumerate(a) if isinstance(x, str))
    # We only get here if there's at least one string
    return a[index]
```

The `find_first_str()` function above returns the first element of
the list `a` that is of type `str`.
This is determined by the `if isinstance(x, str)` statement inside
the generator expression, where `index` always corresponds to the index of
the first `str` object in the list `a`.
However, the type checking performed by `mypy` will raise an error because
`a[index]` is of type `object`.

```bash
find_first_str.py:4: error: Incompatible return value type (got "object", expected "str")  [return-value]
Found 1 error in 1 file (checked 1 source file)
```

To correct an erronous result of `mypy`, `typing.cast()` can be used to cast
the type of `a[index]` into type `str`.

```python
from typing import cast

def find_first_str(a: list[object]) -> str:
    index = next(i for i, x in enumerate(a) if isinstance(x, str))
    # We only get here if there's at least one string
    return cast(str, a[index])
```

### *Policy of using `typing.cast()` call and `# type: ignore` comment*

* `mypy` is usually correct when it reports an error. Don't use `typing.cast()`
    too frequently.
* `typing.cast()` is more informative compared to the `# type: ignore` comment.
* Using the `Any` type is contagious because `Any` is *consistent-with* all
    types.
* Use `# type: ignore` and `Any` only when `typing.cast()` cannot solve the problem.

## Reading Type Hints at Runtime

At import time, Python reads the type hint of imported functions, classes and
modules.
The type hints are stored in `__annotations__` attributes.
For example, the `clip()` function has type annotation of parameters and
return value.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='f248baf418c49e5748bd503dd992b52440ed236b'
     path='15-more-types/clip_annot.py'
     language='python'
     line='21-21'
></div>

The type hint of the `clip()` function can be read by using the following code.

```python
>>> from clip_annot import clip
>>> clip.__annotations__
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
```

The `return` key maps to the return type of the `clip()` function.
This is because the `return` keyword is reserved for indicating the value that
is returned from a function, and using it as a parameter name raises
a `SyntaxError`.

The `__annotations__` properties are evaluated by the Python interpreter at
import time. This is why the above value of `clip.__annotations__` are
the Python classes `str` and `int`.

The import time evaluation of type annoatations is default at Python 3.11.
The evaluation process may change if PEP 563[^pythonorg__pep563] or
PEP 649[^pythonorg__pep649] is implemented.

### Problems with Annotations at Runtime

Type Annotations causes the following two problems.
1. CPU and memory resources for importing modules are increased by using more
    type hints in the module.
    * CPU resources are used when evaluating the `__annotations__` property.
    * Memory resources are used when storing the `__annotations__` property.
2. Referring to types not yet defined stores type annotation information as
    strings instead of actual types.
    * This problem is called the "Forward reference problem".

#### *Example of Forward reference problem*

```python
class Rectangle:
    # ... lines omitted ...
    def stretch(self, factor: float) -> 'Rectangle':
        return Rectangle(width=self.width * factor)
```

Referencing a class `Rectangle` in the type hint of the `stretch` method is
an example of the "Forward Reference" problem.
The "Forward Reference" problem occurs when referencing a certain type before
its definition.

In the above example, the `stretch` method must reference
the class `Rectangle` type before the definition of the class `Rectangle` is
done.
Because referencing a type before its definition is impossible,
the `stretch` method annotates its return type using the string `'Rectangle'`.

#### *Annotation using string object*

```python
>>> class Rectangle:
...     # ... lines omitted ...
...     def stretch(self, factor: float) -> 'Rectangle':
...         return Rectangle(width=self.width * factor)
...
>>> Rectangle.stretch.__annotations__
{'factor': <class 'float'>, 'return': 'Rectangle'}
```

`Rectangle.stretch.__annotations__` stores type hint of return value as
a string object. Static type checker can deal with these type hints stored
in string object.

To get type hint as the actual type, the following function can be used:

1. `typing.get_type_hints(obj, globalns=None, localns=None, include_extras=False)`
    [^pythonorg__typing_get_type_hints]
    * Available in Python 3.5 or later.
2. `inspect.get_annotations(obj, *, globals=None, locals=None, eval_str=False)`
    [^pythonorg__inspect_get_annotations]
    * Available in Python 3.10 or later.

```python
>>> typing.get_type_hints(Rectangle.stretch)
{'factor': <class 'float'>, 'return': <class '__main__.Rectangle'>}
>>> inspect.get_annotations(Rectangle.stretch)
{'factor': <class 'float'>, 'return': 'Rectangle'}
```

#### *PEP 563—Postponed Evaluation of Annotations*

With PEP 563[^pythonorg__pep563], type hints are not evaluated at import time.
To enable a functionality of the PEP 563, use the following `import` statement:

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/clip_annot_post.py'
     language='python'
     line='1-1'
></div>

With this `import` statement, `__annotations__` property holds type hints as string objects.

```python
>>> import clip_annot_post
>>> clip_annot_post.clip.__annotations__
{'text': 'str', 'max_len': 'int', 'return': 'str'}
```

To convert type hints stored as string objects into the actual type,
`typing.get_type_hints()` or `inspect.get_annotations()` can be used to obtain
the actual type.

```python
>>> import clip_annot_post
>>> clip_annot_post.clip.__annotations__
{'text': 'str', 'max_len': 'int', 'return': 'str'}
>>> typing.get_type_hints(clip_annot_post.clip)
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
>>> inspect.get_annotations(clip_annot_post.clip)
{'text': 'str', 'max_len': 'int', 'return': 'str'}
>>> inspect.get_annotations(clip_annot_post.clip, eval_str=True)
{'text': <class 'str'>, 'max_len': <class 'int'>, 'return': <class 'str'>}
```

`typing.get_type_hints()` has the following limitations[^pythonorg__pep563]:

1. Usage of `typing.get_type_hints()` is generally costly at runtime.
2. `typing.get_type_hints()` cannot properly handle non-global contexts such as
    inner classes or classes within functions.
3. `typing.get_type_hints()` does not properly handle classes with methods
    accepting or returning objects of their own type, if a class generator is
    used.

### Dealing with the Problem

If reading type annotations at runtime is needed, the following approaches can
be recommended:
1. Use `inspect.get_annotations()` (from Python 3.10) or
    `typing.get_type_hints()` (since Python 3.5) instead of reading
    the `__annotations__` property directly.
2. Create a simple wrapper function that calls `inspect.get_annotations()` or
    `typing.get_type_hints()` and invoke the wrapper function when reading
    type annotations.
    This approach localizes future changes related to reading type annoations
    into a single function.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='80f7f84274a47579e59c29a4657691525152c9d5'
     path='24-class-metaprog/checked/metaclass/checkedlib.py'
     language='python'
     line='118-123'
></div>

Python documentation pages[^pythonorg__annotations_best_practices]
provides advices when using type annotations at runtime.

## Implementing a Generic Class

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/lotto/generic_lotto_demo.py'
     language='python'
     line='6-11'
></div>

The above example explains the usage of a **Generic Class**.
* `<1>` : A generic class can be instantiated by using a pair of brackets with
    an actual type parameter, like `[int]`.
* `<2>` : `first` is recognized as an `int` object by `mypy`.
* `<3>` : `remain` is recognized as a `tuple[int, ...]` object by `mypy`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/lotto/generic_lotto_errors.py'
     language='python'
     line='1-17'
></div>

* `<1>` : An error is raised by `mypy` because `LottoBlower[int]` cannot be
    initialized by using `float` objects.
* `<2>` : An error is raised by `mypy` because the method `load` of `LottoBlower[int]`
    accepts only iterable of `int` objects.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='13-protocol-abc/tombola.py'
     language='python'
     line='3-31'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/lotto/generic_lotto.py'
     language='python'
     line='1-29'
></div>

The above two code blocks are implementations of a generic class `LottoBlower`.
In this paragraph, details of the class `LottoBlower` definition are discussed.
* `<1>` : A generic class can be defined by using multiple inheritance,
    including `typing.Generic`.
    * `typing.Generic` must decorated with a formal type parameter with
        brackets `[T]`.
    * A type variable `T`, which is defined by using `typing.TypeVar`, is
        used to express a formal type parameter.
* `<2>` : The `items` argument of the `__init__` method is annotated as
    `Iterable[T]`, which becomes `Iterable[int]` when an instance is
    initialized by using a parameterized type `LottoBlower[int]`.
* `<3>` : The `load` method is bounded in the same way as the `__init__` method.
* `<4>` : The return type of the `pick` method becomes `int` in
    a `LottoBlower[int]`.
* `<6>` : The return type of the `inspect` method becomes `tuple[int, ...]` in
    a `LottoBlower[int]`.

### Basic Jargon for Generic Types

Generic type
: A type declared with one or more type variables. The followings are examples
of Generic types.
* `LottoBlower[T]` (A generic type with one type variables)
* `abc.Mapping[KT, VT]` (A generic type with two type variables)

Formal type parameter
: Type variables that appear in a generic type declaration.
* `T` in `LottoBlower[T]`
* `KT` and `VT` in `abc.Mapping[KT, VT]`

Parameterized type
: A type derived from a generic type, declared with actual type parameters.
* `LottoBlower[int]`
* `abc.Mapping[str, float]`

Actual type parameter
: An actual type used as a parameter in the declaration of a parameterized type.
* `int` in `LottoBlower[int]`
* `str` and `float` in `abc.Mapping[str, float]`

## Variance

In this subchapter, the following variance of generic class will be discussed.

1. Invariant types
2. Covariant types
3. Contravariant types

To discuss about variance, we will introduce `:>` and `<:` symbols to denote relationships between two types.

`A :> B`
: This means that `A` is *supertype-of* or the same as `B`.

`B <: A`
: This means that `B` is *subtype-of* or the same as `A`.

### An Invariant Dispenser

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/invariant.py'
     language='python'
     line='2-24'
></div>

* `<1>` : A relationship `Beverage :> Juice :> OrangeJuice` is satisfied.
* `<2>` : A generic type using this simple `typing.TypeVar()` will be
    an invariant type.
* `<3>` : `BeverageDispenser` is parameterized on the type of beverage.
* `<4>` : The `install` method accepts only the type of
    `BeverageDispenser[Juice]`.

The `BeverageDispenser` will be an invariant type; therefore there is no
*supertype-of* or *subtype-of* relationship between the following types:
* `BeverageDispenser[Beverage]`
* `BeverageDispenser[Juice]`
* `BeverageDispenser[OrangeJuice]`

For example, the following code does not raise error.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/invariant.py'
     language='python'
     line='30-31'
></div>

But, `mypy` will raise errors on the following codes.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/invariant.py'
     language='python'
     line='38-42'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/invariant.py'
     language='python'
     line='49-53'
></div>

### A Covariant Dispenser

By defining a class with `TypeVar(covariant=True)`, the class will be
a covariant generic type.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/covariant.py'
     language='python'
     line='1-28'
></div>

* `<1>` : To define a covariant generic type, a type variable generated by
    `TypeVar(covariant=True)` is needed.
* `<2>` : Inheriting `Generic[T_co]` makes `BeverageDispenser`
    a covariant generic type.
* `<3>` : The `install()` function can accepts `BeverageDispenser[Juice]` or
    a subtype of `BeverageDispenser[Juice]`.

This `BeverageDispenser` will be a covariant type; therefore there are
the following relationships between the parameterized types.
* `BeverageDispenser[Beverage] :> BeverageDispenser[Juice]`
* `BeverageDispenser[Juice] :> BeverageDispenser[OrangeJuice]`

Therefore, the following code does not raise any errors.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/covariant.py'
     language='python'
     line='34-38'
></div>

But, the following code raises an error because the `beverage_dispenser`,
which is the type of the `BeverageDispenser[Beverage]`, is *supertype-of*
`BeverageDispenser[Juice]`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/covariant.py'
     language='python'
     line='44-48'
></div>

### A Contravariant Trash Can

By defining a class with `TypeVar(contravariant=True)`, the class will be
a contravariant generic type.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/contravariant.py'
     language='python'
     line='2-20'
></div>

* `<1>` :  A relationship `Refuse :> Biodegradable :> Compostable` is satisfied.
* `<2>` : To define a contravariant generic type, a type variable generated by
    `TypeVar(contravariant=True)` is needed.
* `<3>` : Inheriting `Generic[T_contra]` makes `TrashCan` a contravariant
    generic type.

The `deploy()` function can accepts `TrashCan[Biodegradable]` or any supertype
of `TrashCan[Biodegradable]`. Therefore, the following types can be accepted by
the `deploy()` function:
* `TrashCan[Biodegradable]`
* `TrashCan[Refuse]`

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/contravariant.py'
     language='python'
     line='28-32'
></div>

The type `TrashCan[Compostable]` cannot be accepted by the `deploy()` function.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/cafeteria/contravariant.py'
     language='python'
     line='39-43'
></div>

### Variance Review

Assume that `A` is a *supertype-of* `B`, which means `A :> B`.

#### Invariant types

If the type `L` is invariant, `L[A] :> L[B]` or `L[B] :> L[A]` does not hold
true.

For example, the following built-in type `list` is invariant.
The methods of `list` include methods that accept the formal type parameter `_T`
as an argument and methods that return it as a return value at the same time.
This aspect of the built-in type `list` makes it an invariant type.

| Method       | Where the formal type parameter `_T` appears |
| ------------ | -------------------------------------------- |
| `__init__()` | Argument                                     |
| `append()`   | Argument                                     |
| `extend()`   | Argument                                     |
| `pop()`      | Return value                                 |

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/builtins.pyi'
     language='python'
     line='70-72'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/builtins.pyi'
     language='python'
     line='743-752'
></div>

#### Covariant types

If the type `C` is covariant, `C[A] :> C[B]` holds true.

For example, immutable containers such as `tuple` and `frozenset` is covariant
types. The methods of `tuple` and `frozenset` return the formal type parameter
`_T_co` as a return value. This aspect of the built-in types `tuple` and
`frozenset` makes them invariant type.

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/builtins.pyi'
     language='python'
     line='70-72'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/builtins.pyi'
     language='python'
     line='711-733'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/builtins.pyi'
     language='python'
     line='872-895'
></div>

#### Contravariant types

If the type `K` is contravariant, `K[A] <: K[B]` holds true.

There are several contravariant types. For example,
1. `typing.Callable[[ParamType, ...], ReturnType]` is contravariant on
    the `ParamType`. [^chapter08__callable]
2. A write-only data structure, also known as "sink", is a contravariant
    container type.
3. `Generator`, `Coroutine`, `AsyncGenerator`, which have a contravariant type
    parameter, is a contravariant type.

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='89-97'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='199-202'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='227-239'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='268-272'
></div>

#### Variance rules of thumb

To sum up the above discussion, the following rules holds true on variance:

* If a formal type parameter defines a type for data that comes out of
    the object, it can be covariant.
* If a formal type parameter defines a type for data that goes into the object
    after its initial construction, it can be contravariant.
* If a formal type parameter defines a type for data that comes out of
    the object and the same parameter defines a type for data that goes into
    the object, it must be invariant.
*  As a precaution, it is advisable to make formal type parameters invariant.

## Implementing a Generic Static Protocol

A generic static protocol is a static protocol that can specify the type of
an argument or a return value of a static protocol using formal type parameters.

Variance discussed in the above contents can be specified on a generic static
protocol. The following code is the implementation of `SupportsAbs` in
the `typing` module.

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='89-97'
></div>

<div class='embed-github-src'
     repo='python/typeshed'
     branch='bfc83c365a0b26ab16586beac77ff16729d0e473'
     path='stdlib/typing.pyi'
     language='python'
     line='156-159'
></div>

The following code is an example of usage of `SupportsAbs`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/protocol/abs_demo.py'
     language='python'
     line='3-32'
></div>

* `<1>` : The class `Vector2d` supports the `abs` method that returns a `float`.
* `<2>` : The function `is_unit()` accepts an argument `v`, which is an object
    that support the `abs` method which returns a `float`(`SupportsAbs[float]`)
     or a subtype of `SupportsAbs[float]`.
* `<3>` : `mypy` accepts the `abs` method on `v`, and `abs(v)` as
    the first argument of `math.isclose()` because `v` is a type of
    `SupportsAbs[float]`.
* `<4>` : Thanks to `@runtime_checkable` on `SupportsAbs`, this assert
    statement can be used on runtime.
* `<5>` : The remaining code pass `mypy` checks and runtime assertion check.
* `<6>` : `int` type also have `__abs__` method and returns `int`, which is
    *consistent-with* the `float` type.
    Therefore, `int` is *consistent-with* `SupportAbs[float]` and can be
    assigned the `v` argument of `is_unit()` function.


On the following codes are `RandomPicker` protocol which is discussed in
the chapter 13.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/typing/randompick.py'
     language='python'
     line='1-5'
></div>

`RandomPicker` protocol can be a covariant generic protocol because
the return type of the `pick()` method can be specified by
a formal type parameter.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='15-more-types/protocol/random/generic_randompick.py'
     language='python'
     line='1-7'
></div>

## Chapter Summary

* `@typing.overload` [^pythonorg__typing_overload] can be used to type-annotate
    for complex methods. This can be used for object methods.
* The `typing.TypeVar()` function generates a type variable. Type variables are
    used to express common types in arguments and return value of a certain
    function.
    This can be used in type-annotation which is using the `@typing.overload`.
* `typing.TypedDict` [^pythonorg__typing_typeddict] can be used to provide
    type hints for `dict` object.
    Be careful that `typing.TypedDict` has no runtime type checking facility.
    * If runtime type checking facility is needed for a `dict` object, consider
        using the `pydantic` library to achieve it.
* `typing.cast()` is a function that is used to supress incorrect errors or
    warnings generated by `mypy`[^mypy__documentation_casts].
* To read type annotations at runtime, use
    the function `inspect.get_annotations()` (from Python 3.10) or
    the function `typing.get_type_hints()` (since Python 3.5)
    instead of reading the `__annotations__` property directly.
* The generic class can be defined by inheriting the `typing.Generic` class and
    using formal type parameters.
* The generic class can have variance.
* The generic static protocol also can have variance.

## Further Reading

| PEP  | Title                                                                                                    | Python | Year |
| ---- | -------------------------------------------------------------------------------------------------------- | ------ | ---- |
| 3107 | [Function Annotations](https://peps.python.org/pep-3107)                                                 | 3.0    | 2006 |
| 483* | [The Theory of Type Hints](https://peps.python.org/pep-483)                                              | n/a    | 2014 |
| 484* | [Type Hints](https://peps.python.org/pep-484)                                                            | 3.5    | 2014 |
| 482  | [Literature Overview for Type Hints](https://peps.python.org/pep-482)                                    | n/a    | 2015 |
| 526* | [Syntax for Variable Annotations](https://peps.python.org/pep-526)                                       | 3.6    | 2016 |
| 544* | [Protocols: Structural subtyping (static duck typing)](https://peps.python.org/pep-544)                  | 3.8    | 2017 |
| 557  | [Data Classes](https://peps.python.org/pep-557)                                                          | 3.7    | 2017 |
| 560  | [Core support for typing module and generic types](https://peps.python.org/pep-560)                      | 3.7    | 2017 |
| 561  | [Distributing and Packaging Type Information](https://peps.python.org/pep-561)                           | 3.7    | 2017 |
| 563  | [Postponed Evaluation of Annotations](https://peps.python.org/pep-563)                                   | 3.7    | 2017 |
| 586* | [Literal Types](https://peps.python.org/pep-586)                                                         | 3.8    | 2018 |
| 585  | [Type Hinting Generics In Standard Collections](https://peps.python.org/pep-585)                         | 3.9    | 2019 |
| 589* | [TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://peps.python.org/pep-589)       | 3.8    | 2019 |
| 591* | [Adding a final qualifier to typing](https://peps.python.org/pep-591)                                    | 3.8    | 2019 |
| 593  | [Flexible function and variable annotations](https://peps.python.org/pep-593)                            | ?      | 2019 |
| 604  | [Allow writing union types as `X | Y`](https://peps.python.org/pep-604)                                  | 3.10   | 2019 |
| 612  | [Parameter Specification Variables](https://peps.python.org/pep-612)                                     | 3.10   | 2019 |
| 613  | [Explicit Type Aliases](https://peps.python.org/pep-613)                                                 | 3.10   | 2020 |
| 645  | [Allow writing optional types as `x?`](https://peps.python.org/pep-645)                                  | ?      | 2020 |
| 646  | [Variadic Generics](https://peps.python.org/pep-646)                                                     | ?      | 2020 |
| 647  | [User-Defined Type Guards](https://peps.python.org/pep-647)                                              | 3.10   | 2021 |
| 649  | [Deferred Evaluation Of Annotations Using Descriptors](https://peps.python.org/pep-649)                  | ?      | 2021 |
| 655  | [Marking individual TypedDict items as required or potentially-missing](https://peps.python.org/pep-655) | ?      | 2021 |

PEP with numbers marked with * are important enough to be mentioned in the
opening paragraph of the typing documentation.

## Links

[^bible]:[*Fluent Python* by Luciano Ramalho](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/)
[^pythonorg__typing_overload]:[`typing` — Support for type hints : `@typing.overload`](https://docs.python.org/3/library/typing.html#typing.overload)
[^pythonorg__typing_typeddict]:[`typing` — Support for type hints : `class typing.TypedDict`](https://docs.python.org/3/library/typing.html#typing.TypedDict)
[^pythonorg__typing_get_type_hints]:[`typing` — Support for type hints : `typing.get_type_hints`](https://docs.python.org/3/library/typing.html#typing.get_type_hints)
[^pythonorg__builtinfunctions_sum]:[Built-in Functions : `sum()`](https://docs.python.org/3/library/functions.html#sum)
[^pythonorg__inspect_get_annotations]:[`inspect` — Inspect live objects : `inspect.get_annotations`](https://docs.python.org/3/library/inspect.html#inspect.get_annotations)
[^pythonorg__pep570]:[PEP 570 – Python Positional-Only Parameters](https://peps.python.org/pep-0570/)
[^pythonorg__pep570_syntax_and_semantics]:[PEP 570 – Python Positional-Only Parameters : Syntax and Semantics](https://peps.python.org/pep-0570/#syntax-and-semantics)
[^pythonorg__pep589]:[PEP 589 – TypedDict: Type Hints for Dictionaries with a Fixed Set of Keys](https://peps.python.org/pep-0589/)
[^pythonorg__pep563]:[PEP 563 – Postponed Evaluation of Annotations](https://peps.python.org/pep-0563/)
[^pythonorg__pep649]:[PEP 649 – Deferred Evaluation of Annoations Using Descriptors](https://peps.python.org/pep-0649/)
[^github__python_typeshed_stdlib_2and3_builtins_pyi]:[Github : `python/typeshed/stdlib/2and3/builtins.pyi`](https://github.com/python/typeshed/blob/a8834fcd46339e17fc8add82b5803a1ce53d3d60/stdlib/2and3/builtins.pyi)
[^pypiorg__pydantic]:[PyPI - pydantic](https://pypi.org/project/pydantic/)
[^mypy__documentation_casts]:[mypy documentation : Type narrowing - Casts](https://mypy.readthedocs.io/en/stable/type_narrowing.html#casts)
[^pythonorg__annotations_best_practices]:[Annotations Best Practices](https://docs.python.org/3/howto/annotations.html)
[^chapter08__callable]:[Chapter 8 - Types Usable in Annotations - Callable](../chapter-08/#callable)
