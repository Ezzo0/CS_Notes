## <span style="color:#92d050">Introduction to Inheritance:</span>
- Inheritance is very important concept in OOP that allows class (sub-class, child, derived) to inherit from another - (super, parent, base) class.
- It promotes code reusability and establishes a hierarchical relationship between classes.
- ## <span style="color:#92d050">Key Terminology:</span>
	- **<span style="color:#b336ec">Superclass</span>(Parent/Base Class):** The class whose properties and behaviors are inherited.
	- **<span style="color:#b336ec">Subclass</span>(Child/Derived Class)**: The class that inherits properties and behaviors from the superclass.
	- **<span style="color:#b336ec">extends Keyword:</span>** Used in Java to establish inheritance between classes. The subclass extends the superclass.
```java
class Superclass {
    // superclass members
}

class Subclass extends Superclass {
    // subclass members
}

```
- ## <span style="color:#92d050">Access Modifiers and Inheritance:</span> 
	- <span style="color:#00b0f0">Inheritance affects member access in subclasses:</span>
		- #Public members of the superclass are accessible in the subclass.
		- #Private members of the superclass are not accessible in the subclass.
		- #Protected members of the superclass are accessible in the subclass and within the same package.  (java only)
		- #Default (package-private) members of the superclass are accessible in the subclass if they are in the same package.
	
- #### <span style="color:#92d050">Types of Inheritance:</span>
	1. **Single Inheritance**: A subclass extends only one superclass.
	2. **Multilevel Inheritance**: A subclass extends another subclass, creating a chain of inheritance.
	3. **Hierarchical Inheritance**: Multiple subclasses inherit from a single superclass.
	4. **Multiple Inheritance <span style="color:#ff0000">(Not Supported in Java)</span>**: A class inherits from multiple classes. Java doesn't support this directly to avoid the diamond problem, but it can be achieved through interfaces. <span style="color:#7030a0">(found in c++)</span>

#### <span style="color:#92d050">Constructors in Inheritance:</span>
- Constructors <span style="color:#ff0000">are not inherited</span> but are <span style="color:#00b0f0">called implicitly</span> when an object of a subclass is created.
- The superclass constructor is <span style="color:#00b0f0">invoked</span> before the subclass constructor.
- If the superclass has multiple constructors, the default constructor is called implicitly <span style="color:#ff0000">if no other constructor is explicitly called.
</span>`
``` java
	public class Gamer extends Games {

    private String player;

    private String playerType;

    Gamer() {

    }

    Gamer(String gameName, int gameYearRelease, int gamePlayTime,

            String gameCategory, boolean GOTY, String player, String playerType) {

        super(gameName, gameYearRelease, gamePlayTime, gameCategory, GOTY);

        this.player = player;

        this.playerType = playerType;

        System.out.println("child constructor");

    }
}
```
#### <span style="color:#92d050">Method Overriding:</span> 
- Subclasses can override (redefine) methods inherited from the superclass to provide specialized behavior.
	```java
	class Animal {
	    void makeSound() {
	        System.out.println("Animal makes a sound");
	    }
	}
	
	class Dog extends Animal {
	    // Overrides the makeSound method from Animal
	    void makeSound() {
	        System.out.println("Dog barks");
	    }
	}
	
	public class Main {
	    public static void main(String[] args) {
	        Animal animal = new Animal();
	        animal.makeSound(); // Output: Animal makes a sound
	        
	        Dog dog = new Dog();
	        dog.makeSound(); // Output: Dog barks
	    }
	}
	
	```
- Method signature (name, parameter types, return type) <span style="color:#ff0000">must match</span> the superclass method.
- Annotation `@Override` is used to indicate that a method is intended to <span style="color:#ff0000">override a superclass method.</span>
> [!override] #Override
>	The `@Override` annotation is used in Java to explicitly indicate that a method in a subclass is intended to override a method in the superclass.
	  It helps in catching errors at compile-time by informing the compiler to check if the annotated method is indeed overriding a method from the superclass.
#### <span style="color:#92d050">Super Keyword:</span>

- The `super` keyword is used to refer to superclass members (fields, methods, constructors) from within a subclass.
- It can be used to call superclass constructors, access superclass methods, and access superclass fields hidden by the subclass.
- [Super](https://www.youtube.com/watch?v=Qb_NUn0TSAU)
#### <span style="color:#92d050">Final Classes and Methods:</span>

- Use the `final` keyword to prevent a class from being sub classed (final class) or a method from being overridden (final method).
- Final classes cannot have subclasses, and final methods cannot be overridden in subclasses.
```java
public class FinalExample {
    // Final variable
    final int constantValue = 10;

    // Final method
    final void finalMethod() {
        // Method implementation
    }

    public static void main(String[] args) {
        // Final local variable
        final int localVar = 20;
        // localVar = 30; // Error: Cannot assign a value to final variable localVar

        FinalExample obj = new FinalExample();
        // obj.constantValue = 20; // Error: Cannot assign a value to final variable constantValue

        // Calling final method
        obj.finalMethod();
    }
}

```
- <span style="color:#92d050">Explanation:</span>

	- The `final` keyword is used to declare constants, methods, and classes.( <span style="color:#ff0000">same as C</span> `constant`)
	- In the example:
	- `constantValue` is a final variable, and once assigned, its value cannot be changed.
	- `finalMethod()` is a final method, and it cannot be overridden in any subclass.
	- `localVar` is a final local variable, and its value cannot be changed after initialization.
	- Attempting to modify the value of a final variable or override a final method will result in a compilation error.
