---
tags:
  - software-engineering/languages/java
---

> [!IMPORTANT]
> **Java Enums are full-fledged `final` classes, not integer aliases.**  
> In languages like C/C++, enums are syntactic sugar over primitive integers (`0, 1, 2`). In Java, every enum constant is an **immutable, type-safe singleton object** instantiated by the JVM at class-loading time.

---

# What Happens Under the Hood

When you define an enum:

```java
public enum CatalogType {
    REST,
    GLUE
}
```

The Java compiler (`javac`) lowers/desugars this code into a standard class hierarchy:

```java
// Equivalent decompiled bytecode representation
public final class CatalogType extends java.lang.Enum<CatalogType> {

    // 1. Each constant is a public static final INSTANCE of the class
    public static final CatalogType REST = new CatalogType("REST", 0);
    public static final CatalogType GLUE = new CatalogType("GLUE", 1);

    private static final CatalogType[] $VALUES = new CatalogType[] { REST, GLUE };

    // 2. Private constructor prevents external instantiation via 'new'
    private CatalogType(String name, int ordinal) {
        super(name, ordinal);
    }

    public static CatalogType[] values() {
        return (CatalogType[]) $VALUES.clone();
    }

    public static CatalogType valueOf(String name) {
        return (CatalogType) java.lang.Enum.valueOf(CatalogType.class, name);
    }
}
```

### Why `method(CatalogType type)` Accepts `CatalogType.REST`
* In `void setCatalog(CatalogType type)`, the parameter requires an instance of `CatalogType`.
* `CatalogType.REST` is a static field whose evaluated type is literally `CatalogType`.
* At the bytecode level (`invokevirtual`), this is identical to passing any standard reference type.

---

# Parameter Passing & Polymorphism Patterns

### Pattern A: Concrete Enum Type (Strict Domain Boundaries)
Use when the method belongs exclusively to one domain model:
```java
public void configureCatalog(CatalogType type) {
    Objects.requireNonNull(type, "CatalogType cannot be null");
    // Switch expression (exhaustiveness checked at compile-time in modern Java)
    switch (type) {
        case REST -> initRest();
        case GLUE -> initGlue();
    }
}
```

### Pattern B: Interface-Driven Extensibility (The Open-Closed Pattern)
Because enums implicitly extend `java.lang.Enum<E>`, **Java does not support enum inheritance** (`enum A extends B` is illegal). To pass different enum families into the same parameter, make them implement a common interface:

```java
// 1. Define common contract
public interface CatalogFormat {
    String protocol();
}

// 2. Both enums implement the interface independently
public enum IcebergCatalog implements CatalogFormat {
    REST { @Override public String protocol() { return "https"; } },
    HIVE { @Override public String protocol() { return "thrift"; } }
}

public enum DeltaCatalog implements CatalogFormat {
    UNITY { @Override public String protocol() { return "grpc"; } }
}

// 3. Consumer programs against the interface, not the concrete enum
public void mountCatalog(CatalogFormat format) {
    System.out.println("Connecting via: " + format.protocol());
}

// Call site accepts constants from completely different enums:
mountCatalog(IcebergCatalog.REST);
mountCatalog(DeltaCatalog.UNITY);
```

### Pattern C: Generic Enum Handling (`Enum<?>` & Bounded Types)
When writing infrastructure/framework code (e.g., serializers, loggers, metrics collectors):

```java
// Accept any enum constant in the JVM
public void logEnumState(Enum<?> enumVal) {
    logger.info("Enum: {} | Ordinal: {}", enumVal.name(), enumVal.ordinal());
}

// Type-safe generic self-bound pattern
public <E extends Enum<E>> E parseConfig(Class<E> enumClass, String value) {
    return Enum.valueOf(enumClass, value.trim().toUpperCase());
}
```

---

# VM Guarantees & Edge Cases

| Feature                 | JVM Guarantee / Behavior                                                                                | Senior Note                                                                                                           |
| :---------------------- | :------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------- |
| **Singleton Guarantee** | The JVM guarantees exactly one instance per constant per ClassLoader.                                   | Safe against reflection attacks (`Constructor.newInstance()` explicitly throws `IllegalArgumentException` for enums). |
| **Serialization**       | Special-cased by the JVM Specification. Only the `name` is written/read; `readResolve()` is not needed. | Backwards compatible as long as constant names don't change.                                                          |
| **Thread Safety**       | Class initialization lock guarantees thread-safe instantiation.                                         | Safe for lazy/eager initialization patterns (e.g., Joshua Bloch's Enum Singleton).                                    |
| **Memory Efficiency**   | Fast reference equality checks                                                                          | Always use == for enum comparison (null-safe at compile time).                                                        |

---

# Architectural Best Practices & Anti-Patterns

### Best Practices
1. Always use == instead of `.equals()`:
   `type == CatalogType.REST` avoids `NullPointerException` if `type` is null and provides compile-time type safety.
2. **Use `EnumMap` and `EnumSet`:**
   High-performance collections optimized specifically for enums. Internally implemented as bit vectors / arrays with zero hash-collision overhead.
3. **Static Reverse-Lookup Map:**
   Avoid iterating `values()` repeatedly inside search methods; cache a lookup map:
```java
   private static final Map<String, CatalogType> LOOKUP = Arrays.stream(values()).collect(Collectors.toUnmodifiableMap(Enum::name, Function.identity()));
```

### Anti-Patterns to Avoid
1. **Never use `ordinal()` for business logic or persistence:**
   `ordinal()` depends on the declared order of constants in source code. Reordering constants will silently corrupt database records or serialization protocols.
2. **Avoid mutable fields in enums:**
   Enums are global singletons. Adding mutable state (`private int counter;`) introduces concurrency race conditions across threads. All enum fields should be `private final`.