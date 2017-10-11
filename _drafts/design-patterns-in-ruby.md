---
layout: post
title:  "OO Design Patterns in Ruby"
date:   2017-10-09
categories: software
tags: ruby design-patterns
excerpt_separator: <!--more-->
---

These are my notes on OO design patterns from reading the following sources:
Head First OOP design patterns
https://github.com/piscolomo/ruby-patterns/blob/master/README.md
https://github.com/nslocum/design-patterns-in-ruby
https://github.com/kamranahmedse/design-patterns-for-humans
https://bogdanvlviv.github.io/posts/ruby/patterns/design-patterns-in-ruby.html

OO design patterns can be divided into 3 categories - creational, structural, and behavioral.
<!--more-->

Creational
- Abstract Factory
- Builder
- Factory Method
- Prototype
- Singleton

Structural
- Adapter
- Bridge
- Composite
- Decorator
- Facade
- Flyweight
- Proxy

Behavioral
- Chain of responsibility
- Command
- Interpreter
- Iterator
- Mediator
- Memento
- Observer
- State
- Strategy
- Template Method
- Visitor

Pattern of patterns
- MVC

### OO Principles

- Use composition over inheritance
  - RubberDuck IS A Duck -> this can use inheritance
  - RubberDuck HAS A NoFlyBehavior -> this can use composition
- Encapsulate what varies
- Program to interfaces, not implementation
- Strive for loosely coupled designs between objects that interact like subject and owner in Observer pattern.
- Classes should be open for extension, but closed for modification (Open-Closed principle).
- Depend on abstraction, not on concrete classes (aka dependency inversion principle).
- Only talk to your friend, not 2nd tier friends (law of demeter).




### Creational

#### Abstract Factory
A factory of factories. Provides an interface allowing a client to create families of related or dependent objects without specifying their concrete classes.

UML from Ross + check HF
Example?

#### Builder
Separates object construction from its representation. Separate the construction of a complex object from its representation so that the same construction processes can create different representations.

UML from Ross
Example?

#### Factory Method
Defines an interface for creating an object but lets subclasses decide which class to instantiate.
(Key point - subclasses decide which concrete classes to create)

UML Ross
Example?

#### Prototype
A fully initialized instance to be copied or cloned. Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype. Used when creation is expensive and cloning an instance then modifying it is faster.

#### Singleton
Ensures a class has one and only one instance and provides a global point of access to it. Becareful in multi-threaded applications as it's easy to have race conditions during initial instantiation creating more than one instance - use a lock or synchronize.
Ruby has a Singleton module.

UML
Example? HF
In Ruby use Singleton module



### Structural

#### Adapter
Converts (wraps) interface of a class into another interface the clients expect. Let's classes work together that couldn't because of different interfaces.

UML
Example? HF


#### Bridge
Bridge pattern is about preferring composition over inheritance. Implementation details are pushed from a hierarchy to another object with a separate hierarchy.

#### Composite
Compose objects into tree structures to represent part-whole hiearchies. Lets clients treat collections of objects and individual objects uniformly.

UML
Example? HF

#### Decorator
Wraps an object to provide new behavior by attaching additional responsibilities to the object dynamically. It provides a flexible alternative to subclassing for extending functionality.
Ruby has a forwardable module that might be useful depending on the circumstances.

UML
Example? HF

#### Facade
Provides a unified interface to a set interfaces in a subsystem. It defines a higher level interface that makes the subsystem easier to use i.e. simplifies the interface of a set of classes.

UML
Example? HF

#### Flyweight
It is used to minimize memory usage or computational expenses by sharing as much as possible with similar objects.

#### Proxy
Provides a surrogate or placeholder for another object to control access to it. There is remote proxy and virtual proxy. Remote proxy is when you're remotely controlling an object running on a different machine. Virtual proxy allow us to delay the creation of an object until necessary like the image placeholder while the actual image is still being downloaded in an activity feed.

UML
Example? HF



### Behavioral

#### Chain of responsibility
It helps building a chain of objects. Request enters from one end and keeps going from object to object till it finds the suitable handler.

#### Command
Encapsulates a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.

UML
Example? HF

#### Interpreter
Provides an interpreter to deal with an abstract language. Using classes we can understand the inputs for parsing them.

UML
Example? HF

#### Iterator
Provides a way to traverse a collection of objects without exposing its underlying implementation.
Ruby has the Enumerable module.

UML
Example? HF

#### Mediator
Adds a third party object (called mediator) to control the interaction between two objects (called colleagues). It helps reduce the coupling between the classes communicating with each other. Because now they don't need to have the knowledge of each other's implementation.

#### Memento
It's about capturing and storing the current state of an object in a manner that it can be restored later on in a smooth manner.

#### Observer
Defines a one-to-many dependency between objects that allows the many objects to be notified when state of the one object changes.
Ruby has Observable module that implements this pattern.

UML
Example? HF

#### State
Encapsulates state-based behaviors and uses delegation to switch between behaviors. In other words, allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

UML
Example? HF

#### Strategy
Defines a family of algorithms (or interchangeable behaviors), encapsulates each one, and makes them interchangeable. It uses delegation to decide which one to use and lets the algorith vary independently from clients that use it.

UML
Example? HF

#### Template Method
Defines the skeleton of an algorithm in an operation, deferring some steps to subclasses. Lets subclasses decide how to implement steps in an algorithm without changing the algorithm's structure.

UML
Example? HF Recipe for tea/coffee

#### Visitor
Lets you add further operations to objects without having to modify them.
