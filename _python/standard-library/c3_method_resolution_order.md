---
layout: distill
title: C3 Method Resolution Order
description: Details of method resolution order in complex class hierarchy
# img: assets/img/12.jpg
# importance: 1
category: Standard-Library

toc:
  - name: Abstract
  - name: Basic Definitions
  - name: An example of the nonexistence of linearization
  - name: The C3 Method Resolution Order
  - name: MRO calculation examples
  - name: Bad Method Resolution Orders
  - name: MRO Calculation Codes
  - name: Links
---

## Abstract

In this article, the method resolution order introduced in the Python 2.3
[^MRO-in-Python-2-3] will be discussed.

## Basic Definitions

1. Method Resolution Order(MRO)
: Given a class `C` in a complicated multiple inheritance hierarchy, it is
a non-trivial task to specify the order in which methods are overridden,
i.e. to specify the order of the ancestors of `C`.
**Method Resolution Order(MRO)** defines the order in which a method is
searched for in a classes hierarchy.

2. A list of the ancestors of a class `C` and the linearization of `C`
: The list of the ancestors of a class `C`, including the class itself, ordered
from the nearest ancestor to the furthest in MRO, is called
**the class precedence list** or **the linearization of `C`**.

3. The Method Resolution Order (MRO) and Linearization
: The Method Resolution Order (MRO) is the set of rules that construct the linearization. In the Python literature, the idiom "the MRO of `C`" is also used as a synonymous for the linearization of the class `C`.

4. Example of MRO and Linearization
: For instance, in the case of single inheritance hierarchy, if `C` is a
subclass of `C1`, and `C1` is a subclass of `C2`, then the linearization of `C`
is simply the list `[C, C1, C2]`.
However, with multiple inheritance hierarchies, the construction of the
linearization is more cumbersome, since it is more difficult to construct
a linearization that respects local precedence ordering and monotonicity.

5. Monotonicity on MRO
: I will discuss the local precedence ordering later, but I can give
the definition of monotonicity here. A MRO is **monotonic** when the following
is true:
    1. If `C1` precedes `C2` in the linearization of `C`, then `C1` precedes
    `C2` in the linearization of any subclass of `C`.
    2. Otherwise, the innocuous operation of deriving a new class could change
    the resolution order of methods, potentially introducing very subtle bugs.
    Examples where this happens will be shown later.

6. Existance of linearization
: Not all classes admit a linearization. There are cases, in complicated
hierarchies, where it is not possible to derive a class such that
its linearization respects all the desired properties.

## An example of the nonexistence of linearization

Consider the following source code defining classes `O`, `X`, `Y`, `A`, `B` and
`C`. In this case, it is not possible to derive a new class `C` from class `A`
and `B` due to the following reasons:

1. The definition of class `A` has `X` preceding `Y`, but in the definition of
    class `B`, `Y` precedes `X`.
2. As a result, the method resolution order would be ambiguous in class `C`.

Python 2.3 raises an exception in this situation
(`TypeError: MRO conflict among bases Y, X`) preventing the creation of
ambiguous hierarchies by naive programmers.
On the other hand, Python 2.2 instead does not raise an exception, but chooses
an *ad hoc* ordering (`CABXYO` in this case).

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-3">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/OXYAB.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes O, X, Y, A and B." %}
    </div>
    <div class="col-sm-9">
        <pre class="language-python">
            <code class="language-python hljs">O = object

class X(O): pass
class Y(O): pass

class A(X,Y): pass
class B(Y,X): pass

class C(A,B): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes O, X, Y, A and B.</figcaption>
    </div>
</div>

## The C3 Method Resolution Order

### Notations

This chapter describes the C3 Method Resolution Order. In order to illustrates
class hierarchy for a certain method resolution order, the following shortcut
notation is used to indicate the list of classes `[C1, C2, ..., CN]`

$$
C_1\;C_2\;\ldots\;C_N
$$

In this notation, the *head* of the list means its first element:

$$
\textrm{head} = C_1
$$

The *tail* of the list means the rest element of the list:

$$
\textrm{tail} = C_2 \ldots C_N
$$

In order to denote the sum of the lists `[C] + [C1, C2, ..., CN]`,
the following notation will be used.

$$
C + (C_1\;C_2\;\ldots\;C_N) = C\;C_1\;C_2\;\ldots\;C_N
$$

### How the MRO works in the Python 2.3

Consider a class `C` in a complex multiple inheritance hierarchy, with `C`
inheriting from the base classes `B1`, `B2`, ..., `BN`. We want to compute
the linearization $L\left[C\right]$ of the class `C`. The rule is the following.

The linearization of `C`, denotes as
$L\left[C\left(B_1\;B_2\;\ldots\;B_N\right)\right]$ is the sum of

1. $C$ and
2. the $merge$ of

    1. the linearizations of the parents ($L\left[C\left(B_1\;B_2\;\ldots\;B_N\right)\right]$)
    2. the list of the parents ($B_1\;B_2\;\ldots\;B_N$).

This can be denote by the following equations.

$$
\begin{split}
&L\left[C\left(B_1\;B_2\;\ldots\;B_N\right)\right] \\
&= C + merge\left(L[B_1]\;L[B_2]\;\ldots\;L[B_N],\;B_1\;B_2\;\ldots\;B_N\right)
\end{split}
$$

In order to calculate $merge$, the following procedures are needed:

1. Take the head of the first list, idx.e $L[B_1][0]$
2. If this head is not in the tail of any of the other lists, then add
    the first list to the linearization of $C$ and remove it from the lists in
    the $merge$ operation.
    Otherwise, look at the head of the next list and take it, if it is a good
    head.
3. Then repeat the operation until all the class are removed or it is
    impossible to find good heads.

This prescription ensures that the $merge$ operation preserves the ordering,
if the ordering can be preserved.
On the other hand, if the order cannot be preserved (as in the example of
serious order disagreement discussed above) then the $merge$ cannot be computed.

## MRO calculation examples

### Simple Example of Computation in case of Single Inheritance

The following computation of the merge is trivial if $C$ has only one parent
(single inheritance).

$$
\begin{split}
L[C] = L[C(B)] &= C + merge\left(L[B],\;B\right) \\
&=C + L[B]
\end{split}
$$

In special case of $C = object$, the linearization of $C$(equals $object$) is
represented by the following equation because $object$ has no parent.

$$
L[object] = object
$$

### First Complex example of multiple inheritance

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-4">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/OFEDCBA.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes O, F, E, D, C, B and A." %}
    </div>
    <div class="col-sm-6">
        <pre class="language-python">
            <code class="language-python hljs">O = object

class F(O): pass
class E(O): pass
class D(O): pass

class C(D,F): pass
class B(D,E): pass

class A(B,C): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes O, F, E, D, C, B and A.</figcaption>
    </div>
</div>

The linearization of $O$, $D$, $E$, $F$(represented by $L[O]$, $L[D]$, $L[E]$,
$L[F]$) are derived like the following equation:

$$
\begin{split}
L[O] &= O \\
L[D] &= L[D(O)] = D + merge(L[O], O) = D\;L[O] = DO\\
L[E] &= L[E(O)] = E + merge(L[O], O) = E\;L[O] = EO\\
L[F] &= L[F(O)] = F + merge(L[O], O) = F\;L[O] = FO
\end{split}
$$

The linearization of $B$ (represented by $L[B]$ or $L[B(D\;E)]$ in a more
explicit form) can be computed using the following equation:

$$
\begin{split}
L[B] = L[B(D\;E)] &= B + merge[L[D],\;L[E],\;DE] \\
&= B + merge[DO,\;EO, DE] \\
&= BD + merge[O,\;EO, E] \\
&= BDE + merge[O,\;O] \\
&= BDEO
\end{split}
$$

The linearization of $C$ (represented by $L[C]$ or $L[C(D\;F)]$ in a more
explicit form) can be computed using the following equation:

$$
\begin{split}
L[C] = L[C(D\;F)] &= C + merge[L[D],\;L[F],\;DF] \\
&= C + merge[DO,\;FO, DF] \\
&= CD + merge[O,\;FO, F] \\
&= CDF + merge[O,\;O] \\
&= CDFO
\end{split}
$$

Now we can compute the linearization of $A$ (represented by $L[A]$ or
$L[A(B\;C)]$ in a more explicit form) using the following equation:

$$
\begin{split}
L[A] = L[A(B\;C)] &= A + merge[L[B],\;L[C],\;BC]\\
&= A + merge[BDEO,\;CDFO,\;BC] \\
&= AB + merge[DEO,\;CDFO,\;C] \\
&= ABC + merge[DEO,\;DFO] \\
&= ABCD + merge[EO,\;FO] \\
&= ABCDE + merge[O,\;FO] \\
&= ABCDEF + merge[O,\;O] \\
&= ABCDEFO \\
\end{split}
$$

### Second Complex example of multiple inheritance

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-4">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/OFEDCBA2.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes O, F, E, D, C, B and A." %}
    </div>
    <div class="col-sm-6">
        <pre class="language-python">
            <code class="language-python hljs">O = object

class F(O): pass
class E(O): pass
class D(O): pass

class C(D,F): pass
class B(E,D): pass

class A(B,C): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes O, X, Y, A and B.</figcaption>
    </div>
</div>

The linearization of $O$, $D$, $E$, $F$ (represented by $L[O]$, $L[D]$, $L[E]$,
$L[F]$) are derived like the following equation:

$$
\begin{split}
L[O] &= O \\
L[D] &= L[D(O)] = D + merge(L[O], O) = D L[O] = DO\\
L[E] &= L[E(O)] = E + merge(L[O], O) = E L[O] = EO\\
L[F] &= L[F(O)] = F + merge(L[O], O) = F L[O] = FO
\end{split}
$$

The linearization of $B$ (represented by $L[B]$ or $L[B(E\;D)]$ in a more
explicit form) can be computed using the following equation:

$$
\begin{split}
L[B] = L[B(E\;D)] &= B + merge[L[E],\;L[D],\;ED] \\
&= B + merge[EO,\;DO, ED] \\
&= BE + merge[O,\;DO, D] \\
&= BED + merge[O,\;O] \\
&= BEDO
\end{split}
$$

The linearization of $C$ (represented by $L[C]$ or $L[C(D\;F)]$ in a more
explicit form) can be computed using the following equation:

$$
\begin{split}
L[C] = L[C(D\;F)] &= C + merge[L[D],\;L[F],\;DF] \\
&= C + merge[DO,\;FO, DF] \\
&= CD + merge[O,\;FO, F] \\
&= CDF + merge[O,\;O] \\
&= CDFO \\
\end{split}
$$

Now we can compute the linearization of $A$ (represented by $L[A]$ or
$L[A(B\;C)]$ in a more explicit form) using the following equation:

$$
\begin{split}
L[A] = L[A(B\;C)] &= A + merge[L[B],\;L[C],\;BC]\\
&= A + merge[BEDO,\;CDFO,\;BC] \\
&= AB + merge[EDO,\;CDFO,\;C] \\
&= ABE + merge[DO,\;CDFO,\;C] \\
&= ABEC + merge[DO,\;DFO] \\
&= ABECD + merge[O,\;FO] \\
&= ABECDF + merge[O,\;O] \\
&= ABECDFO
\end{split}
$$

The results of $L[A(B\;C)] = ABECDFO$ can be checked by obtaining MRO from
`A.mro()` method in Python interpreter.
The following are examples of interpreter output:

```python
>>> O = object
>>> class F(O): pass
...
>>> class E(O): pass
...
>>> class D(O): pass
...
>>> class C(D,F): pass
...
>>> class B(E,D): pass
...
>>> class A(B,C): pass
...
>>> A.mro()
[
    <class '__main__.A'>, <class '__main__.B'>,
    <class '__main__.E'>, <class '__main__.C'>,
    <class '__main__.D'>, <class '__main__.F'>,
    <class 'object'>
]
```

### An example of the nonexistence of linearization

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-3">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/OXYAB.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes O, X, Y, A and B." %}
    </div>
    <div class="col-sm-9">
        <pre class="language-python">
            <code class="language-python hljs">O = object

class X(O): pass
class Y(O): pass

class A(X,Y): pass
class B(Y,X): pass

class C(A,B): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes O, X, Y, A and B.</figcaption>
    </div>
</div>

The linearization of $O$, $X$, $Y$ (represented by $L[O]$, $L[X]$, $L[Y]$) are
derived like the following equation:

$$
\begin{split}
L[O] &= O \\
L[X] &= L[X(O)] = X + merge(L[O], O) = X L[O] = XO\\
L[Y] &= L[Y(O)] = Y + merge(L[O], O) = Y L[O] = YO
\end{split}
$$

The linearization of $A$ (represented by $L[A]$ or $L[A(X\;Y)]$ in a more
explicit form) can be computed using the following equation:

$$
\begin{split}
L[A] = L[A(X\;Y)] &= A + merge[L[X],\;L[Y],\;XY] \\
&= A + merge[XO,\;YO, XY] \\
&= AX + merge[O,\;YO, Y] \\
&= AXY + merge[O,\;O] \\
&= AXYO
\end{split}
$$

Now we can compute the linearization of $B$ (represented by $L[B]$ or
$L[B(Y\;X)]$ in a more explicit form) using the following equation:

$$
\begin{split}
L[B] = L[B(Y\;X)] &= B + merge[L[Y],\;L[X],\;YX]\\
&= B + merge[YO,\;XO,\;YX] \\
&= BY + merge[O,\;XO,\;X] \\
&= BYX + merge[O,\;O] \\
&= BYXO
\end{split}
$$

Now we can compute the linearization of $C$ (represented by $L[C]$ or
$L[C(A\;B)]$ in a more explicit form) using the following equation:

$$
\begin{split}
L[C] = L[C(A\;B)] &= C + merge[L[A],\;L[B],\;AB]\\
&= C + merge[AXYO,\;BYXO,\;AB]\\
&= CA + merge[XYO,\;BYXO,\;B]\\
&= CAB + merge[XYO,\;YXO]
\end{split}
$$

$merge[XYO,\;YXO]$ cannot be calculated since $X$ is in the tail of $YXO$
whereas $Y$ is in the tail of $XYO$.
Therefore there are no good heads and the C3 algorithm stops. Python 2.3 raises
an error and refuses to create the class $C$ like the following results of
the python interpreter execution.

```python
>>> O = object
>>> class X(O): pass
...
>>> class Y(O): pass
...
>>> class A(X,Y): pass
...
>>> class B(Y,X): pass
...
>>> class C(A,B): pass
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Cannot create a consistent method resolution
order (MRO) for bases X, Y
```

## Bad Method Resolution Orders

### Class Defining Built-in Function

`class type(name, bases, dict, **kwds)`
: Details of this built-in function can be found in this article[^Built-in-Functions__type].
* `name` : a string of the class name and becomes the `__name__` attribute.
* `bases` : a tuple contains the base lasses and becomes the `__bases__`
attribute.
* `dict` : a dictionary containts attribute and method definitions for
the class body

### An Example of Ambiguous Hierarchies.

Using this `type` function, the following codes are defining a class $F$, $E$
and $G$.

```python
>>> F=type('Food',(),{'remember2buy':'spam'})
>>> F.remember2buy
'spam'
>>> E=type('Eggs',(F,),{'remember2buy':'eggs'})
>>> E.remember2buy
'eggs'
>>> G=type('GoodFood',(F,E),{}) # under Python 2.3 this is an error!
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Cannot create a consistent method resolution
order (MRO) for bases Food, Eggs
```

<div class="row row-cols-2 align-items-center">
    <div class="col-sm">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/FEG.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes F, E and G." %}
    </div>
</div>

The linearization of $G$ (represented by $L[G]$ or $L[G(F\;E)]$ in a more
explicit form) can be calculated by the following equation:

$$
\begin{split}
L[F] = L[F(O)] &= F + merge(L[O],\;O) = F + merge(O,\;O) = FO\\
L[E] = L[E(F)] &= E + merge(L[F],\;F) = E + merge(O,\;F) = EFO
\end{split}
$$

$$
\begin{split}
L[G] = L[G(F\;E)] &= G + merge(L[F],\;L[E],\;FE)\\
&= G + merge(FO,\;EFO,\;FE)
\end{split}
$$

In C3 algorithm, $merge(FO,\;EFO,\;FE)$ cannot be calculated because $F$ is
the tail of $EFO$ and $E$ is the tail of $FE$.
This is the reason of `TypeError: Cannot create a consistent method resolution order (MRO) for bases Food, Eggs`.

On the other hands, class $G$ can be defined in Python 2.2 without `TypeError`
and the linearization of $G$ ($L[G]$) can be calculated and retrived from
`G.mro()`. The results of `G.mro()` represents the following linearization of $G$.

```python
Python 2.2.3 (#1, Jul 17 2023, 01:41:16)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> F=type('Food',(),{'remember2buy':'spam'})
>>> F.remember2buy
'spam'
>>> E=type('Eggs',(F,),{'remember2buy':'eggs'})
>>> E.remember2buy
'eggs'
>>> G=type('GoodFood',(F,E),{}) # No error occurs on this definition in Python 2.2
>>> G.remember2buy
'eggs'
>>> G.mro()
[<class '__main__.GoodFood'>, <class '__main__.Eggs'>, <class '__main__.Food'>, <type 'object'>]
```

$$
L[G] = GEFO
$$

As a general rule, hierarchies such as the previous one should be avoided,
because it is unclear whether $F$ should override $E$ or vice versa.
Python 2.3 solves this ambiguity by raising an exception in the creation of
class $G$, effectively preventing the programmer from generating ambiguous
hierarchies.
The reason for that is that the C3 algorithm fails when the
$merge(FO,\;EFO,\;FE)$.

The correct solution is to design a non-ambiguous hierarchy, idx.e. to derive $G$
from $E$ and $F$ (with the more specific class $G$ placed at first)
instead of deriving $G$ from $F$ and $E$.
In this case the MRO will be $GEF$ without any doubt.

### An Example of Error-prone Hierarchies

Python 2.3 forces the programmer to write good hierarchies (or, at least, less error-prone ones).

On a related note, let me point out that the Python 2.3 algorithm is smart
enough to recognize obvious mistakes, as the duplication of classes in
the list of parents:

```python
Python 3.11.1 (main, Feb  4 2023, 17:05:54) [Clang 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> class A(object): pass
...
>>> class C(A,A): pass
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: duplicate base class A
```

The reason of `TypeError` on this code can be checked by the following
equations:

$$
\begin{split}
L[A(O)] &= A + merge(L[O], O) = A + merge(O, O) = AO\\
L[C(A\;A)] &= C + merge(L[A],\;L[A],\;AA) \\
&= C + merge(AO, AO, AA)
\end{split}
$$

The results of linearization of $C$ (represented by $L[C(A\;A)]$) yields
$merge(AO, AO, AA)$ which has no good head because $A$ is the tail of $AA$.


On the other hands, Python 2.2 (both for classic classes and new style classes)
in this situation, would not raise any exception.

Finally, I would like to point out a lesson we have learned from this example:

Despite the name, the MRO determines the resolution order of attributes,
not only methods.

```python
Python 2.2.3 (#1, Jul 17 2023, 01:41:16)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> class A(object): pass
...
>>> class C(A,A): pass # No error occurs in this definition!
...
>>> C.mro()
[<class '__main__.C'>, <class '__main__.A'>, <type 'object'>]
```

### An example without Monotonicity

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-4">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/CABD.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes C, A, B, and D." %}
    </div>
    <div class="col-sm-6">
        <pre class="language-python">
            <code class="language-python hljs">C = object

class A(C): pass
class B(C): pass

class D(A,B): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes C, A, B, and D.</figcaption>
    </div>
</div>

$$
\begin{split}
L[A(C)] &= A + merge(L[C], C) = A + merge(C,C) = AC\\
L[B(C)] &= B + merge(L[C], C) = B + merge(C,C) = BC\\
L[D(A\;B)] &= D + merge(L[A],\;L[B],\;AB)\\
&= D + merge(AC,\;BC,\;AB)\\
&= DA + merge(C,\;BC,\;B)\\
&= DAB + merge(C,\;C)\\
&= DABC
\end{split}
$$

Python 2.2 yields MRO as $DABC$ as described above equations.

```python
Python 2.2.3 (#1, Jul 17 2023, 02:59:21)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> C = object
>>> class A(C): pass
...
>>> class B(C): pass
...
>>> class D(A,B): pass
...
>>> D.mro()
[
    <class '__main__.D'>, <class '__main__.A'>,
    <class '__main__.B'>, <type 'object'>
]
```

Python 3.11 yields MRO as $DABC$ as described above equations.

```python
Python 3.11.1 (main, Feb  4 2023, 17:05:54) [Clang 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> C = object
>>> C.mro()
[<class 'object'>]
>>> class A(C): pass
...
>>> class B(C): pass
...
>>> A.mro()
[<class '__main__.A'>, <class 'object'>]
>>> B.mro()
[<class '__main__.B'>, <class 'object'>]
>>> class D(A,B): pass
...
>>> D.mro()
[
    <class '__main__.D'>, <class '__main__.A'>,
    <class '__main__.B'>, <class 'object'>
]
```

On the other hands, Python 2.1 yields $L[D] = DACBC$.

### A complex example

#### Class Definition

<div class="row row-cols-2 align-items-center">
    <div class="col-sm-6">
        {% include figure.html path="python/standard-library/c3_method_resolution_order/ABCDEK1K2K3Z.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="UML diagram of classes A, B, C, D, E, K1, K2, K3, and Z" %}
    </div>
    <div class="col-sm-6">
        <pre class="language-python">
            <code class="language-python hljs">O = object

class A(O): pass
class B(O): pass
class C(O): pass
class D(O): pass
class E(O): pass

class K1(A,B,C): pass
class K2(D,B,E): pass
class K3(D,A):   pass

class Z(K1,K2,K3): pass
            </code>
        </pre>
        <figcaption class="caption">Source code of classes A, B, C, D, E, K1, K2, K3, and Z.</figcaption>
    </div>
</div>

#### The results of C3 MRO

The results of linearization of class $A$, $B$, $C$, $D$ and $E$
(represented by $L[A(O)]$, $L[B(O)]$, $L[C(O)]$, $L[D(O)]$, $L[E(O)]$) can be
derived by the following equations with C3 MRO.

$$
\begin{split}
L[O] &= O + merge(L[O,\; O]) = O\\
L[A(O)] &= A + merge(L[O],\; O) = A + merge(O,\;O) = AO\\
L[B(O)] &= B + merge(L[O],\; O) = B + merge(O,\;O) = BO\\
L[C(O)] &= C + merge(L[O],\; O) = C + merge(O,\;O) = CO\\
L[D(O)] &= D + merge(L[O],\; O) = D + merge(O,\;O) = DO\\
L[E(O)] &= E + merge(L[O],\; O) = E + merge(O,\;O) = EO\\
\end{split}
$$

The results of linearization of $K_1$, $K_2$, and $K_3$
(represented by $L[K_1(A,\;B,\;C)]$, $L[K_2(D,\;B,\;E)]$, $L[K_3(D,\;A)]$)
can be derived by the following equations.

$$
\begin{split}
L[K_1(A,\;B,\;C)] &= K_1 + merge(L[A],\;L[B],\;L[C],\;ABC)\\
&= K_1 + merge(AO,\;BO,\;CO,\;ABC)\\
&= K_1A + merge(O,\;BO,\;CO,\;BC)\\
&= K_1AB + merge(O,\;O,\;CO,\;C)\\
&= K_1ABC + merge(O,\;O,\;O)\\
&= K_1ABCO
\end{split}
$$

$$
\begin{split}
L[K_2(D,\;B,\;E)] &= K_2 + merge(L[D],\;L[B],\;L[E],\;DBE)\\
&= K_2 + merge(DO,\;BO,\;EO,\;DBE)\\
&= K_2D + merge(O,\;BO,\;EO,\;BE)\\
&= K_2DB + merge(O,\;O,\;EO,\;E)\\
&= K_2DBE + merge(O,\;O,\;O)\\
&= K_2DBEO
\end{split}
$$

$$
\begin{split}
L[K_3(D,\;A)] &= K_3 + merge(L[D],\;L[A],\;DA)\\
&= K_3 + merge(DO,\;AO,\;DA)\\
&= K_3D + merge(O,\;AO,\;A)\\
&= K_3DA + merge(O,\;O)\\
&= K_3DAO
\end{split}
$$

The results of linearization of $Z$ (represented by $L[Z(K_1,\;K_2,\;K_3)]$)
can be derived by the following equations.

$$
\begin{split}
L[Z(K_1,\;K_2,\;K_3)] &= Z + merge(L[K_1],\;L[K_2],\;L[K_3],\;K_1K_2K_3)\\
&= Z + merge(K_1ABCO,\;K_2DBEO,\;K_3DAO,\;K_1K_2K_3)\\
&= ZK_1 + merge(ABCO,\;K_2DBEO,\;K_3DAO,\;K_2K_3)\\
&= ZK_1K_2 + merge(ABCO,\;DBEO,\;K_3DAO,\;K_3)\\
&= ZK_1K_2K_3 + merge(ABCO,\;DBEO,\;DAO)\\
&= ZK_1K_2K_3D + merge(ABCO,\;BEO,\;AO)\\
&= ZK_1K_2K_3DA + merge(BCO,\;BEO,\;O)\\
&= ZK_1K_2K_3DAB + merge(CO,\;EO,\;O)\\
&= ZK_1K_2K_3DABC + merge(O,\;EO,\;O)\\
&= ZK_1K_2K_3DABCE + merge(O,\;O,\;O)\\
&= ZK_1K_2K_3DABCEO
\end{split}
$$

To summarize the results of linearization of each classes by C3 MRO are
represented by the following equations.

$$
\begin{split}
L[A(O)] &= AO \\
L[B(O)] &= BO \\
L[C(O)] &= CO \\
L[D(O)] &= DO \\
L[E(O)] &= EO \\
L[K_1(A,\;B,\;C)] &= K_1ABCO \\
L[K_2(D,\;B,\;E)] &= K_2DBEO \\
L[K_3(D,\;A)] &= K_3DAO \\
L[Z(K_1,\;K_2,\;K_3)] &= ZK_1K_2K_3DABCEO
\end{split}
$$

#### Source Code for checking MRO

These results can be checked by the following Python code on Python 2.2 or
a later version.
This is because the Python 2.1 or earlier versions do not have `object` type.

```python
O = object
class A(O): pass
class B(O): pass
class C(O): pass
class D(O): pass
class E(O): pass

class K1(A,B,C): pass
class K2(D,B,E): pass
class K3(D,A):   pass

class Z(K1,K2,K3): pass

A.mro()
B.mro()
C.mro()
D.mro()
E.mro()
K1.mro()
K2.mro()
K3.mro()
Z.mro()
```

#### Results in Python 2.2

```python
Python 2.2.3 (#1, Jul 18 2023, 04:37:32)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> O = object
>>> class A(O): pass
...
>>> class B(O): pass
...
>>> class C(O): pass
...
>>> class D(O): pass
...
>>> class E(O): pass
...
>>> class K1(A,B,C): pass
...
>>> class K2(D,B,E): pass
...
>>> class K3(D,A):   pass
...
>>> class Z(K1,K2,K3): pass
...
>>> A.mro()
[<class '__main__.A'>, <type 'object'>]
>>> B.mro()
[<class '__main__.B'>, <type 'object'>]
>>> C.mro()
[<class '__main__.C'>, <type 'object'>]
>>> D.mro()
[<class '__main__.D'>, <type 'object'>]
>>> E.mro()
[<class '__main__.E'>, <type 'object'>]
>>> K1.mro()
[
    <class '__main__.K1'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <type 'object'>
]
>>> K2.mro()
[
    <class '__main__.K2'>, <class '__main__.D'>,
    <class '__main__.B'>, <class '__main__.E'>,
    <type 'object'>
]
>>> K3.mro()
[
    <class '__main__.K3'>, <class '__main__.D'>,
    <class '__main__.A'>, <type 'object'>
]
>>> Z.mro()
[
    <class '__main__.Z'>, <class '__main__.K1'>,
    <class '__main__.K3'>, <class '__main__.A'>,
    <class '__main__.K2'>, <class '__main__.D'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <class '__main__.E'>, <type 'object'>
]
```

To summarize the results of executing the above source code,
the linearizations of each class are represented by the following equations.

$$
\begin{split}
L[A(O)] &= AO \\
L[B(O)] &= BO \\
L[C(O)] &= CO \\
L[D(O)] &= DO \\
L[E(O)] &= EO \\
L[K_1(A,\;B,\;C)] &= K_1ABCO \\
L[K_2(D,\;B,\;E)] &= K_2DBEO \\
L[K_3(D,\;A)] &= K_3DAO \\
L[Z(K_1,\;K_2,\;K_3)] &= ZK_1K_3AK_2DBCEO
\end{split}
$$

The result of linearization of class $Z$($L[Z(K_1,\;K_2,\;K_3)]$)
is different from the results of C3 MRO
($L[Z(K_1,\;K_2,\;K_3)] = ZK_1K_2K_3DABCEO$).

It is clear that this linearization is wrong, since $A$ comes before $D$
whereas in the linearization of $K_3$ $A$ comes after $D$.
In other words, in $K_3$, methods derived by $D$ override methods derived by $A$,
but in $Z$, which still is a subclass of $K_3$, methods derived by $A$ override
methods derived by $D$.

This is a violation of monotonicity. Moreover, the Python 2.2 linearization of
$Z$ is also inconsistent with local precedence ordering,
since the local precedence list of the class $Z$ is `[K1, K2, K3]`
($K_2$ precedes $K_3$),
whereas in the linearization of $Z$, $K_2$ follows $K_3$.
These problems explain why the 2.2 rule has been dismissed in favor of
the C3 MRO rule.

#### Results in Python 2.3

```python
Python 2.3.7 (#1, Jul 18 2023, 04:48:37)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> O = object
>>> class A(O): pass
...
>>> class B(O): pass
...
>>> class C(O): pass
...
>>> class D(O): pass
...
>>> class E(O): pass
...
>>> class K1(A,B,C): pass
...
>>> class K2(D,B,E): pass
...
>>> class K3(D,A):   pass
...
>>> class Z(K1,K2,K3): pass
...
>>> A.mro()
[<class '__main__.A'>, <type 'object'>]
>>> B.mro()
[<class '__main__.B'>, <type 'object'>]
>>> C.mro()
[<class '__main__.C'>, <type 'object'>]
>>> D.mro()
[<class '__main__.D'>, <type 'object'>]
>>> E.mro()
[<class '__main__.E'>, <type 'object'>]
>>> K1.mro()
[
    <class '__main__.K1'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <type 'object'>
]
>>> K2.mro()
[
    <class '__main__.K2'>, <class '__main__.D'>,
    <class '__main__.B'>, <class '__main__.E'>,
    <type 'object'>
]
>>> K3.mro()
[
    <class '__main__.K3'>, <class '__main__.D'>,
    <class '__main__.A'>, <type 'object'>
]
>>> Z.mro()
[
    <class '__main__.Z'>, <class '__main__.K1'>,
    <class '__main__.K2'>, <class '__main__.K3'>,
    <class '__main__.D'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <class '__main__.E'>, <type 'object'>
]
```

To summarize the results of executing the above source code,
the linearizations of each class are represented by the following equations.

$$
\begin{split}
L[A(O)] &= AO \\
L[B(O)] &= BO \\
L[C(O)] &= CO \\
L[D(O)] &= DO \\
L[E(O)] &= EO \\
L[K_1(A,\;B,\;C)] &= K_1ABCO \\
L[K_2(D,\;B,\;E)] &= K_2DBEO \\
L[K_3(D,\;A)] &= K_3DAO \\
L[Z(K_1,\;K_2,\;K_3)] &= ZK_1K_2K_3DABCEO
\end{split}
$$

The result of linearization of class $Z$($L[Z(K_1,\;K_2,\;K_3)]$)
is same as the results of C3 MRO
($L[Z(K_1,\;K_2,\;K_3)] = ZK_1K_2K_3DABCEO$).

#### Results in Python 3.11

Results in Python 3.11 are same as the results in Python 2.3

The result of linearization of class $Z$($L[Z(K_1,\;K_2,\;K_3)]$)
is same as the results of C3 MRO
($L[Z(K_1,\;K_2,\;K_3)] = ZK_1K_2K_3DABCEO$).

```python
Python 3.11.1 (main, Feb  4 2023, 17:05:54) [Clang 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> O = object
>>> class A(O): pass
...
>>> class B(O): pass
...
>>> class C(O): pass
...
>>> class D(O): pass
...
>>> class E(O): pass
...
>>> class K1(A,B,C): pass
...
>>> class K2(D,B,E): pass
...
>>> class K3(D,A):   pass
...
>>> class Z(K1,K2,K3): pass
...
>>> A.mro()
[<class '__main__.A'>, <class 'object'>]
>>> B.mro()
[<class '__main__.B'>, <class 'object'>]
>>> C.mro()
[<class '__main__.C'>, <class 'object'>]
>>> D.mro()
[<class '__main__.D'>, <class 'object'>]
>>> E.mro()
[<class '__main__.E'>, <class 'object'>]
>>> K1.mro()
[
    <class '__main__.K1'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <class 'object'>
]
>>> K2.mro()
[
    <class '__main__.K2'>, <class '__main__.D'>,
    <class '__main__.B'>, <class '__main__.E'>,
    <class 'object'>
]
>>> K3.mro()
[
    <class '__main__.K3'>, <class '__main__.D'>,
    <class '__main__.A'>, <class 'object'>
]
>>> Z.mro()
[
    <class '__main__.Z'>, <class '__main__.K1'>,
    <class '__main__.K2'>, <class '__main__.K3'>,
    <class '__main__.D'>, <class '__main__.A'>,
    <class '__main__.B'>, <class '__main__.C'>,
    <class '__main__.E'>, <class 'object'>
]
```

## MRO Calculation Codes for Python 2.2

```python
#<mro.py>

"""C3 algorithm by Samuele Pedroni (with readability enhanced by me)."""

class __metaclass__(type):
    "All classes are metamagically modified to be nicely printed"
    __repr__ = lambda cls: cls.__name__

class ex_2:
    "Serious order disagreement" #From Guido
    class O: pass
    class X(O): pass
    class Y(O): pass
    class A(X,Y): pass
    class B(Y,X): pass
    try:
        class Z(A,B): pass #creates Z(A,B) in Python 2.2
    except TypeError:
        pass # Z(A,B) cannot be created in Python 2.3

class ex_5:
    "My first example"
    class O: pass
    class F(O): pass
    class E(O): pass
    class D(O): pass
    class C(D,F): pass
    class B(D,E): pass
    class A(B,C): pass

class ex_6:
    "My second example"
    class O: pass
    class F(O): pass
    class E(O): pass
    class D(O): pass
    class C(D,F): pass
    class B(E,D): pass
    class A(B,C): pass

class ex_9:
    "Difference between Python 2.2 MRO and C3" #From Samuele
    class O: pass
    class A(O): pass
    class B(O): pass
    class C(O): pass
    class D(O): pass
    class E(O): pass
    class K1(A,B,C): pass
    class K2(D,B,E): pass
    class K3(D,A): pass
    class Z(K1,K2,K3): pass

def merge(seqs):
    print '\n\nCPL[%s]=%s' % (seqs[0][0],seqs),
    res = []; i=0
    while 1:
      nonemptyseqs=[seq for seq in seqs if seq]
      if not nonemptyseqs: return res
      i+=1; print '\n',i,'round: candidates...',
      for seq in nonemptyseqs: # find merge candidates among seq heads
          cand = seq[0]; print ' ',cand,
          nothead=[s for s in nonemptyseqs if cand in s[1:]]
          if nothead: cand=None #reject candidate
          else: break
      if not cand: raise "Inconsistent hierarchy"
      res.append(cand)
      for seq in nonemptyseqs: # remove cand
          if seq[0] == cand: del seq[0]

def mro(C):
    "Compute the class precedence list (mro) according to C3"
    return merge([[C]]+map(mro,C.__bases__)+[list(C.__bases__)])

def print_mro(C):
    print '\nMRO[%s]=%s' % (C,mro(C))
    print '\nP22 MRO[%s]=%s' % (C,C.mro())

print_mro(ex_9.Z)

#</mro.py>
```

The results of `print_mro(ex_9.Z)` in Python 2.2.3 is described in below codes.

```python
Python 2.2.3 (#1, Jul 19 2023, 03:00:12)
[GCC 9.4.0] on linux5
Type "help", "copyright", "credits" or "license" for more information.
>>> print_mro(ex_9.Z)


CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[A]=[[A], [O, <type 'object'>], [O]]
1 round: candidates...   A
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[B]=[[B], [O, <type 'object'>], [O]]
1 round: candidates...   B
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[C]=[[C], [O, <type 'object'>], [O]]
1 round: candidates...   C
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[K1]=[[K1], [A, O, <type 'object'>], [B, O, <type 'object'>], [C, O, <type 'object'>], [A, B, C]]
1 round: candidates...   K1
2 round: candidates...   A
3 round: candidates...   O   B
4 round: candidates...   O   O   C
5 round: candidates...   O
6 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[D]=[[D], [O, <type 'object'>], [O]]
1 round: candidates...   D
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[B]=[[B], [O, <type 'object'>], [O]]
1 round: candidates...   B
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[E]=[[E], [O, <type 'object'>], [O]]
1 round: candidates...   E
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[K2]=[[K2], [D, O, <type 'object'>], [B, O, <type 'object'>], [E, O, <type 'object'>], [D, B, E]]
1 round: candidates...   K2
2 round: candidates...   D
3 round: candidates...   O   B
4 round: candidates...   O   O   E
5 round: candidates...   O
6 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[D]=[[D], [O, <type 'object'>], [O]]
1 round: candidates...   D
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[<type 'object'>]=[[<type 'object'>], []]
1 round: candidates...   <type 'object'>

CPL[O]=[[O], [<type 'object'>], [<type 'object'>]]
1 round: candidates...   O
2 round: candidates...   <type 'object'>

CPL[A]=[[A], [O, <type 'object'>], [O]]
1 round: candidates...   A
2 round: candidates...   O
3 round: candidates...   <type 'object'>

CPL[K3]=[[K3], [D, O, <type 'object'>], [A, O, <type 'object'>], [D, A]]
1 round: candidates...   K3
2 round: candidates...   D
3 round: candidates...   O   A
4 round: candidates...   O
5 round: candidates...   <type 'object'>

CPL[Z]=[[Z], [K1, A, B, C, O, <type 'object'>], [K2, D, B, E, O, <type 'object'>], [K3, D, A, O, <type 'object'>], [K1, K2, K3]]
1 round: candidates...   Z
2 round: candidates...   K1
3 round: candidates...   A   K2
4 round: candidates...   A   D   K3
5 round: candidates...   A   D
6 round: candidates...   A
7 round: candidates...   B
8 round: candidates...   C
9 round: candidates...   O   E
10 round: candidates...   O
11 round: candidates...   <type 'object'>
MRO[Z]=[Z, K1, K2, K3, D, A, B, C, E, O, <type 'object'>]

P22 MRO[Z]=[Z, K1, K3, A, K2, D, B, C, E, O, <type 'object'>]
```

## Links

[^MRO-in-Python-2-3]:[the method resolution order introduced in the Python 2.3](https://www.python.org/download/releases/2.3/mro/)
[^Built-in-Functions__type]:[Built-in Functions : `class type(name, bases, dict, **kwds)`](https://docs.python.org/3/library/functions.html#type)
