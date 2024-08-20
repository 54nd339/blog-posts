---
title: OOP and SOLID - Four Pillars and Five Principles
description: Object-oriented programming models a system as objects that bundle data with behaviour. The four pillars — encapsulation, abstraction, inheritance, polymorphism — and the five SOLID principles that keep an object design from rotting.
date: 2024-08-20
draft: false
slug: /interview/oop
tags:
  - System Design
  - Interview Prep
  - OOP
---

Code that works and code that survives five years of changes are different achievements. **Object-oriented programming** is one way to aim for the second: model the system as **objects** that bundle data (fields) with the behaviour that acts on it (methods), so each piece is self-contained and the connections between pieces are explicit.

OOP rests on four pillars. **SOLID** is five principles for applying them without the design turning brittle. This covers both, with short Java examples.

## The four pillars

### Encapsulation

Bundle data and its methods in a class, and restrict direct access to the data — callers go through a public interface, so internals can change and invariants can be enforced.

```java
public class Person {
    private String name;
    private int age;

    public void setAge(int age) {
        if (age > 0) this.age = age;   // validation lives with the data
    }
    public int getAge() { return age; }
}
```

`person.age = -5` will not compile; the setter is the only way in, and it can refuse bad input.

### Abstraction

Expose *what* an object does, hide *how*. Callers depend on a small interface, not the implementation behind it.

```java
public interface Vehicle {
    void start();
    String getFuelType();
}

public class Car implements Vehicle {
    public void start() { /* ignition, fuel pump, starter motor... */ }
    public String getFuelType() { return "Petrol"; }
}
```

Code holding a `Vehicle` reference knows it can `start()` and nothing about engines.

### Inheritance

A subclass acquires the fields and methods of a superclass, establishing an "is-a" relationship and letting shared behaviour be written once.

```java
public class Animal {
    protected String name;
    public void eat() { System.out.println(name + " is eating."); }
    public void makeSound() { System.out.println("Some generic sound."); }
}

public class Dog extends Animal {
    @Override
    public void makeSound() { System.out.println(name + ": Woof!"); }
}
```

`Dog` gets `eat()` for free and replaces `makeSound()`.

### Polymorphism

"Many forms" — one call site, behaviour chosen by the object. Java has two kinds:

- **Overriding (runtime)** — a subclass replaces a superclass method; the actual object type decides which runs. `Animal a = new Dog(); a.makeSound();` prints "Woof".
- **Overloading (compile-time)** — several methods, same name, different parameters; the compiler picks by argument types. `add(int, int)` versus `add(double, double)`.

This is what lets a method take an `Animal[]` and work correctly for whatever concrete animals are in it.

## SOLID

Five principles, acronym coined by Michael Feathers from Robert C. Martin's work, for keeping object designs flexible.

### S — Single Responsibility

A class should have **one reason to change**. Mixing responsibilities means a change to one risks breaking the other, and the class is harder to test. A `UserEmailService` that both changes emails *and* validates address format is doing two jobs — split the validation into an `EmailValidator` it depends on.

### O — Open/Closed

Software entities should be **open for extension, closed for modification** — add behaviour without editing tested code.

```java
interface Shape { double area(); }

class Circle implements Shape {
    private double r;
    public double area() { return Math.PI * r * r; }
}

class AreaCalculator {
    double total(Shape[] shapes) {
        double t = 0;
        for (Shape s : shapes) t += s.area();   // never changes
        return t;
    }
}
```

Adding a `Triangle` means writing a new class that implements `Shape`. `AreaCalculator` is untouched. The alternative — a `switch` on shape type inside the calculator — has to be edited and re-tested for every new shape.

### L — Liskov Substitution

A subtype must be usable **anywhere its base type is expected**, without breaking the caller. The classic violation: `Ostrich extends Bird` overriding `fly()` to throw an exception. Any code that calls `fly()` on a `Bird` now breaks for ostriches. The fix is to model the capability, not force it into the hierarchy — a `Flyable` interface that only flying birds implement. LSP violations show up as `instanceof` checks and special cases scattered through the code.

### I — Interface Segregation

Clients should not be forced to depend on methods they do not use. A fat `Worker { work(); eat(); sleep(); }` interface forces `RobotWorker` to implement `eat()` and `sleep()` with empty bodies or exceptions. Split it into `Workable`, `Eatable`, `Sleepable`; the robot implements only `Workable`.

### D — Dependency Inversion

High-level modules and low-level modules should both depend on **abstractions**, not on each other.

```java
interface MessageService { void send(String msg); }

class NotificationService {
    private final MessageService service;
    NotificationService(MessageService service) { this.service = service; }  // injected
    void notify(String msg) { service.send(msg); }
}
```

`NotificationService` never names `EmailService` or `SmsService` — it takes a `MessageService` in its constructor. Swapping email for SMS is a change at the call site, and tests can pass a fake. This is dependency injection, the usual way DIP is realised.

## The principles are a direction, not a checklist

OOP's pillars give you the tools; SOLID points them at maintainability — one responsibility per class, extension without edits, substitutable subtypes, narrow interfaces, dependencies on abstractions. Applied literally to every class they produce their own kind of mess (an interface and a factory for everything), so the judgement is *where* rigidity is actually costing you and applying them there. The [design patterns](/citadel/interview/design-patterns) are the recurring shapes you reach for once SOLID tells you a seam is needed.
