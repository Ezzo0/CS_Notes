# Explanation
- We classify the approaches of creating a web app as the following:
	1. _Apps where the backend provides the fully prepared view in response to a client’s request_.
		- The browser directly interprets the data received from the backend and displays this information to the user in these apps.
		![[backendfullyview.PNG]]
	2. _Apps using frontend-backend separation._ 
		- For these apps, the backend only serves raw data. 
		- The browser doesn’t display the data in the backend’s response directly.
		- The browser runs a separate frontend app that gets the backend responses, processes the data, and instructs the browser what to display.
		- Sometimes developers refer to the frontend-backend separation approach as being a **modern** approach.
		![[frontendbackendseparation.PNG]]
### Using a servlet container in web app development
- One of the most important things to consider is the communication between the client and the server. 
- A web browser uses a protocol named **Hypertext Transfer Protocol (HTTP)** to **communicate** with the server over the network. 
- This protocol accurately **describes** how the client and the server **exchange data** over the network.
- You’ll use a component already designed to understand HTTP.
- In fact, what you need is _not only something that understands_ HTTP, but something that can **translate** the HTTP request and response to a Java app.
- This something is a _servlet container_ (sometimes referred to as a web server): a **translator of the HTTP** messages for your Java app.
- This way, your Java app doesn’t need to take care of implementing the communication layer.
	![[servletcontainer.PNG]]
- But if this is everything a servlet container does, why name it _servlet_ container? What is a _servlet_? 
- A servlet is nothing more than a **Java object** that directly **interacts** with the servlet container. 
- When the servlet container gets an **HTTP request**, it calls a **servlet object’s method** and provides the **request as a parameter**. 
- The same method also gets a **parameter representing the HTTP response** used by the servlet to **set the response** sent back to the client that made the request.
- Some time ago, the servlet was the most critical component of a backend web app from the developer’s point of view. 
- Suppose a developer had to implement a new page accessible at a specific path in the URL (e.g., /home/profile/edit, etc.) for a web app. 
- The developer needed to create **a new servlet instance**, configure it in the **servlet container**, and assign it to a specific path. 
- The servlet contained the **logic** associated with the user’s request and the ability to prepare a **response**, including info for the browser on how to display the response.
- For any path the web client could call, the developer needed to add the instance in the servlet container and configure it. 
- Because such a component manages servlet instances you add into its **context**, we name it a **servlet container**. 
- It basically has a context of servlet instances it **controls**, just as Spring does with its beans. For this reason, we call a component such as Tomcat a servlet container.
	![[servletinstances.PNG]]
### The magic of Spring Boot
- To create a Spring web app, we need to configure a servlet container, create a servlet instance, and then make sure we correctly configure this servlet instance such that Tomcat calls it for any client request. What a headache to write so many configurations!
- Spring Boot is now one of the most appreciated projects in the Spring ecosystem. 
- It helps you create Spring apps more efficiently and focus on the business code you write by eliminating a huge part of the code you used to write for configurations.
- Listed here are what we consider the most critical Spring Boot features, and what they offer:
	- _Simplified project creation_
		- You can use a project initialization service to get an empty but configured skeleton app.
	- _Dependency starters_
		- Spring Boot groups certain dependencies used for a specific purpose with dependency starters. 
		- You don’t need to figure out all the must-have dependencies you need to add to your project for one particular purpose nor which versions you should use for compatibility.
	- _Autoconfiguration based on dependencies_
		- Based on the dependencies you added to your project, Spring Boot defines some default configurations. 
		- Instead of writing all the configurations yourself, you only need to change the ones provided by Spring Boot that don’t match what you need. 
		- Changing the configs likely requires less code (if any).
##### Using a project initialization service to create a Spring Boot project
- Now we discuss the main things Spring Initializr configured into your Maven project:
	- The Spring app main class
	- The Spring Boot POM parent
	- The dependencies
	- The Spring Boot Maven plugin
	- The properties file
	![[springInitializer.PNG]]
- You can observe that Spring Initializr added the `Main` class to your app and also some configurations in the `pom.xml` file. 
- The `Main` class of a Spring Boot app is annotated with the `@SpringBootApplication` annotation, and it looks similar to the next code snippet:
```Java
// This annotation defines the Main class of a Spring Boot app.
@SpringBootApplication  
public class Main {
    public static void main(String[] args) {
       SpringApplication.run(Main.class, args);  
    }
}
```
- If you open your project’s pom.xml file, you’ll find that the project initialization service also added some details here. 
- One of the most important details you’ll find is the Spring Boot parent node, which looks similar to the next code snippet:
```Java
<parent> 
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.3.4.RELEASE</version> 
	<relativePath/> 
</parent>
```
- One of the essential things this parent does is **provide you with compatible versions** for the dependencies you’ll add to your project.
- You’ll observe that we don’t specify a version for a dependency we use in most cases. We let Spring Boot choose the version of a dependency to make sure we don’t run into incompatibilities.
- You find Spring Boot Maven plugin also configured in the `pom.xml` file. 
- The next code snippet shows the plugin declaration, which you usually find at the end of the `pom.xml` file inside the `<build> <plugins> … </plugins></build>` tags. 
- This plugin is responsible for adding part of the default configurations you’ll observe in your project:
```
<build>  
    <plugins>
	    <plugin>
		    <groupId>org.springframework.boot</groupId>
		    <artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```
- Also in the `pom.xml` file, you find the dependency you added when creating the project in start.spring.io, Spring Web. It is a dependency starter named spring-boot-starter-web.
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
- The last essential thing Spring Initializr added to your project is a file named `application.properties`. This file is used to configure property values your app needs during its execution.
##### Using dependency starters to simplify the dependency management
- A _dependency starter_ is a **group** of dependencies you add to **configure your app** for a specific purpose.
- In your project’s `pom.xml` file, the starter looks like a normal dependency, as presented in the next code snippet. 
- Observe the name of the dependency: A starter name **usually starts** with `spring-boot-starter-` followed by a **relevant name** that describes the capabilities it added to the app:
```
<dependency> 
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-web</artifactId> 
</dependency>
```
- Say you want to add web capabilities to your app. In the past, to configure a Spring web app you had to add all the needed dependencies to your `pom.xml` file yourself and make sure their versions were compatible one with the other. 
- Configuring all the dependencies you need is not an easy job. Taking care of the version compatibility is even more complicated.
- With dependency starters, we **don’t** request dependencies directly. We request **capabilities**.
- You add a dependency starter for a particular capability you need, say web functionalities, a database, or security. 
- Spring Boot makes sure to add the right dependencies to your app with the proper **compatible version for your requested capability**.
- We can say that dependency starters are **capability-oriented groups** of compatible dependencies.
	![[capabilities.PNG]]
##### Using autoconfiguration by convention based on dependencies
- Spring Boot also provides autoconfiguration for your application. We say that it applies the _convention-over-configuration_ principle.
- You can start the app, and you’ll find your app boots a Tomcat instance **by default** accessible on port 8080. 
- In your console, you find something similar to the next snippet:
	![[tomcat.PNG]]
- Based on the dependencies you added, Spring Boot realizes what you expect from your app and **provides you some default configurations**. 
- Spring Boot gives you the configurations, which are generally used for the capabilities you requested when adding the dependencies. 
- For example, Spring knows when you added the web dependency you need for a servlet container and configures you a Tomcat instance because, in most cases, developers use this implementation. 
- For Spring Boot, Tomcat is the convention for a servlet container. The convention represents the most-used way to configure the app for a specific purpose. 
- Spring Boot configures the app by convention such that you now only need to change those places where your app needs a more particular configuration. 
- With this approach, you’ll write less code for configuration (if any).
### Implementing a web app with Spring MVC
- It’s true we already have a Spring Boot project with the default configurations, but this app only starts a Tomcat server. 
- These configurations don’t make our app a web app yet! We still have to implement the pages that someone can access using a web browser.
- We continue implementing the project to add a web page with static content. With these changes, you’ll learn to implement a web page and how your Spring app works behind the scenes.
- To add a web page to your app, you follow two steps:
	1. Write an **HTML document** with the content you want to be displayed by the browser.
	2. Write a **controller** with an action for the web page created at point 1.
	![[webpage.PNG|800]]
- We first start adding a static web page with the content we want to display in the browser. 
- This web page is just an HTML document, and for our example the page only displays a short text in a heading.
- You need to add the file in the `resources/static` folder of your Maven project.
- This folder is the **default place** where the Spring Boot app expects to **find the pages** to render.
- The second step you take is writing a **controller with a method** that _links_ the HTTP request to the _page_ you want your app to provide in response.
- The controller is a **component of the web app** that contains methods (often named actions) **executed for a specific HTTP request**.
- In the end, the controller’s action **returns a reference** to the web page the app returns in response.
- We’ll keep our first example simple, and we won’t make the controller execute any specific logic for the request for now.
- We’ll just configure an action to return in response to the content of the `home.html` document we created and stored in the `resources/static` folder in the first step.
- To mark a class as a controller, you only need to use the `@Controller` annotation, a stereotype annotation (like [[0x00_The Spring context_ Defining beans#Using stereotype annotations to add beans to the Spring context|@Component]] and [[0x02_The Spring context_  Using abstractions#Focusing on object responsibilities with stereotype annotations|@Service]]). 
- This means that Spring will also **add a bean** of this class to its context to manage it.
- Inside this class, you can define **controller actions**, which are _methods associated_ with specific HTTP requests.
- Say you want the browser to display this page’s content when the user accesses the `/home` path. 
- To achieve this result, you **annotate** the action method with the `@RequestMapping` annotation specifying the path as a value of the annotation: `@RequestMapping("/home")`.
- The method needs to return, as a string, the **name of the document** you want the app to send as a response.
- By running the app and writing the following in your browser and the below result will appear:
	![[localhost.PNG]]
	![[result.PNG]]
- Now that you’ve seen the app’s behavior, let’s discuss the mechanism behind it. 
- Spring has a set of components that interact with each other to get the result you observed:
	1. The client makes an HTTP request.
	2. Tomcat gets the client’s HTTP request. Tomcat has to call a **servlet component** for the HTTP request. 
		- In the case of Spring MVC, Tomcat calls a servlet Spring Boot configured. We name this servlet _dispatcher servlet_.
	3. The dispatcher servlet is the **entry point** of the Spring web app. Tomcat calls the dispatcher servlet for any HTTP request it gets. 
		- Its responsibility is to **manage** the request further inside the Spring app. 
		- It has to **find what controller action** to call for the request and what to send back in response to the client. 
		- This servlet is also referred to as a **front controller**.
	4. The first thing the dispatcher servlet needs to do is **find a controller action** to call for the request. 
		- To **find out** which controller action to call, the dispatcher servlet delegates to a component named **handler mapping**. 
		- The handler mapping finds the controller action you associated with the request with the `@RequestMapping` annotation.
		- NOTE that The handler mapping also searches by something named the **HTTP method**.
	5. After finding out which controller action to call, the dispatcher servlet calls that specific controller action. 
		- If the handler mapping **couldn’t find** any action associated with the request, the app responds to the client with an HTTP `404 Not Found` status. 
		- The controller **returns** the page name it needs to render for the response to the dispatcher servlet. We refer to this HTML page also as **the view**.
	6. At this moment, the dispatcher servlet needs to **find the view** with the name received from the controller to get its content and send it as response. 
		- The dispatcher servlet delegates the responsibility of getting the view content to a component named **View Resolver**.
	7. The dispatcher servlet returns the rendered view in the HTTP response.
	![[springservletcomponents.PNG|900]]
- Spring (with Spring Boot) considerably simplifies the development of a web app by arranging this setup. 
- You only need to write controller actions and map them to requests using annotations. 
- A large part of the logic is hidden in the framework, and this helps you write the apps faster and cleaner.
# Sources
- Spring Start Here - Chapter 7.
- [Spring Start Here - Chapter 7 - Episode 12](https://www.youtube.com/watch?v=abr4bdK9Z_4&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=1&pp=iAQB).
- [Spring Start Here - Chapter 7 - Episode 13](https://www.youtube.com/watch?v=sJf_otREa04&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=2&pp=iAQB).