# Explanation
- Key to understanding Java’s implementation of lambda expressions are two constructs. The first is the _lambda expression_, itself. The second is the _functional interface_.
- A _lambda expression_ is, essentially, an anonymous method. However, this method is **not executed** on its own. Instead, it is used to **implement** a method defined by a functional interface.
- Thus, a lambda expression results in a form of **anonymous class**. Lambda expressions are also commonly referred to as _closures_.
- A _functional interface_ is an interface that contains **one and only one** abstract method. For example, the standard interface [[0x02_Multithreaded  Programming#Implementing Runnable|Runnable]] is a functional interface because it defines only one method: `run()`.
- Furthermore, a functional interface defines the _target type_ of a lambda expression.
### Lambda Expression Fundamentals
- The lambda expression introduced a new syntax element and operator into the Java language. New operator referred to as the _lambda operator_ or the _arrow operator_, is `->`.
- It divides a lambda expression into two parts. 
	- The left side specifies any parameters required by the lambda expression. 
	- On the right side is the lambda body, which specifies the actions of the lambda expression.
- The simplest type of lambda expression you can write is: `() -> 123.45`. 
- This lambda expression takes no parameters, thus the parameter list is empty. It returns the constant value `123.45`. Therefore, it is similar to the following method: `double myMeth() { return 123.45; }`.
- When a lambda expression requires a parameter, it is specified in the parameter list on the left side of the lambda operator. Here is a simple example: 
	 `(n) -> (n % 2)==0`.
### Functional Interfaces
- Let’s work through an example that shows how a lambda expression can be used. First, a reference to the functional interface `MyNumber` is declared. Next, a lambda expression is assigned to that interface reference::
```Java
interface MyNumber { 
	double getValue(); 
}

// Create a reference to a MyNumber instance. 
MyNumber myNum;
// Use a lambda in an assignment context. 
myNum = () -> 123.45;
```
- When a lambda expression occurs in a target type context, an instance of a **class is automatically created** that implements the functional interface, with the lambda expression defining the behavior of the abstract method declared by the functional interface. 
- When that method is called through the target, the lambda expression is executed. Thus, a lambda expression gives us a way to transform a code segment into an object.
- In the preceding example, the lambda expression becomes the implementation for the `getValue()` method. As a result, the following displays the value `123.45`:
```java
// Call getValue(), which is implemented by the previously assigned 
// lambda expression.
System.out.println(myNum.getValue());
```
- In order for a lambda expression to be used in a target type context, the type of the abstract method and the type of the lambda expression must be **compatible**. 
- For example, if the abstract method specifies two **int** parameters, then the lambda must specify two parameters whose type either is explicitly **int** or can be implicitly inferred as **int** by the context. 
- In general, the type and number of the lambda **expression’s parameters** must be **compatible** with the method’s parameters; the **return types** must be compatible; and **any exceptions** thrown by the lambda expression must be acceptable to the method.
- Let's put the pieces together:
```java
interface NumericTest {
	boolean test(int n); 
}

class LambdaDemo2 { 
	public static void main(String[] args) {
		// A lambda expression that tests if a number is even.
		NumericTest isEven = (n) -> (n % 2)==0;
		
		if(isEven.test(10)) System.out.println("10 is even");
		if(!isEven.test(9)) System.out.println("9 is not even");
		
		// Now, use a lambda expression that tests if a number 
		// is non-negative.
		NumericTest isNonNeg = (n) -> n >= 0;
		
		if(isNonNeg.test(1)) System.out.println("1 is non-negative"); 
		if(!isNonNeg.test(-1)) System.out.println("-1 is negative");.
	}
}
```
- An important point about multiple parameters: If you need to explicitly declare the type of a parameter, then all of the parameters must have declared types. For example, this is legal: `(int n, int d) -> (n % d) == 0`. But this is not: `(int n, d) -> (n % d) == 0`.
### Block Lambda Expressions
- The preceding examples consist of a single expression. These types of lambda bodies are referred to as _expression bodies_, and lambdas that have expression bodies are sometimes called _expression lambdas_. 
- In an expression body, the code on the right side of the lambda operator must consist of a **single expression**.
- Java supports a second type of lambda expression in which the code on the right side of the lambda operator consists of a block of code that can contain more than one statement. 
- This type of lambda body is called a _block body_. Lambdas that have block bodies are sometimes referred to as _block lambdas_.
- Here is an example that uses a block lambda to compute and return the factorial of an **int** value:
```Java
// A block lambda that computes the factorial of an int value.
interface NumericFunc { 
	int func(int n); 
}

class BlockLambdaDemo { 
	public static void main(String[] args) {
		// This block lambda computes the factorial of an int value.
		NumericFunc factorial = (n) -> { 
			int result = 1; 
			for(int i = 1; i <= n; i++) 
			result = i * result; 
			return result;
		}; 
		System.out.println("The factoral of 3 is " + factorial.func(3));
		System.out.println("The factoral of 5 is " + factorial.func(5));
	}
}
```
### Generic Functional Interfaces
- A lambda expression **cannot** be generic. However, the functional interface associated with a lambda expression **can be generic**.
```Java
// Use a generic functional interface with lambda expressions. 
// A generic functional interface.
interface SomeFunc<T> {
	T func(T t); 
}

class GenericFunctionalInterfaceDemo { 
	public static void main(String[] args) {
		// Use a String-based version of SomeFunc.
		SomeFunc<String> reverse = (str) -> { 
			String result = ""; 
			int i; 
			for(i = str.length()-1; i >= 0; i--) 
				result += str.charAt(i); 
			return result; 
		};
		System.out.println("Lambda reversed is " + 
							reverse.func("Lambda"));
		System.out.println("Expression reversed is " +
							reverse.func("Expression"));
							
		// Now, use an Integer-based version of SomeFunc.
		SomeFunc<Integer> factorial = (n) -> { 
			int result = 1; 
			for(int i=1; i <= n; i++) 
				result = i * result; 
			return result; 
		};
		System.out.println("The factoral of 3 is " + factorial.func(3));
		System.out.println("The factoral of 5 is " + factorial.func(5));
	}
}
```
### Passing Lambda Expressions as Arguments
- To pass a lambda expression as an argument, the type of the parameter receiving the lambda expression argument must be of a **functional interface** type compatible with the lambda.
```Java
// Use lambda expressions as an argument to a method.
interface StringFunc { 
	String func(String n); 
}

class LambdasAsArgumentsDemo {
	// This method has a functional interface as the type of 
	// its first parameter. Thus, it can be passed a reference to 
	// any instance of that interface, including the instance created 
	// by a lambda expression.
	// The second parameter specifies the string to operate on. 
	static String stringOp(StringFunc sf, String s) {
		return sf.func(s); 
	}
	public static void main(String[] args) { 
		String inStr = "Lambdas add power to Java"; 
		String outStr;
		System.out.println("Here is input string: " + inStr);
		// Here, a simple expression lambda that uppercases a string
		// is passed to stringOp( ).
		outStr = stringOp((str) -> str.toUpperCase(), inStr);
		System.out.println("The string in uppercase: " + outStr);

		// This passes a block lambda that removes spaces.
		outStr = stringOp((str) -> {
						String result = ""; 
						int i; 
						for(i = 0; i < str.length(); i++) 
							if(str.charAt(i) != ' ') 
								result += str.charAt(i); 
						return result;
						}, inStr);
		System.out.println("The string with spaces removed: " + outStr);
		
		// Of course, it is also possible to pass a StringFunc instance
		// created by an earlier lambda expression. For example,
		// after this declaration executes, reverse refers to an
		// instance of StringFunc.
		StringFunc reverse = (str) -> { 
			String result = ""; 
			int i; 
			for(i = str.length()-1; i >= 0; i--) 
				result += str.charAt(i); 
			return result; 
		};
		
		// Now, reverse can be passed as 
		// the first parameter to stringOp() 
		// since it refers to a StringFunc object.
		
		System.out.println("The string reversed: " + 
						   stringOp(reverse, inStr));
```
### Lambda Expressions and Exceptions
- If lambda expression throws a [[0x01_Exception Handling#^44d5fe|checked exception]], then that exception must be compatible with the exception(s) listed in the `throws` clause of the abstract method in the functional interface.
```Java
// Throw an exception from a lambda expression.
interface DoubleNumericArrayFunc { 
	double func(double[] n) throws EmptyArrayException; 
}

class EmptyArrayException extends Exception {
	EmptyArrayException() { 
		super("Array Empty"); 
	}
}

class LambdaExceptionDemo {
	public static void main(String[] args) throws EmptyArrayException {
		double[] values = { 1.0, 2.0, 3.0, 4.0 };
		// This block lambda computes the average of an array of doubles.
		DoubleNumericArrayFunc average = (n) -> { 
			double sum = 0;
			if(n.length == 0)
				throw new EmptyArrayException();
			for(int i = 0; i < n.length; i++) 
				sum += n[i];
			return sum / n.length;
		};
		System.out.println("The average is " + average.func(values));
		// This causes an exception to be thrown.
		System.out.println("The average is " + 
							average.func(new double[0]));
	}
}
```
- Notice that the parameter specified by `func()` in the functional interface `DoubleNumericArrayFunc` is an array. However, the parameter to the lambda expression is simply `n`, rather than `n[]`. Remember, the type of a lambda expression parameter **will be inferred** from the target context.
- It is **not** necessary (or legal) to specify it as `n[]`. It would be legal to explicitly declare it as `double[] n`, but doing so gains nothing in this case.
### Lambda Expressions and Variable Capture
- Variables defined by the enclosing scope of a lambda expression are accessible within the lambda expression. 
- For example, a lambda expression can use an instance or **static** variable defined by its enclosing class. 
- A lambda expression also has access to `this`, which refers to the invoking instance of the lambda expression’s enclosing class.
- However, when a lambda expression uses a local variable from its enclosing scope, a special situation is created that is referred to as a _variable capture_.
- In this case, a lambda expression may only use local variables that are _effectively final_. An effectively final variable is one whose value does **not change** after it is first assigned.
- The `this` parameter of an enclosing scope is automatically **effectively final**, and lambda expressions **do not have** a `this` of their own.
- It is important to understand that a local variable of the enclosing scope **cannot be modified** by the lambda expression. 
- Doing so would remove its effectively final status, thus rendering it illegal for capture.
- The following program illustrates the difference between effectively final and mutable local variables:
```Java
// An example of capturing a local variable from the enclosing scope.
interface MyFunc { 
	int func(int n); 
}

class VarCapture { 
	public static void main(String[] args) {
		// A local variable that can be captured. 
		int num = 10;
		MyFunc myLambda = (n) -> {
			// This use of num is OK. It does not modify num. 
			int v = num + n;
			
			// However, the following is illegal because it attempts 
			// to modify the value of num.
			// num++;
			return v;
		}
		// The following line would also cause an error, because 
		// it would remove the effectively final status from num.
		// num = 9;
	}
}
```
- As the comments indicate, `num` is effectively final and can, therefore, be used inside `myLambda`. However, if `num` were to be modified, either inside the lambda or outside of it, `num` would lose its effectively final status. 
- This would cause an error, and the program would not compile.
- It is important to emphasize that a lambda expression can **use and modify** an instance variable from its **invoking class**. 
- It just can’t use a local variable of its enclosing scope unless **that variable is effectively final**.
### Method References
- A _method reference_ provides a way to refer to a method without executing it. It relates to lambda expressions because it, too, requires a target type context that consists of a compatible functional interface.
- There are different types of method references:
	- **Method References to static Methods**
	- **Method References to Instance Methods**
	- **Method References with Generics**
##### Method References to static Methods
- To create a static method reference, use this general syntax: `ClassName::methodName`.
- The following program demonstrates a **static** method reference:
```Java
// Demonstrate a method reference for a static method.
// A functional interface for string operations.
interface StringFunc { 
	String func(String n); 
}

// This class defines a static method called strReverse(). 
class MyStringOps {
	// A static method that reverses a string. 
	static String strReverse(String str) {
		String result = ""; 
		int i; 
		for(i = str.length()-1; i >= 0; i--) 
			result += str.charAt(i); 
		return result;
	}
}

class MethodRefDemo {
	// This method has a functional interface as the type of 
	// its first parameter. Thus, it can be passed any instance 
	// of that interface, including a method reference.

	static String stringOp(StringFunc sf, String s) { 
		return sf.func(s);
	}
	
	public static void main(String[] args) { 
		String inStr = "Lambdas add power to Java"; 
		String outStr;
		// Here, a method reference to strReverse 
		// is passed to stringOp(). 
		outStr = stringOp(MyStringOps::strReverse, inStr);
		
		System.out.println("Original string: " + inStr); 
		System.out.println("String reversed: " + outStr); 
	} 
}	
```
##### Method References to Instance Methods
- To pass a reference to an instance method on a specific object, use this basic syntax: `objRef::methodName`
- Here is the previous program rewritten to use an instance method reference:
```Java
// Demonstrate a method reference to an instance method
// A functional interface for string operations.
interface StringFunc { 
	String func(String n); 
}

// Now, this class defines an instance method called strReverse().
class MyStringOps { 
	String strReverse(String str) { 
		String result = ""; 
		int i; 
		for(i = str.length()-1; i >= 0; i--) 
			result += str.charAt(i); 
		return result; 
	}
}

class MethodRefDemo2 {
	// This method has a functional interface as the type of 
	// its first parameter. Thus, it can be passed any instance 
	// of that interface, including method references.
	static String stringOp(StringFunc sf, String s) { 
		return sf.func(s);
	}
	
	public static void main(String[] args) { 
		String inStr = "Lambdas add power to Java"; 
		String outStr;
		
		// Create a MyStringOps object. 
		MyStringOps strOps = new MyStringOps( );
		
		// Now, a method reference to the instance method strReverse 
		// is passed to stringOp().
		outStr = stringOp(strOps::strReverse, inStr);
		
		System.out.println("Original string: " + inStr);
		System.out.println("String reversed: " + outStr);
	}
}
```
- It is also possible to handle a situation in which you want to specify an instance method that can be used with any object of a given class—not just a specified object. 
- In this case, you will create a method reference as shown here: `ClassName::instanceMethodName`.
- One other point: you can refer to the superclass version of a method by use of super, as shown here: `super::name`.
- The name of the method is specified by name. Another form is `typeName.super::name` where `typeName` refers to an enclosing class or super interface.
##### Method References with Generics
- Consider the following program:
```Java
// Demonstrate a method reference to a generic method
// declared inside a non-generic class.

// A functional interface that operates on an array
// and a value, and returns an int result.
interface MyFunc<T> {
    int func(T[] vals, T v);
}

// This class defines a method called countMatching() that
// returns the number of items in an array that are equal
// to a specified value. Notice that countMatching()
// is generic, but MyArrayOps is not.
class MyArrayOps {
    static <T> int countMatching(T[] vals, T v) {
        int count = 0;
        for (int i = 0; i < vals.length; i++)
            if (vals[i] == v) count++;
        return count;
    }
}

class GenericMethodRefDemo {
    // This method has the MyFunc functional interface as the
    // type of its first parameter. The other two parameters
    // receive an array and a value, both of type T.
    static <T> int myOp(MyFunc<T> f, T[] vals, T v) {
        return f.func(vals, v);
    }

    public static void main(String[] args) {
        Integer[] vals = {1, 2, 3, 4, 2, 3, 4, 4, 5};
        String[] strs = {"One", "Two", "Three", "Two"};

        int count;
        count = myOp(MyArrayOps::<Integer>countMatching, vals, 4);
        System.out.println("vals contains " + count + " 4s");

        count = myOp(MyArrayOps::<String>countMatching, strs, "Two");
        System.out.println("strs contains " + count + " Twos");
    }
}
```
- It is important to point out, however, that explicitly specifying the type argument is not required in this situation (and many others) because the type argument would have been automatically inferred. 
- In cases in which a generic class is specified, the type argument follows the class name and precedes the `::`.
### Constructor References
- Similar to the way that you can create references to methods, you can create references to constructors. Here is the general form of the syntax that you will use: `classname::new`.
- This reference can be assigned to any functional interface reference that defines a method compatible with the constructor. Here is a simple example:
```Java
// Demonstrate a Constructor reference.
// MyFunc is a functional interface whose method returns
// a MyClass reference.
interface MyFunc {
    MyClass func(int n);
}

class MyClass {
    private int val;

    // This constructor takes an argument.
    MyClass(int v) {
        val = v;
    }

    // This is the default constructor.
    MyClass() {
        val = 0;
    }

    // ...
    int getVal() {
        return val;
    }
}

class ConstructorRefDemo {
    public static void main(String[] args) {
        // Create a reference to the MyClass constructor.
        // Because func() in MyFunc takes an argument, new
        // refers to the parameterized constructor in MyClass,
        // not the default constructor.
        MyFunc myClassCons = MyClass::new;

        // Create an instance of MyClass via that constructor reference.
        MyClass mc = myClassCons.func(100);

        // Use the instance of MyClass just created.
        System.out.println("val in mc is " + mc.getVal());
    }
}
```
- `MyFunc myClassCons = MyClass::new;` Here, the expression `MyClass::new` creates a constructor reference to a `MyClass` constructor.
- The following illustrates this by modifying the previous example so that MyFunc and MyClass are generic.
```Java
// Demonstrate a constructor reference with a generic class.
// MyFunc is now a generic functional interface.
interface MyFunc<T> {
    MyClass<T> func(T n);
}

class MyClass<T> {
    private T val;

    // A constructor that takes an argument.
    MyClass(T v) { 
        val = v; 
    }

    // This is the default constructor.
    MyClass() { 
        val = null; 
    }

    T getVal() { 
        return val; 
    }
}

class ConstructorRefDemo2 {
    public static void main(String[] args) {
        // Create a reference to the MyClass<T> constructor.
        MyFunc<Integer> myClassCons = MyClass<Integer>::new;

        // Create an instance of MyClass<T> via 
        // that constructor reference.
        MyClass<Integer> mc = myClassCons.func(100);

        // Use the instance of MyClass<T> just created.
        System.out.println("val in mc is " + mc.getVal());
    }
}

```
### Predefined Functional Interfaces
- In many cases, you won’t need to define your own functional interface because the package called `java.util.function` provides several predefined ones. Here is a sampling: ![[Predefined Functional Interfaces.PNG]]
```Java
// Use the Function built-in functional interface. 
// Import the Function interface.
import java.util.function.Function;

class UseFunctionInterfaceDemo { 
	public static void main(String[] args) {
		// This block lambda computes the factorial of an int value. 
		// This time, Function is the functional interface.
		Function<Integer, Integer> factorial = (n) -> { 
			int result = 1; 
			for(int i=1; i <= n; i++) 
				result = i * result; 
			return result; 
		}; 
		
		System.out.println("The factoral of 3 is " + factorial.apply(3));
		System.out.println("The factoral of 5 is " + factorial.apply(5)); 
	} 
}
```
# Sources
- Java The Complete Reference - Chapter 15.