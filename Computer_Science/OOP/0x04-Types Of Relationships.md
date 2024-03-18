## Association

- **Definition**: Association represents a relationship between two or more classes.
- **Characteristics**:
    - Can be one-to-one, one-to-many, or many-to-many.
    - Classes remain independent and can exist without each other.
- **UML Representation**:
	- A single teacher has multiple students.
		 ![[Pasted image 20240318174319.png]]
		 -  A single student can associate with many teachers.
		  ![[Pasted image 20240318174350.png]]
```java
public class Car {
    private Engine engine;

    // Constructor
    public Car(Engine engine) {
        this.engine = engine;
    }

    // Other attributes and methods
}

public class Engine {
    // Engine class implementation
}

// Usage example:
Engine engine = new Engine();
Car car = new Car(engine);

```
- **Garbage Collector**: In association, objects referenced by one class may be automatically cleaned up by the garbage collector if there are no other references to them, even if the association is severed.
## Composition

- **Definition**: Composition is a strong form of association where one class (whole) is composed of other classes (parts).
- **Characteristics**:
    - Parts are created and destroyed with the whole.
    - Parts have no meaning or purpose outside the whole.
- **UML Representation**:![[Pasted image 20240318174634.png]]
```java
public class Car {
    private Engine engine; // Composition relationship

    // Constructor
    public Car() {
        this.engine = new Engine(); // Create engine object along with the car
    }

    // Other attributes and methods
}

public class Engine {
    // Engine class implementation
}

// Usage example:
Car car = new Car();

```
- **Garbage Collector**: In composition, objects that are part of the whole (e.g., engine in a car) are automatically cleaned up by the garbage collector when the whole object is no longer in use.
## Aggregation

- **Definition**: Aggregation is a weak form of association where one class (whole) has a relationship with another class (part).
- **Characteristics**:
    - Parts can exist independently of the whole.
    - Parts may belong to multiple wholes.
- **UML Representation**:![[Pasted image 20240318174625.png]]
```java
public class Department {
    private List<Employee> employees; // Aggregation relationship

    // Constructor
    public Department() {
        this.employees = new ArrayList<>(); // Initialize the list of employees
    }

    // Other attributes and methods
}

public class Employee {
    // Employee class implementation
}

// Usage example:
Department department = new Department();

```
- **Garbage Collector**: In aggregation, objects that are part of the whole (e.g., employees in a department) are not automatically cleaned up by the garbage collector when the whole object is no longer in use. They may exist independently and be managed separately.
## **Composition Vs Aggregation**
- The composition and aggregation are two subsets of association. In both of the cases, the object of one class is owned by the object of another class; the only difference is that in composition, the child does not exist independently of its parent, whereas in aggregation, the child is not dependent on its parent i.e., standalone. An aggregation is a special form of association, and composition is the special form of aggregation.
![[Pasted image 20240318174545.png]]![[Screenshot (58).png]]