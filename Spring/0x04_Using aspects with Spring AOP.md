# Explanation
- Aspects are a way the framework **intercepts** method calls and possibly **alters** the execution of methods.
### How aspects work in Spring
- An aspect is simply a **piece of logic** the framework executes when you **call specific methods** of your choice. When designing an aspect, you define the following:
	- _What_ code you want Spring to execute when you call specific methods. This is named an _aspect_.
	- _When_ the app should execute this logic of the aspect (e.g., before or after the method call, instead of the method call). This is named the _advice_.
	- _Which_ methods the framework needs to intercept and execute the aspect for them. This is named a _pointcut_.
- With aspects terminology, you’ll also find the concept of a _join point_, which defines the event that **triggers the execution** of an aspect. But with Spring, this event is **always a method call**.
- As in the case of the dependency injection, to use aspects you need the framework to manage the objects for which you want to apply aspects.
- The [[0x00_The Spring context_ Defining beans|bean]] that declares the method intercepted by an aspect is named the _target object_.
	 ![[aspect.PNG]]
- But how does Spring intercept each method call and apply the aspect logic? As discussed earlier, the object needs to be a bean in the Spring context. 
- But because you made the object an aspect target, Spring **won’t directly give** you an instance reference for the bean when you request it from the context. 
- Instead, Spring gives you an object that **calls the aspect logic** instead of the actual method. We say that Spring gives you a _proxy_ object instead of the real bean.
- You will now receive the proxy instead of the bean anytime you get the bean from the context, either if you directly use the `getBean()` method of the context or if you use DI. This approach is named _weaving_.
	![[weaving.PNG]]
	 ![[aspect_comparison.PNG]]
### Implementing aspects with Spring AOP
- In addition to the `spring-context` dependency, for this example we also need the `spring-aspects` dependency.
```Java
<dependency> 
	<groupId>org.springframework</groupId> 
	<artifactId>spring-context</artifactId> 
	<version>5.2.8.RELEASE</version> 
</dependency> 

<dependency> 
	<groupId>org.springframework</groupId> 
	<artifactId>spring-aspects</artifactId> 
	<version>5.2.8.RELEASE</version> 
</dependency>
```
- To make our example shorter and allow you to focus on the syntax related to aspects, we’ll only consider one service object named `CommentService` and a use case it defines named `publishComment(Comment comment)`. 
- This method, defined in the `CommentService` class, receives a parameter of type `Comment`. `Comment` is a model class and is presented in the next code snippet:
```Java
public class Comment { 
	private String text; 
	private String author;

	// Omitted getters and setters
}
```
- We annotate the `CommentService` class with the `@Service` stereotype annotation to make it a bean in the Spring context.
```Java
@Service
public class CommentService { 
	private Logger logger =
		Logger.getLogger(CommentService.class.getName()); 
	
	public void publishComment(Comment comment) {
		logger.info("Publishing comment:" + comment.getText()); 
	} 
}
```
- We also need to add a configuration class to tell Spring where to look for the classes annotated with stereotype annotations.
```Java
@Configuration 
@ComponentScan(basePackages = "services")
public class ProjectConfig {
	
}
```
- Let’s write the `Main` class that calls the `publishComment()` method in the service class and observe the current behavior, as shown in the following:
```Java
public class Main {  
  
	public static void main(String[] args) {  
	    var c = 
		    new AnnotationConfigApplicationContext(ProjectConfig.class);  
  
	    var service = c.getBean(CommentService.class);  
  
	    Comment comment = new Comment();  
	    comment.setText("Demo comment");  
	    comment.setAuthor("Natasha");  
	    service.publishComment(comment);  
  }  
}
```
- Let’s now enhance the project with an aspect class that intercepts the method call and adds an output before and after the call. To create an aspect, you follow these steps:
	1. Enable the aspect mechanism in your Spring app by annotating the configuration class with the `@EnableAspectJAutoProxy` annotation.
	2. Create a new class, and annotate it with the `@Aspect` annotation. Using either `@Bean` or stereotype annotations, add a bean for this class in the Spring context.
	3. Define a method that will implement the aspect logic and tell Spring when and which methods to intercept using an advice annotation.
	4. Implement the aspect logic.
	![[aspect_steps.PNG|750]]
- The `@Aspect` annotation isn’t a stereotype annotation. Using `@Aspect`, you tell Spring that the class implements the definition of an aspect, but Spring **won’t** also create a bean for this class. 
- We need to explicitly use one of the syntaxes we learned to create a bean for your class and allow Spring to manage it this way.
- Other than using the `@Around` annotation, you also observe we’ve written an unusual string expression as the value of the annotation, and we have added a parameter to the aspect method. What are these?
- The peculiar expression used as a parameter to the `@Around` annotation tells Spring **which method calls to intercept**.
- This expression language is called **AspectJ pointcut** language. When you need to write such an expression, you can refer to [this documentation ](http://mng.bz/4K9g).
- The expression we means Spring intercepts **any method** defined in a class that is in the services package, **regardless** of the method’s return type, the class it belongs to, the name of the method, or the parameters the method receives.
	![[AspectJ pointcut.PNG]]
- Now let’s look at the second element we’ve added to the method: the `ProceedingJoinPoint` parameter, which represents the **intercepted method**. 
- The main thing you do with this parameter is tell the aspect when it should delegate further to the actual method.
- In the following, we’ve added the logic for our aspect.
```java
@Aspect
public class LoggingAspect {
	// Defines which are the intercepted methods
	@Around("execution(* services.*.*(..))") 
	public void log(ProceedingJoinPoint joinPoint) throws Throwable {
		// Prints a message in the console before 
		// the intercepted method’s execution
		logger.info("Method will execute");.

		//Delegates (Calls) to the actual intercepted method
		joinPoint.proceed(); 
		
		// Prints a message in the console after 
		// the intercepted method’s execution
		logger.info("Method executed");
	}
}
```
- Now the aspect
	1. Intercepts the method
	2. Displays something in the console before calling the intercepted method
	3. Calls the intercepted method
	4. Displays something in the console after calling the intercepted method
	![[aspects behavior.png]]
- The method `proceed()` of the `ProceedingJoinPoint` parameter calls the intercepted method, `publishComment()`, of the `CommentService` bean. 
- If you don’t call `proceed()`, the aspect never delegates further to the intercepted method.
	![[notCalling_proceed.PNG]]
- You can even implement logic where the actual method isn’t called anymore. For example, an aspect that applies some authorization rules decides whether to delegate further to a method the app protects. If the authorization rules aren’t fulfilled, the aspect doesn’t delegate to the intercepted method it protects.
- Also, observe that the `proceed()` method throws a `Throwable`. The method `proceed()` is designed to throw any exception _coming from the intercepted method_.
##### Altering the intercepted method’s parameters and the returned value
- Aspects, not only, can they intercept a method and alter its execution, but they can also intercept the parameters used to call the method and possibly alter them or the value the intercepted method returns.
- Suppose you want to log the parameters used to call the service method and what the method returned.
- Because we also refer to what the method returns, we changed the service method and made it return a value, as presented in the next code snippet:
```Java
@Service
public class CommentService { 
	private Logger logger =
		Logger.getLogger(CommentService.class.getName());

	public String publishComment(Comment comment) {
		logger.info("Publishing comment:" + comment.getText());
		return "SUCCESS"; 
	} 
}
```
- The aspect can easily find the name of the intercepted method and the method parameters with `ProceedingJoinPoint` parameter of the aspect method. `ProceedingJoinPoint` represents the intercepted method. 
- You can use this parameter to get any information related to the intercepted method (parameters, method name, target object, and so on).
- The next code snippet shows you how to get the method name and the parameters used to call the method before intercepting the call:
```java
String methodName = joinPoint.getSignature().getName(); 
Object [] arguments = joinPoint.getArgs();
```
- Now we can change the aspect also to log these details. In the next listing, you find the change you need to make to the aspect method.
```Java
@Aspect
public class LoggingAspect { 
	private Logger logger =
		Logger.getLogger(LoggingAspect.class.getName());

	@Around("execution(* services.*.*(..))") 
	public Object log(ProceedingJoinPoint joinPoint) throws Throwable {

		// Obtains the name and parameters of the intercepted method
		String methodName = joinPoint.getSignature().getName(); 
		Object [] arguments = joinPoint.getArgs(); 

		// Logs the name and parameters of the intercepted method
		logger.info("Method " + methodName + " with parameters " +
					Arrays.asList(arguments) + " will execute");

		// Calls the intercepted method
		Object returnedByMethod = joinPoint.proceed();
		logger.info("Method executed and returned " + returnedByMethod);

		// Returns the value returned by the intercepted method
		return returnedByMethod; 
	} 
}
```
- Figure 6.10 makes it easier to visualize the flow.
	![[flow.PNG]]
- We’ve changed the `main()` method to print the value returned by `publishComment()`, as presented in the following:
```Java
public class Main {  
  
	public static void main(String[] args) {  
	    var c = 
		    new AnnotationConfigApplicationContext(ProjectConfig.class);  
  
	    var service = c.getBean(CommentService.class);  
  
	    Comment comment = new Comment();  
	    comment.setText("Demo comment");  
	    comment.setAuthor("Natasha");  
	    service.publishComment(comment);  

		String value = service.publishComment(comment);
		logger.info(value);
  }  
}
```
- But aspects are even more powerful. They can alter the execution of the intercepted method by:
	- Changing the value of the parameters sent to the method.
	- Changing the returned value received by the caller.
	- Throwing an exception to the caller or catching and treating an exception thrown by the intercepted method.
	- You can be extremely flexible in altering the call of an intercepted method. You can even change its behavior completely.
	![[alterUsingAspect.PNG]]
- When you call the `proceed()` method without sending any parameters, the aspect sends the original parameters to the intercepted method. But you can choose to provide a parameter when calling the `proceed()` method. 
- This parameter is an array of objects that the aspect sends to the intercepted method instead of the original parameter values. 
- The aspect logs the value returned by the intercepted method, but it returns to the caller a different value.
```Java
@Aspect
public class LoggingAspect { 
	private Logger logger =
		Logger.getLogger(LoggingAspect.class.getName());

	@Around("execution(* services.*.*(..))") 
	public Object log(ProceedingJoinPoint joinPoint) throws Throwable {

		// Obtains the name and parameters of the intercepted method
		String methodName = joinPoint.getSignature().getName(); 
		Object [] arguments = joinPoint.getArgs(); 

		// Logs the name and parameters of the intercepted method
		logger.info("Method " + methodName + " with parameters " +
					Arrays.asList(arguments) + " will execute");

		Comment comment = new Comment(); 
		comment.setText("Some other text!");
		Object [] newArguments = {comment};

		// We send a different comment instance 
		// as a value to the method’s parameter.
		Object returnedByMethod = joinPoint.proceed(newArguments);
		logger.info("Method executed and returned " + returnedByMethod);

		// We log the value returned by the intercepted method, 
		// but we return a different value to the caller.
		return "FAILED"; 
	} 
}
```
##### Intercepting annotated methods
- You can also use annotations to **mark the methods** you want an **aspect to intercept** with a comfortable syntax that allows you also to avoid writing complex AspectJ pointcut expressions.
- In the `CommentService` class, we’ll add three methods: `publishComment()`, `deleteComment()`, and `editComment()`.
- We want to define a custom annotation and log only the execution of the methods we mark using the custom annotation. To achieve this objective, you need to do the following:
	1. Define a custom annotation, and make it accessible at runtime. We’ll call this annotation `@ToLog`.
	2. Use a different AspectJ pointcut expression for the aspect method to tell the aspect to intercept the methods annotated with the custom annotation.
	![[customAnnotations.PNG]]
- The definition of the retention policy with `@Retention(RetentionPolicy.RUNTIME)` is critical. By default, in Java annotations **cannot be intercepted** at runtime. 
- You need to **explicitly** specify that someone can intercept annotations by setting the retention policy to `RUNTIME`.
- The `@Target` annotation specifies which language elements we can use this annotation for. 
- By default, you can annotate any language elements, but it’s always a good idea to restrict the annotation to only what you make it for—in our case, methods:
```Java
// Enables the annotation to be intercepted at runtime
@Retention(RetentionPolicy.RUNTIME)
// Restricts this annotation to only be used with methods
@Target(ElementType.METHOD)
public @interface ToLog {
	
}
```
- In the following, you find the definition of the `CommentService` class, which now defines three methods. We annotated only the `deleteComment()` method, so we expect the aspect will intercept only this one.
```Java
@Service
public class CommentService { 
	private Logger logger =
		Logger.getLogger(CommentService.class.getName()); 

	public void publishComment(Comment comment) { 
		logger.info("Publishing comment:" + comment.getText());
	} 

	// We use the custom annotation for 
	// the methods we want the aspect to intercept.
	@ToLog
	public void deleteComment(Comment comment) { 
		logger.info("Deleting comment:" + comment.getText()); 
	} 
	
	public void editComment(Comment comment) { 
		logger.info("Editing comment:" + comment.getText()); 
	} 
}
```
- To weave the aspect to the methods annotated with the custom annotation, we use the following AspectJ pointcut expression: `@annotation(ToLog)`. 
- This expression refers to any method annotated with the annotation named `@ToLog` (which is, in this case, our custom annotation).
	![[weavingToLog.PNG|800]]
- Changing the pointcut expression to weave aspect to annotated method:
```Java
@Aspect
public class LoggingAspect { 
	private Logger logger =
		Logger.getLogger(LoggingAspect.class.getName());

	// Weaving the aspect to the methods annotated with @ToLog
	@Around("@annotation(ToLog)") 
	public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
		 // Omitted code 
	} 
}
```
- When you run the app, only the annotated method (`deleteComment()`, in our case) is intercepted, and the aspect logs the execution of this method in the console.
##### Other advice annotations you can use
- So far we’ve used the advice annotation `@Around`. This is indeed the most used of the advice annotations in Spring apps because you can cover any implementation case: you can do things before, after, or even instead of the intercepted method. You can alter the logic any way you want from the aspect.
- But you don’t necessarily always need all this flexibility. A good idea is to look for the most straightforward way to implement what you need to implement.
- For simple scenarios, Spring offers four alternative advice annotations that are **less powerful** than `@Around`.
	- `@Before`—Calls the method defining the aspect logic before the execution of the intercepted method.
	- `@AfterReturning`—Calls the method defining the aspect logic after the method successfully returns, and provides the returned value as a parameter to the aspect method. 
		- The aspect method **isn’t called** if the intercepted method **throws** an exception.
	- `@AfterThrowing`—Calls the method defining the aspect logic if the intercepted method throws an exception, and provides the exception instance as a parameter to the aspect method.
	- `@After`—Calls the method defining the aspect logic only after the intercepted method execution, whether the method successfully returned or threw an exception.
- You use these advice annotations **the same way** as for `@Around`. 
- You provide them with an **AspectJ pointcut** expression to **weave** the aspect logic to specific method executions.
- In the above cases the aspect methods **don’t receive** the `ProceedingJoinPoint` parameter, and they **cannot decide** when to delegate to the intercepted method.
- This event already happens based on the annotation’s purpose (for example, for `@Before`, the intercepted method call will always happen after the aspect logic execution).
- In the next code snippet, you find the `@AfterReturning` annotation used.
```Java
@Aspect
public class LoggingAspect {

	private Logger logger =
		Logger.getLogger(LoggingAspect.class.getName());

	@AfterReturning(value = "@annotation(ToLog)", 
					returning = "returnedValue") 
	public void log(Object returnedValue) { 
		logger.info("Method executed and returned " + returnedValue);
	} 
}
```
- The _AspectJ pointcut expression_ specifies which methods this aspect logic _weaves to_.
- Optionally, when you use `@AfterReturning`, you can get the value returned by the intercepted method. 
- In this case, we add the `returning` attribute with a value that corresponds to the _name of the method’s parameter_ where this value will be provided.
- The parameter name should be **the same** as the value of the `returning` attribute of the annotation or **missing** if we **don’t need** to use the returned value.
### The aspect execution chain
- In a real-world app, a method is often intercepted by **more than one aspect**. For example, we have a method for which we want to log the execution and apply some security constraints.
- There’s nothing wrong with having as _many aspects_ as we need, but when this happens, we need to ask ourselves the following questions:
	- In which order does Spring execute these aspects?
	- Does the execution order matter?
- When you have multiple aspects weaved to the same method, they need to execute one after another. 
- One way is to have the `SecurityAspect` execute first and then delegate to the `LoggingAspect`, which further delegates to the intercepted method.
- The second option is to have the `LoggingAspect` execute first and then delegate to the `SecurityAspect`, which eventually delegates further to the intercepted method.
- This way, the aspects create an **execution chain**.
- The order in which the aspects execute is important because executing the aspects in different orders can have different results. 
- Take our example: we know that the `SecurityAspect` doesn’t delegate the execution in all the cases, so if we choose this aspect to execute first, sometimes the `LoggingAspect` won’t execute. 
- If we expect the `LoggingAspect` to log the executions that failed due to security restrictions, this isn’t the way we need to go.
	![[executionOrder.PNG]]
- Can we define this order then? By default, Spring **doesn’t guarantee the order** in which two aspects in the same execution chain are called. 
- If the execution order is not relevant, then you just need to define the aspects and leave the framework to execute them in whatever order. 
- If you need to define the aspects’ execution order, you can use the `@Order` annotation.
- This annotation receives an **ordinal** (a number) representing the order in the execution chain for a specific aspect. The **smaller** the number, the **earlier** that aspect executes.
- If two values are the **same**, the order of execution is again **not defined**.
- The implementation of the `LoggingAspect` class:
```Java
@Aspect
public class LoggingAspect { 
	private Logger logger =
		Logger.getLogger(LoggingAspect.class.getName()); 
	
	@Around(value = "@annotation(ToLog)") 
	public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
		logger.info("Logging Aspect: Calling the intercepted method");

		// The proceed() method here delegates further in
		// the aspect execution chain. It can either call the
		// next aspect or the intercepted method.
		Object returnedValue = joinPoint.proceed(); 
		
		logger.info("Logging Aspect: Method executed and returned " +
					returnedValue); 
		return returnedValue; 
	} 
}
```
- The implementation of the `SecurityAspect` class:
```Java
@Aspect
public class SecurityAspect { 
	private Logger logger =
		Logger.getLogger(SecurityAspect.class.getName()); 
	
	@Around(value = "@annotation(ToLog)") 
	public Object secure(ProceedingJoinPoint joinPoint) throws Throwable{
		logger.info("Security Aspect: Calling the intercepted method"); 
		
		Object returnedValue = joinPoint.proceed(); 
		
		logger.info("Security Aspect: Method executed and returned " +
					returnedValue); 
		
		return returnedValue; 
	} 
}
```
- The `CommentService` class is similar to the one we defined in the previous examples:
```Java
@Service
public class CommentService { 
	private Logger logger =
		Logger.getLogger(CommentService.class.getName());

	@ToLog
	public String publishComment(Comment comment) {
		logger.info("Publishing comment:" + comment.getText()); 
		return "SUCCESS"; 
	} 
}
```
- Also, remember that both aspects need to be beans in the Spring context:
```Java
@Configuration 
@ComponentScan(basePackages = "services") 
@EnableAspectJAutoProxy
public class ProjectConfig {

	@Bean
	public LoggingAspect loggingAspect() { 
		return new LoggingAspect(); 
	}

	@Bean
	public SecurityAspect securityAspect() { 
		return new SecurityAspect(); 
	} 
}
```
- The `main()` method calls the `publishComment()` method of the `CommentService` bean. In my case, the output after the execution looks like the one in the next code snippet:
	![[snippetResult.PNG]]
- To order the execution of `LoggingAspect` and `SecurityAspect`, we use the `@Order` annotation.
```Java
@Aspect 
@Order(1)
public class SecurityAspect { 
	// Omitted code 
}

@Aspect 
@Order(2)
public class LoggingAspect { 
	// Omitted code 
}
```
# Sources
- Spring Start Here - Chapter 6.
- [Spring Start Here - Chapter 6 - Episode 10](https://www.youtube.com/watch?v=dwVO9qVzQDQ&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=2&pp=iAQB).
- [Spring Start Here - Chapter 6 - Episode 11](https://www.youtube.com/watch?v=kz1-kA5BadE&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=2&pp=iAQB).