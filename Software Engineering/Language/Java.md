---
tags:
  - software-engineering
  - languages
---
#### var
Lets Java infer the local variable type from the initializer and is useful when the type is obvious and long. **Does not make Java dynamically typed like Javascript.**

```java
var context = new AnnotationConfigApplicationContext();
var name = "Name"
```

- Only for local variables
- Compiler infers the type from the initializer
- Still statically typed
- Cannot use with `null` only
- Equivalent to explicit type when type is obvious

#### .class
Gves the Class object for a type. Frameworks use this when they need metadata about a class rather than an instance.
```java
ProjectConfig.class
```

- Class literal
- Produces a `Class<ProjectConfig>`
- Refers to the type itself, not an instance
- Useful in frameworks, reflection, annotations, testing