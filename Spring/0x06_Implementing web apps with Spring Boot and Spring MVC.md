# Explanation
- Now, we’ll implement dynamic views using template engines. A **template engine** is a **dependency** that allows you to easily **get and display variable data** the controller sends.
### Implementing web apps with a dynamic view
- Let’s assume for now we want to send a name and print it with a specific color. In a real-world scenario, you’d maybe need to print the name of the user somewhere on the page. 
- How you do that? How do you get data that could be different from one request to another and print it on the page?
- We’ll create a [[0x05_Understanding Spring Boot and Spring MVC#The magic of Spring Boot|Spring Boot]] project and add a template engine to the dependencies in the `pom.xml` file. We’ll use a template engine named **Thymeleaf**.
- The next code snippet shows the dependency you need to add to the `pom.xml` file:
```
<dependency> 
	<groupId>org.springframework.boot</groupId> 
	<artifactId>spring-boot-starter-thymeleaf</artifactId> 
</dependency> 
```
- Below, you find the definition of the controller. We annotate the method to map the action to a specific request path using `@RequestMapping`. We now also define a parameter to the method. 
- This parameter of type `Model` stores the data we want the controller to send to the view. 
- In this `Model` instance, we add the values we want to send to the view and identify each of them with a unique name (also referred to as key). 
- To add a new value that the controller sends to the view, we call the `addAttribute()` method. 
- The first parameter of the `addAttribute()` method is the **key**; the second parameter is the **value** you send to the view.
```Java
@Controller
public class MainController { 
	@RequestMapping("/home") 
	public String home(Model page) { 
		page.addAttribute("username", "Katy"); 
		page.addAttribute("color", "red");
		return "home.html"; 
	} 
}
```
- To define the view, you need to add a new `home.html` file to your Spring Boot project’s `resources/templates` folder.
- Below is the content of the `home.html` file. The first important thing to notice in the file’s content is the `<html>` tag where I added the attribute `xmlns:th="http://www.thymeleaf.org"`.
- This definition is equivalent to an `import` in Java. It allows us further to use the prefix `th` to **refer to specific features** provided by **Thymeleaf** in the view.
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="UTF-8">
		<title>Home Page</title>
	</head>
	
	<body> 
		<h1>Welcome <span th:style="'color:' + ${color}"
							th:text="${username}"></span>!</h1>
	</body>
</html>
```
- A little bit further in the view, you find two places where we used this `th` prefix to refer to the **controller’s data** to the view. 
- With the `${attribute_key}` syntax, you refer to any of the **attributes** you send from the controller **using the Model instance**. 
- For example, we used the `${username}` to get the value of the `username` attribute and `${color}` to get the value of the `color` attribute.
##### Getting data on the HTTP request
- In most cases, to send data through the HTTP request you use one of the following ways:
	- _An HTTP request parameter_ represents a simple way to send values from client to server in a **key-value(s) pair** format. 
		- To send HTTP request parameters, you **append them to the URL** in a request query expression. 
		- They are also called _query parameters_. You should use this approach only for sending a small quantity of data.
	- An HTTP _request header_ is **similar** to the **request parameters** in that the request headers are sent through the HTTP header. 
		- The big difference is that they **don’t appear** in the URL, but you still **cannot** send large quantities of data using HTTP headers.
	- A _path variable_ sends data through the **request path itself**. 
		- It is the **same as** for the **request parameter** approach: you use a path variable to send a **small quantity** of data. 
		- But we should use path variables when the value you send is mandatory.
	- The _HTTP request body_ is mainly used to send a **larger quantity of data** (formatted as a string, but sometimes even binary data such as a file). 
##### Using request parameters to send data from client to server
- You use request parameters in the following scenarios:
	- The _quantity of data you send is not large_. 
		- You set the request parameters using query variables. This approach limits you to about 2,000 characters.
	- _You need to send optional data_. 
		- A request parameter is a clean way to deal with a value the client might not send. 
		- The server can expect to not get a value for specific request parameters.
- An often-encountered use case for request parameters used is defining some search and filtering criteria.
	![[requestparam.PNG|800]]
- Let’s use a request parameter by changing the previous example we discussed to get the color in which the username is displayed from the client.
- To get the value from a request parameter, you need to add one more parameter to the controller’s action method and annotate that parameter with the `@RequestParam` annotation. 
- The `@RequestParam` annotation tells Spring it needs to get the value from the HTTP request parameter with the **same name** as the method’s parameter name.
	![[requestparam2.PNG]]
```Java
@Controller
public class MainController { 
	@RequestMapping("/home") 
	public String home(
		@RequestParam String color, 
		Model page) { 
		page.addAttribute("username", "Katy"); 
		page.addAttribute("color", color);
		return "home.html"; 
	} 
}
```
- To set the request parameter’s value, you need to use the next snippet’s syntax: `http://localhost:8080/home?color=blue`.
- When setting HTTP request parameters, you extend the path with a `?` symbol followed by pairs of `key=value` parameters separated by the `&` symbol. 
- For example, if I want to also send the name as a request parameter, I write: `http://localhost:8080/home?color=blue&name=Jane`.
	![[urlrequestparam.PNG]]
- NOTE that A request parameter is **mandatory by default**. If the client doesn’t provide a value for it, the server sends back a response with the status HTTP `400 Bad Request`. 
- If you wish the value to be **optional**, you need to **explicitly** specify this on the annotation using the optional attribute: `@RequestParam(optional=true)`.
##### Using path variables to send data from client to server
- Instead of using the HTTP request parameters, you directly set variable values in the path:
	- Using request parameters: `http://localhost:8080/home?color=blue`.
	- Using path variables: `http://localhost:8080/home/blue`.
- You don’t identify the value with a key anymore. You just **take that value** from a **precise position** in the path. On the server side, you **extract** that value from the path **from the specific position**.
- You may have more than one value provided as a path variable, but it’s generally better to avoid using more than a couple. Also, you shouldn’t use path variables for optional values.
	![[comparison_RP_PV.PNG|800]]
- To reference a path variable in the controller’s action, you simply give it a name and add it to the path between curly braces, as presented below.
- You then use the `@PathVariable` annotation to mark the controller’s action parameter to get the path variable’s value.
```Java
@Controller
public class MainController { 
	@RequestMapping("/home/{color}")
	public String home(
		@PathVariable String color, 
		Model page) { 
		page.addAttribute("username", "Katy"); 
		page.addAttribute("color", color);
		return "home.html"; 
	} 
}
```
- Figure 8.8 visually represents the link between the code and the request path.
	![[Usingpathvariables.PNG]]
### Using the GET and POST HTTP methods
- We’ve relied on the request path to reach a specific action of the controller, but in a more complex scenario you can assign the same path to multiple actions of the controller as long as you use **different** HTTP methods.
- The HTTP method is defined by a verb and represents the client’s intention.
	![[HTTPMethods.PNG]]
- Now let’s implement an example that uses more than just HTTP GET. The scenario is the following: We have to create an app that stores a list of products. Each product has a name and a price.
- The web app displays a list of all products and allows the user to add one more product through an HTML form.
- Observe the two use cases described by the scenario. The user needs to do the following:
	- View all products in the list; here, we’ll continue using **HTTP GET**.
	- Add products to the list; here, we’ll use **HTTP POST**.
- In the project, we create a `Product` class to describe a product with its name and price attributes. The `Product` class is a model class so we’ll create it in a package named `model`.
- Now that we have a way to represent a product, let’s create the list where the app stores the products. 
- The web app will display the product in this list on a web page, and in this list the user can add more products.
```java
@Service
public class ProductService { 
	private List<Product> products = new ArrayList<>(); 
	
	public void addProduct(Product p) { products.add(p); } 
	public List<Product> findAll() { return products; } 
}
```
- A controller will call the use cases implemented by the service. The controller gets data about a new product from the client and adds it to the list by calling the service, and the controller gets the list of products and sends it to the view.
```Java
@Controller
public class ProductsController { 
	private final ProductService productService;

	public ProductsController(ProductService productService) {
		this.productService = productService;
	}

	// We map the controller action to the /products path. 
	// The @RequestMapping annotation, by default, 
	// uses the HTTP GET method.
	
	@RequestMapping("/products") 
	public String viewProducts(Model model) { 
		var products = productService.findAll();
		model.addAttribute("products", products);
		return "products.html"; 
	}

	// We map the controller action to the /products path. 
	// We use the method attribute of the @RequestMapping annotation to
	// change the HTTP method to POST.
	
	@RequestMapping(path = "/products", method = RequestMethod.POST)
	public String addProduct(  
		@RequestParam String name,  
		@RequestParam double price,  
		Model model  
	) {
		Product p = new Product();  
		p.setName(name);  
		p.setPrice(price);  
		productService.addProduct(p);
		
		var products = productService.findAll();  
		model.addAttribute("products", products);  
		return "products.html";  
	}
}
```
- Developers usually use dedicated annotations for each HTTP method instead of `@RequestMapping`.
- For apps, you’ll often find developers using `@GetMapping` to map a **GET** request to an action, `@PostMapping` for a request using HTTP **POST**, and so on. 
	- `@RequestMapping("/products")` ==> `@GetMapping("/products")`.
	- `@RequestMapping(path = "/products", method = RequestMethod.POST)` ==> `@PostMapping("/products")`.
- To display the products in the view, we define the products.html page in the `resources/templates` folder of the project.
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="UTF-8">
		<title>Home Page</title>
	</head>
	<body>
		<h1>Products</h1>
		<h2>View products</h2>
		
		<table>
			<tr>
				<!-- We define a static header for our table. -->
				<th>PRODUCT NAME</th>
				<th>PRODUCT PRICE</th>
			</tr>
			<!-- We use the th:each feature from Thymeleaf to -->
			<!-- iterate on the collection and display a table row -->
			<!-- for each product in the list. -->
			<tr th:each="p: ${products}" >
				<!-- We display the name and the price of -->
				<!-- each product on one row. -->
				<td th:text="${p.name}"></td>
				<td th:text="${p.price}"></td>
			</tr>
		</table>


		<h2>Add a product</h2>
		<!-- When submitted, the HTML form makes a -->
		<!-- POST request for path /products. -->
		<form action="/products" method="post">
			<!-- An input component allows the user to set -->
			<!-- the name of the product. -->
			<!-- The value in the component is sent as -->
			<!-- a request parameter with the key “name.” -->
			Name: <input 
						type="text" 
						name="name"><br /> 
			Price: <input
						type="number"
						step="any"
						name="price"><br />
			<button type="submit">Add product</button> 
		</form>
	</body>
</html>
```
- The flow for calling the `/products` path with **HTTP GET** on the Spring MVC diagram:
	1. The client sends an HTTP request for the `/products` path.
	2. The dispatcher servlet uses the handler mapping to find the controller’s action to call for the `/products` path.
	3. The dispatcher servlet calls the controller’s action.
	4. The controller requests the product list from the service and sends it to be rendered with the view.
	5. The view is rendered into an HTTP response.
	6. The HTTP response is sent back to the client.
	![[callingproducts.PNG]]
- In our example, we used the `@RequestParameter` annotation. We used this annotation here to make it clear how the client sends the data. 
- But sometimes Spring allows you to **omit** code. For example, you could use a Product as a parameter of the controller’s action directly, as presented below.
- Because the request **parameters’ names are the same as the Product class attributes’ names**, Spring knows to **match them** and automatically creates the object.
```Java
@Controller
public class ProductsController {
	// Omitted code 
	
	@PostMapping("/products") 
	public String addProduct( 
		Product p,
		Model model
	) { 
		productService.addProduct(p);
		var products = productService.findAll();
		model.addAttribute("products", products); 
		return "products.html"; 
	} 
}
```
# Sources
- Spring Start Here - Chapter 8.
- [Spring Start Here - Chapter 8 - Episode 14](https://www.youtube.com/watch?v=xR77mF4IIpQ&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=1&pp=iAQB).