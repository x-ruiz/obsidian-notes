---
tags:
  - frameworks
  - spring-projects
  - spring
  - software-engineering
---
A toolkit that makes using Spring ecosystem fast, automatic, and production-ready. Introduces the concept of *convention over configuration*. Instead of requiring you to setup the framework yourself, Spring Boot offers a default configuration that can be customized. 

The result is that in general, less code is written because conventions are followed and each application is similar to others. Easier to start with a default configuration and only change whats different between use cases and deviate from the convention.
## Auto‑Configuration
Spring Boot looks at your classpath and automatically configures Spring components.
Example:
- You add spring-boot-starter-web

→ Boot auto-configures Spring MVC, Tomcat, JSON serialization, etc.
## Starters
Boot provides curated dependency bundles called starters. Each starter bundles dependencies together and can map to the original spring framework or spring projects.

| Spring Boot Starter              | What It Pulls In                        | Key Underlying Libraries                                                                 |
| -------------------------------- | --------------------------------------- | ---------------------------------------------------------------------------------------- |
| spring-boot-starter              | [[Spring Core]] + logging               | spring-boot, spring-boot-autoconfigure, spring-core, spring-context, slf4j, logback      |
| spring-boot-starter-web          | [[Spring MVC]] + JSON + embedded Tomcat | spring-web, spring-webmvc, spring-boot-starter-json, jackson-databind, tomcat-embed-core |
| spring-boot-starter-webflux      | Reactive web stack                      | spring-webflux, reactor-core, reactor-netty                                              |
| spring-boot-starter-data-jpa     | [[Spring Data]] JPA + Hibernate         | spring-data-jpa, hibernate-core, jakarta.persistence-api, spring-jdbc, spring-tx         |
| spring-boot-starter-jdbc         | JDBC support                            | spring-jdbc, spring-tx, HikariCP                                                         |
| spring-boot-starter-security     | [[Spring Security]]                     | spring-security-core, spring-security-web, spring-security-config                        |
| spring-boot-starter-test         | Testing stack                           | spring-test, junit-jupiter, mockito-core, assertj-core, json-path                        |
| spring-boot-starter-validation   | Bean validation                         | jakarta.validation-api, hibernate-validator                                              |
| spring-boot-starter-thymeleaf    | Thymeleaf templating                    | thymeleaf, thymeleaf-spring6, spring-webmvc                                              |
| spring-boot-starter-actuator     | Metrics + health checks                 | spring-boot-actuator, Micrometer                                                         |
| spring-boot-starter-data-mongodb | MongoDB support                         | spring-data-mongodb, MongoDB Java driver                                                 |
| spring-boot-starter-data-redis   | Redis support                           | spring-data-redis, Lettuce client                                                        |
| spring-boot-starter-amqp         | RabbitMQ support                        | spring-amqp, spring-rabbit                                                               |
| spring-boot-starter-batch        | [[Spring Batch]] processing             | spring-batch-core, spring-batch-infrastructure                                           |

## Embedded Servers
- Embedded Tomcat
- Embedded Jetty
- Embedded Netty