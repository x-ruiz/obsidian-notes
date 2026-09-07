---
tags:
  - design-patterns
  - creational-patterns
  - software-engineering
---
Separates the construction of a complex object from its representation, allowing the same construction process to produce different representations or configurations.
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
            return new TableConfig(this);
        }
    }
}
```

### Notes & Pitfalls
- **Solves the "Telescoping Constructor" Problem:** Eliminates constructor cascades (`new Foo(a)`, `new Foo(a, b)`, `new Foo(a, b, c)`).
- **Enables Immutability:** You can build an object with 15 optional parameters and still make all fields `final` without exposing mutable setters.
- **Overhead:** Creating the builder instance incurs slight memory and CPU overhead; avoid using it for trivial objects with only 1 or 2 mandatory fields.