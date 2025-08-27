# Explanation
- Now it’s time you take a step forward and use what you’ve learned in the previous chapters with an example closer to what happens in the real world.
- Say you are implementing an app a team uses to manage their tasks. One of the app’s features is allowing the users to leave comments for the tasks. 
- When a user publishes a comment, it is stored somewhere (e.g., in a database), and the app sends an email to a specific address configured in the app.
- We need to design the objects and find the right responsibilities and abstractions for implementing this feature.
### Implementing the requirement without using a framework
- First, we need to identify the objects (responsibilities) to implement. In standard real-world applications, we usually refer to the objects implementing uses cases as _services_.
- We’ll need a service that implements the `publish comment` use case. Let’s name this object `CommentService`.
- When analyzing the requirement again, we observe that the use case consists of two actions: storing the comment and sending the comment by mail.
- As they are quite different from one another, we consider these actions to be two different responsibilities, and thus we need to implement two different objects.
- When we have an object working directly with a database, we generally name such an object _repository_. Sometimes you also find such objects referred to as _data access objects_ (**DAO**).
- Let’s name the object that implements the storing comment responsibility `CommentRepository`.
- Finally, in a real-world app, when implementing objects whose responsibility is to establish communication with something outside the app, we name these objects _proxies_. 
- So let’s name the object whose responsibility is sending the email `CommentNotificationProxy`.
	 ![[app.PNG]]
- But wait! Didn’t we say we shouldn’t use direct coupling between implementations? We need to make sure we **decouple** the implementations by **using interfaces**.
- In the end, the `CommentRepository` might now use a database to store the comments. 
- But in the future, maybe this needs to be changed to use some other technology or an external service. 
- We can say the same for the `CommentNotificationProxy` object. 
- Now it sends the notification by email, but maybe in a future version the comment notification needs to be sent through some other channel.
- We certainly want to make sure we decouple the `CommentService` from the implementations of its dependencies so that when we need to change the dependencies, we don’t need to change the object using them as well.
	 ![[app using interface.PNG]]
### Using dependency injection with abstractions
- Remember that the main reason to add an object to the Spring context is to allow Spring to control it and further augment it with functionalities the framework provides. 
- So the decision should be easy and based on the question, “Does this object need to be managed by the framework?
- In our case, we need to add the object to the Spring context if it either **has a dependency** we need to inject from the context or if it’s a **dependency itself**.
- Looking at our implementation, you’ll observe that the only object that doesn’t have a dependency and is also not a dependency itself is `Comment`. 
- The other objects in our class design are as follows:
	- `CommentService`—Has two dependencies, the `CommentRepository` and the `CommentNotificationProxy`.
	- `DBCommentRepository`—Implements the `CommentRepository` interface and is a dependency of the `CommentService`.
	- `EmailCommentNotificationProxy`—Implements the `CommentNotificationProxy` interface and is a dependency of the `CommentService`.
- But why not add the Comment instances as well? Adding objects to the Spring context without needing the framework to manage them adds unnecessary complexity to your app, making the app both more challenging to maintain and less performant.
- Observe that the two interfaces in _figure 4.6_ are not marked with `@Component`. We use stereotype annotations for the **classes** that Spring needs to create instances and add these instances to its context. 
- It doesn’t make sense to add stereotype annotations on interfaces or abstract classes because these **cannot be instantiated**. Syntactically, you can do this, but it is not useful.
```Java
public interface CommentRepository { 
	void storeComment(Comment comment); 
}
```
```Java
public interface CommentNotificationProxy { 
	void sendComment(Comment comment); 
}
```
```Java
@Component
public class DBCommentRepository implements CommentRepository {
	@Override
	public void storeComment(Comment comment) {
		System.out.println("Storing comment: " + comment.getText()); 
	} 
}
```
```Java
@Component
public class EmailCommentNotificationProxy 
	implements CommentNotificationProxy {

	@Override
	
	public void sendComment(Comment comment) { 
		System.out.println( "Sending notification for comment: " +
							comment.getText()); 
	} 
}
```
```Java
@Component
public class CommentService {

	private final CommentRepository commentRepository;
	private final CommentNotificationProxy commentNotificationProxy;

	// We would have to use @Autowired 
	// if the class had more than one constructor.
	public CommentService( 
		CommentRepository commentRepository, 
		CommentNotificationProxy commentNotificationProxy) {
		this.commentRepository = commentRepository;
		this.commentNotificationProxy = commentNotificationProxy; 
	}

	public void publishComment(Comment comment) {
		commentRepository.storeComment(comment);
		commentNotificationProxy.sendComment(comment); 
	} 
}
```
- The `CommentService` class declares the dependencies to the other two components through the interfaces `CommentRepository` and `CommentNotificationProxy`. 
- Spring sees the attributes are defined with interface types and is smart enough to **search in its context** for beans created with classes that implement these interfaces.
```Java
@Configuration 
@ComponentScan( 
	basePackages = {"proxies", "services", "repositories"} 
) 
public class ProjectConfiguration {	
}
```
- In this example, we use the `basePackages` attribute of the `@ComponentScan` annotation. Spring also offers the feature of directly specifying the classes (by using the `basePackageClasses` attribute of the same annotation).
```Java
public class Main { 
	public static void main(String[] args) { 
		var context = new AnnotationConfigApplicationContext( 
							ProjectConfiguration.class);
		
		var comment = new Comment(); 
		comment.setAuthor("Laurentiu"); 
		comment.setText("Demo comment");

		var commentService = context.getBean(CommentService.class);
		commentService.publishComment(comment); 
	} 
}
```
- Running the application, you’ll observe the output presented in the following code snippet, which demonstrates that the two dependencies were accessed and correctly called by the `CommentService` object:
```
Storing comment: Demo comment 
Sending notification for comment: Demo comment
```
- By using the DI feature, we don’t **create the instance** of the `CommentService` object and **its dependencies** _ourselves_, and we don’t need to explicitly make the **relationship** between them.
##### Choosing what to auto-wire from multiple implementations of an abstraction
- Suppose we have two beans created with two different classes that implement the `CommentNotificationProxy` interface.
- Spring uses a mechanism for deciding which bean to choose:
	- Using the `@Primary` annotation to mark one of the beans for implementation as the default.
	- Using the `@Qualifier` annotation to name a bean and then refer to it by its name for DI.
```Java
// Using @Primary to mark the implementation as default
@Component 
@Primary
public class CommentPushNotificationProxy 
	implements CommentNotificationProxy {
	@Override
	public void sendComment(Comment comment) { 
		System.out.println( "Sending push notification for comment: " 
							+ comment.getText()); 
	} 
}

// The following code snippets show you how to 
// use the @Qualifier annotation to name specific implementations.

// The CommentPushNotification class:

@Component
// Using @Qualifier, we name this implementation “PUSH.”
@Qualifier("PUSH")
public class CommentPushNotificationProxy 
	implements CommentNotificationProxy { 
	// Omitted code 
}

// The EmailCommentNotificationProxy class:

@Component 
@Qualifier("EMAIL")
public class EmailCommentNotificationProxy 
	implements CommentNotificationProxy { 
		// Omitted code 
}
```
```Java
@Component
public class CommentService {
	private final CommentRepository commentRepository;
	private final CommentNotificationProxy commentNotificationProxy;

	public CommentService( 
		CommentRepository commentRepository, 
		@Qualifier("PUSH") 
			CommentNotificationProxy commentNotificationProxy) { 
		this.commentRepository = commentRepository;
		this.commentNotificationProxy = commentNotificationProxy; 
	} 
	
	// Omitted code 
}
```
### Focusing on object responsibilities with stereotype annotations
- Thus far, when discussing stereotype annotations, we have only used `@Component` in our examples. 
- But with real-world implementations, you’ll find out that developers sometimes use other annotations for the same purpose.
- Using `@Component` is generic and gives you no detail about the responsibility of the object you’re implementing. 
- But developers generally use objects with some known responsibilities. Two of the responsibilities we discussed are the *service* and the *repository*.
- Spring offers us the `@Service` annotation to mark a component that takes the responsibility of a **service** and the `@Repository` annotation to mark a component that implements a **repository** responsibility.
	![[service and repository.PNG]]
- All three (`@Component`, `@Service`, and `@Repository`) are stereotype annotations and instruct Spring to create and add an instance of the annotated class to its context.
# Sources
- Spring Start Here - Chapter 4.
- [Spring Start Here - Chapter 4 - Episode 6](https://www.youtube.com/watch?v=YinPemPVTFQ&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=4&pp=iAQB).
- [Spring Start Here - Chapter 4 - Episode 7](https://www.youtube.com/watch?v=PDVBkuwBwwk&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=3&pp=iAQB).
