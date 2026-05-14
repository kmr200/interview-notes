# Design Patterns

## What is a Design Pattern?

A design pattern is a proven, reusable solution to a common software design problem. It is not a class or library you include directly — it is a conceptual approach that can be implemented in different ways across different languages.

**Advantages:**
- Reduce complexity by providing ready-made abstractions for common problems.
- Improve communication between developers through shared vocabulary.
- Standardize solution structure across modules and components.
- Enable reuse of effective, battle-tested approaches.

**Disadvantages:**
- Blindly following a pattern can introduce unnecessary complexity.
- Using a pattern without a real need complicates the project for no benefit.

---

## Main Characteristics of a Pattern

| Characteristic     | Description                                              |
|--------------------|----------------------------------------------------------|
| **Name**           | A unique identifier for the pattern.                     |
| **Purpose**        | The intent — what the pattern is trying to achieve.      |
| **Problem**        | The specific problem the pattern addresses.              |
| **Solution**       | The approach the pattern suggests for the given context. |
| **Participants**   | The components involved in the pattern.                  |
| **Consequences**   | The trade-offs and effects of applying the pattern.      |
| **Implementation** | A possible way to realize the pattern in code.           |

---

## Types of Design Patterns

### 1. Fundamental Patterns
Basic building blocks that form the foundation for other patterns. Most design patterns incorporate these in some way.

### 2. Creational Patterns
Abstract the object creation process, making a system independent of how objects are created, composed, and represented.
- **Class-based** — use inheritance to vary the class of object created.
- **Object-based** — delegate instantiation to another object.

### 3. Structural Patterns
Define how objects and classes are composed to form larger structures. They streamline development by simplifying relationships between entities.

### 4. Behavioral Patterns
Define how objects communicate and collaborate, increasing flexibility in interactions.

---

## Fundamental Patterns

| Pattern                 | Description                                                                                                                                                                                                               |
|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Delegation**          | An entity expresses certain behavior outwardly but delegates the actual execution to an associated object.                                                                                                                |
| **Functional Design**   | Ensures each entity has a single responsibility with minimal side effects on others.                                                                                                                                      |
| **Immutable Interface** | Creates an object that cannot be modified after construction.                                                                                                                                                             |
| **Interface**           | A general method for structuring entities to make them easier to understand and use.                                                                                                                                      |
| **Marker Interface**    | Tags an object by implementing (or not implementing) a specific interface. Replaced by annotations in modern languages.                                                                                                   |
| **Property Container**  | Allows additional properties to be added to an entity via a container, rather than extending the entity itself.                                                                                                           |
| **Event Channel**       | A centralized channel for events. Uses a proxy for subscription and another for publishing, decoupled from actual publishers/subscribers. A subscriber can receive events from multiple sources through a single channel. |

---

## Creational Patterns

| Pattern              | Description                                                                                                                     |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------|
| **Abstract Factory** | Provides an interface for creating families of related objects without specifying their concrete classes.                       |
| **Builder**          | Separates the construction of a complex object from its representation, allowing the same process to produce different results. |
| **Factory Method**   | Delegates object creation to subclasses, letting the program work with abstractions rather than concrete classes.               |
| **Prototype**        | Creates new objects by cloning an existing instance rather than using constructors.                                             |
| **Singleton**        | Ensures a class has only one instance and provides a global access point to it.                                                 |

---

## Structural Patterns

| Pattern       | Description                                                                                                                                     |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| **Adapter**   | Allows two objects with incompatible interfaces to work together by wrapping one with a compatible interface.                                   |
| **Bridge**    | Decouples an abstraction from its implementation so that both can vary independently.                                                           |
| **Composite** | Composes objects into tree structures to represent part-whole hierarchies, letting clients treat individual objects and compositions uniformly. |
| **Decorator** | Dynamically adds behavior to an object without modifying its class or using inheritance.                                                        |
| **Facade**    | Provides a simplified, unified interface to a set of interfaces in a subsystem.                                                                 |
| **Flyweight** | Shares a common instance among many consumers to reduce memory usage, while appearing unique to each consumer.                                  |
| **Proxy**     | An intermediary that controls access to another object, potentially adding restrictions or additional behavior.                                 |

---

## Behavioral Patterns

| Pattern                     | Description                                                                                                                           |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| **Chain of Responsibility** | Passes a request along a chain of handlers, each deciding to process it or forward it to the next.                                    |
| **Command**                 | Encapsulates an action and its parameters as an object, allowing parameterization, queuing, and undoable operations.                  |
| **Interpreter**             | Defines a grammar for a language and provides an interpreter to handle expressions in that language.                                  |
| **Iterator**                | Provides sequential access to elements of a collection without exposing its internal structure.                                       |
| **Mediator**                | Centralizes communication between multiple objects, reducing direct dependencies between them.                                        |
| **Memento**                 | Captures and externalizes an object's internal state so it can be restored later without violating encapsulation.                     |
| **Observer**                | Establishes a one-to-many dependency so that when one object changes state, all its dependents are notified automatically.            |
| **State**                   | Allows an object to alter its behavior when its internal state changes, appearing to change its class.                                |
| **Strategy**                | Defines a family of interchangeable algorithms, encapsulates each one, and makes them substitutable without changing client code.     |
| **Template Method**         | Defines the skeleton of an algorithm in a base class, deferring specific steps to subclasses without changing the overall structure.  |
| **Visitor**                 | Adds new operations to an object hierarchy without modifying the existing classes, by separating the operation into a visitor object. |