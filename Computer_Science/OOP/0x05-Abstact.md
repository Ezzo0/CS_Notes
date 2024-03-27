### Abstract Classes and Methods in Java

- **Abstract Classes:**

	- An abstract class in Java is a class that cannot be instantiated on its own and is meant to be sub classed.
	- It serves as a blueprint for other classes and can contain both abstract and concrete methods.
	- Abstract classes are declared using the `abstract` keyword.
- **Abstract Methods:**

	- An abstract method is a method declared without an implementation (body).
	- It is meant to be overridden (implemented) by subclasses.
	- Abstract methods are declared using the `abstract` keyword and terminated with a semicolon (;) instead of a method body.
```java

abstract class AbstractClass {
    // Abstract method declaration
    abstract void abstractMethod();

    // Concrete method
    void concreteMethod() {
        // Method body
    }
}

```
- **Rules for Abstract Classes:**
	1. Abstract classes cannot be instantiated directly. They are meant to be extended.
	2. Abstract classes can contain both abstract and concrete methods.
	3. If a class contains at least one abstract method, the class must be declared as abstract.
	4. Abstract classes can have constructors and member variables.
- **Rules for Abstract Methods:**
	1. Abstract methods cannot have a method body.
	2. Subclasses must override all abstract methods of their abstract superclass unless the subclass is also declared abstract.
	3. Abstract methods cannot be declared as `final` or `private`.
```java
abstract class Shape {
    // Abstract method to calculate area
    abstract double calculateArea();

    // Concrete method
    void display() {
        System.out.println("Displaying shape...");
    }
}

class Circle extends Shape {
    double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    // Implementation of abstract method
    double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Main {
    public static void main(String[] args) {
        Circle circle = new Circle(5);
        System.out.println("Area of circle: " + circle.calculateArea());
        circle.display();
    }
}

```
- **Benefits of Abstract Classes and Methods:**

	- Encourages code reusability and promotes consistency in class hierarchies.
	- Provides a way to define a common interface for a group of related subclasses.
	- Allows for the implementation of generic functionality in the abstract class and specific functionality in subclasses.