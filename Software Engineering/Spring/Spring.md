---
tags:
  - spring
  - frameworks
  - spring-frameworks
  - spring-projects
  - software-engineering
---
Spring is often called a “framework,” but in reality it is an ecosystem of frameworks and projects. At the center is the Spring Framework, a set of core modules that define the programming model.
Surrounding it is a large family of Spring Projects, each solving specialized problems such as data access, security, cloud integration, messaging, and application bootstrap. This layered architecture is what gives Spring its flexibility and power.

![[spring-ecosystem.excalidraw.md]]
Both the Spring Framework and all Spring Projects are fully open source under the Apache 2.0 license. The distinction between “framework modules” and “projects” is architectural and organizational, not licensing.

- Spring Framework = the original, monolithic set of core modules
- Spring Projects = independent, specialized extensions built on top
- Spring Boot = one of the Spring Projects, providing auto‑configuration and application bootstrap
- The ecosystem is designed so the core provides the programming model, while the projects provide domain‑specific capabilities
- Spring is designed to be modular, only installing dependencies of functionality you need, instead of the whole ecosystem

This structure is why Spring remains one of the most flexible and widely used ecosystems in modern backend development.

***There are alternatives for each framework and project that should be considered***
- Spring -> Jakarta EE
- [[Spring Data]] -> JPA
- [[Spring Security]] -> Apache Shiro
- [[Spring MVC]] -> Play Framework

---
# Spring Frameworks

These are the original modules that defined Spring. They all live in the same repository, share the same release cycle, and form the foundation of everything else in the ecosystem.

**Core Modules:**
- [[Spring Core]]
	- IoC container, Dependency Injection, bean lifecycle
- [[Spring MVC]]
	- Web framework for REST and MVC applications
- [[Spring Data Access]]
	- Abstractions over JDBC, transactions, ORM integration
- [[Spring AOP]]
	- Aspect‑oriented programming support
- [[Spring Testing]]
	- Test utilities, mocks, integration test support

These modules define the core programming model that all other Spring technologies rely on.

---

# Spring Projects

As Spring grew, the team created independent, open‑source projects that extend the core framework. These live in separate repositories, have independent versioning, and provide higher‑level or domain‑specific capabilities.

**Projects**
- [[Spring Data]]
	- Unified data access across databases
- [[Spring Security]]
	- Authentication, authorization, OAuth2
- [[Spring Cloud]]
	- Microservices patterns, service discovery, config server
- [[Spring Batch]] 
	- Batch processing
- [[Spring Integration]] 
	- Enterprise integration patterns
- [[Spring Kafka / AMQP]]
	- Messaging abstractions
- [[Spring for GraphQL]]
	- GraphQL support
- [[Spring Boot]] 
	- Auto‑configuration, starters, embedded servers, opinionated defaults

These projects depend on Spring Core, but are not part of the Spring Framework itself.

---

# Use Cases
## Development of a Backend Application
![[Screenshot 2026-03-14 at 11.22.38 PM.png]]
![[Screenshot 2026-03-14 at 11.24.11 PM.png]]
## Development of an Automation Testing Framework
![[Screenshot 2026-03-14 at 11.40.06 PM.png]]

# When Not to Use Spring
- Needing to implement functionality with a small footprint (small % of memory occupied by app files and dependencies)
	- e.g serverless functions, which require to be small and have fast start up times to be effective.
- Specific security requirements that force specific implementation without open source frameworks
- Having to make so many modifications to the base configurations (mentioned in [[Spring Boot]]) that you will write more code modifying than building from scratch or using a different tool.
- You already have a functioning app and refactoring to a framework does not provide a benefit that outweighs the effort and time
