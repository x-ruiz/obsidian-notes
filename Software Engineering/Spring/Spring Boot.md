---
tags:
  - frameworks
  - spring-projects
  - spring
---
A toolkit that makes using Spring ecosystem fast, automatic, and production-ready. Introduces the concept of *convention over configuration*. Instead of requiring you to setup the framework yourself, Spring Boot offers a default configuration that can be customized. 

The result is that in general, less code is written because conventions are followed and each application is similar to others. Easier to start with a default configuration and only change whats different between use cases and deviate from the convention.
## Auto‑Configuration
Spring Boot looks at your classpath and automatically configures Spring components.
Example:
- You add spring-boot-starter-web

→ Boot auto-configures Spring MVC, Tomcat, JSON serialization, etc.
## Starters
Boot provides curated dependency bundles called starters:

- spring-boot-starter-web
- spring-boot-starter-data-jpa
- spring-boot-starter-security

Each starter pulls in the correct Spring modules + third‑party libs.
## Embedded Servers
- Embedded Tomcat
- Embedded Jetty
- Embedded Netty