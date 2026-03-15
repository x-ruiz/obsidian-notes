---
tags:
  - spring
  - frameworks
  - software-engineering
---
Place in memory of the application in which all object instances that the framework manages lives. Object instances that are included in the application context are called [[Spring Beans]] - **not all objects in the application need to be added to the application context.**

# Context Implementations

There are various context implementations, however in most cases, the ```AnnotationConfigApplicationContext``` class is used (annotation driven with the ```@Bean``` annotation). 

# Configuration Classes

Classes that define various Spring-related configurations for the project to be added to the context. Can be used to add new object instances ([[Spring Beans]]) to the Spring application context (see [[Spring Beans#@Bean Annotation]]).