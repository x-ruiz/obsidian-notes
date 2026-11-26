---
tags:
  - software-engineering/design-patterns/creational-patterns
---
Separates the construction of a complex object from its representation, allowing the same construction process to produce different representations or configurations. **Used for simplifying the construction of a complex object**.

### Key Components
- **Product:** The complex object being built (usually immutable).
- **Builder:** An interface or class defining the step-by-step methods to configure attributes.
- **Director (Optional in modern Java):** An orchestrator that executes building steps in a specific sequence.

```java
public final class TableConfig {
    private final String tableName;
    private final String location;       // optional
    private final int commitRetries;     // optional with default
    private final boolean purgeOnDelete; // optional with default

    private TableConfig(Builder builder) {
        this.tableName = builder.tableName;
        this.location = builder.location;
        this.commitRetries = builder.commitRetries;
        this.purgeOnDelete = builder.purgeOnDelete;
    }

    public static Builder builder(String tableName) {
        return new Builder(tableName);
    }

    public static class Builder {
        private final String tableName; // Required
        private String location;
        private int commitRetries = 3;  // Default
        private boolean purgeOnDelete = false;

        public Builder(String tableName) {
            if (tableName == null || tableName.isBlank()) {
                throw new IllegalArgumentException("tableName cannot be empty");
            }
            this.tableName = tableName;
        }

		// fluent setters
        public Builder location(String location) {
            this.location = location;
            return this; // this is to chain setters
        }

        public Builder commitRetries(int retries) {
            this.commitRetries = retries;
            return this;
        }

        public Builder purgeOnDelete(boolean purge) {
            this.purgeOnDelete = purge;
            return this;
        }

        public TableConfig build() {
            // Validation invariants before instantiation
            // occurs here in order to validate final fluent setter values.
            return new TableConfig(this);
        }
    }
}
```

### Notes & Pitfalls
- **Solves the "Telescoping Constructor" Problem:** Eliminates constructor cascades (`new Foo(a)`, `new Foo(a, b)`, `new Foo(a, b, c)`).
- **Enables Immutability:** You can build an object with 15 optional parameters and still make all fields `final` without exposing mutable setters.
- **Overhead:** Creating the builder instance incurs slight memory and CPU overhead; avoid using it for trivial objects with only 1 or 2 mandatory fields.

> [!IMPORTANT]
> Do not use records with the builder pattern unless the builder is optional as records force the constructor to be public. Use POJO for traditional builder pattern classes where the constructor of the product class must remain hidden

# Object Creation Dilemma

The builder pattern gives the important ability to make fields final in a class, while allowing for optional fields to be set. Without the builder pattern, making an object with final fields would look like:

```java
public class Table {
	final String name;
	final String schema;
	
	public Table(String name, String schema) {
		this.name = name;
		this.schema = schema;
	}
}
```

In order for the client to only set one or the other, they must pass in null to one of the values. This introduces risk of passing in the wrong value to the wrong field as nothing enforces the order.

In order to solve this, normal objects use setters, including the name of the field in the method to ensure the correct field is being set. However, this means that the fields cannot be final and makes the object mutable post instantiation.

The builder pattern addresses this by combining the power of setters to non final fields within the builder class, and doing a one time set of all the [[Design Patterns#Product Class|product]] class fields during class instantiation (cannot set final fields with setters as the fields must be populated before object is created, and setters are only available after an object is created).

```mermaid
%%{init: {"theme": "base", "themeCSS": "svg { background-color: #09090B !important; border-radius: 8px; }", "themeVariables": {"darkMode": true, "background": "#09090B", "mainBkg": "#18181B", "textColor": "#F4F4F5", "primaryColor": "#18181B", "primaryTextColor": "#F4F4F5", "primaryBorderColor": "#7C3AED", "lineColor": "#A855F7", "secondaryColor": "#16141F", "tertiaryColor": "#0F0F14", "edgeLabelBackground": "#18181B", "clusterBkg": "#0F0F14", "clusterBorder": "#52525B", "titleColor": "#D8B4FE", "fontFamily": "Inter, ui-sans-serif, system-ui, -apple-system, sans-serif", "fontSize": "13px"}, "flowchart": {"curve": "basis", "nodeSpacing": 42, "rankSpacing": 56, "padding": 14}}}%%
flowchart TB
    subgraph Dilemma["1. The Object Creation Dilemma (Trade-Off Trap)"]
        direction LR
        Telescoping["Approach 1: Telescoping Constructor\nnew Table · positional null args\nPositional parameter binding"]
        JavaBeans["Approach 2: JavaBeans / Setters\ntable.setName · post-construction\nMutable state exposure"]

        TelescopingPro["Pro: 100% Immutable\nfinal fields · Thread-safe\nAtomic initialization"]
        TelescopingCon["Con: Positional Blindness\nNull padding · Arg-swap risk\nConstructor explosion"]

        JavaBeansPro["Pro: Named Methods\nNo argument swapping\nClean optional fields"]
        JavaBeansCon["Con: Forbids final Fields\nMutable post-creation\nHalf-initialized states"]

        Telescoping -->|"Strength"| TelescopingPro
        Telescoping -.->|"Failure Mode"| TelescopingCon
        JavaBeans -->|"Strength"| JavaBeansPro
        JavaBeans -.->|"Failure Mode"| JavaBeansCon
    end

    subgraph Solution["2. Builder Pattern Resolution (Two-Phase Lifecycle)"]
        direction TB
        Phase1["Phase 1: Assembly in Builder\nNon-final mutable fields · Defaults\nNamed fluent setters: .name 'events'"]
        BuildCall{"Transition: .build\nInvariant Validation Gate\nAtomic private constructor handoff"}
        Phase2["Phase 2: Immutable Product\n100% final fields assigned once\nThread-safe · Zero post-creation mutation"]

        Phase1 ==>|"Accumulated state"| BuildCall
        BuildCall ==>|"Invariants verified"| Phase2
    end

    Dilemma ==>|"Resolved by two-phase construction"| Solution

    classDef core fill:#2E1065,stroke:#A855F7,stroke-width:2px,color:#F5F3FF
    classDef service fill:#18181B,stroke:#7C3AED,stroke-width:1.5px,color:#F4F4F5
    classDef external fill:#27272A,stroke:#71717A,stroke-width:1.5px,color:#E4E4E7
    classDef async fill:#1E1B2E,stroke:#6B7280,stroke-width:1.25px,stroke-dasharray:4 3,color:#D4D4D8
    classDef guard fill:#3B0764,stroke:#E879F9,stroke-width:2px,stroke-dasharray:5 3,color:#FAF5FF

    class Phase2 core
    class TelescopingPro,JavaBeansPro,Phase1 service
    class Telescoping,JavaBeans external
    class TelescopingCon,JavaBeansCon async
    class BuildCall guard

    style Dilemma fill:#18181B,stroke:#52525B,stroke-width:1.25px,stroke-dasharray:5 4,color:#A1A1AA
    style Solution fill:#16141F,stroke:#6D28D9,stroke-width:1.25px,stroke-dasharray:4 3,color:#C4B5FD
```