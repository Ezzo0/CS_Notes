# Explanation
- REST services are one of the most often encountered ways to implement communication between two apps. 
- REST offers access to functionality the server exposes **through endpoints** a client can call.
### Using REST services to exchange data between apps
- REST endpoints are as simple as implementing a **controller action** mapped to an HTTP method and a path.
- An app calls this controller action through HTTP. Because it’s how an app exposes a service through a web protocol, we call this endpoint a **web service**.
- Spring uses the same mechanism you learned for web apps for exposing REST endpoints. 
- The only difference is that for REST services we’ll tell the Spring MVC dispatcher servlet **not to look** for a view.
- The server sends back, in the HTTP response to the client, **directly** what the controller’s action returns.
	![[springMVCREST.PNG]]
- But before starting with our first example, we’d like to make you aware of some communication issues the REST endpoint might bring:
	- If the controller’s action takes **a long time** to complete, the HTTP call to the endpoint might **time out and break** the communication.
	- Sending a **large quantity of data** in one call (through the HTTP request) might cause the call to **time out and break** the communication. Sending more than a **few megabytes** through a REST call usually isn’t the right choice.
	- Too many **concurrent calls** on an endpoint exposed by a backend component might put too much pressure on the app and cause it to **fail**.
	- The network supports the HTTP calls, and the network is **never 100% reliable**. There’s always a chance a REST endpoint call might **fail** because of the network.
### Implementing a REST endpoint
- The below code shows you a controller class that implements a simple action. The only new thing you find in this listing is the use of the `@ResponseBody` annotation.
- The `@ResponseBody` annotation tells the dispatcher servlet that the controller’s action **doesn’t return a view name** but the data sent directly in the HTTP response.
```Java
@Controller
public class HelloController {
	@GetMapping("/hello")
	@ResponseBodypublic
	String hello() {
		return "Hello!";
	}
}
```
- Repeating the `@ResponseBody` annotation on every method becomes annoying. A best practice is avoiding code duplication. We want to somehow prevent repeating the `@ResponseBody` annotation for each method.
- To help us with this aspect, Spring offers the `@RestController` annotation, a combination of `@Controller` and `@ResponseBody`.
- You use `@RestController` to instruct Spring that all the controller’s actions are REST endpoints.
### Managing the HTTP response
- The HTTP response is how the backend app sends data back to the client due to a client’s request. The HTTP response holds data as the following:
	- _Response headers_—Short pieces of data in the response (usually not more than a few words long)
	- The _response body_—A larger amount of data the backend needs to send in the response
	- The _response status_—A short representation of the request’s result
##### Sending objects as a response body
- The only thing you need to do to send an object to the client in a response is make the controller’s action **return that object**.
- In the following example, we define a model object named `Country` with the attributes `name` and `population`. We implement a controller action to return an instance of type `Country`.
- When we use an object (such as `Country`) to model the data transferred between two apps, we name this object a **data transfer object (DTO)**. 
- We can say that `Country` is our **DTO**, whose instances are returned by the REST endpoint we implement in the HTTP response body.
```Java
public class Country {
	private String name;
	private int population;
	
	public static Country of( String name, int population) {
		Country country = new Country();
		country.setName(name);
		country.setPopulation(population); 
		return country;
	}
	
	// Omitted getters and setters
}
```
- The following shows the implementation of a controller’s action that returns an instance of type `Country`.
```Java
@RestController
public class CountryController {
	@GetMapping("/france")
	public Country france() {
		Country c = Country.of("France", 67);
		return c;
	}
}
```
- What happens when you call this endpoint? How would the object look in the HTTP response body? 
- By default, Spring creates a string representation of the object and formats it as **JSON**. 
- JavaScript Object Notation (JSON) is a simple way to format strings as _attribute-value_ pairs.
- Using JSON is the most common way to represent objects when working with REST endpoints. Although you aren’t constrained to use JSON as an object representation, you’ll probably never see someone using something else. 
- Spring offers the possibility of using other ways to format the response body (like XML or YAML) if you’d like, by plugging in a custom converter for your objects. However, the chances you’ll need this in a real-world scenario are so small.
##### Setting the response status and headers
- Sometimes it’s more comfortable to send part of the data in the response headers. The response status is also an essential flag in the HTTP response you use to signal the request’s result.
- By default, Spring sets some common HTTP statuses:
	- _200 OK_ if no exception was thrown on the server side while processing the request.
	- _404 Not Found_ if the requested resource doesn’t exist.
	- _400 Bad Request_ if a part of the request could not be matched with the way the server expected the data.
	- _500 Error_ on server if an exception was thrown on the server side for any reason while processing the request. 
		- Usually, for this kind of exception, the client can’t do anything, and it’s expected someone should solve the problem on the backend.
- However, in some cases, the requirements ask you to configure a custom status. How could you do that? The easiest and most common way to customize the HTTP response is using the `ResponseEntity` class.
- This class provided by Spring allows you to specify the **response body, status, and headers** on the HTTP response.
```Java
@RestController
public class CountryController { 
	@GetMapping("/france")
	public ResponseEntity<Country> france() {
		Country c = Country.of("France", 67);
		return ResponseEntity
					// Changes the HTTP response status to 202 Accepted
					.status(HttpStatus.ACCEPTED)
					// Adds three custom headers to the response
					.header("continent","Europe")
					.header("capital", "Paris")
					.header("favorite_food", "cheese and wine")
					// Sets the response body
					.body(c);
	}
}
```
##### Managing exceptions at the endpoint level
- One of the ways you can manage exceptions is catching them in the controller’s action and using the `ResponseEntity` class to send a different configuration of the response when the exception occurs.
- For our scenario, we define an exception named `NotEnoughMoneyException`, and the app will throw this exception when it cannot fulfill the payment because the client doesn’t have enough money in their account. 
- The next code snippet shows the class defining the exception:
```Java
public class NotEnoughMoneyException extends RuntimeException {
	
}
```
- We also implement a service class that defines the use case. For our test, we directly throw this exception. 
- In a real-world scenario, the service would implement the complex logic for making the payment. 
- The next code snippet shows the service class we use for our test:
```Java
@Service
public class PaymentService {
	public PaymentDetails processPayment() {
		throw new NotEnoughMoneyException();
	}
}
```
- `PaymentDetails`, the returned type of the `processPayment()` method, is just a model class describing the response body we expect the controller’s action to return for a successful payment. 
- The next code snippet presents the `PaymentDetails` class:
```Java
public class PaymentDetails {
	private double amount;
	// Omitted getters and setters
}
```
- When the app encounters an exception, it uses another model class named `ErrorDetails` to inform the client of the situation.
- The `ErrorDetails` class is also simple and only defines the error message as an attribute. The next code snippet presents the `ErrorDetails` model class:
```Java
public class ErrorDetails {
	private String message;
	// Omitted getters and setters
}
```
- How could the controller decide what object to send back depending on how the flow executed? 
- When there’s no exception (the app successfully completes the payment), we want to return an HTTP response with the status `Accepted` of type `PaymentDetails`.
- Suppose the app encountered an exception during the execution flow. In that case, the controller’s action returns an HTTP response with the status `400 Bad Request` and an `ErrorDetails` instance containing a message that describes the issue.
	![[managingexceptions.PNG]]
```Java
@RestController
public class PaymentController {
	private final PaymentService paymentService;
	
	public PaymentController(PaymentService paymentService) {
		this.paymentService = paymentService;
	}
	
	@PostMapping("/payment")
	public ResponseEntity<?> makePayment() {
		try { 
			PaymentDetails paymentDetails =
				paymentService.processPayment();
			return ResponseEntity
					.status(HttpStatus.ACCEPTED)
					.body(paymentDetails);
		} catch (NotEnoughMoneyException e) {
			ErrorDetails errorDetails = new ErrorDetails();
			errorDetails
				.setMessage("Not enough money to make the payment.");
			return ResponseEntity
					.badRequest()
					.body(errorDetails);
		}
	}
}
```
- This approach is good, and you’ll often find developers using it to manage the exception cases. 
- However, in a more complex application, you would find it more comfortable to separate the responsibility of exception management. 
	- First, sometimes the **same exception** has to be managed for **multiple endpoints**, and, as you guessed, we don’t want to introduce **duplicated code**.
	- Second, it’s **more comfortable** to know you find the **exception logic all in one place** when you need to understand how a specific case works.
- For these reasons, we prefer using a **REST controller advice**, an **aspect** that **intercepts exceptions** thrown by controllers’ actions and applies custom logic you define according to the intercepted exception.
	![[RESTControllerAdivce.PNG]]
- Now, The controller action is much simplified because it no longer treats the exception case.
```Java
@RestController
public class PaymentController {
	private final PaymentService paymentService;
	
	public PaymentController(PaymentService paymentService) {
		this.paymentService = paymentService;
	}
	
	@PostMapping("/payment")
	public ResponseEntity<PaymentDetails> makePayment() {
		PaymentDetails paymentDetails = paymentService.processPayment();
		return ResponseEntity
			.status(HttpStatus.ACCEPTED)
			.body(paymentDetails);
	}
}
```
- Instead, we created a separate class named `ExceptionControllerAdvice` that implements what happens if the controller’s action throws a `NotEnoughMoneyException`.
- The `ExceptionControllerAdvice` class is a _REST controller advice_. To mark it as a REST controller advice, we use the `@RestControllerAdvice` annotation.
- The method the class defines is also called an **exception handler**. You specify what **exceptions trigger** a controller advice method using the `@ExceptionHandler` annotation over the method.
```Java
@RestControllerAdvice
public class ExceptionControllerAdvice {

	// We use the @ExceptionHandler method to 
	// associate an exception with the logic the method implements.
	@ExceptionHandler(NotEnoughMoneyException.class)
	public ResponseEntity<ErrorDetails> exceptionNotEnoughMoneyHandler(){
		ErrorDetails errorDetails = new ErrorDetails();
		errorDetails.setMessage("Not enough money to make the payment.");
		
		return ResponseEntity
				.badRequest()
				.body(errorDetails);
	}
}
```
- In production apps, you sometimes need to send information about the exception that occurred, from the controller’s action to the advice. 
- In this case, you can add a **parameter** to the advice’s exception handler method of the **type of the handled exception**. 
- Spring is smart enough to pass the **exception reference** from the controller to the advice’s exception handler method. You can then use any details of the exception instance in the advice’s logic.
### Using a request body to get data from the client
- The HTTP request has a request body, and you can use it to send data from the client to the server. The HTTP request body is often used with REST endpoints.
- To use the request body, you just need to annotate a parameter of the controller’s action with `@RequestBody`.
- By default, Spring assumes you **used JSON** to represent the parameter you annotated and will try to **decode** the JSON string into an instance of your parameter type.
- In the case Spring cannot decode the JSON-formatted string into that type, the app sends back a response with the status `400 Bad Request`.
```Java
@RestController
public class PaymentController {
	private static Logger logger =
			Logger.getLogger(PaymentController.class.getName());
	
	@PostMapping("/payment")
	public ResponseEntity<PaymentDetails> makePayment(
		@RequestBody PaymentDetails paymentDetails) {
		logger.info("Received payment " + paymentDetails.getAmount());
		return ResponseEntity
				.status(HttpStatus.ACCEPTED)
				.body(paymentDetails);
	}
}
```
# Sources
- Spring Start Here - Chapter 10.
- [Spring Start Here - Chapter 10 - Episode 16](https://www.youtube.com/watch?v=C6H1YWl-GzY&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=1&pp=iAQB).
