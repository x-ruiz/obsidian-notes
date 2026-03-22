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

1. Add multiple beans to the configuration class
```java
import org.springframework.context.annotation.Configuration;  
import org.springframework.context.annotation.Bean;
 
@Configuration  
public class ProjectConfig {  
      
    @Bean  
    @Primary // will return if multiple are present in context
    public Parrot parrot() {  
        var p = new Parrot();  
        p.setName("Koko");  
        return p;  
    }
    
    @Bean(name = "miki") // can change name of bean
    public Parrot parrot2() {  
        var p = new Parrot();  
        p.setName("Miki");  
        return p;  
    }  
}
```

When adding multiple beans of the same type to the [[Spring Context]] you can no longer retrieve them by [[Java#.class]]: Spring will throw an exception telling you to be precise (if no `@Primary` annotation is present).

2. Retrieve beans by instance type (name of bean registered)
```java
public static void main(String[] args) {  
    var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
    
	// Get the bean of class Parrot  
    Parrot p = context.getBean("miki", Parrot.class);
  
    System.out.println(p.getName());
    // Output: Koko
}
```

# Stereotype Annotations

Alternative ways to add beans to [[Spring Context]] with less boilerplate code and does not require directly adding to context through configuration classes. With stereotype annotations, you add the annotation above the class you want to have an instance of in the context.

## @Component Annotation

The component annotation marks a class to be directly added to the context. With [[Spring Context#AnnotationConfigApplicationContext]] and setting up with context manually, in order for components to be added, a corresponding `@ComponentScan` annotation must be added to the configuration class. If using [[Spring Context#SpringApplication]] with [[Spring Boot]], boot has an annotation `@SpringBootApplication` that packages annotations including the `@ComponentScan` automatically to discover `@Component` annotations and such.

```java
// Configuration Class
@Configuration
@ComponentScan(basePackages = "main") // where to find beans/components
public class ProjectConfig {}

// Component Class
@Component
public class Parrot {  
    private String name;  
  
    public String getName() { return this.name; }  
  
    public void setName(String name) {this.name = name; }  
}

// output: null
// outputs null because component creates the instance without a name
```

For real world scenarios, stereotype annotations are preferred as they provide less boilerplate code. @Bean annotations are used when you cannot add beans through stereotype annotations.
# Comparison Table

| @Bean                                                                                              | Stereotype Annotations                                                          |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Full control over instance creation and properties set before being added to context               | Only have control over the instance after framework creates and adds to context |
| Can add more instances of the same type to context                                                 | Can add only one instance of the class to context                               |
| Can use annotation to create beans of any class, even classes you don't own like String or Integer | Can only add beans of classes you created within your package                   |
| Need to write a separate method for each bean, which is an increase of boilerplate code            | Can directly add annotation to the class creation, reducing boilerplate code    |
