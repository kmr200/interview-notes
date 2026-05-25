# Java Core

## Access Modifiers

| Modifier                  | Accessibility                          |
|---------------------------|----------------------------------------|
| `private`                 | Only within the class                  |
| default (package-private) | Within the package (no keyword needed) |
| `protected`               | Within the package and in subclasses   |
| `public`                  | Everywhere                             |

---

## `final` Modifier

Can be applied to variables, method arguments, fields, methods, and classes:

- **Class** — cannot be extended
- **Method** — cannot be overridden
- **Field** — cannot change its value after initialization
- **Method argument / local variable** — cannot be reassigned

---

## Default Values of Variables

| Type      | Default value |
|-----------|---------------|
| `byte`    | `(byte) 0`    |
| `short`   | `(short) 0`   |
| `int`     | `0`           |
| `long`    | `0L`          |
| `float`   | `0f`          |
| `double`  | `0d`          |
| `char`    | `\u0000`      |
| `boolean` | `false`       |
| Objects   | `null`        |

---

## Logical Operators

| Symbol | Description         |
|--------|---------------------|
| `&`    | Logical AND         |
| `&&`   | Short-circuit AND   |
| `\|`   | Logical OR          |
| `\|\|` | Short-circuit OR    |
| `^`    | Logical XOR         |
| `!`    | Logical unary NOT   |
| `&=`   | AND with assignment |
| `\|=`  | OR with assignment  |
| `^=`   | XOR with assignment |
| `==`   | Equals              |
| `!=`   | Not equals          |
| `?:`   | Ternary operator    |

> **Ternary operator** — a shorthand for simple if-else:
> `condition ? expression1 : expression2`
> Both expressions must be of the same type.

---

## `abstract` Modifier

- An **abstract class** cannot be instantiated; it can only be used as a base for other classes.
- An **abstract method** has no implementation. A class with at least one abstract method must itself be abstract.
- Abstract classes allow defining a blueprint while providing shared default behavior for subclasses.

---

## Interfaces

- Defined with the `interface` keyword — a fully abstract contract.
- All methods are implicitly `public`.
- Fields are implicitly `public static final`.
- Since Java 8, interfaces can have `default` methods with implementations.
- Marker interfaces (e.g., `Cloneable`) have no methods and serve as type tags only.
- Interface methods cannot be `final` because they are implicitly abstract and must be implemented elsewhere.
- Interfaces allow type structures without strict hierarchy.

**Why can't an interface method be `final`?**  
`final` prevents overriding, but interface methods are implicitly abstract — they *must* be overridden somewhere.

**Which has the highest level of abstraction?**  
Interface > Abstract class > Class.

---

## Abstract Class vs Interface

|                      | Abstract Class                | Interface                                         |
|----------------------|-------------------------------|---------------------------------------------------|
| Instantiation        | Cannot be instantiated        | Cannot be instantiated                            |
| Multiple inheritance | Single class inheritance only | A class can implement multiple interfaces         |
| Fields               | Any field types               | Implicitly `public static final`                  |
| Methods              | Can have concrete methods     | All abstract (or `default`/`static` since Java 8) |
| Relationship         | "is-a"                        | "can-do" / capability                             |
| Purpose              | Shared partial implementation | Contract / capability                             |

Use an abstract class when building a hierarchy of closely related classes that share implementation. Prefer interfaces in all other cases.

---

## Accessing `private` Members

- Inside the class: direct access, no restrictions.
- Nested classes: full access to all private members of the enclosing class.
- From outside: only via methods provided by the class (e.g., `getX()`, `setX()`).
- Via Reflection API:

```java
Victim victim = new Victim();
Field field = Victim.class.getDeclaredField("field");
field.setAccessible(true);
int fieldValue = (int) field.get(victim);
```

---

## Initialization Blocks

Blocks of code inside a class, outside any method or constructor.

- **Static** — runs once when the class is loaded by the class loader.
- **Non-static** — runs before the constructor each time an object is created.
- Multiple blocks execute in source order.
- Can throw checked exceptions if all constructors declare them.
- It Can be used in anonymous classes.

**Order of execution (with inheritance):**

```
Parent static block(s)
→ Child static block(s)
→ Grandchild static block(s)
→ Parent non-static block(s) → Parent constructor
→ Child non-static block(s) → Child constructor
→ Grandchild non-static block(s) → Grandchild constructor
```

---

## `static` Modifier

Can be applied to: fields, methods, nested classes, initialization blocks, and import members.

**Static vs instance members:**

|                               | Static           | Instance          |
|-------------------------------|------------------|-------------------|
| Belongs to                    | The class itself | A specific object |
| Accessible without instance   | Yes              | No                |
| Can access non-static members | No               | Yes               |
| Can use `this` / `super`      | No               | Yes               |

**Can static methods be overridden?**  
No. Method selection happens at compile time (early binding), so the parent's method always runs. They *can* be overloaded.

**Can non-static methods overload static ones?**  
Yes. The static version belongs to the class; the non-static version belongs to an instance.

**Where can static/non-static fields be initialized?**

- Static fields: at declaration, in static blocks, or in non-static blocks.
- Non-static fields: at declaration, in non-static blocks, or in constructors.

---

## Method Overriding

**Rules:**
- Access modifier cannot be *narrowed* (but can be widened).
- Return type can only be *narrowed* (covariant return — a subtype of the parent's return type).
- Changing argument type/count/order creates an *overload*, not an override.
- `throws` clause: can omit, add subclasses of declared exceptions, or add `RuntimeException`s. Order doesn't matter.

**Accessing overridden parent methods:**

```java
super.method();
```

**Can a method be both `abstract` and `static`?**  
No — the compiler throws `"Illegal combination of modifiers: 'abstract' and 'static'"`. `abstract` means "implemented elsewhere"; `static` means "accessible by class name".

---

## Types of Classes in Java

- **Top-level class** — standard class; can be `abstract` or `final`
- **Interface**
- **Enum**
- **Nested classes:**
    - Static nested class
    - Member inner class
    - Local inner class
    - Anonymous inner class

### Nested Classes

A nested class should only exist to serve its enclosing class. If it's useful elsewhere, it should be a top-level class. Nested classes have access to all private members of the outer class (but not vice versa).

| Type | Has reference to outer instance | Can have static members | When to use |
|---|---|---|---|
| Static nested | No | Yes | Doesn't need outer class state |
| Member inner | Yes | No | Needs outer class state, used in multiple places |
| Local inner | Yes (effectively final vars) | No | Needed only within one method |
| Anonymous | Yes (context-dependent) | No | One-off, single-use, no new methods needed |

**Accessing outer class fields from inner class:**

```java
Outer.this.field
```

**Anonymous class limitations:**
- Used only at the point of creation.
- Cannot declare new methods (no named type to call them through).
- Implements only the methods of its interface or superclass.

---

## `assert`

Used to verify assumptions during development and testing. Throws `AssertionError` on failure.

```java
assert booleanExpression;
assert booleanExpression : "message";
```

- Disabled in production by default.
- Do not use inside `assert` expressions anything that changes program state.

---

## Stack vs Heap

> "Primitives are always on the stack" — **not quite true.**

- Primitive *local variables* are on the stack.
- Primitive *fields of an object* are stored in the heap (with the object).
- All objects created with `new` are stored in the heap.

---

## Parameter Passing

Java is **always pass-by-value**:

- Primitives: a copy of the value is passed.
- References: a copy of the *reference* is passed. You can mutate the object through the copy, but you cannot change what the original reference points to.

---

## Garbage Collection

Garbage collection frees heap memory by removing objects with no live references. The JVM decides when to run it.

**GC algorithms in HotSpot VM:**

| GC          | Notes                                            |
|-------------|--------------------------------------------------|
| Serial GC   | Simple, single-threaded. `-XX:+UseSerialGC`      |
| Parallel GC | Multi-threaded throughput. `-XX:+UseParallelGC`  |
| CMS GC      | Low-pause, concurrent. `-XX:+UseConcMarkSweepGC` |
| G1 GC       | Server-optimized, replaces CMS. `-XX:+UseG1GC`   |

---

## String Pool

- A special region in the heap storing string literals and interned strings.
- Possible because `String` is immutable (Flyweight pattern).
- Using `""` syntax: JVM checks the pool first, reuses if found, else adds.
- Using `new String(...)`: always creates a new heap object. Use `intern()` to add it to the pool.

> The String pool lives in the heap (moved from PermGen in Java 7) and is therefore subject to garbage collection.
> However, string literals are referenced by the class's constant pool
> and are never collected as long as their class loader is alive.
> Only strings added to the pool programmatically via intern()
> can be garbage collected once no other references to them remain.

---

## `finalize()`

Called by GC before an object is destroyed. It Can be overridden to release resources — but:

- Not guaranteed to be called.
- Runs in a separate `Finalizer` thread; if it hangs, it blocks the finalization queue.
- An object can "resurrect" itself inside `finalize()` — but only once. This is an antipattern.
- **Prefer** `try-with-resources` and `Cleaner` in modern code. Deprecated since Java 9.

---

## Type Casting

| Type                | Description                                                                        |
|---------------------|------------------------------------------------------------------------------------|
| Identity            | Same type — always allowed, automatic                                              |
| Widening primitive  | Smaller → larger (e.g., `byte → int`). Implicit, safe                              |
| Narrowing primitive | Larger → smaller (e.g., `int → byte`). Explicit, may lose data                     |
| Widening reference  | Subclass → superclass. Always allowed, implicit                                    |
| Narrowing reference | Superclass → subclass. Explicit; throws `ClassCastException` if invalid            |
| To String           | Any type → `String` via `toString()`                                               |
| Forbidden           | Between unrelated classes or between primitive and reference types (except String) |

> For narrowing reference to work the object's runtime type must be assignment-compatible with the target type, which means it must be the target type itself or a subtype of it. If not, a ClassCastException is thrown. You can always check first with instanceof to avoid that.
> 
---

## Autoboxing

Implicit conversion of primitives to their wrapper types (e.g., `int → Integer`).

Occurs when:
- Assigning a primitive to a wrapper variable.
- Passing a primitive to a method expecting a wrapper.

**Rules:**
- Exact type match required (e.g., `byte` won't autobox directly to `Short`).
- The compiler can narrow/widen literals before boxing.
- JVM caches `Integer` values from **-128 to 127** — same object reference returned for values in this range.

---

## String

**Why is `String` immutable and `final`?**

- **String pool** — immutability makes sharing safe.
- **Security** — passwords, URLs, and class names cannot be tampered with.
- **Thread safety** — safe to share across threads without synchronization.
- **Hashing** — `hashCode()` is computed once and cached, making `String` an ideal `HashMap` key.
- **ClassLoader** — class names must be stable.

**Why prefer `char[]` over `String` for passwords?**  
`String` stays in the pool until GC'd; you can't clear it. A `char[]` can be explicitly zeroed out after use.

**`intern()`** — adds the string to the pool (or returns the existing reference if already present).

**`String` in `switch`** — supported since Java 7. Uses `equals()` internally, so be careful with `null`.

### String vs StringBuffer vs StringBuilder

| | Mutable | Thread-safe | Speed |
|---|---|---|---|
| `String` | No | Yes (immutable) | — |
| `StringBuffer` | Yes | Yes (synchronized) | Slower |
| `StringBuilder` | Yes | No | Faster |

---

## `Object` Class Methods

Every class implicitly extends `Object`.

| Method               | Description                           |
|----------------------|---------------------------------------|
| `equals(Object obj)` | Compares by value                     |
| `hashCode()`         | Returns hash code                     |
| `toString()`         | String representation                 |
| `getClass()`         | Runtime class                         |
| `clone()`            | Shallow copy                          |
| `notify()`           | Wakes one waiting thread              |
| `notifyAll()`        | Wakes all waiting threads             |
| `wait()`             | Pauses thread until notified          |
| `wait(long timeout)` | Pauses thread for given duration      |
| `finalize()`         | Called before GC (deprecated Java 9+) |

---

## Constructors

- A special method with no return type, same name as the class.
- Called when a new object is created.
- If no constructor is defined, the compiler generates a no-arg **default constructor**.
- If any constructor is explicitly defined, the default is **not** generated automatically.

| Type          | Description                                  |
|---------------|----------------------------------------------|
| Default       | No-arg constructor                           |
| Parameterized | Takes arguments to initialize fields         |
| Copy          | Takes an existing instance of the same class |

**Uses of a private constructor:**
- Factory methods
- Singleton pattern
- Utility classes (prevent instantiation)
- Access from inner classes

---

## Reflection

Runtime inspection of classes, fields, methods, and constructors via `java.lang.reflect`.

Capabilities:
- Get an object's class.
- Inspect modifiers, fields, methods, constructors, and superclasses.
- Discover implemented interfaces.
- Create instances dynamically.
- Get/set field values (including private).
- Invoke methods.
- Create arrays dynamically.

---

## `equals()` and `hashCode()`

**`==` vs `equals()`:**
- `==` compares references.
- `equals()` (when overridden) compares internal state.

**`equals()` contract:**
- Reflexive: `x.equals(x)` is `true`.
- Symmetric: `x.equals(y) == y.equals(x)`.
- Transitive: if `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`.
- Consistent: same result across calls (unless state changes).
- `x.equals(null)` returns `false`.

**Overriding `equals()` steps:**
1. Check `this == obj` (same reference → true).
2. Check for `null` or wrong type → return `false`.
3. Cast to the correct type.
4. Compare significant fields.

**`hashCode()` contract:**
- Equal objects **must** have the same hash code.
- If `equals()` is overridden, `hashCode()` must be overridden too.
- Different objects *can* share hash codes (collision), but fewer is better.
- Use the same fields in both `equals()` and `hashCode()`.

**Why `31 * x + y` is better than `x + y`?**  
The prime multiplier improves distribution and reduces collisions.

**`instanceof` vs `getClass()`:**
- `instanceof` — true for the class and any subclass.
- `getClass()` — strict exact class match.

**Does `public boolean equals(MyClass that)` override `equals(Object obj)`?**  
No — it *overloads* it (different parameter type).

---

## Object Cloning

**Assignment vs cloning:**  
`=` copies a reference, not the object. Both variables point to the same instance.

**Using `clone()`:**
- Override as `public` and call `super.clone()`.
- Implement `Cloneable` (marker interface); otherwise `CloneNotSupportedException` is thrown.
- Does not invoke a constructor.

|                       | Shallow clone              | Deep clone                    |
|-----------------------|----------------------------|-------------------------------|
| Primitives/immutables | Copied by value            | Copied by value               |
| Mutable references    | References copied (shared) | References cloned recursively |

**Alternatives to `clone()`:**
- **Copy constructor** — safest and most recommended.
- **Factory method** — static method returning a new instance.
- **Serialization** — serialize and deserialize to get an independent copy.

---

## Exceptions

```
Throwable
├── Error          (JVM-level, don't handle)
└── Exception
    ├── Checked    (must catch or declare)
    └── RuntimeException (unchecked)
```

**Common unchecked exceptions:**
`NullPointerException`, `ClassCastException`, `IllegalArgumentException`, `IllegalStateException`, `IndexOutOfBoundsException`, `ArithmeticException`, `ConcurrentModificationException`, `UnsupportedOperationException`, `NoSuchElementException`

**Common checked exceptions:**
`IOException`, `InterruptedException`

**`OutOfMemoryError` variants:**

| Error                                | Cause                                  |
|--------------------------------------|----------------------------------------|
| `Java heap space`                    | Heap exhausted, often a memory leak    |
| `PermGen space`                      | Permanent generation full (pre-Java 8) |
| `GC overhead limit exceeded`         | GC running constantly, freeing nothing |
| `unable to create new native thread` | OS thread limit reached                |

### try-catch-finally

- `try` — code that may throw.
- `catch` — handles the exception.
- `finally` — always runs (with or without exception), unless the JVM is forcefully killed.
- `try-finally` without `catch` is valid.
- Since Java 7: multi-catch with `|` separator.
- `main()` can throw exceptions; uncaught ones are handled by the JVM (stack trace printed, program exits).

**Try-with-resources (Java 7+):**  
Resources declared in `try(...)` are auto-closed. They must implement `AutoCloseable`.

**Catch order:**  
Most specific → most general. `FileNotFoundException` before `IOException`.

---

## Generics

Allow defining parameterized types and methods. Type parameters are replaced with concrete types at usage.

```java
LinkedList<String> list = new LinkedList<>();
```

Using raw types (without type parameters) is allowed but not recommended.

### Type Bounds

Type parameters can be constrained using `extends` and `super`.

**Upper bounded** (`<T extends Type>`) — `T` must be `Type` or a subtype of it:

```java
// Accepts Number, Integer, Double, etc.
public <T extends Number> double sum(List<T> list) { ... }
```

**Lower bounded** (`<T super Type>`) — `T` must be `Type` or a supertype of it. Only valid with wildcards (see below).

### Wildcards (`?`)

A wildcard represents an unknown type. Used in variable declarations and method parameters — **not** in class or method definitions.

| Wildcard | Meaning | Use case |
|---|---|---|
| `<?>` | Any type (unbounded) | Read-only; only `Object` methods available |
| `<? extends T>` | `T` or any subtype (upper bound) | Reading from a structure — **producer** |
| `<? super T>` | `T` or any supertype (lower bound) | Writing into a structure — **consumer** |

```java
List<? extends Number> producers = List.of(1, 2.0, 3L); // can read as Number
List<? super Integer> consumers = new ArrayList<Number>(); // can write Integer
```

### PECS — Producer Extends, Consumer Super

A useful mnemonic for choosing the right wildcard:

- Use `<? extends T>` when you only **read** from the collection (it produces `T`s).
- Use `<? super T>` when you only **write** into the collection (it consumes `T`s).
- Use `<?>` when you do neither (e.g., just printing elements).
```java
// Copies elements from src into dst
public <T> void copy(List<? super T> dst, List<? extends T> src) {
    for (T item : src) dst.add(item);
}
```

### Type Erasure

Generic type information exists only at compile time. At runtime, the JVM erases type parameters and replaces them with their bounds (or `Object` if unbounded). This means:

- `List<String>` and `List<Integer>` are the same type at runtime (`List`).
- You cannot use `instanceof` with parameterized types: `obj instanceof List<String>` is a compile error.
- You cannot create generic arrays: `new T[]` is not allowed.
- You cannot instantiate a type parameter directly: `new T()` is not allowed.
```java
// At runtime, both are just List — this causes an "unchecked cast" warning
List<String> strings = (List<String>) someRawList;
```
 
---

## Proxies: JDK Dynamic Proxy vs CGLIB

| Feature                 | JDK Dynamic Proxy                | CGLIB                                |
|-------------------------|----------------------------------|--------------------------------------|
| Mechanism               | Interface-based (Reflection API) | Subclass-based (bytecode generation) |
| Requires interface      | Yes                              | No                                   |
| Proxies final methods   | No                               | No                                   |
| Proxy creation speed    | Faster                           | Slower                               |
| Method invocation speed | Optimized in modern JVMs         | Historically faster                  |

**Spring defaults:**
- Spring Core: JDK proxy if bean implements an interface; otherwise CGLIB.
- Spring Boot 2.0+: CGLIB by default.
- Force CGLIB: `@EnableAspectJAutoProxy(proxyTargetClass = true)` or `spring.aop.proxy-target-class=true`.

---

## Integer Cache

The JVM caches `Integer` objects for values **-128 to 127**. Autoboxed values in this range return the same object reference, so `==` comparisons may return `true`. Outside this range, new objects are created and `==` will return `false`.

---

## Annotation Retention Policies

Set with `@Retention` using `RetentionPolicy`:

| Policy    | Retained in `.class`? | Available at runtime? | Use case                                                     |
|-----------|-----------------------|-----------------------|--------------------------------------------------------------|
| `SOURCE`  | No                    | No                    | Compiler checks, code generation (e.g., Lombok, `@Override`) |
| `CLASS`   | Yes                   | No                    | Bytecode analysis tools. Default if unspecified              |
| `RUNTIME` | Yes                   | Yes                   | Reflection-based frameworks (Spring, Hibernate, JUnit)       |
