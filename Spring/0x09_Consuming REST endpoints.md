# Explanation
- In a backend solution composed of multiple services, these components need to **speak** to exchange data, so when you implement such a service using Spring, you need to know how to call a REST endpoint exposed by another service.
	![[serviceRESTconsuminig.PNG]]
- There are three ways to call REST endpoints from a Spring app:
	1. _OpenFeign_—A tool offered by the Spring Cloud project. we recommend developers use this feature in new apps for consuming REST endpoints.
	2. _RestTemplate_—A well-known tool developers have used since Spring 3 to call REST endpoints. 
	3. _WebClient_—A Spring feature presented as an alternative to _RestTemplate_. This feature uses a different programming approach named _reactive programming_.
- Suppose you implement an app that allows users to make payments. To make a payment, you need to call an endpoint of another system.
	![[paymentEndpoint1.PNG]]
	![[paymentEndpoint2.PNG]] 
- We’ll model the payment with the Payment class, as presented in the next code snippet:
```Java
public class Payment {
	private String id;
	private double amount;
	
	// Omitted getters and setters
}
```
- Below is the endpoint’s implementation in the controller class.
```Java
@RestController
public class PaymentsController {
	private static Logger logger =
			Logger.getLogger(PaymentsController.class.getName());

	@PostMapping("/payment")
	public ResponseEntity<Payment> createPayment(
		@RequestHeader String requestId,
		@RequestBody Payment payment
	) {
		logger.info("Received request with ID " + requestId + 
					" ;Payment Amount: " + payment.getAmount());

		// The method sets a random value for the payment’s ID.
		payment.setId(UUID.randomUUID().toString());

		// The controller action returns the HTTP response.
		// The response has a header and the response body that 
		// contains the payment with the random ID value set.
		return ResponseEntity 
					.status(HttpStatus.OK)
					.header("requestId", requestId)
					.body(payment);
	}
}		
```
### Calling REST endpoints using Spring Cloud OpenFeign
- With **OpenFeign**, we write in this section, you only need to write an **interface**, and the tool provides you with the implementation.
- We’ll define an interface where we **declare the methods** that consume REST endpoints. 
- The only thing we need to do is **annotate** these methods to **define** the path, the HTTP method, and eventually parameters, headers, and the body of the request.
- The interesting thing is that we don’t need to implement the methods ourselves. You define with the interface methods based on the annotations, and Spring knows to implement them.
	![[OpenFeign.PNG]]
- Your `pom.xml` file needs to define the dependency, as shown by the next code snippet:
```xml
<dependency> 
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
- Once you have the dependency in place, you can create the proxy interface. In OpenFeign terminology, we also name this interface the _OpenFeign client_.
- OpenFeign **implements** this interface, so you **don’t** have to bother **writing** the code that calls the endpoint. 
- You only need to use a few annotations to tell OpenFeign how to send the request. The following shows you how simple the definition of the request is with OpenFeign.
```Java
// We use the @FeignClient annotation to configure the REST client.
// A minimal configuration defines a name and the endpoint base URI.
@FeignClient(name = "payments",
			url = "${name.service.url}")
public interface PaymentsProxy {
	// We specify the endpoint’s path and HTTP method.
	@PostMapping("/payment")
	Payment createPayment(
		@RequestHeader String requestId,
		@RequestBody Payment payment);
}
```
- The first thing to do is annotate the interface with the `@FeignClient` annotation to tell OpenFeign it has to provide an implementation for this contract.
- We have to assign a name to the proxy using the `name` attribute of the `@FeignClient` annotation, which OpenFeign internally uses. The name **uniquely identifies** the client in your app.
- The `@FeignClient` annotation is also where we specify the _base URI of the request_. You can define the base URI as a string using the `url` attribute of `@FeignClient`.
- Ensure you always **store URIs** and other details that might differ from one environment to another in the **properties files** and never hardcode them in the app.
- You can define a property in the project’s `application.properties` file and refer it from the source code using the following syntax: `${property_name}`. 
- Using this practice, you don’t need to recompile the code when you want to run the app in different environments.
- Each method you declare in the interface represents a REST endpoint call.
- OpenFeign needs to know where to find the interfaces defining the client contracts. We use the `@EnableFeignClients` annotation on a configuration class to enable the OpenFeign functionality and tell OpenFeign where to search for the client contracts.
```Java
@Configuration
@EnableFeignClients(
	basePackages = "com.example.proxy")
public class ProjectConfig { }
```
- You can now inject the OpenFeign client through the interface you defined before. Once you enable OpenFeign, it knows to implement the interfaces annotated with `@FeignClient`.
- The following shows you the controller class that injects the FeignClient.
```Java
@RestController  
public class PaymentsController {  
  
  private final PaymentsProxy paymentsProxy;  
  
  public PaymentsController(PaymentsProxy paymentsProxy) {  
    this.paymentsProxy = paymentsProxy;  
  }  
  
  @PostMapping("/payment")  
  public Payment createPayment(  
      @RequestBody Payment payment  
      ) {  
    String requestId = UUID.randomUUID().toString();  
    return paymentsProxy.createPayment(requestId, payment);  
  }  
}
```
- Now start both projects (the payments service and this section’s app) and call the app’s `/payment` endpoint using cURL or Postman.
### Calling REST endpoints using RestTemplate
- We again implement the app that calls the `/payment` endpoint of the payment service, but this time we use a different approach: `RestTemplate`.
- We don’t want you to conclude that `RestTemplate` has any problems. 
- It is being put to sleep not because it’s not working properly or because it’s not a good tool. But as apps evolved, we started to need more capabilities.
- Developers wanted to be able to benefit from different things that aren’t easy to implement with `RestTemplate`, such as the following:
	- Calling the endpoints both synchronously and asynchronously
	- Writing less code and treating fewer exceptions (eliminate boilerplate code)
	- Retrying call executions and implementing fallback operations (logic performed when the app can’t execute a specific REST call for any reason)
- The steps for defining the call are as follows:
	1. Define the HTTP headers by creating and configuring an `HttpHeaders` instance.
	2. Create an `HttpEntity` instance that represents the request data (headers and body).
	3. Send the HTTP call using the `exchange()` method and get the HTTP response.
	![[RestTemplate.PNG|800]]
- Below, you find the definition of the proxy class.
```Java
@Component
public class PaymentsProxy {
	private final RestTemplate rest;
	
	// We take the URL to the payment service from the properties file.
	@Value("${name.service.url}")
	private String paymentsServiceUrl;
	
	// We inject the RestTemplate from the Spring context
	// using constructor DI.
	public PaymentsProxy(RestTemplate rest) {
		this.rest = rest;
	}
	
	public Payment createPayment(Payment payment) {
		String uri = paymentsServiceUrl + "/payment";
		
		// We build the HttpHeaders object to
		// define the HTTP request headers.
		HttpHeaders headers = new HttpHeaders();
		headers.add("requestId", UUID.randomUUID().toString());

		// We build the HttpEntity object to define the request data.
		HttpEntity<Payment> httpEntity = 
				new HttpEntity<>(payment, headers);

		// We send the HTTP request and retrieve the data on 
		// the HTTP response.
		ResponseEntity<Payment> response = 
				rest.exchange(uri, 
							  HttpMethod.POST,
							  httpEntity,
							  Payment.class);
		return response.getBody();
	}
}
```
- Defining a controller class to test the implementation
```Java
@RestController
public class PaymentsController {
	private final PaymentsProxy paymentsProxy;
	
	public PaymentsController(PaymentsProxy paymentsProxy) {
		this.paymentsProxy = paymentsProxy;
	}
	
	@PostMapping("/payment")
	public Payment createPayment(
		@RequestBody Payment payment ) {
		return paymentsProxy.createPayment(payment);
	}
}
```
### Calling REST endpoints using WebClient
- Spring’s documentation recommends using `WebClient`, but that’s only a **valid recommendation for reactive apps**. If you aren’t writing a reactive app, use OpenFeign instead.
- In a nonreactive app, a thread executes a business flow. Multiple tasks compose a business flow, but these tasks are not independent. 
- The same thread executes all the tasks composing a flow. Let’s take an example to observe where this approach might face issues and how we can enhance it.
- Suppose you implement a banking application where a bank’s client has one or more credit accounts. 
- The system component you implement calculates the total debt of a bank’s client. To use this functionality, other system components make a REST call to send a unique ID to the user. 
- To calculate this value, the flow you implement includes the following steps:
	1. The app receives the user ID.
	2. It calls a different service of the system to find out if the user has credits with other institutions.
	3. It calls a different service of the system to get the debt for internal credits.
	4. If the user has external debts, it calls an external service to find out the external debt.
	5. The app sums the debts and returns the value in an HTTP response.
	![[flowSteps.PNG]]
- The app creates a new thread for each request, and this thread executes the steps one by one. 
- The thread has to wait for a step to finish before proceeding to the next one and is blocked every time it waits for the app to perform an I/O call.
	![[flowThread.PNG]]
- We observe two significant issues here:
	1. _The thread is idle while an I/O call blocks it._ 
		- Instead of using the thread, we allow it to stay and occupy the app’s memory.
		- We consume resources without gaining any benefit. With such an approach, you could have cases where the app gets 10 requests simultaneously, but all the threads are idle simultaneously while waiting for details from other systems.
	2. _Some of the tasks don’t depend on one another._
		- For example, the app could execute step 2 and step 3 at the same time.
		- There’s no reason for the app to wait for step 2 to end before executing step 3.
		- The app just needs, in the end, the result of both to calculate the total debt.
- Reactive apps change the idea of having **one atomic flow** in which one thread executes all its tasks from the beginning to the end.
- With reactive apps, we think of tasks as **independent**, and multiple threads can **collaborate to complete** a flow composed of multiple tasks.
- Instead of imagining this functionality as steps on a timeline, imagine it as a backlog of tasks and a team of developers solving them.
- Two developers can implement two different tasks simultaneously if they don’t depend on one another. 
- If a developer gets stuck on a task because of an external dependency, they can leave it temporarily and work on something else. 
- The same developer can get back to the task once it’s not blocked anymore, or another developer can finish solving it.
- Using this approach, you don’t need one thread per each request. You can **solve multiple requests with fewer threads** because the threads don’t have to stay idle.
- When **blocked** on a certain task, the thread **leaves it** and works on some other task that isn’t blocked.
- Technically, in a reactive app, we implement a flow by **defining the tasks** and the **dependencies between them**.
- The reactive app specification offers us two components: the **producer** and the **subscriber** to implement the dependencies between tasks.
- A task returns a producer to **allow other tasks to subscribe** to it, marking the dependency they have on the task.
- A task uses a subscriber to **attach to a producer** of another task and consume that task’s result once it ends.
	![[reactiveTasks.PNG]]
- Because `WebClient` imposes a reactive approach, we need to add a dependency named `WebFlux` instead of the standard web dependency.
- The next code snippet shows the `WebFlux` dependency:
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```
- To call the REST endpoint, you need to use a `WebClient` instance. The best way to create easy access is to put it in the Spring context using the [[0x00_The Spring context_ Defining beans#Using the @Bean annotation to add beans into the Spring context|@Bean]] annotation with a configuration class method.
- The following shows you the app’s configuration class.
```Java
@Configuration
public class ProjectConfig {
	@Bean
	public WebClient webClient() {
		return WebClient
				.builder()
				.build();
	}
}
```
- Below is the proxy class’s implementation, which uses `WebClient` to call the endpoint the app exposes.
```Java
@Component
public class PaymentsProxy {
	private final WebClient webClient;

	// We take the base URL from the properties file.
	@Value("${name.service.url}")
	private String url;

	public PaymentsProxy(WebClient webClient) {
		this.webClient = webClient;
	}
	
	public Mono<Payment> createPayment(
		String requestId,
		Payment payment) {
		// We specify the HTTP method we use when making the call.
		return webClient.post()
			// We specify the URI for the call.
			.uri(url + "/payment")
			// We add the HTTP header value to the request.
			// You can call the header() method multiple times
			//if you want to add more headers.
			.header("requestId", requestId)
			// We provide the HTTP request body.
			.body(Mono.just(payment), Payment.class)
			// We send the HTTP request and obtain the HTTP response.
			.retrieve()
			// We get the HTTP response body.
			.bodyToMono(Payment.class);
	}
}
```
- In our demonstration, we use a class named `Mono`. This class defines a producer.
- Above, you find this case, where the method performing the call doesn’t get the input directly. Instead, we send a `Mono`.
- This way, we can create an independent task that provides the request body value. The `WebClient` subscribed to this task becomes dependent on it.
- The method also doesn’t return a value directly. Instead, it returns a `Mono`, allowing another functionality to subscribe to it.
- This way, the app builds the flow, not by chaining them on a thread, but by linking the dependencies between tasks through producers and consumers.
	![[Mono.PNG]]
- The code before is the proxy method that consumes a `Mono` producing the HTTP request body and returns it to what the `WebFlux` functionality subscribes.
- To prove the call works correctly, as we did in this chapter’s previous examples, we implement a controller class that uses the proxy to expose an endpoint we’ll call to test our implementation’s behavior.
```Java
@RestController
public class PaymentsController {
	private final PaymentsProxy paymentsProxy;
	
	public PaymentsController(PaymentsProxy paymentsProxy) {
		this.paymentsProxy = paymentsProxy;
	}
	
	@PostMapping("/payment")
	public Mono<Payment> createPayment(
		@RequestBody Payment payment) {
		String requestId = UUID.randomUUID().toString();
		return paymentsProxy.createPayment(requestId, payment);
	}
}
```
# Sources
- Spring Start Here - Chapter 11.