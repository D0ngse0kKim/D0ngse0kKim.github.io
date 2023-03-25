---
layout: distill
title: Chapter 10
description: Design Patterns with First-Class Functions
# img: assets/img/12.jpg
# importance: 1
category: Fluent-Python

toc:
  - name: "Case Study: Refactoring Strategy"
  - name: Decorator-Enhanced Strategy Pattern
  - name: The Command Pattern
---

## Abstract

This is summary of chapter 10 of [Fluent Python](https://www.oreilly.com/library/view/fluent-python-2nd/9781492056348/).
In this chapter, examples of refactoring an implementation of the Strategy pattern, using Python decorators, are presented.

## Case Study: Refactoring Strategy

### Classic Strategy

#### Definition of Strategy Pattern

> Define a family of algorithms, encapsulate each one, and make them interchangeable.
> Strategy lets the algorithm vary independently from clients that use it.

#### Requirements

1. Customers with 1,000 or more fidelity points get a global 5% discount per order. (class `FidelityPromo`)
2. A 10% discount is applied to each line item with 20 or more units in the same order. (class `BulkItemPromo`)
3. Orders with at least 10 distinct items get a 7% global discount. (class `LargeOrderPromo`)
4. Assume that only one discount may be applied to an order.

#### UML diagrams

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-10/strategy.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of Strategy pattern"%}
    </div>
</div>

#### Implements

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/classic_strategy.py'
     language="python"
     line='31-105'
></div>

#### Example

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/classic_strategy.py'
     language="python"
     line='7-25'
></div>

* `# <3>` : Passing an instance of `FidelitiPromo` class, `FidelityPromo()`

### Function-Oriented Strategy

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy.py'
     language="python"
     line='31-95'
></div>

* `# <1>` : The type annotation of `promotion` takes the role of the previous `class Promotion(ABC)`.
* `# <3>` : `class Promotion(ABC)` is omitted by `# <1>`.
* `# <4>` : Class implements of each promotion is replaced by functions `fidelity_promo()`, `bulk_item_promo()`, and `large_order_promo()`.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy.py'
     language="python"
     line='7-25'
></div>

* `# <2>` : Instead of passing an instance of `Promotion` class, function `fidelity_promo` is passed.

### Choosing the Best Strategy: Simple Approach

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy_best.py'
     language="python"
     line='29-41'
></div>

To find the best promotion from each promotions automatically, list of whole promotion and function which finds the best promotion is added.

* `# <1>` : each promotion function added manually.

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy_best.py'
     language="python"
     line='19-24'
></div>

### Finding Strategies in a Module

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy_best2.py'
     language="python"
     line='41-55'
></div>

* `# <1>` : imports each promotion functions
* `# <2>` : `globals()` returns a dictionary representing the current global symbol table. This dictionary includes these key and value:
    * `'fidelity_promo' : fidelity_promo`
    * `'bulk_item_promo' : bulk_item_promo`
    * `'large_order_promo' : large_order_promo`


<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy_best3.py'
     language="python"
     line='41-55'
></div>

* `inspect.getmembers(promotions, inspect.isfunction)` inspects `promotions` module and collects only function from the module.

## Decorator-Enhanced Strategy Pattern

<div class='embed-github-src'
     repo='fluentpython/example-code-2e'
     branch='01e717b60a9a14bc1bca4bffd915f7ed2c637e5e'
     path='10-dp-1class-func/strategy_best4.py'
     language="python"
     line='42-88'
></div>

* `# <1>` : defines a list of `Callable[[Order], Decimal]`.
* `# <2>` : defines a decorator which appends decorated function to list `promos[]`.
* `# <3>` : finds the best promotion.
* `# <4>` : appends `fidelity` function to list `promos[]`.

## The Command Pattern

### Design Pattern Command

> Encapsulate a request as an object, thereby letting you parameterize clients
> with different requests, queue or log requests, and support undoable operations.

### UML Diagram

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="python/fluent-python/chapter-10/command.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of Command pattern"%}
    </div>
</div>


### Details

* Command: The Command is an interface that declares the method(s) to be executed. This interface may contain one or more methods, depending on the requirements of the system.
* ConcreteCommand: ConcreteCommand implements the Command interface and contains the actual implementation of the command. It stores the receiver object and invokes the corresponding operation(s) on the receiver.
* Receiver: The Receiver is the object that performs the actual action when the command is executed.
* Invoker: The Invoker is an object that requests the command to be executed.
* Invoking an operation is decoupled from implementation of the operation.
* Each ConcreteCommand like `OpenCommand`, `PasteCommand` and `MacroCommand` is implemented by class.

### Implement ConcreteCommand by Callable

```python
class MacroCommand:
    """A command that executes a list of commands"""

    def __init__(self, commands):
        self.commands = list(commands)

    def __call__(self):
        for command in self.commands:
            command()
```
