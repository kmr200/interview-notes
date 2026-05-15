# JVM

## What is the JVM responsible for?

- Loading, verifying, and executing bytecode.
- Providing a runtime environment for execution.
- Managing memory and garbage collection.

The Java Virtual Machine executes Java code independently of the OS, enabling portability. It handles two categories of data types:

- **Primitive types** — integers, floating points, etc.
- **Reference types** — objects, arrays, interfaces.

### Primitives

`long` and `double` are 64-bit types and occupy **two memory slots**. All other types (`int`, `short`, `byte`, `char`, `boolean`) are extended to 32-bit. `boolean` is stored as a byte where `0 = false` and `1 = true`.

### Reference Types

The JVM supports class, array, and interface types, all referring to dynamically created instances on the heap.

---

## ClassLoader

The ClassLoader dynamically loads Java classes as needed, following a **delegation model**: before attempting to load a class itself, a ClassLoader delegates the request to its parent.

**Three built-in ClassLoaders (in delegation order):**

1. **Bootstrap ClassLoader** — loads core Java libraries (`<JAVA_HOME>/jre/lib`).
2. **Extensions ClassLoader** — loads classes from extension directories (`<JAVA_HOME>/jre/lib/ext`).
3. **System ClassLoader** — loads user-defined classes from the `CLASSPATH`.

**ClassLoader process:**

1. **Loading** — finds and imports the class binary.
2. **Linking** — verifies correctness, prepares static fields, and optionally resolves symbolic references.
3. **Initialization** — executes static initializers and static field assignments.

**Custom ClassLoaders** enable dynamic loading/unloading of classes at runtime, which powers script engines, AOP frameworks, and encrypted class loading.

---

## Runtime Data Areas

The JVM allocates several memory regions at runtime:

| Area                      | Description                                                                                                |
|---------------------------|------------------------------------------------------------------------------------------------------------|
| **PC Register**           | Per-thread; stores the address of the currently executing instruction.                                     |
| **Java Stack**            | Per-thread; holds method frames (local variables, operand stack, etc.). Can be fixed or dynamically sized. |
| **Heap**                  | Shared across all threads; stores all objects and arrays. Managed by the garbage collector.                |
| **Method Area**           | Shared; stores class-level structures — constants, field/method metadata, and bytecode.                    |
| **Runtime Constant Pool** | Per-class; contains class-specific constants and symbolic references resolved at runtime.                  |
| **Native Method Stack**   | Per-thread; supports execution of native (non-Java) methods.                                               |

---

## Frames

A **frame** is created for each method invocation and discarded when the method returns. It holds all the data needed for that method's execution:

- **Local Variables** — stores method parameters and local values. `long` and `double` each occupy two slots.
- **Operand Stack** — a LIFO stack used to store and manipulate intermediate computation results.
- **Dynamic Linking** — resolves symbolic references (e.g., method names) to actual memory addresses at runtime.

**Method termination:**
- **Normal return** — passes a result (if any) back to the caller.
- **Exception return** — exits abruptly if an exception is thrown and not handled within the method.

---

## Execution Engine

The Execution Engine processes bytecode using two strategies:

- **Interpreter** — translates and executes bytecode instructions one at a time. Starts up quickly but is slow for repeated calls since each method is re-interpreted every time.
- **JIT (Just-In-Time) Compiler** — monitors execution and compiles frequently called ("hot") methods into native machine code, which then runs directly without further interpretation.

**JIT components:**

| Component                       | Role                                                                  |
|---------------------------------|-----------------------------------------------------------------------|
| **Profiler**                    | Monitors execution to identify hot spots (frequently called methods). |
| **Intermediate Code Generator** | Produces an intermediate representation of the bytecode.              |
| **Code Optimizer**              | Applies optimizations to the intermediate representation.             |
| **Target Code Generator**       | Produces the final native machine code.                               |

**Garbage Collector** — part of the execution engine; automatically reclaims memory occupied by objects that are no longer reachable, freeing the developer from manual memory management.