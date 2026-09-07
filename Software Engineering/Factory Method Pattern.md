---
tags:
  - creational-patterns
  - design-patterns
  - software-engineering
---

Defines an interface for creating an object, but allows subclasses to decide which class to instantiate. It defers instantiation to subclasses via **inheritance and polymorphism** (virtual constructors).
### Key Components
- **Product:** The interface for the object created.
- **Creator:** Declares the factory method returning a `Product`.
- **Concrete Creators:** Override the factory method to return concrete implementations of `Product.

```java
// Product
public interface TableWriter {
    void write(List<byte[]> records);
}

// Concrete Products
public class ParquetTableWriter implements TableWriter {
    @Override public void write(List<byte[]> records) { /* Parquet encoding */ }
}
public class OrcTableWriter implements TableWriter {
    @Override public void write(List<byte[]> records) { /* ORC encoding */ }
}

// Creator
public abstract class DataPipeline {
    // Core business logic operates on the interface
    public void processAndIngest(List<byte[]> rawData) {
        TableWriter writer = createWriter(); // Delegated to subclass
        writer.write(rawData);
    }

    // The Factory Method
    protected abstract TableWriter createWriter();
}

// Concrete Creators
public class ParquetPipeline extends DataPipeline {
    @Override
    protected TableWriter createWriter() {
        return new ParquetTableWriter();
    }
}
```

### Notes & Pitfalls
- **Open/Closed Principle:** New product types can be introduced without breaking existing client code.
- **Subclass Explosion:** Every new product variation may require a new concrete creator class just to override `createWriter()`.
- **Contrast with Template Method:** Factory Method is frequently used as a step within a Template Method (as in `processAndIngest()` above).
