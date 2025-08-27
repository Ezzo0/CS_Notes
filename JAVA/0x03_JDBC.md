# Explanation
- JDBC stands for **J**ava **D**ata**b**ase **C**onnectivity, which is a standard Java API for database-independent connectivity between the Java and a wide range of databases.
- The JDBC library includes APIs for each of the tasks mentioned below that are commonly associated with database usage.
	- Making a connection to a database.
	- Creating SQL statements.
	- Executing SQL queries in the database.
	- Viewing & Modifying the resulting records.
### JDBC Architecture
- JDBC Architecture consists of two layers:
	- **JDBC API** − This provides the application-to-JDBC Manager connection.
	- **JDBC Driver API** − This supports the JDBC Manager-to-Driver Connection.
- The JDBC API uses a driver manager and database-specific drivers to provide transparent connectivity to heterogeneous databases.
- The JDBC **driver manager** ensures that the **correct driver** is used to access each data source. The driver manager is capable of supporting multiple concurrent drivers connected to multiple heterogeneous databases.
- Following is the architectural diagram, which shows the location of the driver manager with respect to the JDBC drivers and the Java application:
	 ![[jdbc_architecture.jpg]]
### Common JDBC Components
- The JDBC API provides the following interfaces and classes:
	 - **DriverManager:** 
		 - This class manages a list of database drivers. _Matches connection requests_ from the java application with the proper database driver using _communication sub protocol_. 
		 - The first driver that _recognizes_ a certain subprotocol under JDBC will be _used_ to establish a database Connection.
	- **Driver:** 
		- This interface handles the _communications_ with the _database_ server.
	- **Connection:** 
		- This interface with all methods for _contacting_ a database. 
		- The connection object represents communication context, i.e., all communication with database is through connection object only.
	- **Statement:** 
		- You use objects created from this interface to _submit_ the SQL statements to the database.
	- **ResultSet:** 
		- These objects hold _data retrieved_ from a database after you execute an SQL query using Statement objects. It acts as an iterator to allow you to _move through_ its data.
	- **SQLException:** 
		- This class handles _any errors_ that occur in a database application.
- The `java.sql` and` javax.sql` are the primary packages for JDBC 4.0. It offers the main classes for interacting with your data sources.
### Creating JDBC Application
- There are following six steps involved in building a JDBC application:
	 - **Import the packages:** 
		 - Requires that you include the packages containing the JDBC classes needed for database programming. 
		 - Most often, using `import java.sql.*` will suffice.
	- **Open a connection:**
		- Requires using the `DriverManager.getConnection()` method to create a Connection object, which represents a physical connection with the database.
	- **Execute a query:**
		- Requires using an object of type Statement for building and submitting an SQL statement to the database.
	- **Extract data from result set:** 
		- Requires that you use the appropriate `ResultSet.getXXX()` method to retrieve the data from the result set.
	- **Clean up the environment:** 
		- Requires explicitly closing all database resources versus relying on the JVM's garbage collection.
- This sample example can serve as a **template** when you need to create your own JDBC application in the future.
```Java
import java.sql.*;  
public class Main {  
    static final String DB_URL = "jdbc:mysql://localhost/employee_db";  
    static final String USER = "root";  
    static final String PASS = "1899";  
    static final String QUERY = "SELECT id, first, last, age FROM Employees";  
    public static void main(String[] args) {  
        // Open a connection  
        try(Connection conn = DriverManager.
						      getConnection(DB_URL, USER, PASS);  
            Statement stmt = conn.createStatement();) {  
            ResultSet rs = stmt.executeQuery(QUERY);  
            // Extract data from result set  
            while (rs.next()) {  
                // Retrieve by column name  
                System.out.print("ID: " + rs.getInt("id"));  
                System.out.print(", Age: " + rs.getInt("age"));  
                System.out.print(", FirstName: " + 
					             rs.getString("first"));  
                System.out.println(", LastName: " + 
					             rs.getString("last"));  
            }  
        } catch (SQLException e) {  
            System.out.println("Connection Failed");  
            e.printStackTrace();  
        }  
  
    }  
}
```
- In this example, we've four static strings containing a database `connection url`, `username`, `password` and `Query`. 
- Now using `DriverManager.getConnection()` method, we've prepared a database connection. 
- Once connection is prepared, we've created a Statement object using `connection.createStatement()` method, then using `statement.executeQuery()`, the SELECT Query is executed and result is stored in a `resultset`. 
- Now `resultset` is iterated and each record is printed.
### What is JDBC Driver?
- JDBC drivers implement the defined _interfaces_ in the JDBC API, for interacting with your database server.
- The `Java.sql` package that ships with JDK, contains various classes with their behaviors defined and their actual implementations are done in third-party drivers.
- Third party vendors implements the `java.sql.Driver` interface in their database driver.
### Database Connections
- The programming involved to establish a JDBC connection is fairly simple:
	- **Import JDBC Packages:** 
		- To use the standard JDBC package, which allows you to select, [[0x03_Writing#Inserting Data|insert]], [[0x03_Writing#Updating Data|update]], and [[0x03_Writing#Deleting Data|delete]] data in SQL tables, add `import java.sql.* ;`
	- **Register JDBC Driver(Not needed in JDBC 4.0):**  
		- This step causes the JVM to load the desired driver implementation into memory so it can fulfill your JDBC requests.
		- You can register a driver in one of two ways.
			1. _Approach I:_
				- The most common approach to register a driver is to use Java's `Class.forName()` method, to dynamically load the driver's class file into memory, which automatically registers it. 
				- This method is preferable because it allows you to make the driver registration configurable and portable.
				- You can use `getInstance()` method to work around noncompliant JVMs, but then you'll have to code for two extra Exceptions as follows:
			2. _Approach II:_
				- The second approach you can use to register a driver, is to use the static `DriverManager.registerDriver()` method.
				- You should use the `registerDriver()` method if you are using a non-JDK compliant JVM, such as the one provided by Microsoft.
	- **Database URL Formulation:**
		- After you've loaded the driver, you can establish a connection using the `DriverManager.getConnection()` method. The three overloaded `DriverManager.getConnection()` methods are:
			- `getConnection(String url)`.
			- `getConnection(String url, Properties prop)`.
			- `getConnection(String url, String user, String password)`.
		- Here each form requires a database **URL**. A database **URL** is an address that points to your database.
		- Following table lists down the popular JDBC driver names and database URL
			 ![[popular_jdbc.png|800]]
		- All the highlighted part in URL format is static and you need to change only the remaining part as per your database setup.
	    
	- **Create Connection Object:**
		- Finally, code a call to the `DriverManager` object's `getConnection()` method to establish actual database connection.
- At the end of your JDBC program, it is required explicitly to close all the connections to the database to end each database session. 
- However, if you forget, Java's garbage collector will close the connection when it cleans up stale objects.
- To close the above opened connection, you should call `close()` Method that is provided by **Connection** class.
### The Statement Objects
- The JDBC _Statement, CallableStatement,_ and _PreparedStatement_ interfaces define the methods and properties that enable you to send SQL or PL/SQL commands and receive data from your database.
- They also define methods that help bridge data type differences between Java and SQL data types used in a database.
- The following table provides a summary of each interface's purpose to decide on the interface to use.
	 ![[statements_interfaces.png|800]]
##### The Statement Objects
- Before you can use a Statement object to execute a SQL statement, you need to create one using the Connection object's `createStatement()` method, as in the following example:
```Java
Statement stmt = null; 
try { 
	stmt = conn.createStatement(); 
	. . . 
} 
catch (SQLException e) {
	. . . 
} 
finally {
	stmt.close();
}
```
- Once you've created a Statement object, you can then use it to execute an SQL statement with one of its three execute methods.
	- `boolean execute (String SQL)`: 
		- Returns a boolean value of true if a `ResultSet` object can be retrieved; otherwise, it returns false. 
		- Use this method to execute SQL DDL statements or when you need to use truly dynamic SQL.
	- `int executeUpdate (String SQL)`:
		- Returns the number of rows affected by the execution of the SQL statement. 
		- Use this method to execute SQL statements for which you expect to get a number of rows affected - for example, an INSERT, UPDATE, or DELETE statement.
	- `ResultSet executeQuery (String SQL)`:
		- Returns a `ResultSet` object. Use this method when you expect to get a result set, as you would with a SELECT statement.
##### The PreparedStatement Objects
- This statement gives you the flexibility of supplying arguments dynamically.
```Java
PreparedStatement pstmt = null; 
try { 
	String SQL = "Update Employees SET age = ? WHERE id = ?"; 
	pstmt = conn.prepareStatement(SQL); 
	. . . 
} 
catch (SQLException e) { 
	. . . 
} 
finally { 
	pstmt.close();
}
```
- All parameters in JDBC are represented by the `?` symbol, which is known as the **parameter marker**. You must supply values for every parameter before executing the SQL statement.
- The `setXXX()` methods bind values to the parameters, where **XXX** represents the Java data type of the value you wish to bind to the input parameter. If you forget to supply the values, you will receive an **SQLException**.
- Each parameter marker is referred by its **ordinal position**. The first marker represents position 1, the next position 2, and so forth. This method differs from that of Java array indices, which starts at 0.
- All of the **Statement object's** methods for interacting with the database (a) `execute()`, (b) `executeQuery()`, and (c) `executeUpdate()` also work with the `PreparedStatement` object. 
- However, the methods are modified to use SQL statements that **can input the parameters**.
##### The CallableStatement Objects
- Suppose, you need to execute the following:
```SQL
DELIMITER $$ 
DROP PROCEDURE IF EXISTS `EMP`.`getEmpName` $$ 
CREATE PROCEDURE `EMP`.`getEmpName` 
	(IN EMP_ID INT, OUT EMP_FIRST VARCHAR(255)) 
BEGIN 
	SELECT first INTO EMP_FIRST 
	FROM Employees 
	WHERE ID = EMP_ID; 
END $$ 
DELIMITER ;
```
- The following code snippet shows how to employ the `Connection.prepareCall()` method to instantiate a **CallableStatement** object based on the preceding stored procedure:
```Java
CallableStatement cstmt = null; 
try { 
	String SQL = "{call getEmpName (?, ?)}"; 
	cstmt = conn.prepareCall (SQL); 
	. . . 
} 
catch (SQLException e) { 
	. . . 
} 
finally { 
	cstmt.close();
}
```
- Using the **CallableStatement** objects is much like using the **PreparedStatement** objects. You must bind values to all the parameters before executing the statement, or you will receive an **SQLException**.
- If you have IN parameters, just follow the same rules and techniques that apply to a **PreparedStatement** object; use the `setXXX()` method that corresponds to the Java data type you are binding.
- When you use `OUT` and `INOUT` parameters you must employ an additional **CallableStatement** method, `registerOutParameter()`. 
- The `registerOutParameter()` method binds the JDBC data type, to the data type that the stored procedure is expected to return.
- Once you call your stored procedure, you retrieve the value from the `OUT` parameter with the appropriate `getXXX()` method. 
- This method casts the retrieved value of SQL type to a Java data type.
### ResultSets
- The `java.sql.ResultSet` interface represents the result set of a database query.
- A **ResultSet** object maintains a cursor that points to the **current row** in the result set. The term "result set" refers to the row and column data contained in a ResultSet object.
##### Type of ResultSet
- The possible `RSType` are given below. If you do not specify any **ResultSet** type, you will automatically get one that is `TYPE_FORWARD_ONLY`.
	 ![[types of ResultSet.png|800]]
##### Concurrency of ResultSet
- The possible `RSConcurrency` are given below. If you do not specify any Concurrency type, you will automatically get one that is `CONCUR_READ_ONLY`.
	 ![[concurrency of Resultset.png|800]]
```Java
try { 
	Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY); 
} 
catch(Exception ex) { 
	.... 
} 
finally { 
	.... 
}
```
##### Navigating a Result Set
- There are several methods in the ResultSet interface that involve moving the cursor, including:
	 ![[navigating ResultSet.png|800]]
##### Viewing a Result Set
- The **ResultSet** interface contains dozens of methods for getting the data of the current row.
- There is a get method for each of the possible data types, and each get method has two versions:
	 ![[veiwing ResultSet.png|800]]
- Similarly, there are get methods in the **ResultSet** interface for each of the eight Java **primitive types**, as well as common types such as `java.lang.String`, `java.lang.Object`, and `java.net.URL`.
- There are also methods for getting SQL data types `java.sql.Date`, `java.sql.Time`, `java.sql.TimeStamp`, `java.sql.Clob`, and `java.sql.Blob`. Check the documentation for more information about using these SQL data types.
##### Updating a Result Set
- The ResultSet interface contains a collection of update methods for updating the data of a result set.
- For example, to update a String column of the current row of a result set, you would use one of the following `updateString()` methods:
	 ![[update ResultSet.png|800]]
- There are update methods for the eight primitive data types, as well as **String**, **Object**, **URL**, and the SQL data types in the `java.sql` package.
- Updating a row in the result set changes the columns of the current row in the **ResultSet** object, but **not** in the underlying **database**. 
- To update your changes to the row in the database, you need to invoke one of the following methods:
	 ![[update database jdbc.png|800]]
### Transactions
- If your JDBC Connection is in _auto-commit_ mode, which it is by default, then every SQL statement is committed to the database upon its completion.
- That may be fine for simple applications, but there are three reasons why you may want to turn off the auto-commit and manage your own transactions:
	- To increase performance.
	- To maintain the integrity of business processes.
	- To use distributed transactions.
- Transactions enable you to control if, and when, changes are applied to the database. It treats a single SQL statement or a group of SQL statements as one logical unit, and if any statement fails, the whole transaction fails.
- To enable manual- transaction support instead of the _auto-commit_ mode that the JDBC driver uses by default, use the Connection object's `setAutoCommit()` method. 
- If you pass a boolean false to `setAutoCommit()`, you turn off _auto-commit_. You can pass a boolean true to turn it back on again. For example: `conn.setAutoCommit(false);`
##### Commit & Rollback
- Once you are done with your changes and you want to commit the changes then call `commit()` method on connection object as follows:`conn.commit();`.
- Otherwise, to roll back updates to the database made using the Connection named conn, use the following code:`conn.rollback();`.
- The following example illustrates the use of a commit and rollback object:
```Java
try{ 
	//Assume a valid connection object conn 
	conn.setAutoCommit(false); 
	Statement stmt = conn.createStatement(); 
	String SQL = "INSERT INTO Employees " +
				 "VALUES (106, 20, 'Rita', 'Tez')"; 
	stmt.executeUpdate(SQL); 
	//Submit a malformed SQL statement that breaks 
	String SQL = "INSERTED IN Employees " + 
				 "VALUES (107, 22, 'Sita', 'Singh')";
	stmt.executeUpdate(SQL); 
	// If there is no error. 
	conn.commit(); 
}
catch(SQLException se){ 
	// If there is any error. 
	conn.rollback(); 
}
```
##### Using Savepoints
- The new JDBC 3.0 Savepoint interface gives you the additional transactional control. Most modern DBMS, support savepoints within their environments such as Oracle's PL/SQL.
- When you set a savepoint you define a logical rollback point within a transaction. If an **error** occurs past a savepoint, you can use the **rollback** method to **undo** either all the changes or only the changes made after the savepoint.
- The Connection object has two new methods that help you manage savepoints:
	- `setSavepoint(String savepointName)`: 
		- Defines a new savepoint. 
		- It also returns a Savepoint object.
	- `releaseSavepoint(Savepoint savepointName)`: 
		- Deletes a savepoint. 
		- Notice that it requires a Savepoint object as a parameter. 
		- This object is usually a savepoint generated by the `setSavepoint()` method.
- There is one `rollback (String savepointName)` method, which rolls back work to the specified savepoint.
- The following example illustrates the use of a Savepoint object:
```Java
try{ 
	//Assume a valid connection object conn 
	conn.setAutoCommit(false); 
	Statement stmt = conn.createStatement(); 
	//set a Savepoint 
	Savepoint savepoint1 = conn.setSavepoint("Savepoint1"); 
	String SQL = "INSERT INTO Employees " + 
				 "VALUES (106, 20, 'Rita', 'Tez')";
	stmt.executeUpdate(SQL); 
	//Submit a malformed SQL statement that breaks 
	String SQL = "INSERTED IN Employees " + 
				 "VALUES (107, 22, 'Sita', 'Tez')";
	stmt.executeUpdate(SQL); 
	// If there is no error, commit the changes. 
	conn.commit(); 
}
catch(SQLException se){ 
	// If there is any error. 
	conn.rollback(savepoint1); 
}
```
# Sources
- [JDBC Tutorial](https://www.tutorialspoint.com/jdbc/index.htm).