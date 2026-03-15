---
tags:
  - spring
  - frameworks
---
A toolkit that makes using Spring ecosystem fast, automatic, and production-ready. 
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