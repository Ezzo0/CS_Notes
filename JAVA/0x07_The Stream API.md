# Explanation
- The key aspect of the stream API is its ability to perform very sophisticated operations that search, filter, map, or otherwise manipulate data. 
- For example, using the stream API, you can construct sequences of actions that resemble, in concept, the type of database queries for which you might use [[0x00_Querying#SQL|SQL]].
- Furthermore, in many cases, such actions can be performed in _parallel_, thus providing a high level of efficiency, especially when large data sets are involved.
- As a general rule, however, a stream operation by itself **does not modify** the data source.
### Stream Interfaces
- The stream API defines several stream interfaces, which are packaged in `java.util.stream` and contained in the `java.base` module.
- At the foundation is `BaseStream`, which defines the basic functionality available in all streams. `BaseStream` is a generic interface declared like this: 
	 `interface BaseStream<T, S extends BaseStream<T, S>>`
- Here, `T` specifies the type of the elements in the stream, and `S` specifies the type of stream that extends `BaseStream`.
- `BaseStream` extends the `AutoCloseable` interface; thus, a stream can be managed in a **try**-with-resources statement. 
- In general, however, only those streams whose data source requires closing (such as those connected to a file) will need to be closed. 
- In most cases, such as those in which the data source is a **collection**, there is no need to close the stream.
- The methods declared by **BaseStream** are:
	 ![[BaseStreamMethods.PNG|750]]
- From `BaseStream` are derived several types of stream interfaces. The most general of these is `Stream`. It is declared as shown here: `interface Stream<T>`.
- In addition to the methods that it inherits from `BaseStream`, the `Stream` interface adds several of its own, a sampling of which is shown:
	 ![[StreamMethods1.PNG|750]]
	 ![[StreamMethods2.PNG]]
- In both tables, notice that many of the methods are notated as being either _terminal_ or _intermediate_.
- A _terminal_ operation **consumes** the stream. It is used to produce a result, such as finding the minimum value in the stream, or to execute some action, as is the case with the `forEach()` method. Once a stream has been consumed, it cannot be reused.
- _Intermediate_ operations **produce** another stream. Thus, intermediate operations can be used to create a _pipeline_ that performs a sequence of actions.
- One other point: intermediate operations **do not take place immediately**. Instead, the specified action is performed **when a terminal operation is executed** on the new stream created by an intermediate operation.
- This mechanism is referred to as _lazy_ behavior, and the intermediate operations are referred to as _lazy_. The use of lazy behavior enables the stream API to perform more efficiently.
- Another key aspect of streams is that some intermediate operations are _stateless_ and some are _stateful_.
- In a stateless operation, each element is processed **independently** of the others. 
- In a stateful operation, the processing of an element **may depend** on aspects of the other elements.
- For example, sorting is a stateful operation. However, filtering elements based on a stateless predicate is stateless because each element is handled individually.
### How to Obtain a Stream
- The most common is when a stream is obtained for a collection. 
- Beginning with JDK 8, the **Collection** interface was expanded to include two methods that obtain a stream from a collection. The first is `stream()`, shown here: `default Stream<E> stream( )`. Its default implementation returns a sequential stream
- The second method is `parallelStream()`, shown next: 
	 `default Stream<E> parallelStream( )`
- Its default implementation returns a parallel stream, if possible.(If a parallel stream can not be obtained, a sequential stream may be returned instead.)
- A stream can also be obtained from an array by use of the static `stream()` method, which was added to the **Arrays** class. One of its forms is shown here:
	`static <T> Stream<T> stream(T[ ] array)`
##### A Simple Stream Example
```Java
// Demonstrate several stream operations.
import java.util.*;
import java.util.stream.*;

class StreamDemo {
    public static void main(String[] args) {
        // Create a list of Integer values.
        ArrayList<Integer> myList = new ArrayList<>();
        myList.add(7);
        myList.add(18);
        myList.add(10);
        myList.add(24);
        myList.add(17);
        myList.add(5);

        System.out.println("Original list: " + myList);

        // Obtain a Stream to the array list.
        Stream<Integer> myStream = myList.stream();

        // Obtain the minimum and maximum value 
        // by use of min(), max(), isPresent(), and get().
        Optional<Integer> minVal = myStream.min(Integer::compare);
        if (minVal.isPresent()) 
            System.out.println("Minimum value: " + minVal.get());

        // Must obtain a new stream because previous call 
        // to min() is a terminal operation that consumed the stream.
        myStream = myList.stream();
        Optional<Integer> maxVal = myStream.max(Integer::compare);
        if (maxVal.isPresent()) 
            System.out.println("Maximum value: " + maxVal.get());

        // Sort the stream by use of sorted().
        Stream<Integer> sortedStream = myList.stream().sorted();

        // Display the sorted stream by use of forEach().
        System.out.print("Sorted stream: ");
        sortedStream.forEach((n) -> System.out.print(n + " "));
        System.out.println();

        // Display only the odd values by use of filter().
        Stream<Integer> oddVals = myList.stream()
                                        .sorted()
                                        .filter((n) -> (n % 2) == 1);
        System.out.print("Odd values: ");
        oddVals.forEach((n) -> System.out.print(n + " "));
        System.out.println();

        // Display only the odd values that are greater than 5. 
        // Notice that two filter operations are pipelined.
        oddVals = myList.stream()
                        .filter((n) -> (n % 2) == 1)
                        .filter((n) -> n > 5);
        System.out.print("Odd values greater than 5: ");
        oddVals.forEach((n) -> System.out.print(n + " "));
        System.out.println();
    }
}
```
- Recall that `min()` is declared like this: 
	 `Optional<T> min(Comparator<? super T> comp)`
- First, notice that the type of `min()`’s parameter is a **Comparator**. This comparator is used to compare two elements in the stream.
- In the example, `min()` is passed a [[0x05_Lambda Expressions#Method References|method reference]] to **Integer**’s `compare()` method, which is used to implement a **Comparator** capable of comparing two Integers.
- Next, notice that the return type of `min()` is `Optional`. The `Optional` class is a generic class packaged in `java.util` and declared like this: `class Optional<T>`.
- An `Optional` instance can either contain a value of type `T` or be empty. You can use `isPresent()` to determine if a value is present.
- Assuming that a value is available, it can be obtained by calling `get()`, or if you are using JDK 10 or later, `orElseThrow()`.
- One other point about the preceding line: `min()` is a terminal operation that consumes the stream. Thus, `myStream` cannot be used again after `min()` executes.
### Reduction Operations
- Consider the `min()` and `max()` methods in the preceding example program. Both are terminal operations that return a result based on the elements in the stream. 
- In the language of the stream API, they represent _reduction operations_ because each reduces a stream to a single value. The stream API refers to these as _special case_ reductions because they perform a specific function.
- In addition to `min()` and `max()`, other special case reductions are also available, such as `count()`, which counts the number of elements in a stream. 
- However, the stream API generalizes this concept by providing the `reduce()` method. By using `reduce()`, you can return a value from a stream based on any arbitrary criteria. By definition, all reduction operations are terminal operations.
- **Stream** defines three versions of `reduce()`. The two we will use first are shown here
```Java
Optional<T> reduce(BinaryOperator<T> accumulator) 
T reduce(T identityVal, BinaryOperator<T> accumulator)
```
- The first form returns an object of type `Optional`, which contains the result. The second form returns an object of type `T` (which is the element type of the stream).
- In both forms, `accumulator` is a function that operates on two values and produces a result.
- In the second form, `identityVal` is a value such that an accumulator operation involving `identityVal` and any element of the stream yields that element, unchanged. 
- For example, if the operation is addition, then the identity value will be 0 because `0+x` is `x`. For multiplication, the value will be 1, because `1 * x` is `x`.
- `BinaryOperator` is a functional interface declared in `java.util.function` that extends the `BiFunction` functional interface. `BiFunction` defines this abstract method: `R apply(T val, U val2)`
- Here, `R` specifies the result type, `T` is the type of the first operand, and `U` is the type of second operand. Thus, `apply()` applies a function to its two operands and returns the result.
- When `BinaryOperator` extends `BiFunction`, it specifies the same type for all the type parameters. Thus, as it relates to `BinaryOperator`, `apply( )` looks like this: `T apply(T val, T val2)`
- Furthermore, as it relates to `reduce()`, `val` will contain the previous result and `val2` will contain the next element. 
- In its first invocation, `val` will contain either the `identityVal` or the first element, depending on which version of `reduce()` is used.
- It is important to understand that the accumulator operation must satisfy three constraints. It must be
	- **Stateless** 
	- **Non-interfering** -- means that the data source is not modified by the operation.
	- **Associative** -- means that it does not matter which pair of operands are processed first.
- The following program demonstrates the versions of `reduce()` just described:
```Java
// Demonstrate the reduce() method.
import java.util.*;
import java.util.stream.*;

class StreamDemo2 {
    public static void main(String[] args) {
        // Create a list of Integer values.
        ArrayList<Integer> myList = new ArrayList<>();
        myList.add(7);
        myList.add(18);
        myList.add(10);
        myList.add(24);
        myList.add(17);
        myList.add(5);

        // Two ways to obtain the integer product of the elements
        // in myList by use of reduce().
        Optional<Integer> productObj = myList.stream().
									        reduce((a, b) -> a * b);

        if (productObj.isPresent())
            System.out.println("Product as Optional: " +
					           productObj.get());

        int product = myList.stream().reduce(1, (a, b) -> a * b);
        System.out.println("Product as int: " + product);
    }
}
```
### Using Parallel Streams
- One way to obtain a parallel stream is to use the `parallelStream()` method defined by Collection. 
- Another way to obtain a parallel stream is to call the `parallel()` method on a sequential stream. The `parallel()` method is defined by `BaseStream`, as shown here: `S parallel()`. If it is called on a stream that is already parallel, then the invoking stream is returned.
- As a general rule, any operation applied to a parallel stream must be **stateless**. It should also be **non-interfering** and **associative**.
- When using parallel streams, you might find the following version of `reduce()` especially helpful. It gives you a way to specify how partial results are combined: 
```Java
<U> U reduce(U identityVal, BiFunction<U, ? super T, U> accumulator, 
			BinaryOperator<U> combiner)
```
- In this version, `combiner` defines the function that combines two values that have been produced by the `accumulator` function.
- Assuming the preceding program, the following statement computes the product of the elements in `myList` by use of a parallel stream:
```Java
int parallelProduct = myList.parallelStream().reduce(1, (a,b) -> a*b, 
														(a,b) -> a*b);
```
- As you can see, in this example, both the accumulator and combiner perform the same function. 
- However, there are cases in which the actions of the accumulator must differ from those of the combiner. For example, consider the following program:
```Java
// Demonstrate the use of a combiner with reduce()
import java.util.*;
import java.util.stream.*;

class StreamDemo3 {
    public static void main(String[] args) {
        // This is now a list of double values.
        ArrayList<Double> myList = new ArrayList<>();
        myList.add(7.0);
        myList.add(18.0);
        myList.add(10.0);
        myList.add(24.0);
        myList.add(17.0);
        myList.add(5.0);

        double productOfSqrRoots = myList.parallelStream().reduce(
            1.0,
            (a, b) -> a * Math.sqrt(b),
            (a, b) -> a * b
        );

        System.out.println("Product of square roots: " +
					       productOfSqrRoots);
    }
}
```
- If the data source is ordered, then the stream will also be ordered. However, when using a parallel stream, a performance boost can sometimes be obtained by allowing a stream to be unordered. 
- When a parallel stream is unordered, each partition of the stream can be operated on independently, without having to coordinate with the others. 
- In cases in which the order of the operations does not matter, it is possible to specify unordered behavior by calling the `unordered()` method, shown here: 
	 `S unordered( )`
- One other point: the `forEach()` method may not preserve the ordering of a parallel stream. If you want to perform an operation on each element in a parallel stream while preserving the order, consider using `forEachOrdered()`.
### Mapping
- Often it is useful to map the elements of one stream to another. For example, a stream that contains a database of name, telephone, and e-mail address information might map only the name and e-mail address portions to another stream.
- The most general mapping method is `map()`. It is shown here:
	 `<R> Stream<R> map(Function<? super T, ? extends R> mapFunc)`.
- Here, `R` specifies the type of elements of the new stream; `T` is the type of elements of the invoking stream; and `mapFunc` is an instance of **Function**, which does the mapping.
- The map function **must** be _stateless_ and _non-interfering_. Since a new stream is returned, `map()` is an **intermediate** method.
- `Function` is a functional interface declared in `java.util.function`. It is declared as shown here: `Function<T, R>`.
- As it relates to `map()`, `T` is the element type and `R` is the result of the mapping. `Function` has the abstract method shown here: `R apply(T val)`.
- Here, `val` is a reference to the object being mapped. The mapped result is returned.
- The following is a simple example of `map()`.
```Java
// Demonstrate the use of a combiner with reduce()
import java.util.*;
import java.util.stream.*;

class StreamDemo3 {
    public static void main(String[] args) {
        // This is now a list of double values.
        ArrayList<Double> myList = new ArrayList<>();
        myList.add(7.0);
        myList.add(18.0);
        myList.add(10.0);
        myList.add(24.0);
        myList.add(17.0);
        myList.add(5.0);

        // Map the square root of the elements in myList to a new stream. 
        Stream<Double> sqrtRootStrm = myList.stream().
												map((a) -> Math.sqrt(a));

		// Find the product of the square roots.

		double productOfSqrRoots = sqrtRootStrm.
											reduce(1.0, (a,b) -> a*b); 
		System.out.println("Product of square roots is " + 
							productOfSqrRoots);
    }
}
```
- In addition to the version just described, three other versions of `map()` are provided. They return a primitive stream, as shown here:
```Java
IntStream mapToInt(ToIntFunction<? super T> mapFunc) 
LongStream mapToLong(ToLongFunction<? super T> mapFunc) 
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapFunc)
```
- Each `mapFunc` must implement the abstract method defined by the specified interface, returning a value of the indicated type. For example, `ToDoubleFunction` specifies the `applyAsDouble(T val)` method, which must return the value of its parameter as a `double`.
- Here is an example that uses a primitive stream:
```Java
// Map a Stream to an IntStream.
import java.util.*;
import java.util.stream.*;

class StreamDemo6 {
    public static void main(String[] args) {
        // A list of double values.
        ArrayList<Double> myList = new ArrayList<>();
        myList.add(1.1);
        myList.add(3.6);
        myList.add(9.2);
        myList.add(4.7);
        myList.add(12.1);
        myList.add(5.0);

        System.out.print("Original values in myList: ");
        myList.stream().forEach((a) -> {
            System.out.print(a + " ");
        });
        System.out.println();

        // Map the ceiling of the elements in myList to an IntStream.
        IntStream cStrm = myList.stream().
							        mapToInt((a) -> (int) Math.ceil(a));

        System.out.print("The ceilings of the values in myList: ");
        cStrm.forEach((a) -> {
            System.out.print(a + " ");
        });
    }
}
```
- Before leaving the topic of mapping, it is necessary to point out that the stream API also provides methods that support _flat maps_. 
- These are `flatMap()`, `flatMapToInt()`, `flatMapToLong()`, and `flatMapToDouble( )`. 
- The flat map methods are designed to handle situations in which each element in the original stream is mapped to more than one element in the resulting stream.
### Collecting
- Sometimes it is desirable to obtain a collection from a stream. To perform such an action, you will generally use the `collect()` method.
- It has two forms. The one we will use first is shown here:
	`<R, A> R collect(Collector<? super T, A, R> collectorFunc)`
- Here, `R` specifies the type of the result, and `T` specifies the element type of the invoking stream. The internal accumulated type is specified by `A`. 
- The `collectorFunc` specifies how the collection process works. The `collect()` method is a terminal operation.
- The `Collector` interface is declared in `java.util.stream`, as shown here: `interface Collector<T, A, R>`
- The Collectors class defines a number of static collector methods that you can use as-is. The two we will use are `toList()` and `toSet()`, shown here:
```Java
static <T> Collector<T, ?, List<T>> toList() 
static <T> Collector<T, ?, Set<T>> toSet( )
```
- The following program puts the preceding discussion into action:
```Java
// Use collect() to create a List and a Set from a stream.
import java.util.*;
import java.util.stream.*;

class NamePhoneEmail {
    String name;
    String phonenum;
    String email;

    NamePhoneEmail(String n, String p, String e) {
        name = n;
        phonenum = p;
        email = e;
    }
}

class NamePhone {
    String name;
    String phonenum;

    NamePhone(String n, String p) {
        name = n;
        phonenum = p;
    }
}

class StreamDemo7 {
    public static void main(String[] args) {
        // A list of names, phone numbers, and e-mail addresses.
        ArrayList<NamePhoneEmail> myList = new ArrayList<>();
        myList.add(new NamePhoneEmail("Larry", "555-5555",
                "Larry@HerbSchildt.com"));
        myList.add(new NamePhoneEmail("James", "555-4444",
                "James@HerbSchildt.com"));
        myList.add(new NamePhoneEmail("Mary", "555-3333",
                "Mary@HerbSchildt.com"));

        // Map just the names and phone numbers to a new stream.
        Stream<NamePhone> nameAndPhone = myList.stream().map(
                (a) -> new NamePhone(a.name, a.phonenum)
        );

        // Use collect to create a List of the names and phone numbers.
        List<NamePhone> npList = 
					        nameAndPhone.collect(Collectors.toList());

        System.out.println("Names and phone numbers in a List:");
        for (NamePhone e : npList)
            System.out.println(e.name + ": " + e.phonenum);

        // Obtain another mapping of the names and phone numbers.
        nameAndPhone = myList.stream().map(
                (a) -> new NamePhone(a.name, a.phonenum)
        );

        // Now, create a Set by use of collect().
        Set<NamePhone> npSet = nameAndPhone.collect(Collectors.toSet());

        System.out.println("\nNames and phone numbers in a Set:");
        for (NamePhone e : npSet)
            System.out.println(e.name + ": " + e.phonenum);
    }
}
```
- There is a second version that gives you more control over the collection process. It is shown here:
```Java
<R> R collect(Supplier<R> target, BiConsumer<R, ? super T> accumulator,
			  BiConsumer <R, R> combiner)
```
- Here, `target` specifies how the object that holds the result is created. For example, to use a **LinkedList** as the result collection, you would specify its constructor.
- The `accumulator` function adds an element to the result and `combiner` combines two partial results. Thus, these functions work similarly to the way they do in `reduce()`.
- For both, they must be **stateless** and **non-interfering**. They must also be **associative**..
- Note that the `target` parameter is of type `Supplier`. It is a functional interface declared in `java.util.function`. It specifies only the `get()` method, which has no parameters and, in this case, returns an object of type `R`.
- Thus, as it relates to `collect()`, `get()` returns a reference to a mutable storage object, such as a `collection`.
- Note also that the types of `accumulator` and `combiner` are `BiConsumer`. 
- This is a functional interface defined in `java.util.function`. It specifies the abstract method `accept()` that is shown here: `void accept(T obj, U obj2)`.
- This method performs some type of operation on `obj` and `obj2`. As it relates to `accumulator`, `obj` specifies the target collection, and `obj2` specifies the element to add to that collection. 
- As it relates to combiner, `obj` and `obj2` specify two collections that will be combined.
- Using the version of `collect()` just described, you could use a **LinkedList** as the target in the preceding program, as shown here:
```Java
LinkedList<NamePhone> npList = nameAndPhone.collect( 
								() -> new LinkedList<>(), 
								(list, element) -> list.add(element), 
								(listA,listB) -> listA.addAll(listB));
```
- As you may have guessed, it is not always necessary to specify a lambda expression for the arguments to `collect()`. 
- Often, method and/or constructor references will suffice. For example, again assuming the preceding program, this statement creates a `HashSet` that contains all of the elements in the `nameAndPhone` stream:
```Java
HashSet<NamePhone> npSet = nameAndPhone.collect(HashSet::new, 
												HashSet::add, 
												HashSet::addAll);
```
- Notice that the first argument specifies the `HashSet` [[0x05_Lambda Expressions#Constructor References|constructor reference]]. The second and third specify [[0x05_Lambda Expressions#Method References|method references]] to `HashSet`’s `add()` and `addAll()` methods.
### Iterators and Streams
- The stream API supports two types of iterators. The first is the traditional **Iterator**. The second is **Spliterator**.
##### Use an Iterator with a Stream
- Iterators are objects that implement the `Iterator` interface declared in `java.util`
- Its two key methods are `hasNext()` and `next()`. If there is another element to iterate, `hasNext()` returns true, and false otherwise. 
- The `next()` method returns the next element in the iteration.
- To obtain an iterator to a stream, call `iterator()` on the stream. The version used by Stream is shown here. `Iterator<T> iterator( )`
- The following program shows how to iterate through the elements of a stream:
```Java
// Use an iterator with a stream.
import java.util.*;
import java.util.stream.*;

class StreamDemo8 {
    public static void main(String[] args) {
        // Create a list of Strings.
        ArrayList<String> myList = new ArrayList<>();
        myList.add("Alpha");
        myList.add("Beta");
        myList.add("Gamma");
        myList.add("Delta");
        myList.add("Phi");
        myList.add("Omega");

        // Obtain a Stream to the array list.
        Stream<String> myStream = myList.stream();

        // Obtain an iterator to the stream.
        Iterator<String> itr = myStream.iterator();

        // Iterate the elements in the stream.
        while (itr.hasNext())
            System.out.println(itr.next());
    }
}
```
##### Use Spliterator
- **Spliterator** offers an alternative to **Iterator**, especially when parallel processing is involved. 
- **Spliterator** defines several methods, but we only need to use three. 
- The first is `tryAdvance()`. It performs an action on the next element and then advances the iterator. It is shown here: 
	 `boolean tryAdvance(Consumer<? super T> action)`
- Here, `action` specifies the action that is executed on the next element in the iteration. `tryAdvance()` returns true if there is a next element. It returns false if no elements remain.
- `Consumer` declares one method called `accept()` that receives an element of type T as an argument and returns `void`.
- The following version of the preceding program substitutes a `Spliterator` for the `Iterator`:
```java
// Use a Spliterator.
import java.util.*;
import java.util.stream.*;

class StreamDemo9 {
    public static void main(String[] args) {
        // Create a list of Strings.
        ArrayList<String> myList = new ArrayList<>();
        myList.add("Alpha");
        myList.add("Beta");
        myList.add("Gamma");
        myList.add("Delta");
        myList.add("Phi");
        myList.add("Omega");

        // Obtain a Stream to the array list.
        Stream<String> myStream = myList.stream();

        // Obtain a Spliterator.
        Spliterator<String> splitItr = myStream.spliterator();

        // Iterate the elements of the stream.
        while (splitItr.tryAdvance((n) -> System.out.println(n)));
    }
}
```
- In some cases, you might want to perform some action on each element **collectively**, rather than one at a time. 
- To handle this type of situation, `Spliterator` provides the `forEachRemaining()` method, shown here: 
	 `default void forEachRemaining(Consumer<? super T> action)`
- This method applies `action` to each unprocessed element and then returns. For example: `splitItr.forEachRemaining((n) -> System.out.println(n));`
- One other `Spliterator` method of particular interest is `trySplit()`. It splits the elements being iterated in two, returning a new `Spliterator` to one of the partitions. 
- The other partition remains accessible by the original `Spliterator`. It is shown here: `Spliterator<T> trySplit()`
- If it is not possible to split the invoking `Spliterator`, `null` is returned. Otherwise, a reference to the partition is returned. For example:
```Java
// Demonstrate trySplit().
import java.util.*;
import java.util.stream.*;

class StreamDemo10 {
    public static void main(String[] args) {
        // Create a list of Strings.
        ArrayList<String> myList = new ArrayList<>();
        myList.add("Alpha");
        myList.add("Beta");
        myList.add("Gamma");
        myList.add("Delta");
        myList.add("Phi");
        myList.add("Omega");

        // Obtain a Stream to the array list.
        Stream<String> myStream = myList.stream();

        // Obtain a Spliterator.
        Spliterator<String> splitItr = myStream.spliterator();

        // Now, split the first iterator.
        Spliterator<String> splitItr2 = splitItr.trySplit();

        // If splitItr could be split, use splitItr2 first.
        if (splitItr2 != null) {
            System.out.println("Output from splitItr2: ");
            splitItr2.forEachRemaining((n) -> System.out.println(n));
        }

        // Now, use the splitItr.
        System.out.println("\nOutput from splitItr: ");
        splitItr.forEachRemaining((n) -> System.out.println(n));
    }
}
```
# Sources
- Java The Complete Reference - Chapter 30.