# Explanation
- Here we discuss two ways you can establish the relationships among beans:
	- Link the beans by directly calling the methods that create them (which we’ll call _wiring_).
	- Enable Spring to provide us a value using a method parameter (which we’ll call _auto-wiring_).
### Implementing relationships among beans defined in the configuration file
- Say we have two instances in the Spring context: a parrot and a person. We’ll create and add these instances to the context. 
- We want to make the person own the parrot. In other words, we need to link the two instances.
- This straightforward example helps us discuss the two approaches for linking the beans in the Spring context without adding unnecessary complexity. So, for each of the two approaches (_wiring_ and _auto-wiring_), we have two steps:
	1. Add the person and parrot beans to the Spring context.
	2. Establish a relationship between the person and the parrot.
	 ![[has a.PNG]]
- You can now write a Main class, as presented, and check that the two instances aren’t yet linked to one another.
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = 
			new AnnotationConfigApplicationContext (ProjectConfig.class);

		Person person = context.getBean(Person.class); 
		Parrot parrot = context.getBean(Parrot.class);
		System.out.println( "Person's name: " + person.getName());

		System.out.println( "Parrot's name: " + parrot.getName()); 
		System.out.println( "Person's parrot: " + person.getParrot()); 
	}
}
```
- When running this app, you’ll see a console output similar to the one presented in the next code snippet:
```
Person's name: Ella 
Parrot's name: Koko
Person's parrot: null
```
- The most important thing to observe here is that the person’s parrot is `null`. Both the person and the parrot instances are in the context, however. 
- This output is `null`, which means there’s not yet a relationship between the instances.
##### Wiring the beans using a direct method call between the [[0x00_The Spring context_ Defining beans#Using the @Bean annotation to add beans into the Spring context|@Bean]] methods
- The first way (_wiring_) to achieve this is to call one method from another in the configuration class.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	public Parrot parrot() { 
		Parrot p = new Parrot(); 
		p.setName("Koko"); 
		return p; 
	}

	@Bean
	public Person person() { 
		Person p = new Person();
		p.setName("Ella"); 
		p.setParrot(parrot()); 
		return p; 
	} 
}
```
- Now you find (see next snippet) that the second line shows that Ella (the person in the Spring context) owns Koko (the parrot in the Spring context):
```
Person's name: Ella
Person's parrot: {Parrot : Koko}
```
- Doesn’t this mean that we create two instances of Parrot—one instance Spring creates and adds into its context and another one when the `person()` method makes the direct call to the `parrot()` method? 
- No, we actually have **only one parrot** instance in this application overall. It might look strange at first, but Spring is smart enough to understand that by calling the `parrot()` method, you want to refer to the parrot bean in its context.
- If the parrot bean already exists in the context, then instead of calling the `parrot()` method, Spring will directly take the instance from its context. 
- If the parrot bean does not yet exist in the context, Spring calls the `parrot()` method and returns the bean.
##### Wiring the beans using the @Bean annotated method’s parameters
- Instead of directly calling the method that defines the bean we wish to refer to, we **add a parameter** to the method of the corresponding type of object, and we **rely on Spring** to provide us a value through that parameter.
- With this approach, it doesn’t matter if the bean we want to refer to is defined with a method annotated with [[0x00_The Spring context_ Defining beans#Using the @Bean annotation to add beans into the Spring context|@Bean]] or using a [[0x00_The Spring context_ Defining beans#Using stereotype annotations to add beans to the Spring context|stereotype annotation]] like `@Component`.
	![[parameter.PNG|800]]
- Take a look at the `person()` method. It now receives a parameter of type `Parrot`, and we set the reference of that parameter to the returned person’s attribute. 
- When calling the method, Spring knows it has to find a parrot bean in its context and **inject** its value into the parameter of the `person()` method.
- In the above point we used the word “**inject**.” We refer here to what we will from now on call **dependency injection (DI)**. 
- As its name suggests, **DI** is a technique involving the framework **setting a value** into a specific field or parameter. 
- In our case, Spring sets a particular value into the parameter of the `person()` method when calling it and resolves a dependency of this method. 
- DI is an application of the **IoC principle**, and IoC implies that the **framework controls** the application at execution.
	 ![[IoC.PNG]]
### Using the @Autowired annotation to inject beans
- Using the `@Autowired` annotation, we mark an object’s property where we want Spring to inject a value from the context, and we mark this intention directly in the class that defines the object that needs the dependency.
- As you’ll see, there are three ways we can use the @Autowired annotation:
	- Injecting the value in the field of the class, which you usually find in examples and proofs of concept.
	- Injecting the value through the constructor parameters of the class approach that you’ll use most often in real-world scenarios.
	- Injecting the value through the setter, which you’ll rarely use in production-ready code.
##### Using @Autowired to inject the values through the class fields
- Using the annotation **over** the field tell Spring we want to **inject** a value there from its context.
	 ![[wired class fields.PNG]]
- Why is this approach not desired in production code? It’s not totally wrong to use it, but you want to make sure you make your app maintainable and testable in production code. By injecting the value directly in the field:
	- it’s more difficult to manage the value yourself at initialization.
	- you don’t have the option to make the field final (see next code snippet), and this way, make sure no one can change its value after initialization:
```Java
@Component
public class Person { 
	private String name = "Ella";
	
	@Autowired
	private final Parrot parrot;
}
```
- This doesn’t compile. You cannot define a final field without an initial value.
##### Using @Autowired to inject the values through the constructor
- This approach is the one used most often in production code. It enables you to define the **fields as final**, ensuring no one can change their value after Spring initializes them.
	 ![[constructor autowired.PNG]]
- NOTE that Starting with Spring version 4.3, when you only have one constructor in the class, you can **omit** writing the `@Autowired` annotation.
##### Using dependency injection through the setter
- This approach has more disadvantages than advantages: 
	- it’s more challenging to read 
	- it doesn’t allow you to make the field final 
	- it doesn’t help you in making the testing easier.
```Java
@Component
public class Person { 
	private String name = "Ella"; 
	private Parrot parrot;
	
	// Omitted getters and setters 
	
	@Autowired
	public void setParrot(Parrot parrot) {
		this.parrot = parrot; 
	} 
}
```
### Dealing with circular dependencies
- A circular dependency is a situation in which, to create a bean (let’s name it Bean A), Spring needs to **inject another bean** that doesn’t exist yet (Bean B). But Bean B also requests a **dependency to Bean A**. 
- So, to create Bean B, Spring needs first to have Bean A. Spring is now in a _deadlock_. It cannot create Bean A because it needs Bean B, and it cannot create Bean B because it needs Bean A.
	 ![[cicular dependency.PNG]]
- A circular dependency is easy to avoid. You just need to make sure you don’t define objects whose creation depends on the other. 
- Having dependencies from one object to another like this is a bad design of classes. In such a case, you need to rewrite your code.
### Choosing from multiple beans in the Spring context
- Here, we discuss the scenario in which Spring needs to inject a value into a parameter or class field but has multiple beans of the same type to choose from.
- Say you have three Parrot beans in the Spring context. You configure Spring to inject a value of type Parrot into a parameter. How will Spring behave?
- Depending on your implementation, you have the following cases:
	1. The identifier of the parameter matches the name of one of the beans from the context. In this case, Spring will choose the bean for which the name is the same as the parameter.
	2. The identifier of the parameter doesn’t match any of the bean names from the context. Then you have the following options:
		-  You marked one of the beans as primary. In this case, Spring will select the primary bean for injection.
		- You can explicitly select a specific bean using the `@Qualifier` annotation.
		- If none of the beans is primary and you don’t use `@Qualifier`, the app will fail with an exception, complaining that the context contains more beans of the same type and Spring doesn’t know which one to choose.
- The following shows you a configuration class that defines two `Parrot` instances and uses injection through the method parameters.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	public Parrot parrot1() { 
		Parrot p = new Parrot(); 
		p.setName("Koko"); 
		return p; 
	}

	@Bean
	public Parrot parrot2() { 
		Parrot p = new Parrot(); 
		p.setName("Miki"); 
		return p; 
	}

	@Bean
	public Person person(Parrot parrot2) {
		Person p = new Person(); 
		p.setName("Ella"); 
		p.setParrot(parrot2); 
		return p; 
	} 
}
```
- Running the app with this configuration, you’d observe a console output similar to the next code snippet. 
- Observe that Spring linked the `person` bean to the `parrot` named `Miki` because the bean representing this `parrot` has the name `parrot2`
```
Parrot created 
Person's name: Ella
Person's parrot: {Parrot : Miki}
```
- In a real-world scenario, it is recommended to avoid relying on the name of the parameter, which could be easily refactored and changed by mistake by another developer. 
- We usually choose a more visible approach to express our intention to inject a specific bean: using the `@Qualifier` annotation.
```java
@Configuration
public class ProjectConfig {
	@Bean
	public Parrot parrot1() { 
		Parrot p = new Parrot(); 
		p.setName("Koko"); 
		return p; 
	}

	@Bean
	public Parrot parrot2() { 
		Parrot p = new Parrot(); 
		p.setName("Miki"); 
		return p; 
	}

	@Bean
	public Person person( 
				@Qualifier("parrot2") Parrot parrot) {
		Person p = new Person(); 
		p.setName("Ella"); 
		p.setParrot(parrot2); 
		return p; 
	} 
}
```
# Sources
- Spring Start Here - Chapter 3.
- [Spring Start Here - Chapter 3 - Episode 4](https://www.youtube.com/watch?v=CB7Tzu6-16M&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=6&pp=iAQB).
- [Spring Start Here - Chapter 3 - Episode 5](https://www.youtube.com/watch?v=ZoeaLEOjayM&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=5).