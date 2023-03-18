---
layout: distill
title: Relationships
description: UML Class Diagram Relationships
# img: assets/img/12.jpg
# importance: 1
category: UML-diagram

toc:
  - name: Abstract
  - name: Associations
  - name: Aggregation
  - name: Composition
  - name: Differences between aggregation and composition
  - name: Generalization
  - name: Realization
  - name: Dependency
  - name: Inner Class
---

## Abstract

UML class diagrams are a popular tool for modeling object-oriented systems. They provide a graphical representation of classes, interfaces, and their relationships. In this article, I will describe the various types of relationships that can be represented in a UML class diagram.

## Associations

Assosiations represent relationships between two or more objects (instances of class) where one object is aware of the other object.

### Representation

* Assosiation is represented by a solid line between two objects.
* Direction indicates source and target object.
* Multiplicity indicates a number of source or target object.

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/association.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of association relationship"%}
    </div>
</div>

* A `Customer` places zero, one or more `Order`.
* An `Order` contains zero, one or more `OrderItem`

<div class='embed-local-src'
    path='association.puml'
    language='plantuml'
></div>

* Association relationship is represented by the following code in plantuml.
    ```plantuml
    SourceClass "Multiplicity" --> "Multiplicity" TargetClass
    ```

### Direction

* Direction of the line indicates source object and target object.
* The source object knows about the target object.
* Bidirectional can be possible if two objects know each other.

### Multiplicity

The following representations decorates source or target object in association relationship.

* `1` : represents one object.
* `0..1` : represents zero or one object.
* `*` : represents 0 or more objects.

The following representation is examples of derivations of the above representations.

* `2..4` : represents two, three or four objects.
* `0..*` : represents zero or more objects and is equivalant to `*`.

There are various terms that refer to the multiplicity.

* **Optional** : implies lower bound of 0.
* **Mandatory** : impilies lower bound of 1.
* **Single-valued** : implies an upper bound of 1.
* **Multivalued** : implies an upper bound of more than 1 (`*`).

## Aggregation

Aggregation is a special type of association. Aggregation represents **has-a** relationship between objects.

### Representation

* Aggregation relationship is represented by a diamond-shaped hollow arrowhead on the line that connects the two classes.
* The arrowhead points from the component class (the "part") to the container class (the "whole")

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/aggregation.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of aggregate relationship"%}
    </div>
</div>

* `Customer` **has** zero, one or more `Order`. (Aggregation)
* `Order` can belongs to one `Customer`.
* `Customer` has one `CustomerOrderHistory`. (Aggregation)
* `CustomerOrderHistory` can belongs to one `Customer`.

<div class='embed-local-src'
    path='aggregation.puml'
    language='plantuml'
></div>

* Aggregation relationship is represented by the following code in plantuml.
    ```plantuml
    WholeClass "Multiplicity" o-- "Multiplicity" PartClass
    ```

### Lifetime

* The instances of the "part" class can exist independently of the "whole" class.
* For example, `Order` can exist independently of `Customer` class.

## Composition

Composition is a stronger form of aggregation. Composition represents **part-of** relationship between objects.

### Representation

* Composition relationship is represented by a diamond-shaped filled arrowhead on the line that connects the two classes.
* The arrowhead points from the component class (the "part") to the container class (the "whole").

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/composition.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of composition relationship"%}
    </div>
</div>

* `Customer` is part of `Order` class.
    * `Order` has an attribute `customer: Customer` which is an instance of  `Customer` class.
* `Item` is part of `Order` class.
    * `Order` has an attribute `items: list[Item]` which is one or more instances of `Item` class.
* `Address` is part of `Customer` class.
    * `Customer` has an attribute `address: Address` which is an instance of `Address` class.

<div class='embed-local-src'
    path='composition.puml'
    language='plantuml'
></div>

### Lifetime

* The instances of the "part" class cannot exist independently of the "whole" class in composition relationship.
* When an `Order` object is destroyed, its contained `Customer` and `Item` objects are also destroyed.
* Similarly, when a `Customer` object is destroyed, its contained `Address` object is also destroyed.

## Differences between aggregation and composition

### Lifetime

Aggregation
: The part object can exist independently of the container object, meaning it has a longer lifetime than the container object.

Composition
: The lifetime of the part objects is dependent on the lifetime of the container object. If the container object is destroyed, all of its pars are also destroyed.

### Relationship strength

Aggregation
: The part objects in an aggregation relationship are usually created and managed independently of the container object.

Composition
: Composition is a stronger form of aggregation.
The part objects are typically created and managed by the container object.

### Cardinality (Multiplicity)

Aggregation
: The cardinality (i.e., the number of part objects that can be associated with a container object) can be many-to-many. A single part object can be associated with multiple container objects and vice versa.

Composition
: The cardinality is typically one-to-many, meaning that each part object can only be associated with a single container object.

### Navigation

Aggregation
: The part objects can be navigated to directly from other parts of the system, without going through the container object.

Composition
: The part objects are typically only accessible through the container object.

### Shared identity

Aggregation
: The part objects can have their own identity, independent of the container object.

Composition
: The part objects usually share the same identity as the container object, meaning that they are part of the same logical entity.

## Generalization

A generalization relationship is used to represent **inheritance** between classes, where a subclass inherits properties and behavior from a superclass.

### Representation

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/generalization.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of generalization relationship"%}
    </div>
</div>

* `Employee` is subclass of `Person`
* `Manager` is subclass of `Person`

<div class='embed-local-src'
    path='generalization.puml'
    language='plantuml'
></div>

* Generalization relationship is represented by the following code in plantuml.
    ```plantuml
    SuperClass <|-- SubClass
    ```

## Realization

A realization relationship is used to represent the implementation of an interface by a class.

### Representation

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/realization.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of realization relationship"%}
    </div>
</div>

* `Paypal` and `CreditCardGateway` classes implement an interface `PaymentGateway`.
* The realization relationship is represented in the diagram using a dashed line with a closed arrowhead pointing from the implementing class (`PayPal` and `CreditCardGateway`) to the interface (`PaymentGateway`).
* `Payment` can uses two different payment method `PayPal` and `CreditCardGateway` by common interface `PaymentGateway`

<div class='embed-local-src'
    path='realization.puml'
    language='plantuml'
></div>

* Realization relationship is represented by the following two type of code in plantuml.

    ```plaintext
    interface NameInterface {

    }

    class NameClass implements NameInterface{

    }
    ```

    ```plaintext
    interface NameInterface {

    }

    class NameClass{

    }

    NameInterface <|.. NameClass
    ```

## Dependency

A dependency relationship exists between two elements when changes to the definition of one element (the client) may affect the definition of the other element (the supplier).

### Representation

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/dependency.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of dependency relationship"%}
    </div>
</div>

* A dependency relationship is represented in a UML class diagram using a dashed arrow with an arrowhead pointing from the client to the supplier.
* The `DataAnalyzer` class depends on the `DataFetcher` interface in order to function correctly.

<div class='embed-local-src'
    path='dependency.puml'
    language='plantuml'
></div>

* Realization relationship is represented by the following two type of code in plantuml.

    ```plaintext
    DependentClass ..> IndependentClass
    ```

## Nested(Inner) Class

A nested class is a class that is defined inside another class. This can be useful for grouping related classes together, and for controlling the scope of the nested class.

We can encapsulate related functionality and make our code more modular and easier to maintain. Additionally, nested classes can help us to control access to certain fields and methods by limiting their scope to the enclosing class.

### Representation

<div class="column mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="softwaredesign/uml-diagram/relationships/nestedclass.svg" class="img-fluid rounded z-depth-1" zoomable=true caption="Example of nested class relationship"%}
    </div>
</div>

* `Transaction` class declared within the scope of `BankAccount` class.

<div class='embed-local-src'
    path='nestedclass.puml'
    language='plantuml'
></div>

* Nested class relationship is represented by the following two type of code in plantuml.
    ```plaintext
    InnerClass -+ OuterClass
    ```
