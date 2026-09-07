---
tags:
  - software-engineering/design-patterns/creational-patterns
---
Provides an interface for creating familities of related or dependent objects without specifying their concrete classes. Aka: Kit Pattern.

## Applicability
Use the Abstract Factory Pattern when:
- a system should be independent of how its products are created, composed, and represented.
- a system should be configured with one of multiple families of products.
- a family of related product objects is designed to be used together, and you need to enforce this constraint.
- you want to provide a class library of products, and you want to reveal just their interfaces, not their implementations.
- you want to create services, not data objects (objects that perform execution functions)
## Participants
- AbstractFactory (e.g. LakehouseFactory)
	- Declares an [[Interface]] for operations that create abstract product objects
- ConcreteFactory (e.g. IcebergLakehouseFactory)
	- Implements the operations to create concrete [[Design Patterns#Product Class|product]] objects.
- AbstractProduct (e.g. Table, TableManager)
	- Declares an interface for a type of [[Design Patterns#Product Class|product]] object.
- ConcreteProduct (e.g. IcebergTable, IcebergTableManager)
	- Defines a [[Design Patterns#Product Class|product]] object to be created by the corresponding concrete factory.
	- Implements the AbstractProduct interface
- Client
	- Uses only interfaces declared by AbstractFactory and AbstractProduct classes
# Collaborations
- A single instance of a ConcreteFactory class is created at run-time. This concrete factory creates product objects having a specific implementation. To create different product objects, clients should use a different concrete factory.
- AbstractFactory defers creation of product objects to its ConcreteFactory subclass.
# Consequences
## Benefits
- Isolates concrete classes
	- The pattern assists in controlling the classes of objects that an application creates. Due to the factory encapsulating the responsibility and process of creating product objects, it isolates clients from implementation classes.
	- Clients manipulate instances through the abstract interfaces
	- Product class names are isolated in the implementation of the concrete factory, and do not appear in client code.
- Makes exchanging product families easy
	- The class of a concrete factory appears only once in an application, when instantiated. his makes it easy to change the concrete factory an application uses - do not have to go searching throughout the codebase for every instance to swap
	- Due to the abstract factory pattern creating a family of product classes, the entire family swaps when changing concrete factories.
- Promotes consistency among products
	- Allows enforcement of a family of related objects instead of relying on standards

## Liabilities
- Supporting new kinds of products is difficult
	- Extending abstract factories to produce new kinds of product classes is difficult. Supporting new kinds of products requires extending the factory interface, which involves changing the AbstractFactory class and all of the subclasses

