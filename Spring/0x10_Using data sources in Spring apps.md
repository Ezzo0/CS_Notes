# Explanation
- The **data source** is a component that **manages** connections to the server handling the database (the database management system, also known as **DBMS**).
	![[dataSource.PNG]]
- Without an object taking the responsibility of a data source, the app would **need to request** a new connection for each operation with the data.
- This approach is not realistic in a production scenario because communicating through the network for establishing a new connection for each operation would dramatically slow down the application and cause performance issues.
- The data source makes sure your app only requests a new connection when it really **needs it**, improving the app’s performance.
- When working with any tool related to data persistence in a relational database, Spring expects you to define a data source.
- In a Java app, the language’s capabilities to connect to a relational database is named Java Database Connectivity ([[0x03_JDBC|JDBC]]).
- JDBC offers you a way to connect to a DBMS to work with a database. However, the JDK doesn’t provide a specific implementation for working with a particular technology.
- The JDK only gives you the abstractions for objects an app needs to work with a relational database. 
- To gain the implementation of this abstraction and enable your app to connect to a certain DBMS technology, you add a runtime dependency named the **JDBC driver**.
- Every technology vendor provides the JDBC driver you need to add to your app to enable it to connect to that specific technology.
	![[JDBC.PNG]]
- When you learn JDBC in a Java fundamentals tutorial, the examples generally use a class named `DriverManager` to get a connection, as presented in the following code snippet:
```Java
Connection con = DriverManager.getConnection(url, username, password);
```
- The `getConnection()` method uses the `url` provided as a value for the first parameter to identify the database your app needs to access and the `username` and `password` to authenticate the access to the database.
- But requesting a new connection and authenticating each operation again and again for each is a waste of resources and time for both the client and the database server.
- A data source object can **efficiently manage** the connections to minimize the number of unnecessary operations.
- Instead of using the JDBC driver manager directly, we use a data source to retrieve and manage the connections.
- For Java apps, you have multiple choices for data source implementations, but the most commonly used today is the [HikariCP](https://github.com/brettwooldridge/HikariCP) (**Hikari connection pool**) data source.
### Using JdbcTemplate to work with persisted data
- Your app can use a data source to obtain connections to the database server efficiently. But how easily can you write code to work with the data?
- Using JDBC classes provided by the JDK has not proven to be a comfortable way to work with persisted data.
- You have to write verbose blocks of code even for the simplest operations. we’ll use a tool named `JdbcTemplate` that allows you to work with a database with JDBC in a simplified fashion.
- To demonstrate how `JdbcTemplate` is used, we’ll implement an example. We’ll follow these steps:
	1. Create a connection to the DBMS.
	2. Code the repository logic.
	3. Call the repository methods in methods that implement REST endpoints’ actions.
	![[purchase.PNG]]
- We start the implementation as usual, by adding the necessary dependencies.
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>runtime</scope>
</dependency>
```
- We add the `JDBC starter` to get all the needed capabilities to work with databases using JDBC.
- We add the `H2` dependency to get both an in-memory database for this example and a JDBC driver to work with it.
- The app only needs the database and the JDBC driver at **runtime**. The app doesn’t need them for compilation. To instruct Maven we only want these dependencies at runtime, we add the `scope` tag with the value `runtime`.
- Even if you don’t have a database server for this example, the `H2` dependency simulates the database. `H2` is an excellent tool we use both for examples and application tests when we want to test an app’s functionality but exclude its dependency on a database.
- We need to add a table that stores the purchase records. In theoretical examples, it’s easy to create a database structure by adding a file named `schema.sql` to the Maven project’s `resources` folder.
- In this file, you can write all the structural SQL queries you need to define the database structure. 
- You also find developers name these queries “**data description language” (DDL)**. We’ll also add such a file in our project and add the query to create the purchase table, as presented in the next code snippet:
```SQL
CREATE TABLE IF NOT EXISTS purchase (
	id INT AUTO_INCREMENT PRIMARY KEY,
	product varchar(50) NOT NULL,
	price double NOT NULL
);
```
- We need a model class to define the purchase data in our app. Instances of this class map the rows of the `purchase` table in the database, so each instance needs an `ID`, the `product`, and the `price` as attributes. 
- The next code snippet shows the Purchase model class:
```Java
public class Purchase {
	private int id;
	private String product;
	private BigDecimal price;
	
	// Omitted getters and setters
}
```
- You might find it interesting that the `Purchase` class price attribute’s type is `BigDecimal`. Couldn’t we have defined it as a `double`?
- Here’s an important thing I want you to be aware of: in theoretical examples, you often find `double` used for decimal values, but in many real-world examples, using `double` or `float` for decimal numbers isn’t the right thing to do.
- When operating with `double` and `float` values, you **might lose precision** for even simple arithmetic operations such as addition or subtraction.
- This effect is caused by the way Java stores such values in memory. When you work with sensitive information such as prices, you should use the `BigDecimal` type instead.
- To easily get a `PurchaseRepository` instance when we need it in the controller, we’ll also make this object a [[0x00_The Spring context_ Defining beans|bean]] in the Spring context.
```Java
@Repository
public class PurchaseRepository { }
```
- Now that `PurchaseRepository` is a bean in the application context, we can inject an instance of `JdbcTemplate` that we’ll use to work with the database. 
- I know what you’re thinking! “Where is this `JdbcTemplate` instance coming from? Who created this instance so that we can already inject it into our repository?”
- In this example, like in many production scenarios, we’ll benefit once more from Spring Boot’s magic. When Spring Boot saw you added the `H2` dependency in pom.xml, it **automatically configured** a data source and a `JdbcTemplate` instance.
- If you use Spring but not Spring Boot, you need to **define** the `DataSource` bean and the `JdbcTemplate` bean (you can add them in the Spring context using the `@Bean` annotation in the configuration class).
```Java
@Repository
public class PurchaseRepository { 
	private final JdbcTemplate jdbc;

	public PurchaseRepository(JdbcTemplate jdbc) {
		this.jdbc = jdbc;
	}

	public void storePurchase(Purchase purchase) {
		String sql = "INSERT INTO purchase VALUES (NULL, ?, ?)";
		jdbc.update(sql, purchase.getProduct(), purchase.getPrice());
	}
}
```
- Finally, you have a `JdbcTemplate` instance, so you can implement the app’s requirements. `JdbcTemplate` has an `update()` method you can use to execute any query for data mutation: `INSERT`, `UPDATE` or `DELETE`.
- Pass the SQL and the parameters it needs, and that’s it; let `JdbcTemplate` take care of the rest (obtaining a connection, creating a statement, treating the `SQLException`, and so on).
- To retrieve data, this time, you’ll write a `SELECT` query. And to tell `JdbcTemplate` how to transform the data into Purchase objects (your model class), you implement a `RowMapper`: an object responsible for **transforming** a row from the `ResultSet` into a specific object.
	![[RowMapper.PNG]]
- The following shows you how to implement a repository method to get all the records in the purchase table.
```Java
@Repository
public class PurchaseRepository { 
	// Omitted code

	public List<Purchase> findAllPurchases() {
		String sql = "SELECT * FROM purchase";

		// We implement a RowMapper object that tells JdbcTemplate
		// how to map a row in the result set into a Purchase object.
		// In the lambda expression, parameter “r” is the ResultSet
		// (the data you get from the database), 
		// while parameter “i” is an int representing the row number.
		RowMapper<Purchase> purchaseRowMapper = (r, i) -> {
			// We set the data into a Purchase instance.
			// JdbcTemplate will use this logic for
			// each row in the result set.
			Purchase rowObject = new Purchase();
			rowObject.setId(r.getInt("id"));
			rowObject.setProduct(r.getString("product"));
			rowObject.setPrice(r.getBigDecimal("price"));
			return rowObject;
		};

		// We send the SELECT query using the query method, 
		// and we provide the row mapper object for JdbcTemplate to
		// know how to transform the data it gets in Purchase objects.
		return jdbc.query(sql, purchaseRowMapper);
	}
}
```
- Once you have the repository methods and you can store and retrieve records in the database, it’s time to expose these methods through endpoints.
```Java
@RestController
@RequestMapping("/purchase")
public class PurchaseController {
	private final PurchaseRepository purchaseRepository;
	
	public PurchaseController( PurchaseRepository purchaseRepository) {
		this.purchaseRepository = purchaseRepository;
	}

	@PostMapping
	public void storePurchase(@RequestBody Purchase purchase) {
		purchaseRepository.storePurchase(purchase);
	}

	@GetMapping
	public List<Purchase> findPurchases() {
		return purchaseRepository.findAllPurchases();
	}
}
```
- Now, you can test the two endpoints using Postman or cURL.
### Customizing the configuration of the data source
- The H2 database we used before is excellent for examples and tutorials and to get started with implementing the persistence layer for an app. In production apps, however, you need more than an in-memory database, and often you need to configure the data source as well.
- To discuss using a DBMS in real world–type scenarios, we’ll change the example we implemented in before to use a MySQL server.
- You’ll observe the logic in the example doesn’t change, and changing the data source to point to a different database isn’t tricky. These are the steps we’ll follow:
	1. We’ll add a MySQL JDBC driver and configure a data source using the `application.properties` file to point to a MySQL database.
		- We’ll still let Spring Boot define the `DataSource` bean in the Spring context based on the properties we define.
	2. We’ll change the project to define a custom `DataSource` bean and discuss when something like this is needed in real-world scenarios.
##### Defining the data source in the application properties file
- Production-ready applications use external database servers, so having this skill will help you. We follow two simple steps for performing this transformation:
	1. Change the project dependencies to exclude `H2` and add the adequate `JDBC` driver.
	2. Add the connection properties for the new database to the `application.properties`file.
- For step 1, in the `pom.xml` file, exclude the `H2` dependency.
- If you use MySQL you need to add the MySQL JDBC driver. The project now needs to have the dependencies, as presented in the next snippet:
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```
- For step 2, the `application.properties` file should look like the following code snippet.
- We add the `spring.datasource.url` property to define the database location, and the `spring.datasource.username` and `spring.datasource.password` properties to define the credentials the app needs to authenticate and get connections from the DBMS.
- Additionally, we need to use the `spring.datasource.initialization-mode` property with the value `always` to instruct Spring Boot to use the `schema.sql` file and create the `purchase` table.
- You don’t need to use this property with `H2`. For `H2`, Spring Boot runs by default the queries in the `schema.sql` file, if this file exists:
	![[appProp.PNG]]
- With these couple of changes, the application now uses the MySQL database. Spring Boot knows to create the `DataSource` bean using the `spring.datasource` properties you provided in the `application.properties` file. You can start the app and test the endpoints.
##### Using a custom DataSource bean
- Spring Boot knows how to use a `DataSource` bean if you provide the connection details in the `application.properties` file.
- Sometimes this is enough, and as usual, I recommend you go with the simplest solution that solves your problems. But in other cases, you can’t rely on Spring Boot to create your `DataSource` bean.
- In such a case, you need to define the bean yourself. Some scenarios in which you need to define the bean yourself are as follows:
	- You need to use a specific `DataSource` implementation based on a condition you can only get at runtime.
	- Your app connects to more than one database, so you have to create multiple data sources and distinguish them using qualifiers.
	- You have to configure specific parameters of the `DataSource` object in certain conditions your app has only at runtime.
		- For example, depending on the environment where you start the app, you want to have more or fewer connections in the connection pool for performance optimizations.
	- Your app uses Spring framework but not Spring Boot.
- The `DataSource` is just a bean you add to the Spring context like any other bean.
- Instead of letting Spring Boot choose the implementation for you and configure the `DataSource` object, you define a method annotated with `@Bean` in a configuration class and add the object to the context yourself. This way, you have full control over the object’s creation.
```Java
@Configuration  
public class ProjectConfig {  

	// The connection details are configurable, so it’s a good idea to 
	// continue defining them outside of the source code.
	// In this example, we keep them in the “application.properties” file.
	@Value("${custom.datasource.url}")  
	private String datasourceUrl;  
	
	@Value("${custom.datasource.username}")  
	private String datasourceUsername;  
	
	@Value("${custom.datasource.password}")  
	private String datasourcePassword;  
	
	@Bean
	// The method returns a DataSource object.
	// If Spring Boot finds a DataSource already exists in
	// the Spring context it doesn’t configure one.
	public DataSource dataSource() {  
	
		// We’ll use HikariCP as the data source implementation for
		// this example.
		// However, when you define the bean yourself, you can choose
		// other implementations if your project requires something else.
		HikariDataSource dataSource = new HikariDataSource(); 

		// We set the connection parameters on the data source.
		dataSource.setJdbcUrl(datasourceUrl);  
		dataSource.setUsername(datasourceUsername);  
		dataSource.setPassword(datasourcePassword);

		// You can configure other properties as well
		// (eventually in certain conditions).
		// In this case, I use the connection timeout
		// (how much time the data source waits for a connection
		// before considering it can’t get one) as an example.
		dataSource.setConnectionTimeout(1000);
		
		// We return the DataSource instance, and Spring
		// adds it to its context.
		return dataSource;  
	}  
}
```
- Don’t forget to configure values for the properties you inject using the `@Value` annotation.
- In the `application.properties` file these properties should look like the next code snippet.
- We have intentionally used the word `custom` in their name to stress that we chose these names, and they’re not Spring Boot properties. You can give these properties any name:
```
custom.datasource.url=jdbc:mysql://localhost/spring_quickly?
useLegacyDatetimeCode=false&serverTimezone=UTC

custom.datasource.username=root
custom.datasource.password=
```
# Sources
- Spring Start Here - Chapter 12.