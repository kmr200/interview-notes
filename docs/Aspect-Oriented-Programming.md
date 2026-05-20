# Aspect-Oriented Programming (AOP)

## What is AOP?

Aspect-Oriented Programming (AOP) is a programming paradigm that addresses **cross-cutting concerns** — functionality that is spread across many parts of an application and does not belong cleanly inside any single class or module.

AOP allows you to define this shared behavior in one place (an *aspect*) and apply it declaratively across many points in the codebase, without modifying the target classes themselves. This keeps business logic clean and separates infrastructure concerns from domain logic.

---

## Cross-Cutting Concerns

A cross-cutting concern is any behavior that affects multiple layers or components of an application but is not part of their core responsibility. If you find yourself writing the same boilerplate in many unrelated classes, it is likely a cross-cutting concern.

Common examples:

| Concern                    | Description                                                                        |
|----------------------------|------------------------------------------------------------------------------------|
| **Logging**                | Recording method calls, parameters, execution time, and return values.             |
| **Security**               | Checking authentication and authorization before method execution.                 |
| **Transaction management** | Opening, committing, or rolling back database transactions around service methods. |
| **Caching**                | Returning cached results instead of re-executing expensive operations.             |
| **Error handling**         | Centralizing exception translation or retry logic.                                 |
| **Auditing**               | Tracking who performed which operation and when.                                   |
| **Monitoring / metrics**   | Measuring method execution time and reporting to a metrics system.                 |

Without AOP, these concerns are scattered across every class that needs them, creating tight coupling and code duplication. AOP centralizes them in a single aspect, applied wherever needed.

---

## Core AOP Concepts

### Aspect

An **aspect** is a module that encapsulates a cross-cutting concern. It combines *where* to apply behavior (pointcuts) with *what* behavior to apply (advice). In Spring, an aspect is a class annotated with `@Aspect`.

### Join Point

A **join point** is a specific point during program execution where an aspect can be applied — for example, a method call, a method execution, or an exception being thrown. In Spring AOP, join points are always method executions.

### Pointcut

A **pointcut** is an expression that selects which join points an advice should apply to. It is the *predicate* that matches join points.

```java
// Matches any method in any class in the service package
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceLayer() {}
```

Common pointcut designators:

| Designator    | Matches                                              |
|---------------|------------------------------------------------------|
| `execution`   | Method execution join points — the most widely used. |
| `within`      | Join points within certain types or packages.        |
| `@annotation` | Methods carrying a specific annotation.              |
| `@within`     | Types carrying a specific annotation.                |
| `args`        | Methods whose arguments match given types.           |
| `bean`        | Spring beans with a specific name (Spring AOP only). |

### Advice

**Advice** is the action taken by an aspect at a matched join point. It defines *what* happens and *when*.

| Advice Type     | Annotation        | When it runs                                                                                       |
|-----------------|-------------------|----------------------------------------------------------------------------------------------------|
| Before          | `@Before`         | Before the method executes.                                                                        |
| After returning | `@AfterReturning` | After the method returns successfully.                                                             |
| After throwing  | `@AfterThrowing`  | After the method throws an exception.                                                              |
| After (finally) | `@After`          | After the method exits, regardless of outcome.                                                     |
| Around          | `@Around`         | Wraps the method — can run logic before and after, and control whether the method executes at all. |

### Target Object

The **target object** is the object whose method execution is being advised. In Spring AOP, the target is always wrapped in a proxy.

### Proxy

A **proxy** is an object that wraps the target and intercepts method calls to apply the advice. Spring creates either a JDK dynamic proxy (for interface-based beans) or a CGLIB proxy (for class-based beans) at runtime.

### Weaving

**Weaving** is the process of linking aspects with the target objects to create the advised (proxied) object. This can happen at different times in the lifecycle — see the Weaving Types section below.

---

## Weaving Types

Weaving is the mechanism by which aspects are applied to target code. There are three main weaving times, each with different characteristics.

### 1. Compile-Time Weaving (CTW)

Aspect code is woven directly into the compiled bytecode **during compilation**. The output `.class` files already contain the woven aspect logic — no proxies are involved at runtime.

**Used by:** AspectJ (via the `ajc` compiler).

```
Source (.java) + Aspects (.aj / @Aspect)
        ↓  ajc compiler
Woven bytecode (.class)  ← aspects baked in
```

**Characteristics:**
- Fastest runtime performance — no proxy overhead whatsoever.
- Can advise any join point: constructors, field access, static methods, `private` methods — not just public method executions.
- Requires a special compiler (`ajc`) or a build plugin (Maven/Gradle AspectJ plugin).
- Harder to set up and debug than proxy-based approaches.

### 2. Load-Time Weaving (LTW)

Aspects are woven into classes **as they are loaded by the JVM classloader**, using a Java agent (`-javaagent`). The source `.class` files on disk remain unchanged; weaving happens in memory at load time.

**Used by:** AspectJ with a `javaagent`, also configurable in Spring with `@EnableLoadTimeWeaving`.

```
Bytecode (.class) on disk  →  ClassLoader + AspectJ agent  →  Woven class in memory
```

**Characteristics:**
- Same full join point coverage as compile-time weaving (all method types, constructors, fields).
- No need to recompile source — useful when you do not have access to source code.
- Requires JVM startup argument: `-javaagent:aspectjweaver.jar`.
- Slight startup overhead as classes are transformed on load.

### 3. Runtime Weaving (Proxy-Based)

Aspects are applied at **application startup** by wrapping target beans in proxy objects. No bytecode modification occurs — the proxy intercepts method calls and delegates to the aspect before/after calling the real object.

**Used by:** Spring AOP (the default and only weaving mechanism in Spring AOP).

```
Target bean  →  Spring creates Proxy  →  Proxy intercepts calls  →  Advice runs  →  Target method called
```

**Characteristics:**
- Zero additional tooling required — works with standard `javac` and the Spring container.
- Limited to **public method executions** on **Spring-managed beans**. Cannot advise constructors, field access, private methods, or calls within the same class (self-invocation).
- Proxy type chosen automatically:
    - **JDK dynamic proxy** — used when the bean implements at least one interface.
    - **CGLIB proxy** — used when the bean does not implement an interface (subclasses the target class at runtime).
- Slight per-call overhead due to proxy dispatch, negligible in most applications.

---

## Spring AOP vs AspectJ

Spring AOP and AspectJ both use the `@Aspect` annotation style, but they are fundamentally different in how and where they apply aspects.

| Feature                 | Spring AOP                                                           | AspectJ                                                                     |
|-------------------------|----------------------------------------------------------------------|-----------------------------------------------------------------------------|
| Weaving type            | Runtime (proxy-based)                                                | Compile-time or load-time                                                   |
| Join point support      | Public method execution only                                         | Methods, constructors, field access, static initializers, `private` methods |
| Self-invocation         | Not intercepted (bypasses proxy)                                     | Intercepted                                                                 |
| `private` methods       | Not intercepted                                                      | Intercepted                                                                 |
| Spring bean requirement | Target must be a Spring bean                                         | No Spring required — works on any Java object                               |
| Tooling required        | None beyond Spring                                                   | `ajc` compiler or `aspectjweaver` agent                                     |
| Performance             | Proxy dispatch overhead (negligible)                                 | No proxy overhead                                                           |
| Complexity              | Low — works out of the box                                           | Higher — requires build/JVM configuration                                   |
| Best for                | Typical application-level concerns (logging, transactions, security) | Fine-grained, deep, or performance-critical cross-cutting concerns          |

### Self-Invocation — the Key Spring AOP Limitation

Because Spring AOP works through proxies, a method calling another method **on the same object** bypasses the proxy entirely and the advice is never triggered:

```java
@Service
public class OrderService {

    public void placeOrder() {
        // This calls processPayment() directly on 'this', not through the proxy.
        // Any @Around or @Before advice on processPayment() will NOT run.
        processPayment();
    }

    @Transactional  // will NOT be applied when called from placeOrder()
    public void processPayment() {
        // ...
    }
}
```

**Solutions:**
- Inject the bean into itself via `@Autowired` and call the method through the injected reference (calls go through the proxy).
- Restructure so the two methods live in separate beans.
- Switch to AspectJ weaving if this pattern is pervasive.

---

## Spring AOP in Practice

### Setup

Add the dependency (included transitively with `spring-boot-starter-aop`):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Enable AOP in your configuration (auto-configured in Spring Boot; manual in plain Spring):

```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
}
```

### Defining an Aspect

```java
@Aspect
@Component
public class LoggingAspect {

    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);

    // Pointcut — matches all methods in the service package
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    // Before advice
    @Before("serviceLayer()")
    public void logBefore(JoinPoint joinPoint) {
        log.info("Calling: {}", joinPoint.getSignature().getName());
    }

    // After returning advice — captures return value
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        log.info("Returned from: {} with value: {}", joinPoint.getSignature().getName(), result);
    }

    // After throwing advice — captures exception
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Exception ex) {
        log.error("Exception in: {} — {}", joinPoint.getSignature().getName(), ex.getMessage());
    }

    // Around advice — full control over method execution
    @Around("serviceLayer()")
    public Object measureExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        try {
            return joinPoint.proceed(); // invoke the actual method
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            log.info("{} executed in {} ms", joinPoint.getSignature().getName(), elapsed);
        }
    }
}
```

### Custom Annotation Pointcut

A common pattern is to create a marker annotation and advise any method carrying it:

```java
// 1. Define the annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Audited {}

// 2. Apply it on a method
@Service
public class UserService {

    @Audited
    public void deleteUser(Long id) {
        // ...
    }
}

// 3. Advise it in an aspect
@Aspect
@Component
public class AuditAspect {

    @Before("@annotation(com.example.annotation.Audited)")
    public void audit(JoinPoint joinPoint) {
        System.out.println("Auditing method: " + joinPoint.getSignature().getName());
    }
}
```

---

## Advice Execution Order

When multiple aspects apply to the same join point, their order can be controlled with `@Order`. Lower values run first for `@Before` advice and last for `@After` / `@Around` advice (outer-to-inner on the way in, inner-to-outer on the way out).

```java
@Aspect
@Component
@Order(1)   // runs outermost
public class SecurityAspect { ... }

@Aspect
@Component
@Order(2)   // runs inside SecurityAspect
public class LoggingAspect { ... }

@Aspect
@Component
@Order(3)   // runs innermost, closest to the target method
public class TransactionAspect { ... }
```

Execution for a `@Before` chain: `Security → Logging → Transaction → target method`.
Execution for the `@After` chain (unwinding): `Transaction → Logging → Security`.

---

## Summary

| Concept                   | Description                                                                                                     |
|---------------------------|-----------------------------------------------------------------------------------------------------------------|
| **Cross-cutting concern** | Behavior that spans many classes and does not belong to any single one (logging, security, transactions, etc.). |
| **Aspect**                | A module that encapsulates a cross-cutting concern.                                                             |
| **Join point**            | A point during execution where an aspect can be applied (in Spring AOP: always a method execution).             |
| **Pointcut**              | An expression that selects which join points to advise.                                                         |
| **Advice**                | The action to execute at a matched join point (`@Before`, `@After`, `@Around`, etc.).                           |
| **Weaving**               | The process of applying aspects to target code.                                                                 |
| **Compile-time weaving**  | AspectJ bakes aspect logic into bytecode at compile time — full join point coverage, best performance.          |
| **Load-time weaving**     | AspectJ transforms bytecode as classes are loaded — full coverage without recompilation.                        |
| **Runtime weaving**       | Spring AOP wraps beans in proxies at startup — easy setup, limited to public method executions on Spring beans. |
| **Self-invocation**       | A known Spring AOP limitation — intra-class method calls bypass the proxy and are not advised.                  |