# Interview Notes

A collection of Java and Spring interview preparation notes, ordered from foundational to most complex — each document assumes knowledge from those above it.

| # | Document | Description | Requires |
|---|----------|-------------|----------|
| 1 | [Pillars of OOP](docs/Pillars%20of%20OOP.md) | Encapsulation, inheritance, polymorphism, abstraction | — |
| 2 | [Java Core](docs/Java-Core.md) | Access modifiers, keywords, data types, classes, interfaces | Pillars of OOP |
| 3 | [JVM](docs/JVM.md) | Class loading, memory areas (heap/stack), garbage collection | Java Core |
| 4 | [Java 8](docs/Java-8.md) | Lambdas, streams, functional interfaces, Optional | Java Core |
| 5 | [Collections](docs/Collections.md) | List, Set, Map, Queue — implementations and trade-offs | Java Core, Java 8 |
| 6 | [Concurrency](docs/Concurrency.md) | Java Memory Model, threads, synchronization, locks | JVM, Java Core |
| 7 | [Serialization](docs/Serialization.md) | Object serialization/deserialization, the `Serializable` interface | Java Core |
| 8 | [Input/Output Streams](docs/Input-Output-Streams.md) | Java IO vs NIO, byte/character streams, buffers, channels | Java Core, Serialization |
| 9 | [Design Patterns](docs/Design-Patterns.md) | Creational, structural, and behavioral patterns | Pillars of OOP, Java Core |
| 10 | [Databases](docs/Databases.md) | Relational DB concepts, SQL, normalization, transactions, indexing | — |
| 11 | [JDBC](docs/JDBC.md) | JDBC API, connections, statements, result sets | Java Core, Databases |
| 12 | [Hibernate](docs/Hibernate.md) | ORM, entity mapping, sessions, HQL, caching | JDBC, Databases, Design Patterns |
| 13 | [Spring Core](docs/Spring-Core.md) | Beans, IoC container, dependency injection, AOP | Java Core, Design Patterns |
| 14 | [REST Basics](docs/REST-Basics.md) | REST constraints, HTTP methods, status codes | — |
| 15 | [REST with Spring](docs/REST-With-Spring.md) | Spring MVC/REST annotations, request lifecycle, dispatcher servlet | Spring Core, REST Basics |
| 16 | [REST API Documentation](docs/REST-API-Documentation.md) | Swagger, OpenAPI, Postman | REST with Spring |
| 17 | [Microservices](docs/Microservices.md) | Monolithic vs microservices, service communication, patterns | REST with Spring, Databases, Design Patterns |