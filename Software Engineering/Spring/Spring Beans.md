---
tags:
  - spring
  - frameworks
  - software-engineering
---
Beans are object instances (of any kind - e.g String, Integer, or objects of classes custom defined) that are added to the [[Spring Context]]. There are multiple ways to add beans to the application context:
1. Using the ```@Bean``` annotation
2. Using stereotype annotations
3. Programmatically

# @Bean Annotation

The `@Bean` annotation instructs Spring to call this method when [[Spring Context]] initializes and add the returned value to the context.
1. Java stores `@Bean`on the method.
2. Spring scans configuration classes passed to application context.
3. Spring finds methods marked`@Bean`
4. Spring turns each one into a **BeanDefinition**.
5. When needed, Spring calls that method and stores the result as a managed bean.
6. For `@Configuration` classes, Spring often enhances the class with a proxy so bean method calls behave correctly 

Conceptually, a `@Bean` annotation is not executable logic, but rather **data that Spring can inspect**

```java
// example of what spring will evaluate
Method m = ProjectConfig.class.getDeclaredMethod("parrot");  
boolean present = m.isAnnotationPresent(Bean.class);
```

## Adding a Single Bean

Below is the normal way to initialize a class:

```java
public class Main {  
    public static void main(String[] args) {
	    // can use var here because type is easily inferred
	    // standard way to initialize an instance of a class
        Parrot p = new Parrot();  
    }  
}
```


Below is the ```@Bean``` annotation way:
1. Define a configuration class
2. Add a method to the config class that returns the object instance to add to context
```java
import org.springframework.context.annotation.Configuration;  
import org.springframework.context.annotation.Bean;
 
@Configuration  
public class ProjectConfig {  
      
    /* Method does not use verb e.g getParrot because it
	represents the object to be added to context as a whole.
    The method name also becomes the bean name which should be
    a noun */
    @Bean  
    public Parrot parrot() {  
        var p = new Parrot();  
        p.setName("Koko");  
        return p;  
    }  
}
```

3. Initialize [[Spring Context]]  and tell it to use the config class
```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;  
  
public class Main {  
    public static void main(String[] args) {
	
	// Spring Context wants class metadata, not instance of class
	// hence why we pass in ProjectConfig.class
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);  
        Parrot p = new Parrot();  
    }  
}
```

4. Validate that the object instance was added to the context
	1. If the bean of that class does not exist in the context, an exception will be thrown
```java
public static void main(String[] args) {  
    var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
    
	// Get the bean of class Parrot  
    Parrot p = context.getBean(Parrot.class);
  
    System.out.println(p.getName());
    // Output: Koko
}
```

## Adding Multiple Beans of Same Type
- [ ] Fill out section