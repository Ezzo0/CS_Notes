# Explanation
- Spring has multiple different approaches for creating beans and managing their life cycle, and in the Spring world we name these approaches _scopes_.
### Using the singleton bean scope
- The singleton bean scope defines Spring’s default approach for managing the beans in its context. It is also the bean scope you’ll most encounter in production apps.
##### How singleton beans work
- Spring creates a singleton [[0x00_The Spring context_ Defining beans|bean]] when it loads the context and assigns the bean a name (sometimes also referred to as bean ID). 
- We name this scope singleton because you always get the **same instance** when you refer to a **specific bean**.
- But be careful! You can have **more instances** of the same type in the Spring context if they have **different names**.
	 ![[singletone.PNG]]
	 ![[singletone2.PNG]]
##### Singleton beans in real-world scenarios
- Because the singleton bean scope assumes that multiple components of the app can share an object instance, the most important thing to consider is that these beans must be **immutable**.
- Most often, a real-world app executes actions on multiple threads (e.g., any web app). In such a scenario, multiple threads share the same object instance. If these threads change the instance, you encounter a [[0x17_Intro to Concurrency#Why It Gets Worse Shared Data|race-condition]] scenario.
- In case of a race condition, the developer needs to properly synchronize the threads to avoid unexpected execution results or errors.
- If you want mutable singleton beans (whose attributes change), you need to make these beans **concurrent by yourself** (mainly by employing thread synchronization).
- But singleton beans _aren’t_ designed to be synchronized. They’re commonly used to define an app’s backbone class design and delegate responsibilities one to another.
- Technically, synchronization is possible, but it’s not a good practice. Synchronizing the thread on a concurrent instance can dramatically affect the app’s performance.
- So, If you need to make an object bean in the Spring context, it should be singleton **only if it’s immutable**. Avoid designing mutable singleton beans.
##### Using eager and lazy instantiation
- In most cases, Spring creates all singleton beans when it initializes the context—this is Spring’s default behavior. We’ve used only this default behavior, which is also called _eager instantiation_.
- With _lazy instantiation_, Spring doesn’t create the singleton instances when it creates the context. Instead, it creates each instance the first time someone refers to the bean.
- Let’s take an example:
```Java
@Service
public class CommentService { 
	public CommentService() { 
		System.out.println("CommentService instance created!"); 
	} 
}
```
```Java
@Configuration
@ComponentScan(basePackages = {"services"}) 
public class ProjectConfig { 
	
}
```
- In the `Main` class, we only instantiate the Spring context. A critical aspect to observe is that no one uses the `CommentService` bean. However, Spring will create and store the instance in the context.
```Java
public class Main { 
	public static void main(String[] args) {
		var c = new AnnotationConfigApplicationContext(
					ProjectConfig.class
				); 
	} 
}
```
- Even if the app doesn’t use the bean anywhere, when running the app you’ll find the following output in the console: `CommentService instance created!`
- Now change the example by adding the `@Lazy` annotation above the class (for stereotype annotations approach) or above the `@Bean` method (for the `@Bean` method approach). 
- You’ll observe the output no longer appears in the console when running the app because we instructed Spring to create the bean only when someone uses it.
```Java
@Service 
@Lazy
public class CommentService {
	public CommentService() { 
		System.out.println("CommentService instance created!");
	} 
}
```
- When should you use eager instantiation and when should you use lazy? In most cases, it’s more comfortable to let the framework create all the instances at the beginning when the context is instantiated (_eager_). 
- This way, when one instance delegates to another, the second bean already exists in any situation. 
- In a lazy instantiation, the framework has to first **check** if the instance exists and eventually create it if it doesn’t, so from the performance point of view, it’s better to have the instances in the context already (_eager_) because it spares some checks the framework needs to do when one bean delegates to another. 
- Another advantage of eager instantiation is when something is wrong and the framework cannot create a bean; we can **observe this issue** when starting the app.
- With lazy instantiation, someone would observe the issue only when the app is already executing and it reaches the point that the bean needs to be created.
- But lazy instantiation is not all evil. In some cases, a specific client **didn’t use** a big part of the functionality, so instantiating the beans together with the Spring context unnecessarily occupied a lot of memory.
### Using the prototype bean scope
- The Spring behavior for managing prototype beans is straightforward. Every time you request a reference to a prototype-scoped bean, Spring **creates a new object** instance.
- For prototype beans, Spring doesn’t create and manage an object instance directly. 
- The framework manages the object’s type and creates a new instance every time someone requests a reference to the bean.
	 ![[prototype.PNG]]
- We need to use a new annotation named `@Scope` to change the bean’s scope.
- With prototype beans, we no longer have concurrency problems because each thread that requests the bean gets a **different instance**, so defining mutable prototype beans is not a problem.
	![[comparisonSingleton.PNG]]
# Sources
- Spring Start Here - Chapter 5.
- [Spring Start Here - Chapter 5 - Episode 8](https://www.youtube.com/watch?v=Aca1VEXk0dc&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=2&pp=iAQB).
- [Spring Start Here - Chapter 5 - Episode 9](https://www.youtube.com/watch?v=s8X-n_QQUf0&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=1&pp=iAQB).