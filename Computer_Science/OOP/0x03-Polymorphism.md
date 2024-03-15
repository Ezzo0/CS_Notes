# Polymorphism in Java

- ## Definition

- **Polymorphism** is the ability of an object to take on multiple forms. In Java, polymorphism allows objects of different classes to be treated as objects of a common superclass.

- ## <span style="color:#92d050">Types of Polymorphism</span>

	1. **Compile-time Polymorphism:** Also known as static polymorphism, it is achieved through method overloading and method overriding.
	    - **Method Overloading:** Allows a class to have multiple methods with the same name but different parameters.
	    - **Method Overriding:** Allows a subclass to provide a specific implementation of a method that is already defined in its superclass.
	2. **Runtime Polymorphism:** Also known as dynamic polymorphism, it is achieved through method overriding.
	    - Method calls are resolved at runtime based on the actual type of the object.

- ## Method Overloading <span style="color:#92d050">(Early or Static Binding)</span> - <span style="color:#92d050">(compile-time polymorphism)</span>
	
	- **Definition:** Method overloading allows a class to have multiple methods with the same name but different parameters.
	- **Example:**
```java
class Math {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}

```
- ## Method Overriding <span style="color:#92d050">(Late or Dynamic Binding )</span> - <span style="color:#92d050">(run-rime Polymorphism)</span>

	- **Definition:** Method overriding allows a subclass to provide a specific implementation of a method that is already defined in its superclass.
	- **Example:**
```java
class Animal {
    void makeSound() {
        System.out.println("Some sound");
    }
}

class Dog extends Animal {
    void makeSound() {
        System.out.println("Bark");
    }
}

```
**Upcasting in Object-Oriented Programming (OOP)**

- **Definition**: Upcasting is a fundamental concept in OOP, involving the implicit conversion of a reference of a subclass type to its superclass type.
    
- **Superclass-Subclass Relationship**: Inheritance is key, where subclasses inherit attributes and methods from their superclass.
- **Access to Inherited Members**: Once upcasted, only superclass members are accessible, regardless of the actual object type.

- **Polymorphism**: Enables polymorphic behavior, where the same method call behaves differently based on the actual object type at runtime.
    
- **Downcasting**: The reverse process, requiring explicit type casting to convert a superclass reference back to a subclass reference.
```java
// Define a superclass Animal
class Animal {
    void makeSound() {
        System.out.println("Some generic sound");
    }
}

// Define a subclass Dog inheriting from Animal
class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Woof");
    }

    void fetch() {
        System.out.println("Fetching the ball");
    }
}

public class Main {
    public static void main(String[] args) {
        // Upcasting: Create a Dog object and assign it to an Animal reference
        Animal animal = new Dog(); // Upcasting

        // Accessing superclass method
        animal.makeSound(); // Output: "Woof"

        // Error: Cannot access subclass-specific method using superclass reference
        // animal.fetch(); // Compile-time error

        // Downcasting: Explicitly cast the Animal reference back to a Dog reference
        Dog dog = (Dog) animal; // Downcasting

        // Accessing subclass-specific method after downcasting
        dog.fetch(); // Output: "Fetching the ball"
    }
}

```
-  Explanation :
	- [Upcasting & Downcasting](https://www.youtube.com/watch?v=HpuH7n9VOYk)