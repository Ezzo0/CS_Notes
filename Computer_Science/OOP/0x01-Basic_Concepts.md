
1. Class
   - Definition: Blueprint for creating objects
   - Example: Car class with attributes and methods
```                                                                                 java
public class Car {
    // Attributes
    String brand;
    String model;
    int year;
    
    // Method
    void displayInfo() {
        System.out.println("Brand: " + brand);
        System.out.println("Model: " + model);
        System.out.println("Year: " + year);
    }
}

```
1. Objects
   - Definition: Specific instances of a class
   - Example: Creating car objects from the Car class

```                                                                                   java
public class Main {
    public static void main(String[] args) {
        // Creating objects (instances) of the Car class
        Car myCar = new Car();
        Car anotherCar = new Car();
    }
}

```

2. <span style="color:#92d050">Private Data and Data Hiding</span>
   - Definition: Restricting access to data members within a class
   - Example: Declaring private data members in a class
``` java
public class Car {
    // Private attributes
    private String brand;
    private String model;
    private int year;
}

```
3. <span style="color:#92d050">Setter Methods</span>
   - Definition: Methods to set the value of private data members
   - Example: Writing setter methods for the Car class
```java
public class Car { 
    // Private attributes
    private String brand;
    private String model;
    private int year;
    
    // Setter methods
    public void setBrand(String brand) {
        this.brand = brand;
    }
    
    public void setModel(String model) {
        this.model = model;
    }
    
    public void setYear(int year) {
        this.year = year;
    }
}

```
4. <span style="color:#92d050">Getter Methods</span>
```java
public class Car {
    // Private attributes
    private String brand;
    private String model;
    private int year;
    
    // Getter methods
    public String getBrand() {
        return brand;
    }
    
    public String getModel() {
        return model;
    }
    
    public int getYear() {
        return year;
    }
}

```
5. <span style="color:#92d050">toString Method</span>
   - Definition: Method to return a string representation of an object
   - Example: Overriding the toString method in the Car class
```java
public class Car {
    // Attributes and methods...
    
    // Override toString method
    @Override
    public String toString() {
        return "Brand: " + brand + ", Model: " + model + ", Year: " + year;
    }
}

```
***
- <span style="color:#92d050">passing Objects as arguments</span>
	  - <span style="color:#92d050">Pass-by-Value Semantics:</span>
			  - In Java, objects are passed by value, meaning a copy of the reference to the object is passed, not the object itself. Changes to the object's state within the method affect the original object.
``` java
/* Objects Call by value gives effect of call by refrence */
	class MyClass {
    int value;
}

public class Main {
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        obj.value = 5;
        modifyObject(obj);
        System.out.println(obj.value); // Output: 10
    }

    static void modifyObject(MyClass obj) {
        obj.value = 10;
    }
}

```
- <span style="color:#92d050">**Mutability and Immutability:** </span>
	- For mutable objects, changes made to the object's state within the method affect the original object. For immutable objects, methods cannot modify the object's state but can return a new instance with desired changes.
```java
class Immutable {
    final int value;

    Immutable(int value) {
        this.value = value;
    }

    Immutable add(int num) {
        return new Immutable(this.value + num);
    }
}

public class Main {
    public static void main(String[] args) {
        Immutable obj = new Immutable(5);
        obj = obj.add(10);
        System.out.println(obj.value); // Output: 15
    }
}

<span style="color:#92d050">```
- <span style="color:#92d050">**Encapsulation and Access Control:**</span>
	- Object state is encapsulated within classes, and access to this state is controlled by methods. Methods can manipulate the object's state through its public methods, maintaining encapsulation.
```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        account.deposit(100);
        System.out.println(account.getBalance()); // Output: 100.0
    }
}

```
6. <span style="color:#92d050">**Constructors**:</span>
	- **Constructors**: Constructors are special methods used to initialize objects. A non-argument constructor doesn't take any parameters and is used to create objects with default initializations. A parameterized constructor takes parameters to initialize object state with specific values.
		- **<span style="color:#92d050">Default Constructor:</span>**
			- In Java, a default constructor is a constructor that is automatically provided by the compiler if no constructor is explicitly defined in a class. It initializes objects with default values, such as zero for numeric types and null for reference types.
		-<span style="color:#92d050"> **Non-argument Constructor**:</span>
			- Definition: A constructor that doesn't take any arguments.
			- Example: Creating a non-argument constructor for the `Car` class.
			- Note: It must be the same name of the class `Car` with no datatype
	            
	   ```java
		public class Car {
		    private String brand;
		    private String model;
		    private int year;
		    public Car() {
		    	this.brand = "Brand";
		        this.model = "model";
		        this.year = 2020;
		    }
		}       
	
	    ```    
		- <span style="color:#92d050">**Parameterized Constructor**:</span>
			- Definition: A constructor that takes parameters to initialize object state.
			- Example: Creating a parameterized constructor for the `Car` class to initialize attributes.
	```java
		public class Car {
		    // Attributes
		    private String brand;
		    private String model;
		    private int year;
		    
		    // Parameterized constructor
		    public Car(String brand, String model, int year) {
		        this.brand = brand;
		        this.model = model;
		        this.year = year;
		    }
		}
	
	```
	-  **<span style="color:#92d050">Method Overloading**:</span>
		- **Method Overloading**: Method overloading allows you to define multiple methods with the same name but different parameters. The compiler determines which method to call based on the number and types of arguments passed to it. Overloading enables you to provide multiple ways to perform a task with the same method name.
		- Example: Overloading the `displayInfo` method in the `Car` class to handle different types of display.
```java
public class Car {
	// Attributes...
	
	// Method overloading
	public void displayInfo() {
		System.out.println("Brand: " + brand);
		System.out.println("Model: " + model);
		System.out.println("Year: " + year);
	}
	
	public void displayInfo(boolean includeYear) {
		System.out.println("Brand: " + brand);
		System.out.println("Model: " + model);
		if (includeYear) {
			System.out.println("Year: " + year);
		}
	}
}

```
- <span style="color:#00b050">7.Constructor Channing:</span>
	- Constructor chaining in OOP enables one constructor to call another within the same class, reusing initialization logic across different constructors, helping to avoid redundancy and duplication in code.
``` java
		public class Car {
		    // Attributes
		    private String brand;
		    private String model;
		    private int year;
		    private int color;
		    
		    // Parameterized constructor
		    public Car(String brand, String model, int year) {
		        this.brand = brand;
		        this.model = model;
		        this.year = year;
		    }
		    public Car(String brand, String model, int year, string color){
			    /* calling the construct*/
				this (brand, model, year);
				/* assigning a value to an instance variable `color` */
				this.color = color;
		    {
		}
	
```
8. **<span style="color:#00b050">Copy Constructor</span><span style="color:#00b050">:</span>**
	- A <span style="color:#00b0f0">copy constructor</span> is a special constructor in a class that <span style="color:#00b0f0">creates a new object as a copy</span> of an existing object. It is used to initialize a new object with the contents of another object of the same class.
``` java
public class MyClass {
    private int value;
    
    // Constructor
    public MyClass(int value) {
        this.value = value;
    }
    
    // Copy Constructor
    public MyClass(MyClass other) {
        this.value = other.value;
    }
    
    // Getter and Setter methods...
}

```
**<span style="color:#00b050">usage:</span>**
```java
MyClass obj1 = new MyClass(10);
MyClass obj2 = new MyClass(obj1); // Using the copy constructor

```
9.<span style="color:#92d050"> **Finalizer or Destructor:**</span>
	- In Java, a finalizer (or destructor) is a special method that is called by the garbage collector before reclaiming the memory occupied by an object. It is used to perform cleanup operations or release resources associated with the object before it is garbage collected.
```java
public class MyClass {
    // Constructor
    public MyClass() {
        // Initialization code
    }
    
    // Finalizer (Destructor)
    @Override
    protected void finalize() throws Throwable {
        try {
            // Cleanup code or resource release operations
        } finally {
            super.finalize();
        }
    }
}


MyClass obj = new MyClass();
// Use obj...
obj = null; // Set obj to null to make it eligible for garbage collection
/* Finalizer here works automaticaly when you init obj with null*/
```
- **Note:** It's important to note that finalizers are generally not recommended for resource cleanup in Java due to unpredictability and performance implications. It's better to use explicit resource management techniques such as try-with-resources or implementing the `AutoCloseable` interface for resource cleanup.
***
10. **<span style="color:#92d050">Static Keyword</span>**
	- In Java, the `static` keyword is used to declare variables, methods, and nested classes that belong to the class itself rather than to instances (objects) of the class. Static members are shared among all instances of the class and can be accessed using the class name.
	- **Benefits of Using `static`:**
		- **Shared Data:** Static variables are shared among all instances, conserving memory.
		- **Utility Methods:** Static methods don't require object instantiation and are handy for general tasks.
		- **Improved Performance:** Direct invocation of static methods enhances performance.
		- **Namespace Management:** Static nested classes help organize code by encapsulating related functionalities.
		- **Encapsulation:** Static nested classes promote encapsulation by hiding implementation details.
``` 
public class MyClass {
    static int serialNumber = 0; // Static variable
    private String name;

    public void myMethod(String name) {
        this.name = name;
        serialNumber++;
    }

    static void display() { // Static method
        /*can not use non static data but it's generic
	    so u can not use this.data*/
    }
}
```
- static class
```java

public class OuterClass {
    static class InnerClass { // Static nested class
        public int print(int x){
	        
        }
    }
}

/* Usage */

MyClass.serialNumber = 10; // Accessing static variable
MyClass.display(); // Calling static method
OuterClass.InnerClass inner = new OuterClass.InnerClass(); // Creating an instance of a static nested class
/* here is how u can access static class methods*/
System.out.println(inner.print(5));


```
11. equals():
``` java
public class Main {
    public static void main(String[] args) {
        String str1 = new String("seif");
        String str2 = new String("seif");
        
        System.out.println(str1 == str2); // Output: false
    }
}

```
- In this case, even though the content of `str1` and `str2` is the same, they are created using the `new` keyword, which explicitly creates a new string object. Therefore, `str1` and `str2` will have different memory addresses, and `str1 == str2` evaluates to `false`.
- while using `==` might work in certain cases where string interning occurs, it's generally safer to use `.equals()` for comparing the content of strings to ensure consistent behavior across all scenarios.
- <span style="color:#ff0000">conclusion</span>
	- In Java, `==` compares the memory addresses of objects, while `.equals()` compares their content. While `==` might work for comparing strings in some cases due to string interning, it's safer to use `.equals()` for consistent behavior, especially when dealing with dynamically created strings. This ensures that the comparison is based on the actual content of the strings rather than their memory addresses.
- ## <span style="color:#92d050">Returning Objects in Java:</span>
	In Java, methods can return objects of a specific class, allowing them to manipulate and return complex data structures.
```java
public class Math {

    int val;

    // Instance method to add two Math objects
    Math add(Math obj2) {
        Math result = new Math();
        result.val = this.val + obj2.val;
        return result;
    }
}

public class App {
    public static void main(String[] args) {
        Math objMath1 = new Math();
        Math objMath2 = new Math();
        objMath1.val = 5;
        objMath2.val = 5;
        
        // Call the add method on objMath1, passing objMath2 as a parameter
        Math result = objMath1.add(objMath2);
        System.out.println(result.val); // Output: 10
    }
}

```