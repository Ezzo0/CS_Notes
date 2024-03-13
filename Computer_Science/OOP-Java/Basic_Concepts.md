
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

2. Private Data and Data Hiding
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
3. Setter Methods
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
4. Getter Methods
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
5. toString Method
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
6. **Constructors**:
	- **Constructors**: Constructors are special methods used to initialize objects. A non-argument constructor doesn't take any parameters and is used to create objects with default initializations. A parameterized constructor takes parameters to initialize object state with specific values.
	    - **Non-argument Constructor**:
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
		- **Parameterized Constructor**:
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
7. **Method Overloading**:
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
