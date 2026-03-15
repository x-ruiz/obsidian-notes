---
category: Software Engineering
aliases:
  - SE
  - SWE
last-studied: 2026-03-15
tags:
  - software-engineering
  - frameworks
  - languages
  - testing-strategies
  - cicd
  - design-principles
  - design-patterns
---
# Database of Notes
![[Software Engineering.base]]

# Tasks
```tasks
path includes software engineering
```

# Quizzes
## Quiz 1 - Spring Boot Fundamentals

**Q1**:
*Core Ecosystem (Inference)*: Your [Spring](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring.md) note describes Spring as an "ecosystem of frameworks and projects" with the Spring Framework at the center. Why might calling raw Spring a "framework" be imprecise, and how does this relate to projects like Spring Boot? Give a 1-sentence explanation tying to "programming model."

**A1**:
```spoiler-block
	Spring is a collection of core frameworks and spring projects that are open source where spring boot is a spring project that aims to package related dependencies including other spring projects and core frameworks to make development easier.
```

**Q2:**
*Philosophy Deep Dive*: In [Spring Boot](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Boot.md), "convention over configuration" means starting with defaults and customizing deviations. **Hard twist:** If you add `spring-boot-starter-web` but exclude Tomcat via `exclude` in dependencies, what auto-configs _still_ happen (list 3), and why is this "production-ready" per the note?

**A2:**
```spoiler-block
Auto-configs related to other dependencies will still occur (like Spring MVC) and this is still production-ready because you are taking advantage of Spring's modular fashion, where you pick and choose which implementations to develop off of. This follows exactly with Spring Boot's philosophy of defining a convention, and allowing developers to customize to their use case.
```

**Q3:**
*Beans Mechanics:* In [Spring Beans](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Beans.md)'s `@Bean` example, why must you pass `ProjectConfig.class` (not `new ProjectConfig()`) to `AnnotationConfigApplicationContext`? What happens if you pass an instance instead?

**A3:**
```spoiler-block
You must pass ProjectConfig.class because Spring requires the metadata of the class to go find beans that return object instances of that class in the configuration class. If you pass an instance instead, you are passing the object itself instead of what kind of object to look for.
```

**Q4:**
*Auto-Config Edge Case:* Spring Boot scans your classpath for auto-config. **Scenario:** You have `spring-boot-starter-data-jpa` but no database driver (e.g., H2). Does JPA auto-config fully succeed? Explain the failure mode and how "convention over config" influences this.

**A4:**
- [ ] Answer question 4
```spoiler-block

```

**Q5:**
*Starters vs Manual:* Compare `spring-boot-starter-security` to manually adding Spring Security modules in core Spring. Why does Boot's approach lead to "less code written" (quote [Spring Boot](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Boot.md)), and name 1 third-party lib it pulls.

**A5:**
```spoiler-block
The starter approach leads to less code written because your pom.xml file footprint remains small, and the starter library can manage compatabile versions instead of the developer managing it individualy. It pulls spring-security-core and spring-security-web as an example.
```

**Q6:**
Bean Declaration Methods: [Spring Beans](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Beans.md) lists 3 ways to add beans: `@Bean`, stereotype annotations, programmatically. **Hard:** Write a _stereotype annotation_ example for `Parrot` (assume `@Component`), then explain why it's _not_ used in the `@Bean` config class demo.

**A6:**
- [ ] Answer question 6
```spoiler-block

```

**Q7:**
*Context Initialization:* In the [Spring Beans](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Beans.md) `Main` class, after `context = new AnnotationConfigApplicationContext(ProjectConfig.class)`, how do you _retrieve_ the `Parrot` bean? Write the exact code line. Bonus: What if `Parrot` had deps—how does Spring handle?

**A7:**
- [ ] Answer question 7 bonus
```spoiler-block
Parrot parrot = context.getBean(Parrot.class);
```

**Q8:**
*Boot vs Core Comparison:* Table from our chat: Core Spring = "explicit config," Boot = "defaults first." **Prove it:** In pure Spring (no Boot), rewrite the [Spring Beans](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Beans.md) `@Bean` `Parrot` setup using _XML config_ (1-2 lines). Why is Boot's Java config "easier"?

**A8:**
- [ ] Answer question 8
```spoiler-block

```

**Q9:**
*Java-Spring Interop (Var):* From [Spring Beans](obsidian://open?file=Software%20Engineering%2FSpring%2FSpring%20Beans.md) code: `var p = new Parrot();`. **Limitation test:** Why can't you use `var` for `var context = new AnnotationConfigApplicationContext(ProjectConfig.class);` if it were ambiguous? Tie to Java 10+ inference rules.

**A9:**
```spoiler-block
If it were ambiguous, then the compiler does not know what type to assign to var and would not know how to handle subsequent type checks.
```
