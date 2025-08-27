# Explanation
- The term **generics** means _parameterized types_. Parameterized types are important because they enable you to create classes, interfaces, and methods in which the **type of data** upon which they operate is **specified** as a parameter.
### A Simple Generics Example
- The following program defines two classes. The first is the generic class `Gen`, and the second is `GenDemo`, which uses `Gen`.
```Java
// A simple generic class.
// Here, T is a type parameter that
// will be replaced by a real type
// when an object of type Gen is created.
class Gen<T> {
    T ob; // declare an object of type T

    // Pass the constructor a reference to
    // an object of type T.
    Gen(T o) {
        ob = o;
    }

    // Return ob.
    T getOb() {
        return ob;
    }

    // Show type of T.
    void showType() {
        System.out.println("Type of T is " + ob.getClass().getName());
    }
}

// Demonstrate the generic class.
class GenDemo {
    public static void main(String[] args) {
        // Create a Gen reference for Integers.
        Gen<Integer> iOb;

        // Create a Gen<Integer> object and assign its
        // reference to iOb. Notice the use of autoboxing
        // to encapsulate the value 88 within an Integer object.
        iOb = new Gen<Integer>(88);

        // Show the type of data used by iOb.
        iOb.showType();

        // Get the value in iOb. Notice that
        // no cast is needed.
        int v = iOb.getOb();
        System.out.println("value: " + v);
        System.out.println();

        // Create a Gen object for Strings.
        Gen<String> strOb = new Gen<String>("Generics Test");

        // Show the type of data used by strOb.
        strOb.showType();

        // Get the value of strOb. Again, notice
        // that no cast is needed.
        String str = strOb.getOb();
        System.out.println("value: " + str);
    }
}

```
- The output produced by the program is shown here:
```
Type of T is java.lang.Integer 
value: 88

Type of T is java.lang.String 
value: Generics Test
```
- Here, `T` is the name of a _type parameter_. This name is used as a placeholder for the actual type that will be passed to `Gen` when an object is created. 
- Notice that T is contained within `< >`. This syntax can be generalized. Whenever a _type parameter_ is being declared, it is specified within angle brackets.
- `T` is used to declare an object called `ob`, as shown above. As explained, `T` is a _placeholder_ for the actual type that will be specified when a `Gen` object is created. 
- Thus, `ob` will be an object of the type passed to `T`. For example, if type **String** is passed to `T`, then in that instance, `ob` will be of type **String**.
- It’s necessary to state that the Java compiler does not actually create different versions of `Gen`, or of any other generic class.
- Instead, the compiler removes all generic type information, substituting the necessary casts, to make your code _behave as if_ a specific version of `Gen` were created. 
- Thus, there is really only one version of `Gen` that actually exists in your program. The process of removing generic type information is called **erasure**.
### A Generic Class with Two Type Parameters
- You can declare more than one type parameter in a generic type. To specify two or more type parameters, simply use a comma-separated list. For example, the following `TwoGen` class is a variation of the `Gen` class that has two type parameters:
```Java
// A simple generic class with two type 
// parameters: T and V.
class Gen<T, V> {
    T ob1; 
    V ob2;
    
    // Pass the constructor a reference to 
    // an object of type T and an object of type V.
    Gen(T o1, V o2) {
        ob1 = o1;
        ob2 = o2;
    }

    // Show types of T and V.
    void showType() {
        System.out.println("Type of T is " + ob1.getClass().getName());
        System.out.println("Type of V is " + ob2.getClass().getName());
    }
    
    T getOb1() {
        return ob1;
    }
    V getOb2() {
        return ob2;
    }
}

// Demonstrate the generic class.
class SimpGen {
    public static void main(String[] args) {
	    TwoGen<Integer, String> tgObj = 
		    new TwoGen<Integer, String>(88, "Generics");

	// Show the types. 
	tgObj.showTypes();
	
	// Obtain and show values. 
	int v = tgObj.getOb1();
	System.out.println("value: " + v); 
	
	String str = tgObj.getOb2();
	System.out.println("value: " + str);
    }
}
```
- The output from this program is shown here:
```
Type of T is java.lang.Integer
Type of V is java.lang.String
value: 88
value: Generics
```
### Bounded Types
- Assume that you want to create a generic class that contains a method that returns the average of an array of numbers.
```Java
// Stats attempts (unsuccessfully) to 
// create a generic class that can compute
// the average of an array of numbers of // any given type.
//
// The class contains an error!
class Stats<T> {
    T[] nums; // nums is an array of type T

    // Pass the constructor a reference to 
    // an array of type T.
    Stats(T[] o) {
        nums = o;
    }

    // Return the average as a double in all cases.
    double average() {
        double sum = 0.0;
        for (T num : nums) {
            sum += num.doubleValue(); // Error.
        }

        return sum / nums.length;
    }
}
```
- Because all numeric classes, such as **Integer** and **Double**, are subclasses of **Number**, and **Number** defines the `doubleValue()` method, this method is available to all numeric wrapper classes. 
- The trouble is that the compiler has no way to know that you are intending to create `Stats` objects using only numeric types.
- To handle such situations, Java provides _bounded types_. When specifying a type parameter, you can create an upper bound that declares the superclass from which all type arguments must be derived.
- This is accomplished through the use of an `extends` clause when specifying the type parameter, as shown here: `<T extends superclass>`.
- This specifies that T can only be replaced by _superclass, or subclasses of superclass_. Thus, **superclass** defines an inclusive, upper limit.
```Java
// In this version of Stats, the type argument for 
// T must be either Number, or a class derived 
// from Number.
class Stats<T extends Number> {
    T[] nums; // array of Number or subclass

    // Pass the constructor a reference to 
    // an array of type T.
    Stats(T[] o) {
        nums = o;
    }

    // Return type double in all cases.
    double average() {
        double sum = 0.0;
        for(int i=0; i < nums.length; i++) 
	        sum += nums[i].doubleValue();

        return sum / nums.length;
    }
}
```
- In addition to using a class type as a bound, you can also use an **interface** type. In fact, you can specify multiple interfaces as bounds. 
- Furthermore, a bound can include both a class type and one or more interfaces. In this case, the class type must be specified first.
- When a bound includes an interface type, only type arguments that implement that interface are legal. 
- When specifying a bound that has a class and an interface, or multiple interfaces, use the **&** operator to connect them. This creates an _intersection type_. For example: `class Gen<T extends MyClass & MyInterface> { // ...`.
### Using Wildcard Arguments
- Given the `Stats` class shown at the end of the preceding section, assume that you want to add a method called `isSameAvg()` that determines if two Stats objects contain arrays that yield the same average, no matter what type of numeric data each object holds.
- At first, you might think of a solution like this, in which `T` is used as the type parameter:
```Java
// This won't work!
// Determine if two averages are the same.
boolean isSameAvg(Stats<T> ob) { 
	if(average() == ob.average()) 
		return true; 
	return false; 
}
```
- The trouble with this attempt is that it will work only with other `Stats` objects whose type is the same as the invoking object.
- For example, if the invoking object is of type `Stats<Integer>`, then the parameter `ob` must also be of type `Stats<Integer>`. 
- It can’t be used to compare the average of an object of type `Stats<Double>` with the average of an object of type `Stats<Short>`, for example.
- To create a generic `isSameAvg()` method, you must use another feature of Java generics: the _wildcard argument_. The wildcard argument is specified by the `?`, and it represents an unknown type.
```Java
// Determine if two averages are the same. 
// Notice the use of the wildcard.

boolean isSameAvg(Stats<?> ob) { 
	if(average() == ob.average()) 
		return true; 
	return false; 
}
```
- One last point: It is important to understand that the wildcard **does not affect** what type of `Stats` objects can be created. This is governed by the `extends` clause in the `Stats` declaration. The wildcard simply matches any valid `Stats` object.
##### Bounded Wildcards
- Wildcard arguments can be bounded in much the same way that a type parameter can be bounded. A bounded wildcard is especially important when you are creating a generic type that will operate on a class hierarchy.
- To understand why, let’s work through an example.
```Java
// Two-dimensional coordinates. 
class TwoD { 
	int x, y; 
	TwoD(int a, int b) { 
		x = a; y = b; 
	} 
} 

// Three-dimensional coordinates.
class ThreeD extends TwoD { 
	int z;
	ThreeD(int a, int b, int c) { 
		super(a, b); 
		z = c; 
	} 
} 

// Four-dimensional coordinates.
class FourD extends ThreeD { 
	int t;
	FourD(int a, int b, int c, int d) { 
		super(a, b, c); 
		t = d; 
	} 
}
```
- Shown next is a generic class called Coords, which stores an array of coordinates:
```Java
// This class holds an array of coordinate objects. 
class Coords<T extends TwoD> {
	T[] coords; 
	Coords(T[] o) { 
		coords = o; 
	} 
}
```
- what if you want to create a method that displays the `X`, `Y`, and `Z` coordinates of a `ThreeD` or `FourD` object? The trouble is that not all `Coords` objects will have three coordinates, because a `Coords<TwoD>` object will only have `X` and `Y`.
- The answer is the _bounded wildcard argument_. A bounded wildcard specifies either an **upper bound** or a **lower bound** for the type argument. This enables you to restrict the types of objects upon which a method will operate.
- The upper bound is created using an `extends` clause in much the same way it is used to create a bounded type.
```Java
static void showXYZ(Coords<? extends ThreeD> c) { 
	System.out.println("X Y Z Coordinates:"); 
	for(int i=0; i < c.coords.length; i++)
		System.out.println(c.coords[i].x + " " + 
						   c.coords[i].y + " " + 
						   c.coords[i].z); 
	System.out.println(); 
}
```
- You can also specify a lower bound for a wildcard by adding a `super` clause to a wildcard declaration. Here is its general form: `<? super subclass>`.
- In this case, only classes that are **superclasses** of subclass are acceptable arguments. This is an inclusive clause.
### Creating a Generic Method
- As the preceding examples have shown, methods inside a generic class can make use of a class’ type parameter and are, therefore, automatically generic relative to the type parameter. 
- However, it is possible to declare a generic method that uses one or more type parameters of its own. 
- Furthermore, it is possible to create a generic method that is enclosed within a non-generic class.
- The following program declares a non-generic class called `GenMethDemo` and a static generic method within that class called `isIn()`.
```Java
// Demonstrate a simple generic method. 
class GenMethDemo {
	// Determine if an object is in an array.
	static <T extends Comparable<T>, 
			V extends T> boolean isIn(T x, V[] y) 
	{ 
		for(int i=0; i < y.length; i++)
			if(x.equals(y[i])) 
				return true; 
		return false; 
	}
	public static void main(String[] args) { 
		// Use isIn() on Integers. 
		Integer[] nums = { 1, 2, 3, 4, 5 }; 
		
		if(isIn(2, nums)) 
			System.out.println("2 is in nums");
			
		if(!isIn(7, nums)) 
			System.out.println("7 is not in nums"); 
		
		System.out.println(); 
		// Use isIn() on Strings. 
		String[] strs = { "one", "two", "three", "four", "five" };
		
		if(isIn("two", strs)) 
			System.out.println("two is in strs"); 
		
		if(!isIn("seven", strs)) 
			System.out.println("seven is not in strs"); 
		
		// Oops! Won't compile! Types must be compatible. 
		// if(isIn("two", nums)) 
		//	System.out.println("two is in strs"); 
	} 
}
```
- Note that The type parameters are declared _before_ the return type of the method.
- Now, notice how `isIn()` is called within `main()` by use of the normal call syntax, without the need to specify type arguments. 
- This is because the types of the arguments are automatically discerned, and the types of T and V are adjusted accordingly.
- For example, in the first call: `if(isIn(2, nums))`. The type of the first argument is **Integer**, which causes **Integer** to be substituted for `T`. The base type of the second argument is also **Integer**, which makes Integer a substitute for `V`, too.
- Although type inference will be sufficient for most generic method calls, you can **explicitly** specify the type argument if needed. 
- For example, here is how the first call to `isIn()` looks when the type arguments are specified: `GenMethDemo.<Integer, Integer>isIn(2, nums)`.
- Here is the syntax for a generic method:
	 `<type-param-list > ret-type meth-name (param-list) { // …`
##### Generic Constructors
- It is possible for constructors to be **generic**, even if their class is **not**. For example, consider the following:
```Java
// Use a generic constructor. 
class GenCons {
	private double val;
	<T extends Number> GenCons(T arg) { 
		val = arg.doubleValue();
	} 
	
	void showVal() { 
		System.out.println("val: " + val); 
	} 
} 

class GenConsDemo { 
	public static void main(String[] args) { 
		GenCons test = new GenCons(100); 
		GenCons test2 = new GenCons(123.5F); 
		test.showVal(); 
		test2.showVal(); 
	} 
}
```
- Because `GenCons()` specifies a parameter of a generic type, which must be a subclass of Number, `GenCons()` can be called with any numeric type, including **Integer**, **Float**, or **Double**. Therefore, even though `GenCons` is not a generic class, its constructor is generic.
### Generic Interfaces
- Generic interfaces are specified just like generic classes. Here is an example. It creates an interface called `MinMax` that declares the methods `min()` and `max()`, which are expected to return the minimum and maximum value of some set of objects.
```Java
// A generic interface example. 
// A Min/Max interface.
interface MinMax<T extends Comparable<T>> {
	T min(); 
	T max(); 
}

// Now, implement MinMax
class MyClass<T extends Comparable<T>> implements MinMax<T> {
	T[] vals;
	MyClass(T[] o) { vals = o; }

	// Return the minimum value in vals.
	public T min() { 
		T v = vals[0];
		
		for(int i=1; i < vals.length; i++) 
			if(vals[i].compareTo(v) < 0) v = vals[i]; 
		
		return v; 
	}

	// Return the maximum value in vals.
	public T max() { 
		T v = vals[0]; 
		
		for(int i=1; i < vals.length; i++) 
			if(vals[i].compareTo(v) > 0) v = vals[i]; 
		
		return v; 
	} 
}

class GenIFDemo { 
	public static void main(String[] args) { 
		Integer[] inums = {3, 6, 2, 8, 6 }; 
		Character[] chs = {'b', 'r', 'p', 'w' };

		MyClass<Integer> iob = new MyClass<Integer>(inums);
		MyClass<Character> cob = new MyClass<Character>(chs);
		
		System.out.println("Max value in inums: " + iob.max());
		System.out.println("Min value in inums: " + iob.min());
		
		System.out.println("Max value in chs: " + cob.max()); 
		System.out.println("Min value in chs: " + cob.min());
	}
}		
```
- Notice the declaration of `MyClass`, shown here:
	`class MyClass<T extends Comparable<T>> implements MinMax<T> {`
- Because `MinMax` requires a type that implements Comparable, the implementing class (`MyClass` in this case) must specify the same bound. Furthermore, once this bound has been established, there is no need to specify it again in the implements clause.
- In fact, it would be wrong to do so. For example, this line is incorrect and won’t compile: 
```Java
// This is wrong!
class MyClass<T extends Comparable<T>> 
	implements MinMax<T extends Comparable<T>> {
```
- Once the type parameter has been established, it is simply passed to the interface without further modification.
- In general, if a class implements a generic interface, then that class must also be generic, at least to the extent that it takes a type parameter that is passed to the interface. For example, the following attempt to declare `MyClass` is in error:
	 `class MyClass implements MinMax<T> { // Wrong!`
- Because `MyClass` does not declare a type parameter, there is no way to pass one to `MinMax`. In this case, the identifier `T` is simply unknown, and the compiler reports an error. 
- Of course, if a class implements a _specific type_ of generic interface, such as shown here: `class MyClass implements MinMax<Integer> { // OK` then the implementing class does not need to be generic.
- Here is the generalized syntax for a generic interface:
	 `interface interface-name<type-param-list> { // …`
- When a generic interface is implemented, you must specify the type arguments, as shown here: 
``` java
class class-name<type-param-list> 
	implements interface-name<type-arg-list> {
```
### Generic Class Hierarchies
- A generic class can act as a **superclass** or be a **subclass**. The key difference between generic and non-generic hierarchies is that in a generic hierarchy, any **type arguments** needed by a generic superclass must be **passed up** the hierarchy by **all** subclasses.
- This is similar to the way that constructor arguments must be passed up a hierarchy.
##### Using a Generic Superclass
- Here is a simple example of a hierarchy that uses a generic superclass:
```Java
// A simple generic class hierarchy.
class Gen<T> {
    T ob;
    
    Gen(T o) {
        ob = o;
    }

    // Return ob.
    T getOb() {
        return ob;
    }
}

// A subclass of Gen.
class Gen2<T> extends Gen<T> {
    Gen2(T o) {
        super(o);
    }
}
```
- The type parameter `T` is specified by `Gen2` and is also passed to `Gen` in the extends clause. This means that whatever type is passed to `Gen2` will also be passed to `Gen`.
- Of course, a subclass is free to add its own type parameters, if needed. For example:
```Java
// A subclass of Gen that defines a second 
// type parameter, called V.
class Gen2<T, V> extends Gen<T> { 
	V ob2; 
	Gen2(T o, V o2) { 
		super(o); 
		ob2 = o2; 
	} 
	
	V getOb2() { 
		return ob2; 
	} 
}
```
##### A Generic Subclass
- It is perfectly acceptable for a non-generic class to be the superclass of a generic subclass. For example:
```Java
// A non-generic class can be the superclass
// of a generic subclass.

// A non-generic class.
class NonGen {
    int num;

    NonGen(int i) {
        num = i;
    }

    int getnum() {
        return num;
    }
}

// A generic subclass.
class Gen<T> extends NonGen {
    T ob; // declare an object of type T

    // Pass the constructor a reference to
    // an object of type T.
    Gen(T o, int i) {
        super(i);
        ob = o;
    }

    // Return ob.
    T getOb() {
        return ob;
    }
}

// Create a Gen object.
class HierDemo2 {
    public static void main(String[] args) {
        // Create a Gen object for String.
        Gen<String> w = new Gen<String>("Hello", 47);
        System.out.print(w.getOb() + " ");
        System.out.println(w.getnum());
    }
}
```
##### Casting
- You can cast one instance of a generic class into another only if the two are otherwise compatible and their type arguments are the same. For example:
```Java
class Main { 
	public static void main(String[] args) {
		// Create a Gen object for Integers. 
		Gen<Integer> iOb = new Gen<Integer>(88);
		
		// Create a Gen2 object for Integers. 
		Gen2<Integer> iOb2 = new Gen2<Integer>(99);
		
		// Create a Gen2 object for Strings.
		Gen2<String> strOb2 = new Gen2<String>("Generics Test");
	}
}
```
- Assuming the foregoing program, this cast is legal: `(Gen<Integer>) iOb2` because `iOb2` includes an instance of `Gen<Integer>`. But, this cast: `(Gen<Long>) iOb2` is not legal because `iOb2` is not an instance of `Gen<Long>`.
### Erasure
- An important constraint that governed the way that generics were added to Java was the need for compatibility with previous versions of Java. 
- The way Java implements generics while satisfying this constraint is through the use of _erasure_.
- When your Java code is compiled, all generic type information is **removed** (erased). This means replacing type parameters with their bound type, which is **Object** if no explicit bound is specified, and then applying the appropriate casts (as determined by the type arguments) to maintain type compatibility with the types specified by the type arguments.
- The compiler also enforces this type compatibility. This approach to generics means that **no type parameters** exist at run time. They are simply a **source-code mechanism**.
### Ambiguity Errors
- The inclusion of generics gives rise to an error that you must guard against: _ambiguity_. Ambiguity errors occur when erasure causes two seemingly distinct generic declarations to resolve to the same erased type, causing a conflict.
- Here is an example that involves method overloading:
```Java
// Ambiguity caused by erasure on 
// overloaded methods.
class MyGenClass<T, V> { 
	T ob1; 
	V ob2; 
	
	// ...

	// These two overloaded methods are ambiguous 
	// and will not compile.
	void set(T o) { ob1 = o; } 
	void set(V o) { ob2 = o; } 
}
```
- Inside `MyGenClass`, an attempt is made to overload `set()` based on parameters of type `T` and `V`. This looks reasonable because `T` and `V` appear to be different types. However, there are two ambiguity problems here.
- First, as `MyGenClass` is written, there is no requirement that `T` and `V` actually be different types. For example, it is perfectly correct (in principle) to construct a `MyGenClass` object as shown here:
	 `MyGenClass<String, String> obj = new MyGenClass<String, String>()`
- In this case, both `T` and `V` will be replaced by **String**. This makes both versions of `set()` identical, which is, of course, an error..
- The second and more fundamental problem is that the type erasure of `set()` reduces both versions to the following:`void set(Object o) { // ...`.
- Thus, the overloading of `set()` as attempted in `MyGenClass` is inherently ambiguous.
- In the preceding example, it would be much better to use two separate method names, rather than trying to overload `set()`. Often, the solution to ambiguity involves the **restructuring of the code**, because ambiguity frequently means that you have a conceptual error in your design.
# Sources
- Java The Complete Reference - Chapter 14.