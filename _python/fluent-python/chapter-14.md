---
layout: distill
title: Chapter 14
description: Inheritance For Better or For Worse
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstract
  - name: The Super Function
  - name: Subclassing Built-In Types Is Tricky
  - name: Multiple Inheritance and Method Resolution Order
  - name: Method Resolution Order
  - name: Mixin Classes
  - name: Multiple Inheritance in the Real World
  - name: Coping with Inheritance
  - name: Chapter Summary
  - name: Links

---

## Abstract

This is a summary of chapter 14 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/).
This chapter provides an overview of inheritance and tips for inheritance in
Python.


## The `super()` Function

`super()`[^pythonorg__builtinfunctions_super] is a function used to access
the superclass of an object.

### *Recommendation of the `super()` function*

There are two options to use properties or object methods of superclass:

1. calling the name of the superclass explicitly : `SuperClassName.init_this_object()` or `SuperClassName.property`.
2. calling the built-in function `super()` : `super().init_this_class()` or `super().property`.

There are two reasons why it is recommended to use `super()` instead of calling
the name of the superclass explicitly:

1. It can cause a bug if the class inheritance relationships are changed.
2. The method `super()` implements logic to handle class hierarchies with multiple inheritance.

Consistent use of the `super()` built-in function is essential for building
maintainable object-oriented Python programs.

To build maintainable object-oriented programs, it is recommended to use the
`super()` built-in functions for expanding a existing method of the superclass.

In the following example codes, dictionary with order between each dictionary
key-value pairs is implemented by subclassing existing class
[^pythonorg__colletions_ordereddict] and using the existing methods of
superclass [^pythonorg__colletions_ordereddict_move_to_end]:

For example, in the class `FIFOOrderedDict` in the following example codes, you
can use `super()` to inherit and call methods from the parent class.

However, it is not recommended to use the name of the superclass directly in
the subclass, as shown in the class `NotRecommendedFIFOOrderedDict` in the
following example code.

```python
class FIFOOrderedDict(OrderedDict):
    def __setitem__(self, key, value):
        super().__setitem__(key, value)
        self.move_to_end(key)

class NotRecommendedFIFOOrderedDict(OrderedDict):
    def __setitem__(self, key, value):
        OrderedDict.__setitem__(self, key, value)
        self.move_to_end(key)
```

### *Arguments for the `super()` function*

`super()` can accepts two arguments for its calling.

1st Argument - `type`
: The starting class of the search path for the superclass implementing the desired method.
By default, the class of the method which calls the `super()` built-in method is specified.

2nd Argument - `object_or_type`
: The object (for instance method calls) or class (for class method calls with
`@classmethod`) must be specified for this argument.
By default, `self` is specified when calling the `super()` method in an instance method.

These arguments can be omitted.
The equivalant of the statement `super().__setitem__(key, value)` in the
`__setitem__()` method can be written like this.

```python
class FIFOOrderedDict(OrderedDict):
    def __setitem__(self, key, value):
        super(FIFOOrderedDict, self).__setitem__(key, value)
        self.move_to_end(key)
```

The `super()` call returns a dynamic proxy object that finds a specified method
(`__setitem__` for the above example) in a superclass of the `type` parameter.

In Python 3, it is possible to specify the first and second arguments when
calling `super()`, but this is only necessary in special cases such as skipping
the method resolution order for debugging purpose.

## Subclassing Built-In Types Is Tricky

It is possible to subclass built-in types such as `list` or `dict` in Python 2.2 or later.
When subclassing built-in types, it should be considered that the code of the
built-ins (written in C) may **not call methods overriden by user-defined classes**.

In the following example, The overrided `__setitem__` method are ignored in
`dd.update()` of built-in `dict` class.

```python
>>> class DoppelDict(dict):
...     def __setitem__(self, key, value):
...         super().__setitem__(key, [value] * 2)
...
>>> dd = DoppelDict(one=1)
>>> dd
{'one': 1}
>>> dd['two'] = 2
>>> dd
{'one': 1, 'two': [2, 2]}
>>> dd.update(three=3)
>>> dd
{'three': 3, 'one': 1, 'two': [2, 2]}
```

In the following example, the overrided `__getitem__` are ignored in the
following codes which uses built-in implementation.
1. `dd = DoppelDict(one=1)` : When initialization of class object, \
    setting new pairs must used the overridden `__setitem__()` methods.
2. `d.update()` : built-in object methods which uses `__setitem__()`

```python
>>> class AnswerDict(dict):
...     def __getitem__(self, key):
...         return 42
...
>>> ad = AnswerDict(a='foo')
>>> ad['a']
42
>>> d = {}
>>> d.update(ad)
>>> d['a']
'foo'
>>> d
{'a': 'foo'}
```

This behavior(Result of `d.update(ad)`) in built-in types violates a basic rule
of object-oriented programming called "late binding":

The search for methods should always start from the class of
the receiver (`self`),
even when the call happens inside a method implemented in a superclass.

To correct this behavior, one can subclass `collections.UserDict`.
Subclassing a base class coded in Python, like `UserDict` or `MutableMapping`,
will prevent any related issues.

```python
>>> import collections
>>>
>>> class DoppelDict2(collections.UserDict):
...     def __setitem__(self, key, value):
...         super().__setitem__(key, [value] * 2)
...
>>> dd = DoppelDict2(one=1)
>>> dd {'one': [1, 1]}
>>> dd['two'] = 2
>>> dd
{'two': [2, 2], 'one': [1, 1]}
>>> dd.update(three=3)
>>> dd
{'two': [2, 2], 'three': [3, 3], 'one': [1, 1]}
>>>
>>> class AnswerDict2(collections.UserDict):
... def __getitem__(self, key):
...     return 42
...
>>> ad = AnswerDict2(a='foo')
>>> ad['a']
42
>>> d = {}
>>> d.update(ad)
>>> d['a']
42
>>> d
{'a': 42}
```

## Multiple Inheritance and Method Resolution Order

In this chapter, the class `Root`, `A`, `B` and `Leaf` with complex multiple
inheritance will be discussed to describe the method resolution order with
complex class inheritance.

Any programming language implementing multiple inheritance must resolve
potential naming conflicts when superclasses implement a method with
the same name.

In the following example, the complex inheritance hierarchy,
which is hard to deal with, called the "diamond problem", will be discussed.

### *UML Diagrams in the "Diamond Problem"*

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-14/ping.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="The Red Arrows represent the method resolution order of the ping method."
        %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-14/pong.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="The Red Arrows represent the method resolution order of the pong method.
        The pong method of Leaf class is inherited from the pong method in class A."
        %}
    </div>
</div>

### *Source codes*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='cbd13885fc1a7903be433277791ff9d1868be752'
     path='14-inheritance/diamond.py'
     language="python"
     line='27-61'
></div>

* `<1>` : class `Root` provides the following methods.
    * `ping`, `pong` : methods overridden by classes `A`, `B`, and `Leaf`.
    * `__repr__` : returns a descriptive string that indicates which class executed the method.
* `<2>` : class `A` provides the following methods.
    * `ping`, `pong` : methods overrides the `ping` and `pong` method in class `Root`. In these methods, the superclass method will be executed using `super()`.
* `<3>` : class `B` provides the following methods.
    * `ping`, `pong` : methods overrides the `ping` and `pong` method in class `Root`. In `ping` method, the `ping` method of superclass will be executed using `super()`.
* `<4>` : class `Leaf` inherits classes `A` and `B`, and provides a following method.
    * `ping` : methods overrides the `ping` and `pong` method in classes `Root`, `A`, and `B`. In `ping` method, the `ping` method of superclass will be executed using `super()`.

### *`ping` Method resolution order*

```python
>>> leaf1 = Leaf()
>>> leaf1.ping()
<instance of Leaf>.ping() in Leaf
<instance of Leaf>.ping() in A
<instance of Leaf>.ping() in B
<instance of Leaf>.ping() in Root
```

Results of `leaf1.ping()` represent methods resolution order of the red arrow
in the figures above and to the left.

### *`pong` Method resolution order*

```
>>> leaf1.pong()
<instance of Leaf>.pong() in A
<instance of Leaf>.pong() in B
```

Results of `leaf1.pong()` represent methods resolution order of the red arrow
in the figures above and to the right.

## *Method Resolution Order*

### *Method Resolution Order of the class `Leaf`*

The MRO (Method Resolution Order) is computed using the C3 algorithm described
in this article [^blog__c3_method_resolution_order].

By using the C3 algorithm, the MRO of class `Leaf` can be computed using
the following equations.

| Class Name | Class Representation               | Linearization Representation             |
| ---------- | ---------------------------------- | ---------------------------------------- |
| `object`   | Italic Letter $O$                  | $L[O]$                                   |
| `Root`     | The Non-italic Letter $\textrm{R}$ | $L[\textrm{R}]$ or $L[\textrm{R}(O)]$    |
| `A`        | Italic Letter $A$                  | $L[A]$ or $L[A(\textrm{R})]$             |
| `B`        | Italic Letter $B$                  | $L[B]$ or $L[B(\textrm{R})]$             |
| `Leaf`     | $\textrm{L}$                       | $L[\textrm{L}]$ or $L[\textrm{L}(A\;B)]$ |

$$
\begin{split}
L[\textrm{R}] = L[\textrm{R}(O)] &= \textrm{R} + merge(L[O],\;O) \\
&= \textrm{R} + merge(O,\;O) \\
&= \textrm{R}O\\
&\\
L[A] = L[A(\textrm{R})] &= A + merge(L[\textrm{R}],\;\textrm{R}) \\
&= A + merge(\textrm{R}O,\;\textrm{R}) \\
&= A\textrm{R} + merge(O) \\
&= A\textrm{R}O\\
&\\
L[B] = L[B(\textrm{R})] &= B + merge(L[\textrm{R}],\;\textrm{R}) \\
&= B + merge(\textrm{R}O,\;\textrm{R}) \\
&= B\textrm{R} + merge(O) \\
&= B\textrm{R}O\\
&\\
L[\textrm{L}] = L[\textrm{L}(A\;B)] &= \textrm{L} + merge(L[A],\;L[B],\;AB)\\
&= \textrm{L} + merge(A\textrm{R}O,\;B\textrm{R}O,\;AB)\\
&= \textrm{L}A + merge(\textrm{R}O,\;B\textrm{R}O,\;B)\\
&= \textrm{L}AB + merge(\textrm{R}O,\;\textrm{R}O)\\
&= \textrm{L}AB\textrm{R} + merge(O,\;O)\\
&= \textrm{L}AB\textrm{R}O
\end{split}
$$

Therefore, the linearization(Method resolution order) of class `Leaf` is
derived as $\textrm{L}AB\textrm{R}O$ by the C3 algorithm.

The same result is also obtained from the `Leaf.__mro__`.

```python
>>> Leaf.__mro__
(
    <class '__main__.Leaf'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.Root'>,
    <class 'object'>
)
```

### *The Roles of the Method Resolution Order*

The MRO determines the activation order of a method that has an implementation
that calls `super()`.

Consider the experiment with the `pong` method. The `Leaf` class does not
override it; therefore, calling `leaf1.pong()` activates the implementation
in the `A` class, which is the next class in `Leaf.__mro__`.

The method `A.pong()` calls `super().pong()`. This calls the implementation in
the `B` class, which is next in the MRO. Therefore, `B.pong()` is activated.

The `B.pong()` method doesn't call `super().pong()`, so the activation sequence
ends here.

### *Cooperative Method*

*Cooperative method* is a method calls `super()`.
Cooperative methods enable *cooperative multiple inheritance*.
In order to work, multiple inheritance in Python requires the active
cooperation of the methods involved.
For example, In the `B` class, `ping` cooperates, but `pong` does not.

A non-cooperative method can be the cause of subtle bugs.
It is expected that when the method `A.pong` calls `super.pong()`, it will
ultimately activate `Root.pong`.
But if `B.pong` is activated before, the activation sequence ends here,
which is not expected by many coders reading this example.
That's why it is recommended that every method `m` of a nonroot class should
call `super().m()`.

### *Dynamic Aspect of Method Resolution Order*

Python is a dynamic language, so interaction of `super()` with the MRO is also
dynamic.

In the following example, the MRO related to class `A` is determined
dynamically by the inheritance relationship.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='cbd13885fc1a7903be433277791ff9d1868be752'
     path='14-inheritance/diamond2.py'
     language="python"
     line='50-66'
></div>

<details><summary>Details of class A</summary>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='cbd13885fc1a7903be433277791ff9d1868be752'
     path='14-inheritance/diamond.py'
     language="python"
     line='27-46'
></div>
</details>

<p></p>

#### *The MRO of Each Classes*

Let's introduce the following representation:

| Class Name | Class Representation               | Linearization Representation                           |
| ---------- | ---------------------------------- | ------------------------------------------------------ |
| `object`   | Italic Letter $O$                  | $L[O]$                                                 |
| `Root`     | The Non-italic Letter $\textrm{R}$ | $L[\textrm{R}]$ or $L[\textrm{R}(O)]$                  |
| `A`        | Italic Letter $A$                  | $L[A]$ or $L[A(\textrm{R})]$                           |
| `U`        | Italic Letter $U$                  | $L[U]$ or $L[U(O)]$                                    |
| `LeafUA`   | $\textrm{L}_{UA}$                  | $$L[\textrm{L}_{UA}]$$ or $$L[\textrm{L}_{UA}(U\;A)]$$ |
| `LeafAU`   | $\textrm{L}_{AU}$                  | $$L[\textrm{L}_{AU}]$$ or $$L[\textrm{L}_{AU}(A\;U)]$$ |

Then, the linearization of each classes is represented by the following
equations.

$$
\begin{split}
L[\textrm{R}] = L[\textrm{R}(O)] &= \textrm{R} + merge(L[O],\;O) \\
&= \textrm{R} + merge(O,\;O) \\
&= \textrm{R}O\\
&\\
L[A] = L[A(\textrm{R})] &= A + merge(L[\textrm{R}],\;\textrm{R}) \\
&= A + merge(\textrm{R}O,\;\textrm{R}) \\
&= A\textrm{R} + merge(O) \\
&= A\textrm{R}O\\
&\\
L[U] = L[U(O)] &= U + merge(L[O], O)\\
&= U + merge(O, O)\\
&= UO\\
&\\
L[\textrm{L}_{UA}] = L[\textrm{L}_{UA}(U\;A)] &= \textrm{L}_{UA} + merge(L[U],\;L[A],\;UA)\\
&= \textrm{L}_{UA} + merge(UO,\;A\textrm{R}O,\;UA)\\
&= \textrm{L}_{UA}U + merge(O,\;A\textrm{R}O,\;A)\\
&= \textrm{L}_{UA}UA + merge(O,\;\textrm{R}O)\\
&= \textrm{L}_{UA}UA\textrm{R} + merge(O,\;O)\\
&= \textrm{L}_{UA}UA\textrm{R}O\\
&\\
L[\textrm{L}_{AU}] = L[\textrm{L}_{AU}(A\;U)] &= \textrm{L}_{AU} + merge(L[A],\;L[U],\;AU)\\
&= \textrm{L}_{AU} + merge(A\textrm{R}O,\;UO,\;AU)\\
&= \textrm{L}_{AU}A + merge(\textrm{R}O,\;UO,\;U)\\
&= \textrm{L}_{AU}A\textrm{R} + merge(O,\;UO,\;U)\\
&= \textrm{L}_{AU}A\textrm{R}U + merge(O,\;O)\\
&= \textrm{L}_{AU}A\textrm{R}UO
\end{split}
$$

These results can be checked by the `__mro__` properties of each classes.

```python
>>> Root.__mro__
(<class '__main__.Root'>, <class 'object'>)
>>> A.__mro__
(
    <class '__main__.A'>, <class '__main__.Root'>,
    <class 'object'>
)
>>> U.__mro__
(<class '__main__.U'>, <class 'object'>)
>>> LeafUA.__mro__
(
    <class '__main__.LeafUA'>, <class '__main__.U'>,
    <class '__main__.A'>, <class '__main__.Root'>,
    <class 'object'>
)
>>> LeafAU.__mro__
(
    <class '__main__.LeafAU'>, <class '__main__.A'>,
    <class '__main__.Root'>, <class '__main__.U'>,
    <class 'object'>
)
```

#### *Results of `super().ping()` Method of class `U`*

The `AttributeError` is raised when using the `ping` method of class `U`
because the `U.__mro__` is set to `(<class 'U'>, <class 'object'>)`,
which means $L[U] = UO$.
This means that the superclass of class `U` is `object`, which does not have
a `ping` method.

```python
>>> u = U()
>>> u.ping()
<__main__.U object at 0x102f0ca10>.ping() in U
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 4, in ping
AttributeError: 'super' object has no attribute 'ping'
```

On the other hand, no exceptions is raised when using `ping` method over
instances of the class `LeafUA` because the `LeafUA.__mro__` is set to
`(<class 'LeafUA'>, <class 'U'>, <class 'A'>, <class 'Root'>, <class 'object'>)`.
This means that the superclass of the class `U` in the MRO of
the class `LeafUA` is `A`, which has a `ping` method.
Therefore `AttributeError` is not raised in the following example.

```python
>>> leaf2 = LeafUA()
>>> leaf2.ping()
<instance of LeafUA>.ping() in LeafUA
<instance of LeafUA>.ping() in U
<instance of LeafUA>.ping() in A
<instance of LeafUA>.ping() in Root
```

Finally, no exception is raised when using `ping` method on instances of
the class `LeafAU` because the `leaf3.ping()` would never reach `U.ping()`
method.
This is because `Root.ping()` method does not call `super().ping()` method.
In this case, the `LeafAU.__mro__` is set to
`(<class 'LeafAU'>, <class 'A'>, <class 'Root'>, <class 'U'>, <class 'object'>)`.
Instead of the superclass of the class `U` is `object`, no exception is raised
because the `Root.ping()` method does not call the `super().ping()` method.

```python
>>> leaf3 = LeafAU()
>>> leaf3.ping()
<instance of LeafAU>.ping() in LeafAU
<instance of LeafAU>.ping() in A
<instance of LeafAU>.ping() in Root
```

The class `U` in the above example can be called a *mixin class*.
A mixin class is a class intended to be used together with other classes in
multiple inheritance to provide additional functionality.
Details of mixin classes will be described in the following chapter.

## Mixin Classes

A Mixin class is a class that does not provide all functionality for a concrete
object but only adds the behavior of child or sibling classes.
The mixin class is designed to be subclassed along with at least one other
class in a multiple inheritance arrangement.

### Case-Insensitive Mappings

In the following example, a mixin class `UpperCaseMixin` that provides
the functionality of case-insensitive access to a mapping with string keys will
be described.

#### *Definition of `UpperCaseMixin`*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='14-inheritance/uppermixin.py'
     language="python"
     line='112-131'
></div>

* `<1>` : This helper function tries to return `key.upper()` and
    if `key.upper()` property cannot be found and raises an `AttributeError`,
    this helper function returns the unchanged `key`.
* `<2>` : This `UpperCaseMixin` mixin class overrides several methods accessing
    `key` using the helper function `_upper()`. Every overriden method is using
    `super()` method for overriding the methods of other classes.

#### *Usage of `UpperCaseMixin`*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='14-inheritance/uppermixin.py'
     language="python"
     line='135-139'
></div>

* `<1>` : `UpperDict` inherits `UpperCaseMixin` and `collections.UserDict` in
    that order because `UpperClassMixin` must override `__setitem__` and other
    methods, and in order to do this, `UpperClassMixin` must appear before
    `collection.UserDict`.
* `<2>` : `UpperCounter` inherits `UpperCaseMixin` and `collections.Counter` in
    that order, and the reason for the order of inheriting class is the same as
    before.
* `<3>` : Instead of `pass`, it is better to provide a docstring in the body of
    the class definition for improved readability.

#### *Demo of `UpperDict` and `UpperCounter`*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='14-inheritance/uppermixin.py'
     language="python"
     line='8-17'
></div>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='14-inheritance/uppermixin.py'
     language="python"
     line='24-26'
></div>

## Multiple Inheritance in the Real World

In the *Design Patterns* book, there is the only example of multiple
inheritance is the Adapter pattern.
In Python, multiple inheritance is also not popular, but there are some
important examples of it.

### ABCs Are Mixins Too

Among the Python standard library, the `collections.abc`
[^pythonorg__collections_abc] library frequently uses multiple inheritance.

Definition of *Mixin methods*
: *Mixin methods* in the official documentation of `collections.abc`
[^pythonorg__collections_abc] means concrete methods implemented in
the collection ABCs.
The following descriptions are the roles of mixin methods:
1. The mixin methods defines interfaces of collection ABCs.
2. The mixin methods plays a role as mixin classes.
    * For example, the implementation of `collections.UserDict`
    [^github__implementation_userdict] uses `update` method that is
    a mixin method of `collections.abc.MutableMapping`.

### ThreadingMixIn and ForkingMixIn

The `http.server` package[^pythonorg__http_server] provides `HTTPServer` and
`ThreadingHTTPServer` class.

*`class`* `http.server.ThreadingHTTPServer(server_address, RequestHandlerClass)`
: This class is identical to `HTTPServer` but uses threads to handle requests by
using the `ThreadingMixIn`. This is useful to handle web browsers pre-opening
sockets, on which `HTTPServer` would wait indefinitely.

This class is generated by inheriting `ThreadingMixIn` and `HTTPServer`.

<div class='embed-github-src'
     repo='python/cpython'
     branch='17c23167942498296f0bdfffe52e72d53d66d693'
     path='Lib/http/server.py'
     language="python"
     line='144-145'
></div>

To provide functionality that handles each request in a new thread,
the class `ThreadMixIn` defines three methods in the following codes.

<div class='embed-github-src'
     repo='python/cpython'
     branch='17c23167942498296f0bdfffe52e72d53d66d693'
     path='Lib/socketserver.py'
     language="python"
     line='664-701'
></div>

* `process_request_thread` : This method does not call `super()` because it is
    a new method for handling requests in a new thread, not an override.
    This implementation calls three instance methods that `HTTPServer` provides
    or other inheriting classes.
* `process_request` : This method does not call `super()` because it is also
    a new method for handling requests in a new thread.
    This function starts a new thread to process the request.
* `server_close` : This method calls `super().server_close()` to stop taking
    requests, and then wait for stopping the threads to stop using
    `self._threads.join()`.

<div class='embed-github-src'
     repo='python/cpython'
     branch='17c23167942498296f0bdfffe52e72d53d66d693'
     path='Lib/socketserver.py'
     language="python"
     line='542-552'
></div>

The above code is part of the `ForkingMixIn` class. `ForkingMixIn` is
a mixin class that provides functionality to support concurrent servers
based on `os.fork()`[^pythonorg__os_fork].
This mixin class can be used similarly to `ThreadingMixIn`.

### Django Generic Views Mixins

In Django, class-based views were introduced in Django 1.3. The following
classes are examples of classes that inherit mixin classes for implementing
required functionality.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-14/django_view.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="UML Class Diagram for the <code>django.views.generic</code> module."
        %}
    </div>
</div>

Class `TemplateView`[^django__templateview]
: A class that renders a template. The class `View`, along with
the mixin classes `TemplateResponseMixin` and `ContextMixin`, generates
this class.
The MRO of the class `TemplateView` is the following list of classes.
0. `TemplateView`[^django__templateview]
1. `TemplateResponseMixin`[^django__templateresponsemixin]
2. `ContextMixin`[^django__contextmixin]
3. `View`[^django__view]

Class `ListView`[^django__listview]
: A class that renders some list of objects. The class `View`, along with some
mixin classes, generates this class.
The MRO of the class `ListView` is the following list of classes.
0. `ListView`[^django__listview]
1. `MultipleObjectTemplateResponseMixin`[^django__multipleobjecttemplateresponsemixin]
2. `TemplateResponseMixin`[^django__templateresponsemixin]
3. `BaseListView`[^django__baselistview]
4. `MultipleObjectMixin`[^django__multipleobjectmixin]
5. `ContextMixin`[^django__contextmixin]
6. `View`[^django__view]


### Multiple Inheritance in Tkinter

## Coping with Inheritance

There is still no general theory about inheritance that can guide practicing
programmers. It is easy to create incomprehensible designs using inheritance,
even without multiple inheritance.
The following tips are to avoid incomprehensible class designs.

### Favor Object Composition over Class Inheritance

The subtitle of this chapter, 'Favor Object Composition over Class Inheritance',
is from the book *Design Patterns*[^book__designpatterns].

The two most common techniques for reusing functionality in object-oriented
systems are the following:
1. Class inheritance[^blog__relationships_inheritance]
    (often referred to as **white-box reuse**)
2. Object composition[^blog__relationships_composition]
    (often referred to as **black-box reuse**)

Inheritance and composition each have their advantages and disadvantages.

* **Advantages of Inheritance**
    1. Inheritance is easy to understand because inheritance is defined
        statically during compile-time.
    2. Inheritance is easy to use.
    3. Using method overriding makes it easier to modify the implementation
        being reused in inheritance.
* **Disadvantages of Inheritance**
    1. Implementations of inheritance cannot be changed at run-time because
        inheritance is defined at compile-time.
    2. Inheritance reveals the details of implementation of its parent class.
        In other words, it can be said that 'inheritance breaks encapsulation'.
    3. The implementation of a subclass becomes tightly coupled with
        the implementation of its parent class. This may cause any changes in
        the implementation of the parent class to require changes in
        the subclass.
    4. The above implementation dependencies can cause problems when reusing
        a subclass. When these problems occur, the parent class may need to be
        rewritten or replaced, which causes issues when reusing a subclass.
        * it is possible to overcome this issue by using abstract base classes.
* **Advantages of Composition**
    1. When using composition, objects generated from other classes are
        accessed solely through their well-designed interfaces.
        This does not break the encapsulation of class design.
    2. Objects that are involved in composition are accessed through their
        interface, which reduces implementation dependencies.
    3. Composition is defined dynamically at runtime by acquiring objects from
        other classes.
        Therefore, objects that are involved in composition can be replaced at
        runtime by other objects with the same type or objects that have
        the same properties and methods.
    4. Composition helps to keep each class encapsulated and class hierarchies
        simple.
* **Disadvantages of Composition**
    1. Composition requires objects generated from other classes with carefully
        designed interfaces.
    2. Composition requires objects generated dynamically from other classes,
        and this creates interdependencies among these objects, making
        the behavior of the system dependent on them.

To sum up, favoring composition[^blog__relationships_composition] over class
inheritance is desirable for class designs. The most efficient and desirable
class design is to obtain all the required functionality by assembling existing
components using composition. Inheritance must be used only when the required
functionality cannot be implemented by using composition of existing components.

#### *Delegation in Class Design*

Delegation can be used for utilizing composition.
In class design, delegation is a design pattern where an object passes on
responsibilities for certain tasks to another object.
It allows for better separation of concerns, promotes code reusability, and
makes the class design more flexible and maintainable.
The key idea is that instead of a class directly implementing certain
functionalities, it delegates those tasks to other classes that specialize in
handling them.

Let's illustrate this concept with a simple example in Python. Suppose we have
a `PrintFormatter` class that formats and prints data, but we want it to be
able to use different printing strategies.
Instead of embedding the printing strategy within the `PrintFormatter` class,
we'll delegate the printing responsibility to other classes representing
different printing strategies.

```python
import abc

# The base abstract class for the printing strategy
class PrintStrategy(abc.ABC):
    @abc.abstractmethod
    def print_data(self, data):
        pass

# Concrete implementation of PrintStrategy using plain print
class PlainTextStrategy(PrintStrategy):
    def print_data(self, data):
        print("Plain Text:", data)

# Concrete implementation of PrintStrategy using a fancy printer
class FancyTextStrategy(PrintStrategy):
    def print_data(self, data):
        print("Fancy Text: ~~~", data, "~~~")

# The main PrintFormatter class that delegates printing to the chosen strategy
class PrintFormatter:
    def __init__(self, strategy):
        self.strategy = strategy

    def format_and_print(self, data):
        formatted_data = self._format_data(data)
        self.strategy.print_data(formatted_data)

    def _format_data(self, data):
        # Perform any necessary formatting here
        return str(data).upper()

# Usage example
data = "hello, delegation!"

# Using the PlainTextStrategy
plain_strategy = PlainTextStrategy()
formatter = PrintFormatter(plain_strategy)
formatter.format_and_print(data)  # Output: Plain Text: HELLO, DELEGATION!

# Using the FancyTextStrategy
fancy_strategy = FancyTextStrategy()
formatter = PrintFormatter(fancy_strategy)
formatter.format_and_print(data)  # Output: Fancy Text: ~~~ HELLO, DELEGATION! ~~~
```

In this example, `PrintFormatter` delegates the responsibility of actually
printing the data to the `PrintStrategy` classes (`PlainTextStrategy` and
`FancyTextStrategy`).
By doing this, we can easily switch between different printing behaviors
without modifying the `PrintFormatter` class itself.

Delegation is an essential principle in class design, and it can be used in
various contexts to enhance code modularity and maintainability.
It allows for a more flexible and extensible codebase, as well as better
adherence to the Single Responsibility Principle (SRP) in object-oriented
design.

### Understand Why Inheritance Is Used in Each Case

When we plan to use multiple inheritance, it is important to check the reasons
why subclassing or inheritance is needed in each cases. The common reasons are:
1. Inheritance of an interface creates a subtype. This means "is-a"
    relationship. This can be done with abstract base classes.
    * Interface inheritance should use only ABCs as base classes.
2. Inheritance allows us to avoid implimentation code duplication. Mixin classes
    can help with this.
    * This should be replaced by composition and delegation, if possible.

### Make Interfaces Explicit with ABCs

* When designing a class to define an interface, the class should be an
    explicit ABC or a `typing.Protocol` subclass.
* An ABC should subclass only `abc.ABC` or other ABCs.
* Multiple inheritance of ABCs is possible.

### Use Explicit Mixins for Code Reuse

The followings are tips for building mixin classes.
* A class that is designed to provide additional method implementation for code
    reuse by multiple subclasses without an "is-a" relationship should be an
    explicit mixin class.
* Mixin classes does not define a new type and should not be instantiated.
* Mixin classes provides additional method implementations.
* Mixin classes bundles a few and very closely related methods for providing
    a single specific additional functionality.
* Mixin classes should avoid keeping any internal state or properties.
* It is recommended to use `Mixin` suffix for the mixin class name to annotate
    the mixin class.
    * There is no function for annotating mixin class in Python.

### Provide Aggregate Classes to Users

A class that is constructed primarily by inheriting from mixin classes and does
not add its own structure or behavior is called
an *aggregate class*[^book__OOADA_aggregate_class].

For example, the following implementation of the class `ListView` of Django can
be specified as an aggregate class. The body of the class `ListView` is empty
but the class offers useful services.

Note that aggregate classes don't have to be completely empty, but they are
often empty.

<div class='embed-github-src'
     repo='django/django'
     branch='b64db05b9cedd96905d637a2d824cbbf428e40e7'
     path='django/views/generic/list.py'
     language="python"
     line='194-198'
></div>

### Subclass Only Classes Designed for Subclassing

Subclassing any complex class and overriding its methods can cause bugs
because the methods of the superclass may not be designed to be overridden
in the subclass.

To overcome this problem, the following approaches are helpful.
1. Avoid methods overriding.
2. Only subclass classes that are designed to be easily extended.
3. Subclass in a manner consistent with how they were designed to be
extended.

To confirm that a class is designed to be extended by subclass or not,
the following approaches are helpful.
1. Check documentation or docstrings or comments in codes.
2. Check if a class, method, variable or attribute is decorated with
    the `@final` decorator[^pythonorg__typing_final_decorator] or annotated
    with the `Final` annotation[^pythonorg__typing_final_annotation]
    which was introduced in PEP 591 [^pythonorg__PEP591]
    * If a class decorated with the `@final` decorator,
        it should not be subclassed.
    * If a method decorated with the `@final` decorator,
        it should not be overridden.
    * If a variable or attribute annotated with the `Final` annotation,
        it should not be reassigned.

### Avoid Subclassing from Concrete Classes

Subclassing concrete classes is more dangerous than subclassing ABCs and
mixin classes because
1. instances of concrete classes usually have internal state that can easily be
    corrupted when you override methods that depend on that state.
2. Even if your methods cooperate by calling `super()`, and the internal state
    is held in private attributes using the `__x` syntax, there are still
    countless ways a method override can introduce bugs.

The following stratagies for subclassing are helpful.
1. All non-leaf classes should be abstract.
2. Only abstract classes should be subclassed.
3. If you must use subclassing for code reuse, then the code intended for reuse
    should be in mixin methods of ABCs or in explicitly named mixin classes.

## Chapter Summary

* **The `super()` Function**
    * The recommendation of `super()` function in the context of single
        inheritance
* **Subclassing Built-In Types Is Tricky**
    * Native methods of built-in types implemented in C do not call overridden
        methods in subclasses, except in very few special cases.
    * It is strongly recommended inherit `UserList`, `UserDict`, or `UserString`
        instead of `list`, `dict`, or `str` type.
    * These `UserXxxx` types are all defined in the collections module, which
        actually wrap the corresponding built-in types and delegate operations
        to them.
    * If the desired behavior is very different from what the built-ins offer,
        it may be easier to subclass the appropriate ABC from `collections.abc`
        and write your own implementation.
* **Multiple Inheritance and Method Resolution Order**
    * When using multiple inheritance, method resolution order(MRO) may be complex.
* **Method Resolution Order**
    * Method Resolution Order(MRO) is stored in `__mro__` class attribute.
    * The MRO is calculated by C3 algorithm.
    * The MRO determines the activation order of a method that has
        an implementation that calls `super()`.
    * Cooperative method is a method calls `super()`. Cooperative methods
        enable cooperative multiple inheritance.
    * A non-cooperative method can be the cause of subtle bugs.
    * Python is a dynamic language, so interaction of `super()` with the MRO is
        also dynamic.
* **Mixin Classes**
    * A Mixin class is a class that does not provide all functionality for
        a concrete object but only adds the behavior of child or
        sibling classes.
    * The mixin class is designed to be subclassed along with at least one
        other class in a multiple inheritance arrangement.
* **Multiple Inheritance in the Real World**
    * The following examples are described in this chapter.
        * `collections.abc`
        * `ThreadingMixIn` and `ForkingMixIn` in `http.server` package
        * Django Generic Views Mixins
* **Coping with Inheritance**
    * Favor Object Composition over Class Inheritance
    * Understand Why Inheritance Is Used in Each Case
    * Make Interfaces Explicit with ABCs
    * Use Explicit Mixins for Code Reuse
    * Provide Aggregate Classes to Users
    * Subclass Only Classes Designed for Subclassing
    * Avoid Subclassing from Concrete Classes
        * Rejecting inheritance—even single inheritance—is a modern trend.

## Links
[^pythonorg__builtinfunctions_super]:[Built-in Functions : *`class`* **`super`**](https://docs.python.org/3/library/functions.html#super)
[^pythonorg__colletions_ordereddict]:[`collections` — Container datatypes : `class collections.OrderedDict`](https://docs.python.org/3/library/collections.html#collections.OrderedDict)
[^pythonorg__colletions_ordereddict]:[`collections` — Container datatypes : `class collections.OrderedDict`](https://docs.python.org/3/library/collections.html#collections.OrderedDict)
[^pythonorg__colletions_ordereddict_move_to_end]:[`collections` — Container datatypes : `collections.OrderedDict.move_to_end()`](https://docs.python.org/3/library/collections.html#collections.OrderedDict.move_to_end)
[^pythonorg__collections_abc]:[`collections.abc` — Abstract Base Classes for Containers](https://docs.python.org/3/library/collections.abc.html)
[^pythonorg__http_server]:[`http.server` — HTTP servers](https://docs.python.org/3/library/http.server.html)
[^pythonorg__os_fork]:[`os` — Miscellaneous operating system interfaces : `os.fork()`](https://docs.python.org/3/library/os.html#os.fork)
[^pythonorg__typing_final_decorator]:[`typing` — Support for type hints : `@typing.final`](https://docs.python.org/3/library/typing.html#typing.final)
[^pythonorg__typing_final_annotation]:[`typing` — Support for type hints : `typing.Final`](https://docs.python.org/3/library/typing.html#typing.Final)

[^pythonorg__PEP591]:[PEP 591 – Adding a final qualifier to typing](https://peps.python.org/pep-0591/)
[^github__implementation_userdict]:[Implementation of `class UserDict`](https://github.com/python/cpython/blob/8ece98a7e418c3c68a4c61bc47a2d0931b59a889/Lib/collections/__init__.py#L1084)
[^blog__c3_method_resolution_order]:[C3 Method Resolution Order](../../standard-library/c3_method_resolution_order/)
[^blog__relationships_composition]:[Relationships : Composition](../../../softwaredesign/uml-diagram/relationships/#composition)
[^blog__relationships_inheritance]:[Relationships : Generalization](../../../softwaredesign/uml-diagram/relationships/#generalization)
[^django__templateview]:[Django Classy Class-Based Views — Class `TemplateView`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.base/TemplateView/)
[^django__templateresponsemixin]:[Django Classy Class-Based Views — Class `TemplateResponseMixin`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.base/TemplateResponseMixin/)
[^django__contextmixin]:[Django Classy Class-Based Views — Class `ContextMixin`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.base/ContextMixin/)
[^django__view]:[Django Classy Class-Based Views — Class `View`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.base/View/)
[^django__listview]:[Django Classy Class-Based Views — Class `ListView`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.list/ListView/)
[^django__multipleobjecttemplateresponsemixin]:[Django Classy Class-Based Views — Class `MultipleObjectTemplateResponseMixin`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.list/MultipleObjectTemplateResponseMixin/)
[^django__baselistview]:[Django Classy Class-Based Views — Class `BaseListView`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.list/BaseListView/)
[^django__multipleobjectmixin]:[Django Classy Class-Based Views — Class `MultipleObjectMixin`](https://ccbv.co.uk/projects/Django/4.2/django.views.generic.list/MultipleObjectMixin/)
[^book__designpatterns]:[Wikipedia : Design Patterns: Elements of Resuable Object-Oriented Software (1994)](https://en.wikipedia.org/wiki/Design_Patterns)
[^book__OOADA_aggregate_class]:Grady Booch et al., *Object-Oriented Analysis and Design with Applications*, 3rd ed. (Addison-Wesley), p. 109.
