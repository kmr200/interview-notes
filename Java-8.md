# Java 8

## Lambda

### What is a "lambda"? What is the structure and usage specifics of a lambda expression?

A lambda represents a set of instructions that can be stored in a variable and called multiple times in different parts of the program.

A lambda expression consists of the **lambda operator** `→`, which separates it into two parts:

- **Left side**: contains the list of parameters.
- **Right side**: contains the body of the lambda expression where actions are performed.

A lambda expression itself does not execute but serves as an implementation of a method in a **functional interface**, which must have only one abstract method.

Example:

```java
interface Operationable {
    int calculate(int x, int y);
}

public static void main(String[] args) {
    Operationable operation = (x, y) -> x + y;
    int result = operation.calculate(10, 20);
    System.out.println(result); // 30
}
```

Lambdas are essentially a shorter form of anonymous inner classes used previously in Java.

### Deferred Execution

Lambda expressions can be defined once and executed multiple times at different locations as needed.

### Lambda Parameters & Types

The parameter types must match those of the functional interface method:

```java
operation = (int x, int y) -> x + y;
```

Java can infer types, so specifying them is optional:

```java
(x, y) -> x + y;
```

For methods with no parameters, use empty parentheses:

```java
() -> 30 + 20;
```

For a single parameter, parentheses can be omitted:

```java
n -> n * n;
```

### Lambdas Without Return Values

Lambdas are not required to return a value:

```java
interface Printable {
    void print(String s);
}

public static void main(String[] args) {
    Printable printer = s -> System.out.println(s);
    printer.print("Hello, world");
}
```

### Block Lambdas

Block lambdas are enclosed in `{}` and can contain multiple statements, loops, and conditionals. If a return value is required, use `return`:

```java
Operationable operation = (int x, int y) -> {
    if (y == 0) return 0;
    return x / y;
};
```

### Passing Lambdas as Method Parameters

Lambdas can be passed as method arguments:

```java
interface Condition {
    boolean isAppropriate(int n);
}

private static int sum(int[] numbers, Condition condition) {
    int result = 0;
    for (int i : numbers) {
        if (condition.isAppropriate(i)) {
            result += i;
        }
    }
    return result;
}

public static void main(String[] args) {
    // Prints the sum of all non-zero elements
    System.out.println(sum(new int[]{0, 1, 0, 3, 0, 5, 0, 7, 0, 9}, (n) -> n != 0));
}
```

### What variables can lambda expressions access?

Access to variables from the outer scope in a lambda is similar to access from anonymous classes. A lambda can reference:

- **Effectively final local variables** (not explicitly marked `final` but never modified after assignment).
- **Instance fields** of the enclosing class.
- **Static variables**.

**1. Lifetime of Variables (Stack vs. Heap)**

Local variables are stored on the stack and destroyed as soon as the method that created them finishes. However, a lambda may outlive that method (e.g. passed to another thread). To prevent the lambda from accessing a variable that no longer exists, Java **captures (copies)** the value into the lambda object. By requiring the variable to be effectively final, Java ensures the copy and the original are always consistent.

**2. Thread Safety and Concurrency**

Lambdas are often used in concurrent or parallel processing (e.g. parallel streams). If multiple threads could modify the same local variable, it would create data races. Requiring effective finality preserves the thread-safety guarantee that local variables are normally only accessible to the thread executing that method.

> Note: Lambda expressions **cannot** call default methods of the functional interface they implement.

### How to sort a list of strings with a lambda expression?

```java
public static List<String> sort(List<String> list) {
    Collections.sort(list, (a, b) -> a.compareTo(b));
    return list; // Note: return the list, not `this`
}
```

---

## Method References

### What is a "method reference"?

If an existing method already performs the required operation, a method reference can be used to pass that method directly instead of writing a lambda. Method references are written as:

- Static methods: `ClassName::staticMethodName`
- Instance methods: `instanceObject::methodName`
- Constructors: `ClassName::new`

The effect is identical to defining a lambda that calls the method.

Example:

```java
private interface Measurable {
    int length(String string);
}

public static void main(String[] args) {
    Measurable a = String::length;
    System.out.println(a.length("abc")); // Output: 3
}
```

Method references are often more expressive than lambdas and provide better type information for the compiler. Use them instead of lambdas wherever possible.

### Types of method references

- To a static method
- To an instance method
- To a constructor

### Explain the expression `System.out::println`

This illustrates an **instance method reference**: it passes a reference to the `println()` method of the static field `out` from the `System` class.

---

## Functional Interfaces

### What is a "functional interface"?

A functional interface is an interface that defines exactly **one abstract method**.

The `@FunctionalInterface` annotation explicitly marks an interface as functional (similar to `@Override`), preventing accidental addition of a second abstract method. A functional interface may contain any number of `default` methods and still be considered functional, since default methods are not abstract.

### `Function<T, R>`, `DoubleFunction<R>`, `IntFunction<R>`, `LongFunction<R>`

`Function<T, R>` represents a function that takes an input of type `T` and returns a result of type `R`. Default methods like `compose` and `andThen` allow function chaining.

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123"); // "123"
```

Primitive specializations avoid boxing overhead:
- `DoubleFunction<R>` — takes `double`, returns `R`
- `IntFunction<R>` — takes `int`, returns `R`
- `LongFunction<R>` — takes `long`, returns `R`

### `UnaryOperator<T>`, `DoubleUnaryOperator`, `IntUnaryOperator`, `LongUnaryOperator`

These represent functions that take a single argument and return a result of the **same type**. They are specializations of `Function<T, T>`.

```java
UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5)); // Output: 25
```

| Functional Interface  | Input Type | Output Type |
|-----------------------|------------|-------------|
| `DoubleUnaryOperator` | `double`   | `double`    |
| `IntUnaryOperator`    | `int`      | `int`       |
| `LongUnaryOperator`   | `long`     | `long`      |

### `BinaryOperator<T>`, `DoubleBinaryOperator`, `IntBinaryOperator`, `LongBinaryOperator`

These represent functions that take **two arguments of the same type** and return a result of the same type. They are specializations of `BiFunction<T, T, T>`. Commonly used for aggregation (sum, multiplication, min/max).

```java
BinaryOperator<Integer> sum = (a, b) -> a + b;
System.out.println(sum.apply(1, 2)); // Output: 3
```

```java
IntBinaryOperator multiply = (a, b) -> a * b;
System.out.println(multiply.applyAsInt(3, 4)); // Output: 12
```

| Functional Interface   | Input Type | Output Type |
|------------------------|------------|-------------|
| `DoubleBinaryOperator` | `double`   | `double`    |
| `IntBinaryOperator`    | `int`      | `int`       |
| `LongBinaryOperator`   | `long`     | `long`      |

### `Predicate<T>`, `DoublePredicate`, `IntPredicate`, `LongPredicate`

These represent a function that takes one argument and returns a `boolean`. Used to test conditions and filter data. `Predicate<T>` provides default methods `and()`, `or()`, and `negate()` to compose predicates.

```java
Predicate<String> predicate = s -> s.length() > 0;
System.out.println(predicate.test("foo"));          // true
System.out.println(predicate.negate().test("foo")); // false
```

| Functional Interface | Input Type | Return Type |
|----------------------|------------|-------------|
| `DoublePredicate`    | `double`   | `boolean`   |
| `IntPredicate`       | `int`      | `boolean`   |
| `LongPredicate`      | `long`     | `boolean`   |

### `Consumer<T>`, `DoubleConsumer`, `IntConsumer`, `LongConsumer`

These represent an operation that takes a single argument and **returns no result**. Typically used for side effects like logging, printing, or modifying objects.

```java
Consumer<String> hello = name -> System.out.println("Hello, " + name);
hello.accept("world"); // Output: Hello, world
```

| Functional Interface | Input Type | Return Type |
|----------------------|------------|-------------|
| `DoubleConsumer`     | `double`   | `void`      |
| `IntConsumer`        | `int`      | `void`      |
| `LongConsumer`       | `long`     | `void`      |

### `Supplier<T>`, `BooleanSupplier`, `DoubleSupplier`, `IntSupplier`, `LongSupplier`

These represent a function that **takes no arguments** but returns a value. Useful for lazy initialization and deferred execution.

```java
Supplier<LocalDateTime> now = LocalDateTime::now;
System.out.println(now.get()); // Outputs the current timestamp
```

### Bifunctional Interfaces

| Functional Interface  | Parameters | Return Type | Purpose                                                                      |
|-----------------------|------------|-------------|------------------------------------------------------------------------------|
| `BiConsumer<T, U>`    | T, U       | `void`      | Performs an action on two arguments (e.g., logging, modifying objects).      |
| `BiFunction<T, U, R>` | T, U       | R           | Computes a result from two inputs (e.g., calculations, data transformation). |
| `BiPredicate<T, U>`   | T, U       | `boolean`   | Evaluates a condition on two inputs (e.g., filtering, validation).           |

### Primitive Conversion Interfaces

**`_ToFunction`** — converts between primitive types:

| Functional Interface   | Parameters | Return Type | Use Case                          |
|------------------------|------------|-------------|-----------------------------------|
| `DoubleToIntFunction`  | `double`   | `int`       | Rounding, truncation              |
| `DoubleToLongFunction` | `double`   | `long`      | Convert floating-point to long    |
| `IntToDoubleFunction`  | `int`      | `double`    | Convert integer to floating-point |
| `IntToLongFunction`    | `int`      | `long`      | Widen integer to long             |
| `LongToDoubleFunction` | `long`     | `double`    | Convert long to floating-point    |
| `LongToIntFunction`    | `long`     | `int`       | Narrow long to integer            |

**`ToBiFunction`** — take two inputs, return a primitive:

| Functional Interface       | Parameters | Return Type | Use Case                         |
|----------------------------|------------|-------------|----------------------------------|
| `ToDoubleBiFunction<T, U>` | T, U       | `double`    | Operation on two inputs → double |
| `ToIntBiFunction<T, U>`    | T, U       | `int`       | Operation on two inputs → int    |
| `ToLongBiFunction<T, U>`   | T, U       | `long`      | Operation on two inputs → long   |

**`ToFunction`** — single generic input, primitive return:

| Functional Interface  | Parameters | Return Type | Use Case                 |
|-----------------------|------------|-------------|--------------------------|
| `ToDoubleFunction<T>` | T          | `double`    | Extracts a double from T |
| `ToIntFunction<T>`    | T          | `int`       | Extracts an int from T   |
| `ToLongFunction<T>`   | T          | `long`      | Extracts a long from T   |

**`ObjConsumer`** — object + primitive, no return:

| Functional Interface   | Parameters  | Return Type | Use Case                  |
|------------------------|-------------|-------------|---------------------------|
| `ObjDoubleConsumer<T>` | T, `double` | `void`      | Action on object + double |
| `ObjIntConsumer<T>`    | T, `int`    | `void`      | Action on object + int    |
| `ObjLongConsumer<T>`   | T, `long`   | `void`      | Action on object + long   |

---

## StringJoiner

### What is `StringJoiner`?

`StringJoiner` creates a sequence of strings separated by a delimiter, with an optional prefix and suffix.

```java
StringJoiner joiner = new StringJoiner(".", "prefix-", "-suffix");
for (String s : "Hello the brave world".split(" ")) {
    joiner.add(s);
}
System.out.println(joiner); // prefix-Hello.the.brave.world-suffix
```

---

## Default Methods in Interfaces

### What are default methods in an interface?

Java 8 introduced default methods, allowing non-abstract method implementations in interfaces using the `default` keyword.

```java
interface Example {
    int process(int a);

    default void show() {
        System.out.println("default show()");
    }
}
```

**Key features:**
- A class inherits the default method unless it explicitly overrides it.
- If a class implements multiple interfaces with the **same** default method signature, it must override to resolve the conflict.
- A default method **cannot** override a method from `java.lang.Object`.
- Helps extend interfaces without breaking existing implementations.
- Reduces the need for utility classes by placing helper methods directly in the interface.

**Why were default methods introduced?**  
A major motivation was enabling Java 8 collections to support lambda expressions (e.g. `forEach`, `stream`) without breaking existing implementations of `List`, `Collection`, etc.

### How to call a default method from within an implementing class

Use the `InterfaceName.super.methodName()` syntax:

```java
interface Paper {
    default void show() {
        System.out.println("default show()");
    }
}

class Licence implements Paper {
    public void show() {
        Paper.super.show();
    }
}
```

---

## Static Methods in Interfaces

### What is a static method in an interface?

Static methods in interfaces are similar to default methods but **cannot be overridden** by implementing classes. They belong to the interface itself and are typically used for utility operations (null checks, sorting, etc.). Methods from `java.lang.Object` cannot be overridden as static methods.

### How to call a static method in an interface?

Use the interface name directly:

```java
interface Paper {
    static void show() {
        System.out.println("static show()");
    }
}

class Licence {
    public void showPaper() {
        Paper.show();
    }
}
```

---

## Optional

### What is `Optional`?

`Optional<T>` is a container for an object that may or may not be `null`. It helps prevent `NullPointerException` by providing higher-order functions, eliminating repetitive null checks.

```java
Optional<String> optional = Optional.of("hello");

optional.isPresent();                               // true
optional.ifPresent(s -> System.out.println(s.length())); // 5
optional.get();                                     // "hello"
optional.orElse("ops...");                          // "hello"
```

---

## Stream API

### What is a Stream?

`java.util.Stream` represents a sequence of elements supporting various operations. Stream operations are either:

- **Intermediate** — return a `Stream`, allowing method chaining.
- **Terminal** — produce a result and close the stream.

**Key characteristics:**
- Support method chaining.
- **Lazy execution**: intermediate operations run only when a terminal operation is called.
- Created from sources like `java.util.Collection` (Maps are not directly supported).
- Can run sequentially or in parallel.
- **Cannot be reused**: once a terminal operation is called, the stream is closed.

### Primitive Streams

Java provides specialized streams for primitives to avoid boxing overhead:
- `IntStream`
- `LongStream`
- `DoubleStream`

These use specialized functional interfaces (`IntFunction`, `IntPredicate`, etc.) and provide additional terminal operations like `sum()` and `average()`.

### Ways to Create a Stream

| Method             | Example                                      | Description                    |
|--------------------|----------------------------------------------|--------------------------------|
| From a collection  | `Arrays.asList("x","y","z").stream()`        | Stream from a collection.      |
| From values        | `Stream.of("x","y","z")`                     | Stream from individual values. |
| From an array      | `Arrays.stream(new String[]{"x","y","z"})`   | Stream from an array.          |
| From a file        | `Files.lines(Paths.get("input.txt"))`        | Each line is an element.       |
| From a string      | `"0123456789".chars()`                       | `IntStream` of Unicode values. |
| Using builder      | `Stream.builder().add("a").add("b").build()` | Manually constructed stream.   |
| Using `iterate()`  | `Stream.iterate(1, n -> n + 1)`              | Infinite stream via iteration. |
| Using `generate()` | `Stream.generate(() -> "0")`                 | Infinite stream via supplier.  |

### Difference Between Collection and Stream

| Aspect              | Collection                                       | Stream                                         |
|---------------------|--------------------------------------------------|------------------------------------------------|
| Element handling    | Work with individual elements.                   | Process elements as a whole pipeline.          |
| Concept             | A data structure (List, Set, Map).               | An abstraction for computation.                |
| Modification        | Elements can be added/removed/updated.           | Immutable; operations don't modify the source. |
| Reusability         | Can be iterated multiple times.                  | Single-use; closed after terminal operation.   |
| Execution           | Eager — processes immediately.                   | Lazy — executes only on terminal operation.    |
| Parallel processing | Requires explicit handling (`parallelStream()`). | Built-in support via `parallel()`.             |

### Purpose of the `collect()` Method

`collect()` is a terminal operation that transforms stream elements into a collection or other data structure. It accepts a `Collector<T, A, R>` with four stages:

1. **Supplier** — initializes the accumulator.
2. **Accumulator** — processes each element.
3. **Combiner** — merges two accumulators (for parallel execution).
4. **Finisher** (optional) — performs final modifications.

**Common collectors:**

| Collector                               | Description                                              |
|-----------------------------------------|----------------------------------------------------------|
| `toList()`, `toSet()`, `toCollection()` | Converts to a List, Set, or Collection.                  |
| `toMap()`, `toConcurrentMap()`          | Converts to a Map.                                       |
| `averagingInt/Double/Long()`            | Computes the average.                                    |
| `summingInt/Double/Long()`              | Computes the sum.                                        |
| `summarizingInt/Double/Long()`          | Returns `SummaryStatistics` (min, max, avg, count, sum). |
| `partitioningBy(Predicate)`             | Splits into two groups → `Map<Boolean, List<T>>`.        |
| `groupingBy(Function)`                  | Groups elements → `Map<Key, List<T>>`.                   |
| `mapping(Function, Collector)`          | Additional transformation for complex collectors.        |

**Example of a custom collector:**

```java
Collector<String, List<String>, List<String>> toList = Collector.of(
    ArrayList::new,      // Supplier: creates a new list
    List::add,           // Accumulator: adds an element
    (l1, l2) -> {        // Combiner: merges two lists
        l1.addAll(l2);
        return l1;
    }
);
```

### Other Stream Methods

`map()` transforms each element:

```java
Stream.of("12", "22", "4", "444", "123")
    .mapToInt(Integer::parseInt)
    .toArray(); // [12, 22, 4, 444, 123]
```

`flatMap()` transforms one element into multiple elements ("flattening" nested structures):

```java
Stream.of("H e l l o", "w o r l d !")
    .flatMap(p -> Arrays.stream(p.split(" ")))
    .toArray(String[]::new);
// ["H", "e", "l", "l", "o", "w", "o", "r", "l", "d", "!"]
```

Other intermediate operations include `filter()`, `limit()`, `sorted()`, `distinct()`, `skip()`, `peek()`, and their primitive variants (`mapToInt`, `mapToDouble`, `mapToLong`, `flatMapToInt`, etc.).

### Parallel Streams

Parallel streams split data into segments, process them concurrently across multiple cores using the common `ForkJoinPool`, then merge results.

```java
// A stream can switch between sequential and parallel mid-pipeline:
collection.stream()
    .peek(...)      // sequential
    .parallel()
    .map(...)       // parallel
    .sequential()
    .reduce(...);   // sequential
```

By default, `forEach()` on a parallel stream may produce results in non-deterministic order. Use `forEachOrdered()` to preserve order.

**Factors influencing parallel stream performance:**

- **Data size** — large datasets benefit more from parallelism.
- **CPU cores** — more cores improve performance; single-core gains nothing.
- **Data structure** — `ArrayList` is efficient (contiguous memory); `LinkedList` is not.
- **Primitive vs. object types** — primitive operations are generally faster.
- **Avoid blocking operations** — network calls in parallel streams can starve the shared `ForkJoinPool`.
- **Disable ordering when not needed** — `unordered()` can improve performance:

```java
collection.parallelStream()
    .sorted()
    .unordered()
    .collect(Collectors.toList());
```

### Terminal Methods

| Method             | Description                                                            |
|--------------------|------------------------------------------------------------------------|
| `findFirst()`      | Returns the first element.                                             |
| `findAny()`        | Returns any matching element.                                          |
| `collect()`        | Collects elements into a collection or structure.                      |
| `count()`          | Returns the number of elements.                                        |
| `anyMatch()`       | `true` if any element matches the condition.                           |
| `noneMatch()`      | `true` if no elements match the condition.                             |
| `allMatch()`       | `true` if all elements match the condition.                            |
| `min()`            | Returns the minimum element (via `Comparator`).                        |
| `max()`            | Returns the maximum element (via `Comparator`).                        |
| `forEach()`        | Applies a function to each element (order not guaranteed in parallel). |
| `forEachOrdered()` | Applies a function to each element, preserving order.                  |
| `toArray()`        | Returns an array of elements.                                          |
| `reduce()`         | Performs aggregation, returning a single result.                       |

Additional for numeric streams:

| Method      | Description                         |
|-------------|-------------------------------------|
| `sum()`     | Returns the sum of all numbers.     |
| `average()` | Returns the average of all numbers. |

### Intermediate Methods

| Method                                                                | Description                                                       |
|-----------------------------------------------------------------------|-------------------------------------------------------------------|
| `filter()`                                                            | Returns elements matching a condition.                            |
| `skip()`                                                              | Skips the first N elements.                                       |
| `distinct()`                                                          | Removes duplicates (based on `equals()`).                         |
| `map()`                                                               | Transforms each element.                                          |
| `peek()`                                                              | Passes elements through unchanged, applying a side-effect action. |
| `limit()`                                                             | Limits the stream to N elements.                                  |
| `sorted()`                                                            | Sorts elements naturally or via `Comparator`.                     |
| `mapToInt()`, `mapToDouble()`, `mapToLong()`                          | Variants returning primitive numeric streams.                     |
| `flatMap()`, `flatMapToInt()`, `flatMapToDouble()`, `flatMapToLong()` | Transforms one element into multiple.                             |

Additional for numeric streams:

| Method       | Description                                         |
|--------------|-----------------------------------------------------|
| `mapToObj()` | Converts a numeric stream back to an object stream. |

### Stream Practice Tasks

**1. Print 10 random numbers:**
```java
new Random()
    .ints()
    .limit(10)
    .forEach(System.out::println);
```

**2. Print unique squares of numbers:**
```java
Stream.of(1, 2, 3, 2, 1)
    .map(n -> n * n)
    .distinct()
    .forEach(System.out::println);
```

**3. Count empty strings:**
```java
long count = Stream.of("Hello", "", "world", "", "!")
    .filter(String::isEmpty)
    .count();
System.out.println(count); // 2
```

**4. Print 10 random numbers in ascending order:**
```java
new Random()
    .ints()
    .limit(10)
    .sorted()
    .forEach(System.out::println);
```

**5. Find the maximum number:**
```java
int max = Stream.of(5, 3, 4, 55, 2)
    .mapToInt(n -> n)
    .max()
    .getAsInt(); // 55
```

**6. Find the minimum number:**
```java
int min = Stream.of(5, 3, 4, 55, 2)
    .mapToInt(n -> n)
    .min()
    .getAsInt(); // 2
```

**7. Get the sum of all numbers:**
```java
int sum = Stream.of(5, 3, 4, 55, 2)
    .mapToInt(n -> n)
    .sum(); // 69
```

**8. Get the average of all numbers:**
```java
double avg = Stream.of(5, 3, 4, 55, 2)
    .mapToInt(n -> n)
    .average()
    .getAsDouble(); // 13.8
```

---

## Additional Map Methods (Java 8)

| Method                            | Description                                                    |
|-----------------------------------|----------------------------------------------------------------|
| `putIfAbsent(K, V)`               | Adds key-value pair only if key is absent.                     |
| `forEach(BiConsumer)`             | Iterates over entries, performing an action on each.           |
| `compute(K, BiFunction)`          | Computes a new value for a key (creates or updates).           |
| `computeIfPresent(K, BiFunction)` | Updates a value only if the key already exists.                |
| `computeIfAbsent(K, Function)`    | Creates a key with a computed value only if key is absent.     |
| `getOrDefault(K, V)`              | Returns value for key, or a default if key not found.          |
| `merge(K, V, BiFunction)`         | Merges a new value with an existing one, or inserts if absent. |

Examples:

```java
map.putIfAbsent("a", "Aa");

map.forEach((k, v) -> System.out.println(v));

map.compute("a", (k, v) -> String.valueOf(k).concat(v)); // "a" -> "aAa"

map.computeIfPresent("a", (k, v) -> k.concat(v));

map.computeIfAbsent("a", k -> "A".concat(k)); // "a" -> "Aa"

map.getOrDefault("a", "not found");

map.merge("a", "Z", (value, newValue) -> value.concat(newValue)); // "a" -> "AaZ"
```

---

## Date and Time API

### What is `LocalDateTime`?

`LocalDateTime` combines `LocalDate` and `LocalTime`, representing date and time in the ISO-8601 calendar system **without a time zone**, with nanosecond precision. Provides convenience methods like `plusMinutes()`, `plusHours()`, `isAfter()`, `toSecondOfDay()`, etc.

### What is `ZonedDateTime`?

`ZonedDateTime` is the analog of `java.util.Calendar` — the most complete temporal context in ISO-8601. It includes a time zone, so all operations involving time shifts account for it.

### Common Date/Time Operations

**Get the current date:**
```java
LocalDate.now();
```

**Add time units to the current date:**
```java
LocalDate.now().plusWeeks(1);
LocalDate.now().plusMonths(1);
LocalDate.now().plusYears(1);
LocalDate.now().plus(1, ChronoUnit.DECADES);
```

**Get the next Tuesday:**
```java
LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.TUESDAY));
```

**Get the second Saturday of the current month:**
```java
LocalDate
    .of(LocalDate.now().getYear(), LocalDate.now().getMonth(), 1)
    .with(TemporalAdjusters.nextOrSame(DayOfWeek.SATURDAY))
    .with(TemporalAdjusters.next(DayOfWeek.SATURDAY));
```

**Get the current time with millisecond precision:**
```java
new Date().toInstant();
```

**Get the current local time with millisecond precision:**
```java
LocalDateTime.ofInstant(new Date().toInstant(), ZoneId.systemDefault());
```

---

## Repeatable Annotations

### How to define a repeatable annotation?

Create a container annotation for the list of repeatable annotations, then mark the repeatable annotation with `@Repeatable`:

```java
@interface Schedulers {
    Scheduler[] value();
}

@Repeatable(Schedulers.class)
@interface Scheduler {
    String birthday() default "Jan 8 1935";
}
```

---

## Nashorn

### What is Nashorn?

Nashorn is a JavaScript engine developed in Java by Oracle, enabling embedding of JavaScript code within Java applications. Compared to Rhino (Mozilla Foundation), Nashorn provides **2–10× higher performance** by compiling JavaScript to bytecode and loading it directly into the JVM. It can generate Java classes from JavaScript, loaded via a special class loader, and supports calling Java code from JavaScript.

### What is `jjs`?

`jjs` is a command-line utility for executing JavaScript programs directly in the console.

---

## Base64 Encoding/Decoding

### What class was introduced in Java 8 for encoding/decoding?

`Base64` is a thread-safe class implementing an encoder and decoder per RFC 4648 and RFC 2045.

**Six main methods:**
- `getEncoder()` / `getDecoder()` — standard Base64 per RFC 4648.
- `getUrlEncoder()` / `getUrlDecoder()` — URL-safe Base64 per RFC 4648.
- `getMimeEncoder()` / `getMimeDecoder()` — MIME Base64 per RFC 2045.

**Encoding:**
```java
String b64 = Base64.getEncoder().encodeToString("input".getBytes("utf-8")); // aW5wdXQ=
```

**Decoding:**
```java
String original = new String(Base64.getDecoder().decode("aW5wdXQ="), "utf-8"); // input
```