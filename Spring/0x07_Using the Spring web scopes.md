# Explanation
- Spring has custom ways to manage instances for web apps by using the HTTP request as a point of reference.
- In web apps you can use other [[0x03_The Spring context_  Bean scopes and life cycle|bean scopes]] that are relevant **only** to web applications. We call them **web scopes**:
	- _Request scope_
		- Spring **creates an instance** of the bean class for **every** HTTP request. 
		- The instance exists only for that **specific HTTP request**.
	- _Session scope_
		- Spring **creates an instance and keeps** the instance in the server’s memory for the **full HTTP session**. 
		- Spring **links** the instance in the context with the client’s session.
	- _Application scope_
		- The instance is **unique** in the app’s context, and it’s available while the **app is running**.
### Using the request scope in a Spring web app
- A request-scoped bean is an object managed by Spring, for which the framework **creates a new instance for every** HTTP request.
- The app can use the instance **only** for the request that created it. Any new HTTP request (_from the same or other clients_) creates and uses a different instance of the same class.
	![[request_scope.PNG]]
- Before diving into implementing a Spring app that uses request-scoped beans, we’d like to shortly enumerate here the key aspects of using this bean scope. 
- These aspects will help you analyze whether a request-scoped bean is the right approach in a real-world scenario.
	![[request_scope_table.PNG]]
- A login example, such as this one, is excellent for **didactic** purposes. However, in a production-ready app, it’s better to **avoid implementing** authentication and authorization mechanisms **yourself**. 
- In a real-world Spring app, we use **Spring Security** to implement anything related to authentication and authorization.
- Let’s demonstrate the use of a request-scoped bean in an example. We’ll implement a web application’s login functionality, and we’ll use a request-scoped bean to manage the user’s credentials for the login logic.
- Below is the HTML login page that defines the view in our app.
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="UTF-8">
		<title>Login</title>
	</head>
	<body> 
		<form action="/" method="post">
			Username: <input type="text" name="username" /><br />
			Password: <input type="password" name="password" /><br />
			<button type="submit">Log in</button>
		</form>
		<p th:text="${message}"></p>
	</body>
</html>
```
- Let’s define the controller and the action that receives the HTTP request for the page we created.
```Java
@Controller
public class LoginController {
	@GetMapping("/") 
	public String loginGet() { 
		return "login.html"; 
	}
	
	@PostMapping("/")
	public String loginPost( 
		@RequestParam String username, 
		@RequestParam String password, 
		Model model
	) { 
		boolean loggedIn = false;
		if (loggedIn) { 
			model.addAttribute("message", "You are now logged in.");
		} else { 
			model.addAttribute("message", "Login failed!");
		} 
		return "login.html"; 
	}
}
```
- Notice that we haven’t implemented the login logic. 
- In the above controller, we take the request and send a message in response according to a variable representing the request’s result. 
- But this variable (named `loggedIn`) is always `false`.
	![[linkMVC.PNG|800]]
- Now we have a controller and a view, but where is the request scope in all of this?
- The only class we wrote is the `LoginController`, and we left it a [[0x03_The Spring context_  Bean scopes and life cycle#Using the singleton bean scope|singleton]], which is the **default** Spring scope. 
- We don’t need to change the scope for `LoginController` as long as it **doesn’t store any detail** in its attributes. 
- But remember, we need to implement the login logic. The login logic depends on the user’s credentials, and we have to take into consideration two things about these credentials:
	- The credentials are sensitive details, and you **don’t want to store** them in the app’s memory for **longer than** the login request.
	- More users with different credentials might attempt to log in **simultaneously**.
- Considering these two points, we need to make sure that if we use a bean for implementing the login logic, each instance is **unique** for each HTTP request. 
- We need to use a **request-scoped** bean. We’ll extend the app by adding a request-scoped bean `LoginProcessor`, which takes the credentials on the request and validates them.
	![[LoginProcessor.PNG]]
```Java
@Component 
@RequestScope
public class LoginProcessor {
	private String username;
	private String password;

	// omitted getters and setters

	public boolean login() {
		String username = this.getUsername();
		String password = this.getPassword();
		
		if ("natalie".equals(username) && "password".equals(password)) {
			return true;
		} else { 
			return false; 
		} 
	}
}
```
- Now `LoginController` will be as below
```java
@Controller  
public class LoginController {  
  
    private final LoginProcessor processor;  
  
    @Autowired  
    public LoginController(LoginProcessor processor) {  
        this.processor = processor;  
    }  
  
    @GetMapping("/")  
    public String login() {  
        return "login";  
    }  
  
    @PostMapping("/")  
    public String loginPost(
	    @RequestParam String username, 
	    @RequestParam String password, 
	    Model model
	) {  
        processor.setUsername(username);  
        processor.setPassword(password);  
        if (processor.login())  
            model.addAttribute("message", "You are now logged in.");  
        else  
            model.addAttribute("message", "Login failed!");  
        return "login";  
    }  
}
```
### Using the session scope in a Spring web app
- A session-scoped bean is an object managed by Spring, for which Spring **creates an instance and links** it to the HTTP session. 
- Once a client sends a request to the server, the server **reserves** a place in the memory for this request, for the whole **duration of their session**.
- Spring creates an instance of a session-scoped bean when the HTTP session is created for a specific client. 
- That instance can be reused for the same client **while it still has** the HTTP session **active**. 
- The data you store in the session-scoped bean attribute is **available** for all the client’s requests throughout an HTTP session.
- This approach of storing the data allows you to store information about what users do while they’re surfing through the pages of your app.
	![[session_scope.PNG|800]]
- A couple of features you can implement using session-scoped beans include the following examples:
	- A _login:_
		- Keeps details of the authenticated user while they visit different parts of your app and send multiple requests.
	- An _online shopping cart_
		- Users visit multiple places in your app, searching for products they add to the cart. 
		- The cart remembers all the products the client added.
- Let’s analyze the key characteristics of the session-scoped beans you need to consider when planning to use them in a production app.
	![[session_scope_table.PNG]]
- Let’s change the application we implemented before to display a page that only logged-in users can access. 
- Once a user logs in, the app redirects them to this page, which displays a welcome message containing the logged-in username and offers the user the option to log out by clicking a link.
- These are the steps we need to take to implement this change:
	1. Create a session-scoped bean to keep the logged-in user’s details.
	2. Create the page a user can only access after login.
	3. Make sure a user cannot access the page created at point 1 without logging in first.
	4. Redirect the user from login to the main page after successful authentication.
	![[change.PNG]]
- Fortunately, creating a session-scoped bean in Spring is as simple as using the `@SessionScope` annotation with the bean class. 
- Let’s create a new class, `LoggedUserManagementService`, and make it session-scoped
```Java
// We add the @Service stereotype annotation to 
// instruct Spring to manage this class as a bean in its context.
// We use the @SessionScope annotation to 
// change the scope of the bean to session.
@Service
@SessionScope
public class LoggedUserManagementService {
	private String username;
	// Omitted getters and setters
}
```
- Every time a user successfully logs in, we store its name in this bean’s username attribute. We auto-wire the `LoggedUserManagementService` bean in the `LoginProcessor` class, which we implemented to take care of the authentication logic, as shown below.
```Java
@Component  
@RequestScope  
public class LoginProcessor {
	private final LoggedUserManagementService loggedUserManagementService;
    private String username;  
    private String password;

	// We auto-wire the LoggedUserManagementService bean.
	public LoginProcessor(
		LoggedUserManagementService loggedUserManagementService) {
		this.loggedUserManagementService = loggedUserManagementService;
	}

	// Omitted getters and setters
  
    public boolean login() {
	    String username = this.getUsername(); 
	    String password = this.getPassword(); 
	    boolean loginResult = false;
        
        if (username.equals("admin") && password.equals("admin")) {  
            loginResult = true;
            loggedUserManagementService.setUsername(username); 
        }
        return loginResult;
    }
}
```
- Observe that the `LoginProcessor` bean **stays** request-scoped. We still use Spring to create this instance for each login request. 
- We **only need** the username and password attributes’ values during the request to execute the authentication logic.
- Because the `LoggedUserManagementService` bean is session-scoped, the username value will now be accessible throughout the **entire HTTP session**. 
- You can use this value to know if someone is logged in, and who. 
- You don’t have to worry about the case where multiple users are logged in; the application framework makes sure to **link** each HTTP request to the correct session.
- Figure below visually describes the login flow.
	![[login_flow.PNG]]
- Now we create a new page and make sure a user can access it only if they have already logged in. 
- We define a new controller (that we’ll call `MainController`) for the new page. We’ll define an action and map it to the `/main` path. 
- To make sure a user can access this path only if they logged in, we check if the `LoggedUserManagementService` bean stores any username. If it doesn’t, we redirect the user to the login page. 
- To redirect the user to another page, the controller action needs to return the string `redirect:` followed by the path to which the action wants to redirect the user.
```Java
@Controller
public class MainController {
	private final LoggedUserManagementService loggedUserManagementService;

	// We auto-wire the LoggedUserManagementService bean to 
	// find out if the user already logged in.
	public MainController( 
		LoggedUserManagementService loggedUserManagementService) {
		this.loggedUserManagementService = loggedUserManagementService; 
	}

	@GetMapping("/main")
	public String home() {
		String username = loggedUserManagementService.getUsername();
		
		if (username == null) { 
			return "redirect:/";
		}
		return "main.html";
	} 
}
```
- The following listing shows the content of the `main.html` page.
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="UTF-8">
		<title>Welcome</title>
	</head>
	<body>
		<h1>Welcome, <span th:text="${username}"></span></h1>
		<a href="/main?logout">Log out</a>
	</body>
</html>
```
- To allow the user to log out is also easy. You just need to set the username in the `LoggedUserManagementService` session bean as `null`.
- The next listing shows how to get the logout request parameter in the controller’s action and send the username to the view where it is displayed on the page.
```Java
@Controller
public class MainController {
	// Omitted code

	@GetMapping("/main") 
	public String home(
		@RequestParam(required = false) String logout,
		Model model
	) {
		if (logout != null) {
			loggedUserManagementService.setUsername(null);
		}
		
		String username = loggedUserManagementService.getUsername();
		if (username == null) {
			return "redirect:/";
		}
		model.addAttribute("username" , username);
		return "main.html";
	}
}
```
- To complete the app, we’d like to change the `LoginController` to redirect users to the main page once they authenticate.
```Java
@Controller
public class LoginController {
	// Omitted code
	
	@PostMapping("/")
	public String loginPost(
		@RequestParam String username,
		@RequestParam String password,
		Model model
	) {
		loginProcessor.setUsername(username);
		loginProcessor.setPassword(password);
		boolean loggedIn = loginProcessor.login();
		if (loggedIn) {
			return "redirect:/main";
		}
		
		model.addAttribute("message", "Login failed!");
		return "login.html";
	}
}
```
### Using the application scope in a Spring web app
- The application scope is close to how a [[0x03_The Spring context_  Bean scopes and life cycle#How singleton beans work|singleton]] works. The difference is that you **can’t have more instances** of the same type in the context and that we always use the HTTP requests as a reference point when discussing the life cycle of web scopes (including the application scope).
- We face the same **concurrency problems** we discussed for the [[0x03_The Spring context_  Bean scopes and life cycle#Singleton beans in real-world scenarios|singleton]] beans for application-scoped beans: it’s better to have **immutable** attributes for the singleton beans.
- The same advice is applicable to an application-scoped bean. But if you make the attributes immutable, then you can directly use a singleton bean instead.
	![[application_scope.PNG]]
- Let’s change the application we worked on in this chapter and add a feature that counts the login attempts.
- Because we have to count the login attempts from all users, we’ll store the count in an application-scoped bean. Let’s create a `LoginCountService` application-scoped bean that stores the count in an attribute.
```Java
@Service
@ApplicationScope
public class LoginCountService {
	private int count;
	
	public void increment() {
		count++;
	}
	
	public int getCount() {
		return count;
	}
}
```
- The `LoginProcessor` can then **auto-wire** this bean and call the `increment()` method for any new login attempt.
```Java
@Component
@RequestScope
public class LoginProcessor {
	private final 
		LoggedUserManagementService loggedUserManagementService; 
	private final LoginCountService loginCountService;

	private String username;
	private String password;

	public LoginProcessor(
		LoggedUserManagementService loggedUserManagementService,
		LoginCountService loginCountService) {
		this.loggedUserManagementService = loggedUserManagementService;
		this.loginCountService = loginCountService;
	}

	public boolean login() {
		loginCountService.increment();
		
		String username = this.getUsername();
		String password = this.getPassword();
		boolean loginResult = false;
		
		if ("natalie".equals(username) && "password".equals(password)) {
			loginResult = true;
			loggedUserManagementService.setUsername(username);
		}
		
		return loginResult;
	}

	// Omitted code
}
```
- The last thing you need to do is to display this value.
```Java
@Controller
public class MainController {
	// Omitted code
	@GetMapping("/main")
	public String home(
		@RequestParam(required = false) String logout,
		Model model
	) {
		if (logout != null) {
			loggedUserManagementService.setUsername(null);
		}

		String username = loggedUserManagementService.getUsername();
		int count = loginCountService.getCount();
		if (username == null) {
			return "redirect:/";
		}

		model.addAttribute("username" , username);
		model.addAttribute("loginCount", count);
		return "main.html";
	}
}
```
- The following shows you how to display the count value on the page.
```HTML
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="UTF-8">
		<title>Login</title>
	</head>
	<body>
		<h1>Welcome, <span th:text="${username}"></span></h1>
		<h2>
			Your login number is
			<span th:text="${loginCount}"></span>
		</h2>
		<a href="/main?logout">Log out</a>
	</body>
</html>
```
# Sources
- Spring Start Here - Chapter 9.
- [Spring Start Here - Chapter 9 - Episode 15](https://www.youtube.com/watch?v=faib01JwcDo&list=PLEocw3gLFc8W25hvuYb6EERd3F0aZjUQF&index=1&pp=iAQB).