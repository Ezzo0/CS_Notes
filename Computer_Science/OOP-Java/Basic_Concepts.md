
1. Class
   - Definition: Blueprint for creating objects
   - Example: Car class with attributes and methods
```
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

```
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
```
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
```
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
   - Definition: Methods to retrieve the value of private data members
   - Example: Writing getter methods for the Car class
```
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
```
public class Car {
    // Attributes and methods...
    
    // Override toString method
    @Override
    public String toString() {
        return "Brand: " + brand + ", Model: " + model + ", Year: " + year;
    }
}

```
