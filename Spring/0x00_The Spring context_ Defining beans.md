# Explanation
- In this chapter, we start learning how to work with a crucial Spring framework element: the **context** (also known as the **application context** in a Spring app).
- Imagine the context as a **place** in the memory of your app in which we add all the object instances that we want the **framework to manage**.
- By default, Spring _doesn’t know_ any of the objects you define in your application. To enable Spring to see your objects, you need to add them to the context.
- The context is a complex mechanism that enables Spring to control instances you define. This way, it allows you to use the capabilities the framework offers. We’ll name these object instances **beans**.
### Creating a Maven project
- Maven is not a subject directly related to Spring, but it’s a **tool** you use to easily _manage an app’s build process_ regardless of the framework you use.
- Some examples of tasks that are often part of building the app are as follows:
	- Downloading the dependencies needed by your app
	- Running tests
	- Validating that the syntax follows rules that you define
	- Checking for security vulnerabilities
	- Compiling the app
	- Packaging the app in an executable archive
- The default content of the `pom.xml` file immediately after creating the Maven project is:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
		 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion> 
	<groupId>org.example</groupId> 
	<artifactId>sq-ch2-ex1</artifactId> 
	<version>1.0-SNAPSHOT</version>
</project>
```
- To add external dependencies to your project. You write all the dependencies between the `<dependencies> </dependencies>` tags.
- Each dependency is represented by a `<dependency> </dependency>` group of tags where you write the dependency’s attributes: the dependency’s group ID, artifact name, and version.
- Maven will search for the dependency by the values you provided for these three attributes and will download the dependencies from a repository.
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
		 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion> 
	<groupId>org.example</groupId> 
	<artifactId>sq-ch2-ex1</artifactId> 
	<version>1.0-SNAPSHOT</version>
	<dependencies> 
		<dependency> 
			<groupId>org.springframework</groupId> 
			<artifactId>spring-jdbc</artifactId>
			<version>5.2.6.RELEASE</version>
		</dependency> 
	</dependencies>
</project>
```
### Adding new beans to the Spring context
- You can add beans in the context in the following ways:
	- Using the `@Bean` annotation
	- Using stereotype annotations
	- Programmatically
- We’ll consider a class named `Parrot` with only a `String` attribute representing the name of the parrot.
```Java
public class Parrot { 
	private String name;
	// Omitted getters and setters
}
```
- It’s now time to add the needed dependencies to our project. Because we’re using Maven, we’ll add the dependencies in the `pom.xml` file, as presented in the following:
```xml
<?xml version="1.0" encoding="UTF-8"?> 
<project xmlns="http://maven.apache.org/POM/4.0.0"
		 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
		 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion> 
	<groupId>org.example</groupId> 
	<artifactId>sq-ch2-ex1</artifactId> 
	<version>1.0-SNAPSHOT</version>
	
	<dependencies>  
	    <dependency>        
		    <groupId>org.springframework</groupId>  
	        <artifactId>spring-context</artifactId>  
	        <version>5.2.6.RELEASE</version>  
	    </dependency>
	</dependencies>
</project>
```
- A critical thing to observe is that Spring is designed to be **modular**. By modular, I mean that you _don’t need_ to add the whole Spring to your app when you use something out of the Spring ecosystem. You just need to add those parts that you use.
- For this reason, in the above, you see that we’ve only added the `spring-context` dependency, which instructs Maven to pull the needed dependencies for us to use the Spring context.
- With the dependency added to our project, we can create an instance of the Spring context.
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = new AnnotationConfigApplicationContext(); 
		Parrot p = new Parrot();
	} 
}
```
- Now you created an instance of Parrot, added the Spring context dependencies to your project, and created an instance of the Spring context. 
- Your objective is to add the Parrot object to the context, which is the next step.
##### Using the @Bean annotation to add beans into the Spring context
- The steps you need to follow to add a bean to the Spring context using the `@Bean` annotation are as follows:
	1.  Define a configuration class (annotated with `@Configuration`) for your project, which we use to configure the context of Spring.
	2. Add a method to the configuration class that returns the object instance you want to add to the context and annotate the method with the `@Bean`annotation.
		- This lets Spring know that it _needs to call_ this method when it initializes its context and adds the returned value to the context.
	3. Make Spring use the configuration class defined in step 1. As you’ll learn later, we use configuration classes to write different configurations for the framework.
	![[bean_annotation_steps.PNG]]
- NOTE that A configuration class is a **special** class in Spring applications that we use to instruct Spring to do specific actions. 
- For example, we can tell Spring to create beans or to enable certain functionalities.
- Observe that the name I used for the method doesn’t contain a verb. You probably learned that a Java best practice is to put verbs in method names because the methods generally represent actions. 
- But for methods we use to add beans in the Spring context, we don’t follow this convention. Such methods **represent** the object instances they return and that will now be part of the Spring context. 
- The method’s name also **becomes** the bean’s name (the bean’s name is now  `parrot`).
- To verify the Parrot instance is indeed part of the context now, you can refer to the instance and print its name in the console:
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = new AnnotationConfigApplicationContext(
						ProjectConfig.class); 
		Parrot p = context.getBean(Parrot.class);
		System.out.println(p.getName()); 
	} 
}
```
- The next shows you how I changed the configuration class to also add a bean of type `String` and a bean of type `Integer`.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	Parrot parrot() { 
		var p = new Parrot(); 
		p.setName("Koko");
		return p; 
	}

	@Bean
	String hello() { 
		return "Hello"; 
	}

	@Bean
	Integer ten() { 
		return 10; 
	} 
}
```
- You can now refer to these two new beans in the same way we did with the `parrot`.
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = 
			new AnnotationConfigApplicationContext( ProjectConfig.class); 
		Parrot p = context.getBean(Parrot.class);
		System.out.println(p.getName());

		String s = context.getBean(String.class); 
		System.out.println(s);

		Integer n = context.getBean(Integer.class); 
		System.out.println(n);
	} 
}
```
- The following shows you how we’ve declared three beans of type `Parrot` in the configuration class.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	Parrot parrot1() { 
		var p = new Parrot(); 
		p.setName("Koko");
		return p; 
	}

	@Bean
	Parrot parrot2() { 
		var p = new Parrot(); 
		p.setName("Miki");
		return p; 
	}

	@Bean
	Parrot parrot3() { 
		var p = new Parrot(); 
		p.setName("Riki");
		return p; 
	}
}
```
- You **can’t** get the beans from the context anymore by only specifying the type. If you do, you’ll get an **exception** because Spring **cannot guess** which instance you’ve declared you refer to.
- Running such a code throws an exception in which Spring tells you that you need to **be precise**, which is the instance you want to use.
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = 
			new AnnotationConfigApplicationContext(ProjectConfig.class);

		Parrot p = context.getBean(Parrot.class); 
		System.out.println(p.getName()); 
	} 
}
```
- When running your application, you’ll get an exception similar to that:
```
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'main.Parrot' available: expected single matching
bean but found 3:
	parrot1,parrot2,parrot3
	at …
```
- To solve this ambiguity problem, you need to **refer precisely** to one of the instances by using the bean’s name. 
- By default, Spring uses the _names of the methods_ annotated with `@Bean` as the beans’ names themselves.
- Let’s change the main method to refer to one of these beans explicitly by using its name.
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = 
			new AnnotationConfigApplicationContext(ProjectConfig.class);

		Parrot p = context.getBean("parrot2", Parrot.class); 
		System.out.println(p.getName()); 
	} 
}
```
- If you’d like to give another name to the bean, you can use either one of the name or the value attributes of the `@Bean` annotation. 
- Any of the following syntaxes will change the name of the bean in `miki`:
	- `@Bean(name = "miki")`.
	- `@Bean(value = "miki")`.
	- `@Bean("miki")`.
- There’s another option when referring to beans in the context when you have more of the same type. When you have multiple beans of the same kind in the Spring context you can make **one of them primary**.
- You mark the bean you want to be primary using the `@Primary` annotation. 
- A primary bean is the one Spring will choose if it has multiple options and you don’t specify a name; the primary bean is simply Spring’s default choice.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	Parrot parrot1() { 
		var p = new Parrot(); 
		p.setName("Koko");
		return p; 
	}

	@Bean
	@Primary
	Parrot parrot2() { 
		var p = new Parrot(); 
		p.setName("Miki");
		return p; 
	}

	@Bean
	Parrot parrot3() { 
		var p = new Parrot(); 
		p.setName("Riki");
		return p; 
	}
}
```
##### Using stereotype annotations to add beans to the Spring context
- Spring offers multiple stereotype annotations. But we will focus on how to use a stereotype annotation in general. 
- We’ll take the most basic of these, `@Component`, and use it to demonstrate our examples.
- With stereotype annotations, you add the annotation above the class for which you need to have an instance in the Spring context.
- When doing so, we say that you’ve marked the class as a component. When the app creates the Spring context, Spring creates an instance of the class you marked as a component and adds that instance to its context.
- We’ll still have a configuration class when we use this approach to tell Spring **where to look** for the classes annotated with stereotype annotations. Moreover, you can use both the approaches.
	 ![[stereotype_annotations.PNG]]
- By default, Spring doesn’t search for classes annotated with stereotype annotations, so if we just leave the code in step 1 as-is, Spring won’t add a bean of type `Parrot` in its context. 
- To tell Spring it needs to search for classes annotated with stereotype annotations, we use the `@ComponentScan` annotation over the configuration class.
- Also, with the `@ComponentScan` annotation, we tell Spring where to look for these classes.
- NOTE that We don’t need methods anymore to define the beans. And it now looks like this approach is better because you achieve the same thing by writing less code.
- You can continue writing the main method as presented before. But, by running the application, you’ll observe Spring added a `Parrot` instance to its context because the first value printed is the default String representation of this instance. 
- However, the second value printed (`System.out.println(p.getName()); `) is `null` because we did not assign any name to this parrot.
- Spring just creates the instance of the class, but it’s still our duty if we want to change this instance in any way afterward.
- A short comparison of the two most frequently encountered ways of adding beans to the Spring context:
	 ![[comparison.PNG|800]]
- What if we want to execute some instructions right after Spring creates the bean? We can use the `@PostConstruct` annotation. 
- Spring borrows the `@PostConstruct` annotation from Java EE. We can also use this annotation with Spring beans to specify a set of instructions Spring **executes after** the bean creation.
```Java
@Component
public class Parrot { 
	private String name;
	
	@PostConstruct
	public void init() { 
		this.name = "Kiki"; 
	}
	
	// Omitted code 
}
```
- Very similarly, but less encountered in real-world apps, you can use an annotation named `@PreDestroy`. 
- With this annotation, you define a method that Spring calls immediately before closing and clearing the context.
- But generally I recommend developers avoid using it and find a different approach to executing something before Spring clears the context, mainly because you can expect Spring to fail to clear the context. 
- Say you defined something sensitive (like closing a database connection) in the `@PreDestroy` method; if Spring doesn’t call the method, you may get into big problems.
### Programmatically adding beans to the Spring context
- We’ve had the option of programmatically adding beans to the Spring context with Spring 5, which offers great flexibility because it enables you to add new instances in the context directly by calling a method of the context instance.
- You’d use this approach when you want to implement a custom way of adding beans to the context and the `@Bean` or the stereotype annotations are not enough for your needs.
- Say you need to register specific beans in the Spring context depending on specific configurations of your application. 
- With the `@Bean` and stereotype annotations, you can implement most of the scenarios, but you can’t do something like the code presented in the next snippet:
```Java
if (condition) { 
	registerBean(b1); 
} else { 
	registerBean(b2); 
}
```
- To add a bean to the Spring context using a programmatic approach, you just need to call the `registerBean()` method of the `ApplicationContext` instance. 
- The `registerBean()` has four parameters, as presented in the next code snippet:
```Java
<T> void registerBean( 
	String beanName, 
	Class<T> beanClass, 
	Supplier<T> supplier, 
	BeanDefinitionCustomizer... customizers);
```
- Use the first parameter `beanName` to define a name for the bean you add in the Spring context. 
	- If you don’t need to give a name to the bean you’re adding, you can use `null` as a value when you call the method.
- The second parameter is the class that defines the bean you add to the context.
	- Say you want to add an instance of the class `Parrot`; the value you give to this parameter is `Parrot.class`.
- The third parameter is an instance of `Supplier`. The implementation of this `Supplier` needs to return the value of the instance you add to the context.
	- Remember, `Supplier` is a functional interface you find in the `java.util.function` package. 
	- The purpose of a supplier implementation is to return a value you define without taking parameters.
- The fourth and last parameter is a `varargs` of `BeanDefinitionCustomizer`. The `BeanDefinitionCustomizer` is just an interface you implement to configure different characteristics of the bean; e.g., making it `Primary`. 
	- Being defined as a `varargs` type, you can omit this parameter entirely, or you can give it more values of type `BeanDefinitionCustomizer`.
- Example of Using the `registerBean()` method to add a bean to the Spring context
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = 
			new AnnotationConfigApplicationContext(ProjectConfig.class); 

		Parrot x = new Parrot(); 
		x.setName("Kiki");
 
		Supplier<Parrot> parrotSupplier = () -> x;

		context.registerBean("parrot1", Parrot.class, parrotSupplier);

		Parrot p = context.getBean(Parrot.class);
		System.out.println(p.getName());
	} 
}
```
- Use one or more bean configurator instances as the last parameters to set different characteristics of the beans you add. 
- For example, you can make the bean primary by changing the `registerBean()` method call, as shown in the next code snippet.
```Java
context.registerBean("parrot1", Parrot.class, 
					  parrotSupplier, bc -> bc.setPrimary(true));
```
# Sources
- Spring Start Here - Chapter 2.
- [Spring Start Here - Chapter 2 - Episode 2](https://www.youtube.com/watch?v=b8ocrkawS38&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=8&pp=iAQB).
- [Spring Start Here - Chapter 2 - Episode 3](https://www.youtube.com/watch?v=uj9St3Rcehg&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=7).