# Spring Bean Creation: `@Bean`, `@Component`, `@Configuration`

## PasswordEncoder Example

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

This example introduces three important Spring concepts:

- `@Bean`
- `@Component`
- `@Configuration`

Before understanding them, let's first understand `PasswordEncoder` and `BCryptPasswordEncoder`.

---

# PasswordEncoder vs BCryptPasswordEncoder

## Is PasswordEncoder a Class?

No.

`PasswordEncoder` is an **interface** provided by Spring Security.

```java
public interface PasswordEncoder {

    String encode(CharSequence rawPassword);

    boolean matches(
            CharSequence rawPassword,
            String encodedPassword);
}
```

It only defines method signatures.

---

## What is BCryptPasswordEncoder?

`BCryptPasswordEncoder` is a concrete class that implements the `PasswordEncoder` interface.

```java
public class BCryptPasswordEncoder
        implements PasswordEncoder {

    @Override
    public String encode(CharSequence rawPassword) {
        // BCrypt hashing logic
    }

    @Override
    public boolean matches(
            CharSequence rawPassword,
            String encodedPassword) {
        // Password comparison logic
    }
}
```

Relationship:

```text
PasswordEncoder (Interface)
          ↑
          |
implements
          |
BCryptPasswordEncoder (Class)
```

---

## Why Do We Write?

```java
PasswordEncoder passwordEncoder =
        new BCryptPasswordEncoder();
```

instead of

```java
BCryptPasswordEncoder passwordEncoder =
        new BCryptPasswordEncoder();
```

Because of **Polymorphism**.

Programming to an interface keeps the code loosely coupled.

Today:

```java
PasswordEncoder passwordEncoder =
        new BCryptPasswordEncoder();
```

Tomorrow:

```java
PasswordEncoder passwordEncoder =
        new Pbkdf2PasswordEncoder();
```

No code changes are required in dependent classes.

---

## Real-Life Example

### Interface

```java
public interface Vehicle {

    void start();
}
```

### Car Implementation

```java
public class Car implements Vehicle {

    @Override
    public void start() {
        System.out.println("Car Started");
    }
}
```

### Bike Implementation

```java
public class Bike implements Vehicle {

    @Override
    public void start() {
        System.out.println("Bike Started");
    }
}
```

Usage:

```java
Vehicle vehicle = new Car();
vehicle.start();
```

Later:

```java
Vehicle vehicle = new Bike();
vehicle.start();
```

This is the same principle used with `PasswordEncoder`.

---

# What is @Component?

`@Component` is placed on a class.

It tells Spring:

> Create an object of this class automatically and manage it inside the Spring Container.

Example:

```java
@Component
public class UserService {

}
```

During component scanning Spring detects:

```java
UserService
```

and creates:

```java
new UserService();
```

The object becomes a Spring Bean.

---

## Common Stereotype Annotations

These annotations are specialized forms of `@Component`.

```java
@Service
public class UserService {
}
```

```java
@Repository
public class UserRepository {
}
```

```java
@RestController
public class UserController {
}
```

Internally all are treated as Spring Components.

---

# What is @Bean?

`@Bean` is placed on a method.

It tells Spring:

> Execute this method and register the returned object as a Spring Bean.

Example:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Spring executes:

```java
new BCryptPasswordEncoder();
```

and stores the returned object inside the Spring Container.

---

## Why Use @Bean?

Many classes come from external libraries.

Example:

```java
BCryptPasswordEncoder
RestTemplate
ObjectMapper
ModelMapper
```

We cannot modify their source code to add:

```java
@Component
```

Therefore we create and register them manually using `@Bean`.

Example:

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

---

# Difference Between @Component and @Bean

| Feature | @Component | @Bean |
|----------|-----------|--------|
| Applied On | Class | Method |
| Object Creation | Automatic | Manual |
| Component Scanning Required | Yes | No |
| Used for Own Classes | Mostly Yes | Sometimes |
| Used for Third-Party Classes | No | Yes |
| Example | Service, Repository, Controller | PasswordEncoder, RestTemplate |

---

# What is @Configuration?

`@Configuration` is placed on a class.

It tells Spring:

> This class contains bean definitions and should be processed specially.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

---

# Relationship Between @Configuration and @Bean

Typically:

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### What Happens?

1. Spring finds the configuration class.
2. Spring processes all `@Bean` methods.
3. Returned objects are registered in the container.
4. Spring manages bean lifecycle and singleton behavior.

---

# Will @Bean Work Without @Configuration?

Depends on the situation.

---

## Case 1: Plain Java Class

```java
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Result:

❌ Does NOT work.

Reason:

Spring never scans this class.

The `@Bean` method is never executed.

---

## Case 2: Using @Component

```java
@Component
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Result:

✅ Works.

Reason:

Spring detects the class through component scanning and processes the `@Bean` method.

---

# Then Why Use @Configuration Instead of @Component?

Because of proper singleton bean management.

---

## Example

```java
@Configuration
public class AppConfig {

    @Bean
    public A a() {
        return new A();
    }

    @Bean
    public B b() {
        return new B(a());
    }
}
```

---

## With @Configuration

Spring creates a proxy class internally.

```text
AppConfig$$EnhancerBySpringCGLIB
```

When:

```java
a()
```

is called inside another bean method, Spring returns the existing bean from the container.

Result:

```text
A Bean -> One Instance
```

---

## Without @Configuration

```java
@Component
public class AppConfig {

    @Bean
    public A a() {
        return new A();
    }

    @Bean
    public B b() {
        return new B(a());
    }
}
```

Inside:

```java
new B(a());
```

the method executes directly.

Result:

```text
Container A Bean -> Object1
A inside B -> Object2
```

Two separate objects are created.

This breaks singleton behavior.

---

# Internal Difference

## @Configuration

Spring creates a proxy:

```text
AppConfig$$EnhancerBySpringCGLIB
```

Method calls are intercepted and redirected to the Spring Container.

---

## @Component

No special proxying of bean methods.

Regular Java method calls occur.

---

# Spring Container Example

Configuration:

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Injection:

```java
@Autowired
private PasswordEncoder passwordEncoder;
```

Spring searches for a bean of type:

```java
PasswordEncoder
```

Finds:

```java
BCryptPasswordEncoder
```

and injects it automatically.

---

# Interview Answers

## What is @Component?

`@Component` is used on a class to indicate that Spring should automatically detect, instantiate, and manage it as a bean through component scanning.

---

## What is @Bean?

`@Bean` is used on a method to tell Spring that the object returned by the method should be registered and managed as a Spring Bean.

---

## What is @Configuration?

`@Configuration` marks a class as a source of bean definitions and enables special proxy-based processing to ensure correct singleton behavior for beans defined using `@Bean`.

---

## Difference Between @Bean and @Component

`@Component` automatically creates beans through classpath scanning, while `@Bean` manually registers objects returned by methods.

`@Component` is generally used for application classes, whereas `@Bean` is commonly used for third-party library classes.

---

## Why Use PasswordEncoder Instead of BCryptPasswordEncoder?

`PasswordEncoder` is an interface and `BCryptPasswordEncoder` is its implementation.

Using the interface promotes loose coupling, flexibility, and polymorphism.

Example:

```java
PasswordEncoder passwordEncoder =
        new BCryptPasswordEncoder();
```

This allows the implementation to be changed later without affecting dependent code.

---

# Rule to Remember

```text
@Component
    ↓
Automatically creates Spring-managed beans

@Bean
    ↓
Registers objects returned by methods as Spring beans

@Configuration
    ↓
Processes @Bean methods correctly and preserves singleton behavior
```

---

# Auth Service Example

```java
@Configuration
public class AppConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Usage:

```java
@Service
public class AuthService {

    private final PasswordEncoder passwordEncoder;

    public AuthService(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }
}
```

Spring automatically injects the `BCryptPasswordEncoder` bean into `AuthService`.
