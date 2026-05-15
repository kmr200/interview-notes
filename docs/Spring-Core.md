# Spring Core

## Creating  and managing Beans:
To create a Bean, you simply create a POJO class, for example:

```java
public class TestBean {
    private String name;
    
    public String getName() {
        return name;
    }
    
    public TestBean setName(String name) {
        this.name = name;
        return this;
    }
}
```

You will also need a configuration class which will be responsible for creation and configuration of Beans:

```java
@Configuration
@ComponentScan(basePackags = "base.package")
public class AppConfig {
    
    @Bean
    public TestBean testBean() {
        return new TestBean();
    }
    
}
```

@ComponentScan annotation specifies package where the Beans are located.
You specify all your beans in methods with annotation @Bean.
These methods usually just return a new object of the class, but you can configure how the beans are created here.
You can also specify Beans scope here (Singleton, Prototype, etc.)

And then you can create context object:

```java
public class CoreApplication {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        TestBean testBean = context.getBean(TestBean.class);
        
        testBean.setName("Kamron");
        System.out.println(testBean.getName());
    }
}
```

Context object can be used to get the beans from the IoC.

Within the IoC itself,
beans are stored as BeanDefinition objects which among other things contain the following metadata:
- A package-qualified class name: typically, the actual implementation class of the bean is defined.
- Bean behavioral configuration elements, which state how the bean should behave in the container (scope, lifecycle callbacks, and so forth).
- References to other beans that are needed for the bean to do its work. These references are also called collaborators or dependencies.
- Other configuration settings to set in the newly created object - for example, the size limit of the pool or the number of connections to use in a bean that manages a connection pool.

## Naming Beans:
You can name your beans in @Configuration class using @Bean(name = “beanName”) annotation:\

```java
@Configuration
@ComponentScan(basePackags = "base.package")
public class AppConfig {

    @Bean
    public TestBean testBean() {
        return new TestBean();
    }

}
```

The convention is to use the standard Java convention for instance field names when naming beans. That is, bean names
start with a lowercase letter and are camel-cased from there. Examples of such names include accountManager, accountService,
userDao, loginController, and so forth.

Naming beans consistently makes your configuration easier to read and understand. Also, if you use Spring AOP,
it helps a lot when applying advice to a set of beans related by name.

You can `getBean` by specifying its name:

```java
public class CoreApplication {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        TestBean testBean = context.getBean("testBean", TestBean.class);

        testBean.setName("Kamron");
        System.out.println(testBean.getName());
    }
}
```

The second argument specifies beans type which is not mandatory but prevents casting.

## Dependency Injection (DI):

### Constructor based DI:

To use DI you specify classes you depend on as constructor arguments in DOJO class:

```java
public class TestUser {
    
    private TestBean testBean;
    
    public TestUser(TestBean testBean) {
        this.testBean = testBean;
    }
    
    public void test() {
        System.out.println(testBean.getName());
    }
    
}
```

And pass the bean method as an argument in the Configuration class:

```java
@Configuration
@ComponentScan(basePackags = "base.package")
public class AppConfig {

    @Bean
    public TestBean testBean() {
        return new TestBean();
    }
    
    @Bean(name = "testUser")
    public TestUser testUser() {
        return new TestUser(testBean());
    }

}
```

### Setter based DI:

It is pretty much the same thing

```java
public class TestUser {
    
    private TestBean testBean;
    
    public TestUser() {}
    
    public void setTestBean(TestBean testBean) {
        this.testBean = testBean;
    }
    
    public void test() {
        System.out.println(testBean.getName());
    }
    
}
```

```java
@Configuration
@ComponentScan(basePackags = "base.package")
public class AppConfig {

    @Bean
    public TestBean testBean() {
        return new TestBean();
    }
    
    @Bean(name = "testUser")
    public TestUser testUser() {
        TestUser testUser = new TestUser();
        testUser.setTestBean(testBean());
        return testUser;
    }

}
```

### Constructor-based or setter-based DI?

Since you can mix constructor-based and setter-based DI, it is a good rule of thumb to use constructors for mandatory
dependencies and setter methods or configuration methods for optional dependencies. Note that use of
the @Required annotation on a setter method can be used to make the property be a required dependency; however,
constructor injection with programmatic validation of arguments is preferable.

### Circular dependencies

If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.
For example, Class A requires an instance of class B through constructor injection, and class B requires an instance of
class A thorough constructor injection. If you configure beans for classes A and B to be injected into each other,
the Spring IoC container detects this circular reference at runtime, and throws a BeanCurrentlyInCreationException.

### Lazy Beans

In Spring, beans are by default eagerly initialized, meaning they are created as soon as the ApplicationContext is loaded.
However, you can configure beans to be lazily initialized, meaning they will only be created when they are actually needed
(i.e., the first time they are requested from the ApplicationContext).

To configure beans to be lazily initialized when using annotations, you can use the @Lazy annotation.

```java
@Configuration
@ComponentScan(basePackags = "base.package")
public class AppConfig {

    @Lazy
    @Bean
    public TestBean testBean() {
        return new TestBean();
    }

}
```

### @Value annotation

@Value annotation in Spring is commonly used to inject values from properties files, including application.properties,
into Spring-managed beans. This is especially useful for externalizing configuration, such as database connection strings,
file paths, or any other configurable settings that you want to manage outside your code.

Example usage:
Suppose you have the following entries in your application.properties file:

```properties
app.name=MySpringApp
app.version=1.0.0
app.description=This is a sample Spring application
```

You can inject these values into your Spring bean using the @Value annotation like this:

```java
@Component
public class MyAppConfig {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    @Value("${app.description}")
    private String appDescription;
}
```

For non boot applications, you need to specify property source in Config class:

```java
@Configuration
@PropertySource("classpath:application.properties")
@ComponentScan(basePackages = "base.packages")
public class AppConfig {}
```

### Additional features:
- You can also provide a default value if the property is not found:
   ```java
   @Value("${app.name:DefaultAppName}")
   private String appName;
   ```
- The @Value annotation can be used for injecting more complex types, such as lists or maps, by providing comma-separated dddds or using Spring’s SpEL (Spring Expression Language).

## Depends on:
The depends-on attribute in XML forces a specific initialization order. With annotations, @DependsOn can be used to achieve the same effect. 

```java
@DependsOn("someOtherBean")
@Bean
public MyBean myBean() {
    return new MyBean();
}
```

## Autowiring:

The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans)
automatically for your bean by inspecting the contents of the ApplicationContext.
Autowiring has the following advantages:

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template discussed elsewhere in this chapter are also valuable in this regard.)
- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus, autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.

When using XML-based configuration metadata (see Dependency Injection), you can specify the autowire mode for a bean definition with the autowire attribute of the <bean/> element. The autowiring functionality has four modes. You specify autowiring per bean and can thus choose which ones to autowire. The following table describes the four autowiring modes:
Table 2. Autowiring modes

| Mode        | Explanation                                                                                                                                                                                                                                                                                                                                   |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| no          | (Default) No autowiring. Bean references must be defined by ref elements. Changing the default setting is not recommended for larger deployments, because specifying collaborators explicitly gives greater control and clarity. To some extent, it documents the structure of a system.                                                      |
| byName      | Autowiring by property name. Spring looks for a bean with the same name as the property that needs to be autowired. For example, if a bean definition is set to autowire by name and it contains a master property (that is, it has a setMaster(..) method), Spring looks for a bean definition named master and uses it to set the property. |
| byType      | Lets a property be autowired if exactly one bean of the property type exists in the container. If more than one exists, a fatal exception is thrown, which indicates that you may not use byType autowiring for that bean. If there are no matching beans, nothing happens (the property is not set).                                         |
| constructor | Analogous to byType but applies to constructor arguments. If there is not exactly one bean of the constructor argument type in the container, a fatal error is raised.                                                                                                                                                                        |

With byType or constructor autowiring mode, you can wire arrays and typed collections. In such cases, all autowire candidates within the container that match the expected type are provided to satisfy the dependency. You can autowire strongly-typed Map instances if the expected key type is String.

To use the @Autowired annotation, we first need to specify base package:

```java
@Configuration
@ComponentScan(basePackages = "base.package")
public class AppConfig{}
```

And then we specify Beans to be autowired using @Component, @Repository, @Controller, @Service annotations and auto-inject them using @Autowired annotation:

```java
@Component
public class TestBean {
    
    private String name;
    
    public TestBean() {}
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

```java
@Component
public class TestUser {
    
    private final TestBean testBean;
    
    @Autowired
    public TestUser(TestBean testBean) {
        this.testBean = testBean;
    }
    
    public void test() {
        System.out.println(testBean.getName());
    }
    
}
```
Autowiring can be used with any kind of DI (Setter, Constructor, Field)

## Bean scopes:

Bean’s describes the way it is created. There are 6 types of scopes 4 of which are available only if you use a web-aware ApplicationContext.

| Scope       | Description                                                                                                                                                                                                                                                |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| singleton   | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container.                                                                                                                                                       |
| prototype   | Scopes a single bean definition to any number of object instances.                                                                                                                                                                                         |
| request     | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext. |
| session     | Scopes a single bean definition to the lifecycle of an HTTP Session. Only valid in the context of a web-aware Spring ApplicationContext.                                                                                                                   |
| application | Scopes a single bean definition to the lifecycle of a ServletContext. Only valid in the context of a web-aware Spring ApplicationContext.                                                                                                                  |
| websocket   | Scopes a single bean definition to the lifecycle of a WebSocket. Only valid in the context of a web-aware Spring ApplicationContext.                                                                                                                       |

## Creation of a Custom Scope

The bean scoping mechanism is extensible. You can define your own scopes or even redefine existing scopes, although the latter is considered bad practice, and you cannot override the built-in singleton and prototype scopes.

### 1. Implement the `Scope` Interface

To integrate a custom scope into the Spring container, implement `org.springframework.beans.factory.config.Scope`:

```java
import org.springframework.beans.factory.config.Scope;
import org.springframework.beans.factory.ObjectFactory;

import java.util.HashMap;
import java.util.Map;

public class ThreadScope implements Scope {

    private final ThreadLocal<Map<String, Object>> threadScope =
        ThreadLocal.withInitial(HashMap::new);

    /** Return the object with the given name from the underlying scope. */
    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    /** Remove the object with the given name from the underlying scope. */
    @Override
    public Object remove(String name) {
        Map<String, Object> scope = threadScope.get();
        return scope.remove(name);
    }

    /** Register a destruction callback to be executed when the scope ends. */
    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // In a real implementation, store and invoke this when the scope is destroyed
        System.out.println("Destruction callback registered for: " + name);
    }

    /** Resolve a contextual object for the given key, if any. */
    @Override
    public Object resolveContextualObject(String key) {
        return null; // No contextual objects in this simple example
    }

    /** Return the conversation ID for the current underlying scope, if any. */
    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

### 2. Register the Custom Scope with the Spring Container

Use `ConfigurableBeanFactory.registerScope(String scopeName, Scope scope)` to make Spring aware of your custom scope.

#### Programmatic Registration

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ConfigurableApplicationContext context =
            new ClassPathXmlApplicationContext("applicationContext.xml");

        // Register the custom scope under the name "thread"
        context.getBeanFactory().registerScope("thread", new ThreadScope());

        MyBean bean1 = context.getBean(MyBean.class);
        MyBean bean2 = context.getBean(MyBean.class);

        System.out.println(bean1 == bean2); // true — same thread, same instance

        context.close();
    }
}
```

#### Registration via `CustomScopeConfigurer` (Declarative)

Spring provides `CustomScopeConfigurer` as a convenient `BeanFactoryPostProcessor` alternative to manual registration:

```java
import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

@Configuration
public class ScopeConfig {

    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.setScopes(Map.of("thread", new ThreadScope()));
        return configurer;
    }
}
```

### 3. Use the Custom Scope on a Bean

Once registered, reference the scope by name in your bean definition:

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("thread")  // the name registered above
public class MyBean {

    private final String id = java.util.UUID.randomUUID().toString();

    public String getId() {
        return id;
    }
}
```

### Summary

| Step | What to do                                                             |
|------|------------------------------------------------------------------------|
| 1    | Implement `org.springframework.beans.factory.config.Scope` (4 methods) |
| 2    | Register with `beanFactory.registerScope("name", scopeInstance)`       |
| 3    | Annotate beans with `@Scope("name")` or set `scope="name"` in XML      |

## Customizing the nature of the Beans:

### Lifecycle Callbacks:

```java
public class CachingMovieLister {
    
    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization
    }
    
    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction
    }
}
```

@PostConstruct will be called after initialization and @PreDestroy before destruction.
Neither of them is allowed to have any kind of argument as they are lifecycle callback methods triggered automatically.
If you want to pass an argument during initialization, you can achieve a similar result by using @Autowired on a method:

```java
@Autowired
public void initialize(MyRepository repository) {
    // This runs after bean creation and behaves similarly to PostConstruct
    repository.setup();
}
```

However, the recommended way is using constructor injection which makes the class easier to test.

#### @PreDestroy on prototype beans:
For prototype-scoped beans, the @PreDestroy method is not called because
the Spring container does not manage their complete lifecycle.

Why It Doesn't Work:
The primary reason is that Spring only handles the creation and initialization of prototype beans. Once the container instantiates, configures, and hands the prototype instance to the client, it loses track of that specific instance.

- No Long-term Record: Unlike singletons, Spring does not keep a reference to every created prototype bean.
- Memory Management: If Spring were to track all prototype beans to call their destruction methods later, it would prevent them from being garbage-collected, leading to massive memory leaks in applications that request many short-lived instances.
- Undefined End-of-Life: Since prototype beans can be created at any frequency, the container has no way of knowing when they are no longer needed by the client code.

#### Multiple @PostConstruct methods:
In short: Technically no, the Java EE (Jakarta EE) specification states that
only one method in a given class can be annotated with @PostConstruct.
However, the actual behavior depends on which framework you are using:

1. The Official Rule (Jakarta EE / JSR-250)
   According to the official documentation, you must only have one @PostConstruct method per class. If you define more than one, the behavior is "undefined," which usually results in an error or only one of the methods being called randomly.

2. The Spring Framework Exception
   If you are using Spring, the framework is often more lenient than the official specification. In many Spring versions, you can have multiple methods annotated with @PostConstruct in a single class, and Spring will try to execute all of them.
   - Warning: The execution order for multiple @PostConstruct methods in the same class is not guaranteed.
   - Recommendation: Even if Spring allows it, it is considered bad practice because it breaks portability and makes your bean initialization order unpredictable.

3. Multiple Methods via Inheritance
   You can have multiple @PostConstruct methods across a class hierarchy (e.g., one in a parent class and one in a child class).
   - The container will call the @PostConstruct method of the superclass first, then the method of the subclass.
   - If the subclass overrides the parent's @PostConstruct method, only the subclass method will run unless you explicitly call super.methodName().

**Better Alternatives**:
If you need to perform multiple initialization steps, the safest and cleanest approaches are:
- A Single Entry Point: Create one @PostConstruct method that calls multiple private helper methods in the specific order you need.
- Lifecycle Interfaces: In Spring, you can implement the InitializingBean interface or use the init-method attribute in your configuration, though @PostConstruct is generally preferred for its simplicity.

### Annotation based configuration:

#### `@Required`:

This annotation indicates that the affected bean property must be populated at configuration time,
through an explicit property value in a bean definition or through autowiring.
The container throws an exception if the affected bean property has not been populated.
This allows for eager and explicit failure, avoiding NullPointerException instances or the like later on.
We still recommend that you put assertions into the bean class itself (for example, into an init method).
Doing so enforces those required references and values even when you use the class outside the container.

The @Required annotation and RequiredAnnotationBeanPostProcessor are formally deprecated as of Spring Framework 5.1,
in favor of using constructor injection for required settings (or a custom implementation of InitializingBean.afterPropertiesSet() or a custom @PostConstruct method along with bean property setter methods)

```java
public class SimpleMovieLister {
    
    private MovieFinder movieFinder;
    
    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    //...
}
```

#### `@Autowired`:

You can apply the `@Autowired` annotation to constructors, as the following example shows:

```java
public class MovieRecommender {
    
    private final CustomPreferenceDao customPreferenceDao;
    
    @Autowired
    public MovieRecommender(CustomPrefrenceDao customPrefrenceDao) {
        this.customPreferenceDao = customPrefrenceDao;
    }
    //...
}
```

As of Spring Framework 4.3, an @Autowired annotation on such a constructor is no longer necessary if the target bean defines only one constructor to begin with. However, if several constructors are available and there is no primary/default constructor, at least one of the constructors must be annotated with @Autowired in order to instruct the container which one to use

You can also apply the @Autowired annotation to traditional setter methods, as the following example shows:

```java
public class MovieRecommender {
    
    private final CustomPreferenceDao customPreferenceDao;
    
    @Autowired
    public void setCustomPreferenceDao(CustomPrefrenceDao customPrefrenceDao) {
        this.customPreferenceDao = customPrefrenceDao;
    }
    //...
}
```

You can also apply the annotation to methods with arbitrary names and multiple arguments, as the following example shows:

```java
public class MovieRecommender {
    
    private final CustomPreferenceDao customPreferenceDao;
    
    private final MovieCatalog movieCatalog;
    
    @Autowired
    public void setCustomPreferenceDao(
            CustomPrefrenceDao customPrefrenceDao,
            MovieCatalog movieCatalog
    ) {
        this.customPreferenceDao = customPrefrenceDao;
        this.movieCatalog = movieCatalog;
    }
    //...
}
```

You can apply @Autowired to fields as well and even mix it with constructors, as the following example shows:

```java
public class MovieRecommender {
    
    private final CustomPreferenceDao customPreferenceDao;
    
    @Autowired
    private final MovieCatalog movieCatalog;
    
    @Autowired
    public void setCustomPreferenceDao(CustomPrefrenceDao customPrefrenceDao) {
        this.customPreferenceDao = customPrefrenceDao;
    }
    //...
}
```

You can also instruct Spring to provide all beans of a particular type from the ApplicationContext by adding the @Autowired annotation to a field or method that expects an array of that type, as the following example shows:

```java
public class MovieRecommender {
    @Autowired
    private MovieCatalog[] movieCatalogs;
}
```

You can also autowire a Map of Beans.

By default, when you autowire a Map<String, YourInterface>, Spring automatically populates it with all beans that implement YourInterface,
using the bean names as keys and bean instances as values:

```java
public interface NotificationService {
    void send(String message);
}

@Component("emailService")
public class EmailService implements NotificationService { }

@Component("smsService")
public class SmsService implements NotificationService { }

@Service
public class NotificationManager {
    // Spring injects all NotificationService beans here
    // Keys: "emailService", "smsService"
    @Autowired
    private Map<String, NotificationService> services;
}
```

- Key Requirement: The Map's key must be a String.
- Ordering: You can use @Order or implement the Ordered interface to control the iteration order of the values.

The same applies for typed collections, as the following example shows:

```java
public class MovieRecommender {
    @Autowired
    private Set<MovieCatalog> movieCatalogs;
}
```
Your target beans can implement the org.springframework.core.Ordered interface or use the @Order or standard @Priority annotation if you want items in the array or list to be sorted in a specific order. Otherwise, their order follows the registration order of the corresponding target bean definitions in the container.

**Autowiring a specific collection bean**:

If you have defined a specific Map as a bean itself (e.g., using @Bean or XML), using @Autowired might still trigger the default behavior of collecting all beans of the value type. [1, 2, 3]
To inject your specific map bean instead of a collection of all beans, use @Resource or combine @Autowired with @Qualifier. [1, 2, 3]

```java
@Configuration
public class MyConfig {
    public Map<String, String> customConfigMap() {
        return Map.of("key1", "value1");
    }
}

@Service
public class MyService {
    
    @Resource(name = "customConfigMap") // Recommended for specific Map beans
    private Map<String, String> myMap;
}
```

The same is true for list of beans with only difference being autowiring a specific list bean.
If you want to autowire an actual bean that returns a list,
you should make it return ArrayList instead List interface and also request an ArrayList bean.
This way, Spring will inject your specific bean instead of list of beans.

#### `@Primary`:

Because autowiring by type may lead to multiple candidates, it is often necessary to have more control over the selection process.
One way to accomplish this is with Spring’s @Primary annotation.
@Primary indicates that a particular bean should be given preference when multiple beans are candidates to be autowired to a single-valued dependency.
If exactly one primary bean exists among the candidates, it becomes the autowired.

#### `@Resource`:

Spring also supports injection by using the JSR-250 @Resource annotation (javax.annotation.Resource) on fields or bean property setter methods.
This is a common pattern in Java EE: for example, in JSF-managed beans and JAX-WS endpoints.
Spring supports this pattern for Spring-managed objects as well.

```java
public class SimpleMovieLister {
    
    private MovieFinder movieFinder;
    
    @Resource(name = "myMovieFinder")
   public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

## Prototype inside Singleton:

Injecting a prototype bean into a singleton bean in Spring creates a single instance of the prototype at startup, defeating its purpose.
To get a new prototype instance every time, use `@Lookup`, `ObjectFactory<T>`, `ObjectProvider<T>`, `Provider<T>`, or scoped proxies.
The direct `@Autowired` approach fails because the singleton is initialized once, capturing the initial prototype instance.

### @Lookup Annotation (Recommended):
```java
@Component
public class SingletonBean {
    @Lookup
    public PrototypeBean getPrototypeBean() {
        // Spring overrides this
        return null;
    }
}
```

### ObjectFactory\<T>
```java
@Component
@Scope("prototype")
public class ReportGenerator {
    private final String id = UUID.randomUUID().toString();

    public void generate(String reportName) {
        System.out.printf("[%s] Generating report: %s%n", id, reportName);
    }
}
```

```java
@Component
public class ReportService {

    private final ObjectFactory<ReportGenerator> generatorFactory;

    @Autowired
    public ReportService(ObjectFactory<ReportGenerator> generatorFactory) {
        this.generatorFactory = generatorFactory;
    }

    public void runReport(String reportName) {
        generatorFactory.getObject().generate(reportName);
    }
}
```

### ObjectProvider\<T>

`ObjectProvider<T>` is a Spring-specific extension of `ObjectFactory<T>` introduced in Spring 4.3. It adds **lazy resolution**, **optional dependency handling**, and **stream-based retrieval** — making it the most flexible Spring-native option.

```java
@Component
@Scope("prototype")
public class ReportGenerator {
    private final String id = UUID.randomUUID().toString();

    public void generate(String reportName) {
        System.out.printf("[%s] Generating report: %s%n", id, reportName);
    }
}
```

```java
@Component
public class ReportService {

    private final ObjectProvider<ReportGenerator> generatorProvider;

    @Autowired
    public ReportService(ObjectProvider<ReportGenerator> generatorProvider) {
        this.generatorProvider = generatorProvider;
    }

    public void runReport(String reportName) {
        // getObject() creates a fresh prototype instance each time
        generatorProvider.getObject().generate(reportName);
    }

    public void runIfAvailable(String reportName) {
        // Does not throw if the bean is missing — safe for optional dependencies
        generatorProvider.ifAvailable(gen -> gen.generate(reportName));
    }
}
```

> **`ObjectFactory<T>` vs `ObjectProvider<T>`**: Prefer `ObjectProvider<T>` in modern Spring applications. It is a strict superset — it won't throw a `NoSuchBeanDefinitionException` when the bean is absent (use `ifAvailable` / `getIfAvailable`), and it supports iterating over multiple matching beans via `stream()`.

### Provider\<T> (JSR-330 Standard)

`javax.inject.Provider<T>` (or `jakarta.inject.Provider<T>` in Jakarta EE) is the **standards-based** equivalent from JSR-330. It works identically to `ObjectFactory<T>` but keeps your code decoupled from Spring-specific APIs — useful in portable or framework-agnostic components.

```xml
<!-- Add JSR-330 dependency if not already present -->
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

```java
@Component
@Scope("prototype")
public class ReportGenerator {
    private final String id = UUID.randomUUID().toString();

    public void generate(String reportName) {
        System.out.printf("[%s] Generating report: %s%n", id, reportName);
    }
}
```

```java
import javax.inject.Provider;

@Component
public class ReportService {

    private final Provider<ReportGenerator> generatorProvider;

    @Autowired
    public ReportService(Provider<ReportGenerator> generatorProvider) {
        this.generatorProvider = generatorProvider;
    }

    public void runReport(String reportName) {
        // get() creates a fresh prototype instance on every call
        generatorProvider.get().generate(reportName);
    }
}
```

> **When to use `Provider<T>`**: Choose it when writing library code or components that must remain Spring-agnostic. For application code already tied to Spring, `ObjectProvider<T>` is generally preferred due to its richer API.

### Scoped Proxy

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class EmailSender {
    private final String id = UUID.randomUUID().toString();

    public void send(String to, String subject) {
        System.out.printf("[%s] Sending '%s' to %s%n", id, subject, to);
    }
}
```

```java
@Component
public class NotificationService {

    private final EmailSender emailSender;

    @Autowired
    public NotificationService(EmailSender emailSender) {
        this.emailSender = emailSender;
    }

    public void notify(String user, String message) {
        emailSender.send(user, message);
    }
}
```

**Why @Autowired Fails**:
Because the singleton bean is created only once at application startup, the prototype bean is injected into it only at that time.
Consequently, the same prototype instance is reused for all subsequent calls, nullifying its intended per-request lifecycle.

---

**Quick Comparison**:

| Approach            | Type    | New instance per call | Optional/safe | Framework-agnostic |
|---------------------|---------|-----------------------|---------------|--------------------|
| `@Lookup`           | Spring  | +                     | -             | -                  |
| `ObjectFactory<T>`  | Spring  | +                     | -             | -                  |
| `ObjectProvider<T>` | Spring  | +                     | +             | -                  |
| `Provider<T>`       | JSR-330 | +                     | -             | +                  |
| Scoped Proxy        | Spring  | +                     | -             | -                  |
### Custom Conditions

For logic not covered by built-in annotations, you can create a custom condition by implementing the Condition interface.

**Implement the Condition**:
```java
public class OnUnixCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return System.getProperty("os.name").toLowerCase().contains("nix");
    }
}
```

**Apply the Condition**:
```java
@Bean
@Conditional(OnUnixCondition.class)
public MyService unixSpecificService() {
    return new UnixService();
}
```

## Dynamic Bean Registration in Spring Boot

Dynamic bean registration in Spring Boot allows you to create and register beans programmatically rather than using static annotations like @Bean or @Component. This is essential for scenarios like multi-tenancy, registering beans based on external configuration (YAML/DB), or creating many similar beans in a loop.

Depending on your Spring version and use case, there are several ways to achieve this:

---

### 1. The Modern Way: `BeanRegistrar` (Spring Boot 4.0+)

Starting with Spring Framework 7.0 / Spring Boot 4.0, a first-class API called `BeanRegistrar` was introduced to make programmatic registration cleaner.

- **How it works:** You implement the `BeanRegistrar` interface, which gives you direct access to the `BeanRegistry` and the `Environment`.
- **Example Use Case:** Registering different implementation services (e.g., Email vs. SMS) based on application properties.

```java
public interface NotificationService {
    void send(String message);
}

@Component
public class EmailNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

@Component
public class SmsNotificationService implements NotificationService {
    @Override
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}
```

```yaml
# application.yml
notification:
  channel: sms   # switch to "email" to get EmailNotificationService
```

```java
import org.springframework.context.annotation.BeanRegistrar;
import org.springframework.context.annotation.BeanRegistry;
import org.springframework.core.env.Environment;

public class NotificationBeanRegistrar implements BeanRegistrar {

    @Override
    public void register(BeanRegistry registry, Environment env) {
        String channel = env.getProperty("notification.channel", "email");

        if ("sms".equalsIgnoreCase(channel)) {
            registry.registerBean("notificationService", SmsNotificationService.class);
        } else {
            registry.registerBean("notificationService", EmailNotificationService.class);
        }
    }
}
```

```java
@Configuration
@Import(NotificationBeanRegistrar.class)
public class AppConfig { }
```

---

### 2. Traditional Runtime Way: `ImportBeanDefinitionRegistrar`

This is the standard approach for library developers or complex configurations in Spring Boot 2.x and 3.x.

- **Usage:** Implement the interface and use `@Import` to link it to your `@Configuration` class.
- **Core Benefit:** It allows you to inspect annotation metadata on other classes before deciding which beans to register.

```java
// Custom annotation to trigger registration
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(TenantServiceRegistrar.class)
public @interface EnableTenantServices {
    String[] tenants();
}
```

```java
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;

public class TenantServiceRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata,
                                        BeanDefinitionRegistry registry) {

        Map<String, Object> attrs =
            metadata.getAnnotationAttributes(EnableTenantServices.class.getName());

        String[] tenants = (String[]) attrs.get("tenants");

        for (String tenant : tenants) {
            RootBeanDefinition definition = new RootBeanDefinition(TenantService.class);
            definition.getPropertyValues().add("tenantId", tenant);
            registry.registerBeanDefinition("tenantService_" + tenant, definition);
        }
    }
}
```

```java
@Component
public class TenantService {
    private String tenantId;

    public void setTenantId(String tenantId) { this.tenantId = tenantId; }

    public void serve() {
        System.out.println("Serving tenant: " + tenantId);
    }
}
```

```java
// Registering beans for tenants "acme" and "globex" at startup
@EnableTenantServices(tenants = { "acme", "globex" })
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        var ctx = SpringApplication.run(Main.class, args);

        TenantService acme = (TenantService) ctx.getBean("tenantService_acme");
        acme.serve();   // Serving tenant: acme
    }
}
```

---

### 3. Deep Integration: `BeanDefinitionRegistryPostProcessor`

Use this if you need to register beans based on properties loaded into the environment.

- **Mechanism:** It provides a `postProcessBeanDefinitionRegistry` hook that runs very early in the application lifecycle—before any standard beans are even instantiated.
- **Best Practice:** Declare this processor as a `static @Bean` to avoid early initialization issues of your configuration class.

```yaml
# application.yml
datasources:
  - name: primary
    url: jdbc:postgresql://localhost:5432/primary_db
  - name: analytics
    url: jdbc:postgresql://localhost:5432/analytics_db
```

```java
import org.springframework.beans.factory.config.BeanDefinitionCustomizer;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.core.env.Environment;

public class DataSourceRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    private final Environment env;

    public DataSourceRegistryPostProcessor(Environment env) {
        this.env = env;
    }

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
        int i = 0;
        String name;
        while ((name = env.getProperty("datasources[" + i + "].name")) != null) {
            String url = env.getProperty("datasources[" + i + "].url");

            RootBeanDefinition def = new RootBeanDefinition(TenantDataSource.class);
            def.getPropertyValues().add("name", name);
            def.getPropertyValues().add("url", url);
            registry.registerBeanDefinition("dataSource_" + name, def);
            i++;
        }
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
        // no-op
    }
}
```

```java
@Configuration
public class DataSourceConfig {

    // static — ensures this post-processor runs before the configuration class is instantiated
    @Bean
    public static DataSourceRegistryPostProcessor dataSourceRegistryPostProcessor(Environment env) {
        return new DataSourceRegistryPostProcessor(env);
    }
}
```

```java
@Component
public class TenantDataSource {
    private String name;
    private String url;

    public void setName(String name) { this.name = name; }
    public void setUrl(String url)   { this.url = url; }

    @PostConstruct
    public void init() {
        System.out.printf("DataSource ready — name: %s, url: %s%n", name, url);
    }
}
```

---

### 4. Direct Manual Registration: `GenericApplicationContext`

If you need to register a bean "on the fly" after the application has already started (e.g., in response to a user action or an API call):

```java
@Autowired
private GenericApplicationContext context;

public void registerMyBean(String name) {
    context.registerBean(name, MyService.class, () -> new MyService());
}
```

> **Warning:** Manually registered beans might not be automatically injected into existing beans that have already been created.

```java
@RestController
@RequestMapping("/beans")
public class BeanRegistrationController {

    private final GenericApplicationContext context;

    public BeanRegistrationController(GenericApplicationContext context) {
        this.context = context;
    }

    @PostMapping("/{name}")
    public ResponseEntity<String> register(@PathVariable String name) {
        if (context.containsBean(name)) {
            return ResponseEntity.badRequest().body("Bean already exists: " + name);
        }

        context.registerBean(name, MyService.class, () -> new MyService(name));
        return ResponseEntity.ok("Registered bean: " + name);
    }

    @GetMapping("/{name}/invoke")
    public ResponseEntity<String> invoke(@PathVariable String name) {
        MyService service = context.getBean(name, MyService.class);
        return ResponseEntity.ok(service.execute());
    }
}
```

```java
@Component
public class MyService {
    private final String id;

    public MyService(String id) { this.id = id; }

    public String execute() {
        return "MyService[" + id + "] running at " + Instant.now();
    }
}
```

---

### Comparison Table

| Approach                              | Spring Version             | When to Use                                          |
|---------------------------------------|----------------------------|------------------------------------------------------|
| `BeanRegistrar`                       | Boot 4.0+ / Framework 7.0+ | Cleanest modern API; environment-driven registration |
| `ImportBeanDefinitionRegistrar`       | Boot 2.x / 3.x             | Library authors; annotation metadata inspection      |
| `BeanDefinitionRegistryPostProcessor` | Any                        | Early lifecycle; property-driven bulk registration   |
| `GenericApplicationContext`           | Any                        | Runtime / on-the-fly registration after startup      |

## @PostConstruct vs Bean init-method:

The main difference between @PostConstruct and init-method (or initMethod) lies in their priority, standardization, and usage context within the Spring Framework.

Core Comparison:

| Feature         | @PostConstruct                  | init-method             |
|-----------------|---------------------------------|-------------------------|
| Standard        | JSR-250 (Java standard)         | Spring-specific         |
| Execution Order | First (Highest priority)        | Last (Lowest priority)  |
| Configuration   | Annotation on the method itself | Defined in @Bean or XML |
| Dependency      | javax.annotation-api            | Pure Spring framework   |
| Best For        | Internal bean setup logic       | Third-party classes     |




