---
tags:
  - design-principles
  - software-engineering
aliases:
  - IoC
---
## What is It
The principle of delegating control of application execution to the *dependencies* (frameworks such as [[Spring]] instead of the application managing execution and using *dependencies*. **Not to be confused with [[Dependency Inversion Principle]]**. In this context, "delegating control" refers to actions such as:
- creating an instance
- calling a method
- intercepting method executions

Instead of **code** --> *calls* --> **framework** | **framework** --> *calls* --> **code**

![[Screenshot 2026-03-14 at 8.25.12 PM.png]]

## Why it Exists
Solves the problem of tightly coupled code that is hard to test, hard to extend, and hard to maintain.
IoC flips the dependency direction so code becomes:
- more modular
- more testable
- more flexible
- easier to swap implementations

## IoC vs [[Dependency Injection]] (DI)
| Concept | What It Is                                        | Example                                      |
| ------- | ------------------------------------------------- | -------------------------------------------- |
| IoC     | A principle — “don’t control everything yourself” | Framework calling your code                  |
| DI      | A pattern — a concrete way to achieve IoC         | Spring injecting a service into a controller |
DI is one way to implement IoC but not the only way.

## Common Forms of IoC
### [[Dependency Injection]] (DI)
The framework provides your dependencies instead of the code constructing them

```java
// You don't create the service.
// The framework injects it.
// GOOD
public class UserController {
    private final UserService service;

    public UserController(UserService service) {
        this.service = service;
    }
}

// BAD
public class UserController {
	private final UserService service = new UserService();
}
```

### Service Locator
Code asks a central registry for dependencies. Less favored in modern software engineering
- [ ] 🔼 Finish Service Locator
### Event-Driven / Observer Pattern
Code doesn't call the event loop, the event loop calls the code
- [ ] 🔼 Finish Event-Driven / Observer Pattern
### Template Method Pattern
Framework defines the algorithm, code fills in the steps
- [ ] 🔼 Finish Template Method Pattern

## IoC in Real Frameworks

 **[[Spring Boot]]**
- You don't create controllers, spring instantiates them.
- You don't manage lifecycle - spring does.
- You don't wire dependencies - [[Spring IoC Container]] does
**React**
- You don't control when components re-render - react does.
**Express.js**
- You don't call the request handler - express calls the function.
**JUnit**
- You don't run your tests - JUnit finds, runs, and calls the methods.

