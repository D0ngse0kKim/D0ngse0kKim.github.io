---
layout: distill
title: Chapter 16
description: Operator Overloading
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: Abstract
  - name: Chapter Summary
  - name: Further Reading
  - name: Links
---

## Abstract

This is the 16-th chapter of the book Fluent Python, which is about operator
overloading.

## Operator Overloading 101

In Python, there are operators that can be used to perform operations.
For example, the following operators are existing in Python:

1. Infix operators: `+`, `-`, `*`, `/`, `//`, `%`, `**`, `<<`, `>>`, `&`, `^`, `|`
    * Infix operators are operators that are placed between two operands.
2. Unary operators: `+`, `-`, `~`
    * Unary operators are operators that are placed before one operand.
3. Comparison operators: `<`, `<=`, `>`, `>=`, `==`, `!=`
    * Comparison operators are operators that are used to compare two operands.
4. Boolean operators: `and`, `or`, `not`
    * Boolean operators are operators that are used to perform boolean operations.
5. Function invocation: `()`
    * Function invocation is used to call a function.
6. Attribute access: `.`
    * Attribute access is used to access an attribute of an object.
7. Item access/slicing: `[]`, `[:]`
    * Item access/slicing is used to access an item or a slice of an object.
8. Assignment: `=`, `+=`, `-=`, `*=`, `/=`, `//=`, `%=`, `**=`, `<<=`, `>>=`,
    `&=`, `^=`, `|=`
    * Assignment is used to assign a value to a variable.

Operator overloading is a language feature that can be abused, making
the program harder to understand, and resulting in unexpected behavior and bugs.
However, operator overloading can also be used to make the program more
readable, and to make the program more efficient.

Therefore, Python make the following restrictions on operator overloading to
prevent abuse:

1. It is not possible to overload operators for built-in types.
2. It is not possible to make a new operator. However, it is only possible
    to overload the existing operators.
3. It is not possible to overload the following operators:
    * `and`, `or`, `not`
    * `is`, `is not`, `in`, `not in`
    * `[]`, `[:]`, `()`, `.`, `=`
    * `->`, `,`
    * `yield`, `yield from`
    * `if`, `elif`, `else`
    * `for`, `while`
    * `try`, `except`, `finally`
    * `with`, `as`
    * `assert`
    * `import`, `def`, `class`
    * `lambda`, `nonlocal`, `pass`, `raise`, `return`, `global`, `del`
    * `break`, `continue`
    * `from`, `print`
    * `async`, `await`

## Unary Operators

The Python Language Reference [^pythonorg__expressions_unary]
[^pythonorg__datamodel_abs] defines the following unary operators:
1. `-` : Unary arithmetic negation which is implemented by `__neg__`.
    * If `x` is `-2`, then `-x` is `2`.
2. `+` : Unary arithmetic positive which is implemented by `__pos__`.
    * If `x` is `-2`, then `+x` is `-2`.
    * However, `+x` is not always equal to `x`, because `+x` may return
        a new object.
3. `~` : Unary arithmetic inversion which is implemented by `__invert__`.
    * If `x` is `2`, then `~x` is `-3` because the `~` operator on the `int`
        type means bitwise inverse operator.
4. `abs()` : Absolute value which is implemented by `__abs__`.
    * If `x` is `-2`, then `abs(x)` is `2`.

### *Examples of Implementations of Unary Operators*

The unary operators can be implemented by implementing the corresponding
special methods, which take only one argument, `self`.
The following code shows an example of implementing the unary operators `abs()`,
`-`, and `+` on the `Vector` class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='285-292'
></div>

* `<1>` : It is possible to get the components of a `Vector` object by using
    a `for` loop and generator expression, because the `Vector` class implements
    the `__iter__` method.
* `<2>` : The method `__pos__` builds a new `Vector` object by using the
    `Vector` constructor.

### *Example of a case which `x` is not equal to `+x`*

The following code shows an example of a case which `x` is not equal to `+x`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/unary_plus_decimal.py'
     language='python'
     line='4-16'
></div>

* `<1>` : Get a reference to the current global arithmetic context.
* `<2>` : Change the precision of the current global arithmetic context to `40`.
* `<3>` : Create a `Decimal` object `one_third` with the value `1/3`.
* `<4>` : Print the value of `one_third` with the precision of `40`.
* `<5>` : Check whether `+one_third` is equals to `one_third`.
* `<6>` : Change the precision of the current global arithmetic context from `40` into `28`.
* `<7>` : Check whether `+one_third` is equals to `one_third` again and the result of this comparison is `False`.
* `<8>` : The reason of the above comparison is `False` is that `+one_third`
    returns a new `Decimal` object with the precision of `28`.
    * The fact that `+one_third` returns a new `Decimal` object can be verified
        by checking that the `id(one_third)` and `id(+one_third)` are different
        value.

## Overloading `+` for Vector Addition

In this subsection, the `+` operator is overloaded to support vector addition.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='351-359'
></div>

* `pairs = itertools.zip_longest(self, other, fillvalue=0.0)` fills the missing
    values with `0.0` and this makes it possible to add two `Vector` objects
    with different lengths.
* `return Vector(a + b for a, b in pairs)` builds a new `Vector` object.
    By doing this, `+` operator returns a new `Vector` object instead of
    modifying the existing `Vector` object.
* `try:` and `except TypeError:` and `return NotImplemented` are used to get
    the correct exception message.
* `__radd__` method delegates to `__add__` method, and this makes it possible
    to add a `Vector` object and an object of another type.

### *Usage of `+` for `Vector` Class*

The following code shows the basic usage of `+` for the `Vector` class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='202-212'
></div>

### *Implementation of Mixed Type Addition*

Mixed type addition such as adding a `Vector` object to a `tuple` object can
only be supported if the `tuple` object is the right operand of the `+`
operator.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='215-222'
></div>

Mixed type addition, such as adding a `tuple` object to a `Vector` object,
cannot be supported by implementing `__add__` method alone when the `tuple`
object is the left operand of the `+` operator.

```python
>>> v1 = Vector([3, 4, 5])
>>> (10, 20, 30) + v1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can only concatenate tuple (not "Vector") to tuple
>>> from vector2d_v3 import Vector2d
>>> v2d = Vector2d(1, 2)
>>> v2d + v1
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for +: 'Vector2d' and 'Vector'
```

By implementing the `__radd__` method in addition to the `__add__` method, it
is possible to support mixed type addition, such as adding a `tuple` object and
a `Vector` object, regardless of the order of the operands.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='225-232'
></div>

Computing process with `__add__` and `__radd__` method can be visualized as
the following flowchart:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-16/add_radd.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="Flowchart for computing <code>a + b</code> with <code>__add__</code> and <code>__radd__</code>."
        %}
    </div>
</div>

To sum up the above flowchart, the following rules are used to compute `a + b`:
1. If `a` has an `__add__` method, then `a.__add__(b)` is called.
    If the result of `a.__add__(b)` is not `NotImplemented`, then the result of
    `a.__add__(b)` is returned.
2. If `a` does not have `__add__` method or the result of `a.__add__(b)` is
    `NotImplemented`, then `b.__radd__(a)` is called.
    If the result of `b.__radd__(a)` is not `NotImplemented`, then the result of
    `b.__radd__(a)` is returned.
3. If `b` does not have `__radd__` method or the result of `b.__radd__(a)` is
    `NotImplemented`, then a `TypeError` exception is raised.

Note that `NotImplemented` and `NotImplementedError` are different.
* `NotImplemented` is a special singleton value that is used to indicate that
    the operator is not implemented for the operands.
* `NotImplementedError` is an exception that is raised when a method of
    an abstract base class that is not implemented is called.

Without `try` and `except TypeError` and `return NotImplemented` in
the `__add__` method, the following code will raise a misleading `TypeError`
exception message.

```python
'''Implementation of Vector class is omitted here...'''
    def __add__(self, other):
        # Implementation without try and except TypeError and return NotImplemented
        pairs = itertools.zip_longest(self, other, fillvalue=0.0)
        return Vector(a + b for a, b in pairs)

    def __radd__(self, other):
        return self + other
'''Implementation of Vector class is omitted here...'''

>>> v1 = Vector([3, 4, 5])
>>> v1 + 1 Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "vector_v6.py", line 328, in __add__
    pairs = itertools.zip_longest(self, other, fillvalue=0.0)
TypeError: zip_longest argument #2 must support iteration

>>> v1 + 'ABC'
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "vector_v6.py", line 329, in __add__
    return Vector(a + b for a, b in pairs)
  File "vector_v6.py", line 243, in __init__
    self._components = array(self.typecode, components)
  File "vector_v6.py", line 329, in <genexpr>
    return Vector(a + b for a, b in pairs)
TypeError: unsupported operand type(s) for +: 'float' and 'str
```

To prevent this misleading exception message, `TypeError` exception must be
prevented by using `try:` and `except TypeError:` and `return NotImplemented`
to get the correct exception message.

```python
'''Implementation of Vector class is omitted here...'''
    def __add__(self, other):
        # Implementation with try and except TypeError and return NotImplemented
        try:
            pairs = itertools.zip_longest(self, other, fillvalue=0.0)
            return Vector(a + b for a, b in pairs)
        except TypeError:
            return NotImplemented

    def __radd__(self, other):
        return self + other
'''Implementation of Vector class is omitted here...'''
```

By the above implementation, the following code will raise a correct
`TypeError` exception message.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v6.py'
     language='python'
     line='235-244'
></div>

## Overloading `*` for Scalar Multiplication

In this subsection, the `*` operator is overloaded to support scalar
multiplication. To overload the `*` operator, the `__mul__` and `__rmul__`
method must be implemented.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v7.py'
     language='python'
     line='410-418'
></div>

* `factor = float(scalar)` is a duck typing technique to support scalar
    multiplication with an object that is not a `float` object.
* `try:`, `except TypeError:` and `return NotImplemented` are used to get
    the correct exception message.
* `__rmul__` method delegates to `__mul__` method, and this makes it possible
    to multiply a `Vector` object and an object of another type.

### *Usage of `*` for `Vector` Class*

The following code shows the basic usage of `*` for the `Vector` class.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v7.py'
     language='python'
     line='247-253'
></div>

The following code shows an example of a case which `*` is used to multiply
with unusual types.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v7.py'
     language='python'
     line='256-262'
></div>

## Using `@` as an Infix Operator

`@` is a new infix operator that is introduced in Python 3.5 [^pythonorg__pep465].
`@` is used to perform matrix multiplication. To overload the `@` operator,
`__matmul__` and `__rmatmul__` method must be implemented.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v7.py'
     language='python'
     line='420-431'
></div>

* The parameter `other` of the `__matmul__` method must implement `__len__` and
    `__iter__` method. If `other` does not implement `__len__` and `__iter__`,
    then the `__matmul__` returns `NotImplemented`.
* The `__matmul__` method returns the result of the matrix multiplication
    only if the length of `other` is equal to the length of `self`. Otherwise,
    the `__matmul__` method raises `ValueError` with proper exception message.
* `__rmatmul__` method delegates to `__matmul__` method, and this makes it
    possible to perform matrix multiplication with an object of another type.

## Wrapping-Up Arithmetic Operators

The following table shows the arithmetic infix operators and the corresponding
special methods that are used to overload the arithmetic infix operators.

| Operator     | Forward        | Reverse         | In-place        | Description                                                  |
| ------------ | -------------- | --------------- | --------------- | ------------------------------------------------------------ |
| `+`          | `__add__`      | `__radd__`      | `__iadd__`      | Addition                                                     |
| `-`          | `__sub__`      | `__rsub__`      | `__isub__`      | Subtraction                                                  |
| `*`          | `__mul__`      | `__rmul__`      | `__imul__`      | Multiplication or repetition                                 |
| `/`          | `__truediv__`  | `__rtruediv__`  | `__itruediv__`  | True division                                                |
| `//`         | `__floordiv__` | `__rfloordiv__` | `__ifloordiv__` | Floor division                                               |
| `%`          | `__mod__`      | `__rmod__`      | `__imod__`      | Modulo                                                       |
| `divmod()`   | `__divmod__`   | `__rdivmod__`   | `__idivmod__`   | Divmod (Returns tuple of floor division quotient and modulo) |
| `**`,`pow()` | `__pow__`      | `__rpow__`      | `__ipow__`      | Exponentiation                                               |
| `@`          | `__matmul__`   | `__rmatmul__`   | `__imatmul__`   | Matrix multiplication                                        |
| `&`          | `__and__`      | `__rand__`      | `__iand__`      | Bitwise AND                                                  |
| `|`          | `__or__`       | `__ror__`       | `__ior__`       | Bitwise OR                                                   |
| `^`          | `__xor__`      | `__rxor__`      | `__ixor__`      | Bitwise XOR                                                  |
| `<<`         | `__lshift__`   | `__rlshift__`   | `__ilshift__`   | Bitwise left shift                                           |
| `>>`         | `__rshift__`   | `__rrshift__`   | `__irshift__`   | Bitwise right shift                                          |

## Rich Comparison Operators

There are six rich comparison operators `==`, `!=`, `>`, `>=`, `<`, `<=` in
Python.
* These operators are called rich comparison operators because they return
    a rich set of boolean values.
* The same set of methods is used in forward and reverse comparison.
* The only exception is that `==` and `!=` has fallback behavior.
    * `==` : If the reverse method is missing or returns `NotImplemented`,
        then the `__eq__` method compares the `id` of the two objects.
    * `!=` : If the reverse method is missing or returns `NotImplemented`,
        then the `__ne__` method returns the negation of the result of
        `__eq__` method.

| Group    | Infix operator | Forward method call | Reverse method call | Fallback                |
| -------- | -------------- | ------------------- | ------------------- | ----------------------- |
| Equality | `a == b`       | `a.__eq__(b)`       | `b.__eq__(a)`       | `Return id(a) == id(b)` |
| Equality | `a != b`       | `a.__ne__(b)`       | `b.__ne__(a)`       | `Return not (a == b)`   |
| Ordering | `a > b`        | `a.__gt__(b)`       | `b.__lt__(a)`       | `Raise TypeError`       |
| Ordering | `a < b`        | `a.__lt__(b)`       | `b.__gt__(a)`       | `Raise TypeError`       |
| Ordering | `a >= b`       | `a.__ge__(b)`       | `b.__le__(a)`       | `Raise TypeError`       |
| Ordering | `a <= b`       | `a.__le__(b)`       | `b.__ge__(a)`       | `Raise TypeError`       |

### *Example of Implementations of Rich Comparison Operators*

The following code shows an example of implementing `__eq__`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v7.py'
     language='python'
     line='328-330'
></div>

By using this implementation, comparing two `Vector` objects with the same
components will return `True`, regardless of the type of the other object to be
compared.

```python
>>> va = Vector([1.0, 2.0, 3.0])
>>> vb = Vector(range(1, 4))
>>> va == vb
True
>>> vc = Vector([1, 2])
>>> from vector2d_v3 import Vector2d
>>> v2d = Vector2d(1, 2)
>>> vc == v2d
True
>>> t3 = (1, 2, 3)
>>> va == t3
True
```

Comparison between `Vector` objects with different type of iteratable objects
may cause unexpected behavior. In the following example of `__eq__` method,
comparison `Vector` objects with different type of iteratable objects will
return `NotImplemented`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v8.py'
     language='python'
     line='330-335'
></div>

* `<1>` : By using `isinstance` check on `other` object, it is possible to
    prevent comparison between `Vector` object and an object of another type.
* `<2>` : If `other` object is not an instance of `Vector` class, then
    `NotImplemented` is returned.

### *What happens when using `==` operator*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v8.py'
     language='python'
     line='273-285'
></div>

#### *Tests of operator `vc == v2d`*

`vc == v2d` returns `True` on the above example. The following steps show
that how `True` is returned.

1. To evaluate `vc == v2d`, `vc.__eq__(v2d)` is called.
2. `vc.__eq__(v2d)` returns `NotImplemented` because `v2d` is not an instance
    of `Vector` class.
3. `v2d.__eq__(vc)` is called because `NotImplemented` is returned.
4. In the `v2d.__eq__(vc)` method, there is no `isinstance` check on `other`
    object, so the result `True` is returned.

#### *Tests of operator `va == t3`*

`va == t3` returns `False` on the above example. The following steps show
that how `False` is returned.

1. To evaluate `va == t3`, `va.__eq__(t3)` is called.
2. `va.__eq__(t3)` returns `NotImplemented` because `t3` is not an instance
    of `Vector` class.
3. `t3.__eq__(va)` is called because `NotImplemented` is returned.
4. The `t3.__eq__(va)` method cannot handle the comparison between `Vector`
    object and `tuple` object, so `NotImplemented` is returned.
5. `NotImplemented` is returned from `t3.__eq__(va)`, so the fallback behavior
    `id(va) == id(t3)` is used. Since `va` and `t3` are different objects,
    `False` is returned.

### *What happens when using `!=` operator*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='16-op-overloading/vector_v8.py'
     language='python'
     line='288-295'
></div>

There is no implementation of `__ne__` method on the above example. Therefore,
the fallback behavior is used when `va != vb`, `vc != v2d` and `va != (1, 2, 3)`
is evaluated.

The fallback behavior of `!=` operator is `not (a == b)`. Therefore, the
following comparison results are returned when `!=` operator is used.
1. `va != vb` is equivalant to `not (va == vb)`, so `False` is returned.
2. `vc != v2d` is equivalant to `not (vc == v2d)`, so `False` is returned.
3. `va != (1, 2, 3)` is equivalant to `not (va == (1, 2, 3))`, so `True` is
    returned.

## Augmented Assignment Operators

Augmented assignment operators are used to perform an operation and assign
the result to the left operand. For example, the following code shows the
usage of augmented assignment operators `+=` and `*=` on `Vector` objects.

```python
>>> v1 = Vector([1, 2, 3])
>>> v1_alias = v1
>>> id(v1)
4302860128
>>> v1 += Vector([4, 5, 6])
>>> v1
Vector([5.0, 7.0, 9.0])
>>> id(v1)
4302859904
>>> v1_alias
Vector([1.0, 2.0, 3.0])
>>> v1 *= 11
>>> v1
Vector([55.0, 77.0, 99.0])
>>> id(v1)
4302858336
```

### *Syntactic Sugar of Augmented Assignment Operators*

* `a += b` : `a.__iadd__(b)` is called if `a` has `__iadd__` method.
    Otherwise, `a = a + b` is called.
* `a *= b` : `a.__imul__(b)` is called if `a` has `__imul__` method.
    Otherwise, `a = a * b` is called.

The `Vector` objects already supports `+=` and `*=` operators because
the `Vector` class implements `__add__` method and `__mul__` method.

However, the results of calling `id()` on the above example shows that
the results of `v1 += Vector([4, 5, 6])` and `v1 *= 11` are different objects
from `v1_alias`. This is because the `__add__` method and `__mul__` method
returns a new `Vector` object instead of modifying the existing `Vector`
object.

`Vector` class is immutable, therefore `__iadd__` and `__imul__` methods should
not be implemented. However, if `Vector` class is mutable, then `__iadd__` and
`__imul__` methods, which modify the existing `Vector` object, should be
implemented.

### *Implementation of `AddableBingoCage` Class*

In this subchapter, the `AddableBingoCage` class is implemented to support
the infix operator `+` and the augmented assignment operator `+=`.
The `AddableBingoCage` object is mutable, therefore the `+=` operator modifies
the existing object.

`+` Operator
: The `+` operator on the `AddableBingoCage` class accepts only the abstract
    base class `Tombola`. The result of the `+` operator produces a new
    `AddableBingoCage` object.

`+=` Operator
: The `+=` operator on the `AddableBingoCage` class accepts the abc `Tombola`
    and any iterable object. The result of the `+=` operator modifies
    the existing `AddableBingoCage` object and returns `self`.


#### *Details of Implementation*

<details><summary> Details of the <code>Tombola</code> class </summary>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='135eca25d91595e30eb8538a4b9b7f70393ad5a6'
     path='13-protocol-abc/tombola.py'
     language='python'
     line='3-31'
></div>
</details>

<details><summary> Details of the <code>BingoCage</code> class </summary>
<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='13-protocol-abc/bingo.py'
     language='python'
     line='3-26'
></div>
</details>

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='16-op-overloading/bingoaddable.py'
     language='python'
     line='58-81'
></div>

* `<2>` : The `AddableBingoCage` class inherits from the `BingoCage` class.
* `<3>` : The `+` operator on the `AddableBingoCage` object only accepts
    the abstract base class `Tombola`.
* `<4>` : If `other` object is an instance of the `Tombola` class, then
    `other.inspect()` is called to get the contents of `other` object.
* `<5>` : If `other` object is not an instance of `Tombola` class, then
    try to get iterator from the `other` object.
* `<6>` : If acquiring iterator from `other` object is failed and `TypeError`
    is raised, then raises `TypeError` exception with proper exception message.
* `<7>` : Load the contents of `other` object using the iterator acquired from
    `<5>`.
* `<8>` : The `AddableBingoCage` object is mutable, therefore the `__iadd__`
    method returns `self`.

#### *Usage of `AddableBingoCage` Class*

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='16-op-overloading/bingoaddable.py'
     language='python'
     line='11-26'
></div>

* `<1>` : Create an `AddableBingoCage` object `vowels` with the contents
    `AEIOU`.
* `<2>` : Pop one of the contents of `globe` object and verify that the
    popped value is one of the vowels.
* `<3>` : Check the number of contents of `globe` object.
* `<4>` : Create a second instance with three item `'XYZ'`.
* `<5>` : Confirm the number of contents of `globe3` object.
* `<6>` : Mixed type addition is not supported, so this code will raise
    the `TypeError` exception.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e/'
     branch='47cafc801a8e811b7cd7e7a5821f1ad4a65efa52'
     path='16-op-overloading/bingoaddable.py'
     language='python'
     line='35-49'
></div>

* `<1>` : Create an alias for the `globe` object.
* `<2>` : The `globe` has 4 items.
* `<3>` : Adds `globe2`, which is an object of the `AddableBingoCage` class,
    to the `globe` object.
* `<4>` : The augmented assignment operator `+=` can be used to add an iterable
    object to the `globe` object.
* `<5>` : The alias for the `globe` object, `globe_orig`, and `globe` object are
    the same objects.
    * There is no `__eq__` method on the `AddableBingoCage` class, so the `==`
        operator will compare the `id` of the two objects, which is the fallback
        behavior of the `==` operator.
* `<6>` : Trying to add a non-iterable object to the `globe` object will raise
    a `TypeError` exception.

## Chapter Summary

### Unary Operators

| Operator | Method       | Description    |
| -------- | ------------ | -------------- |
| `-`      | `__neg__`    | Negation       |
| `+`      | `__pos__`    | Plus           |
| `~`      | `__invert__` | Bitwise NOT    |
| `abs()`  | `__abs__`    | Absolute value |

### Infix Operators

| Operator     | Forward        | Reverse         | In-place        | Description                  |
| ------------ | -------------- | --------------- | --------------- | ---------------------------- |
| `+`          | `__add__`      | `__radd__`      | `__iadd__`      | Addition                     |
| `-`          | `__sub__`      | `__rsub__`      | `__isub__`      | Subtraction                  |
| `*`          | `__mul__`      | `__rmul__`      | `__imul__`      | Multiplication or repetition |
| `/`          | `__truediv__`  | `__rtruediv__`  | `__itruediv__`  | True division                |
| `//`         | `__floordiv__` | `__rfloordiv__` | `__ifloordiv__` | Floor division               |
| `%`          | `__mod__`      | `__rmod__`      | `__imod__`      | Modulo                       |
| `divmod()`   | `__divmod__`   | `__rdivmod__`   | `__idivmod__`   | Divmod                       |
| `**`,`pow()` | `__pow__`      | `__rpow__`      | `__ipow__`      | Exponentiation               |
| `@`          | `__matmul__`   | `__rmatmul__`   | `__imatmul__`   | Matrix multiplication        |
| `&`          | `__and__`      | `__rand__`      | `__iand__`      | Bitwise AND                  |
| `|`          | `__or__`       | `__ror__`       | `__ior__`       | Bitwise OR                   |
| `^`          | `__xor__`      | `__rxor__`      | `__ixor__`      | Bitwise XOR                  |
| `<<`         | `__lshift__`   | `__rlshift__`   | `__ilshift__`   | Bitwise left shift           |
| `>>`         | `__rshift__`   | `__rrshift__`   | `__irshift__`   | Bitwise right shift          |

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="python/fluent-python/chapter-16/add_radd.svg"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="Flowchart for computing <code>a + b</code> with <code>__add__</code> and <code>__radd__</code>."
        %}
    </div>
</div>

### Rich Comparison Operators

| Group    | Infix operator | Forward method call | Reverse method call | Fallback                |
| -------- | -------------- | ------------------- | ------------------- | ----------------------- |
| Equality | `a == b`       | `a.__eq__(b)`       | `b.__eq__(a)`       | `Return id(a) == id(b)` |
| Equality | `a != b`       | `a.__ne__(b)`       | `b.__ne__(a)`       | `Return not (a == b)`   |
| Ordering | `a > b`        | `a.__gt__(b)`       | `b.__lt__(a)`       | `Raise TypeError`       |
| Ordering | `a < b`        | `a.__lt__(b)`       | `b.__gt__(a)`       | `Raise TypeError`       |
| Ordering | `a >= b`       | `a.__ge__(b)`       | `b.__le__(a)`       | `Raise TypeError`       |
| Ordering | `a <= b`       | `a.__le__(b)`       | `b.__ge__(a)`       | `Raise TypeError`       |


## Further Reading

* [Why operators are useful](https://neopythonic.blogspot.com/2019/03/why-operators-are-useful.html)
* [Tuple ordering and deep comparisons in Python](https://treyhunner.com/2019/03/python-deep-comparisons-and-code-readability/)
* [3. Data model](https://docs.python.org/3/reference/datamodel.html)
* [`numbers` — Numeric abstract base classes : Implementing the arithmetic operations](https://docs.python.org/3/library/numbers.html#implementing-the-arithmetic-operations)

## Links

[^pythonorg__expressions_unary]:[6. Expressions : 6.6. Unary arithmetic and bitwise operations](https://docs.python.org/3/reference/expressions.html#unary-arithmetic-and-bitwise-operations)
[^pythonorg__datamodel_abs]:[3. Data model : `object.__abs__`](https://docs.python.org/3/reference/datamodel.html#object.__abs__)
[^pythonorg__pep465]:[PEP 465 – A dedicated infix operator for matrix multiplication](https://www.python.org/dev/peps/pep-0465/)
