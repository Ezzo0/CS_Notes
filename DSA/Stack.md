# Explanation

+ A stack is a collection that is based on the last-in-first-out (LIFO) policy.
![[Operations on a pushdown stack.png|300]]

# Implementation
+ Array Based implementation: 
```java
public class ResizingArrayStack<Item> implements Iterable<Item> 
{
	private Item[] a = (Item[]) new Object[1];  // stack items 
	private int N = 0;                         // number of items
	
	public boolean isEmpty() { return N == 0; } 
	public int size()        { return N;      }
	
	private void resize(int max)
	{   // Move stack to a new array of size max.|
		Item[] temp = (Item[]) new Object[max]; 
		for (int i = 0; i < N; i++) temp[i] = a[i]; a = temp; 
	} 
	
	public void push(Item item)
	{   // Add item to top of stack.
		if (N == a.length) resize(2*a.length); a[N++] = item; 
	} 
	
	public Item pop()
	{   // Remove item from top of stack.
		Item item = a[--N]; a[N] = null;   // Avoid loitering (see text). 
		if (N > 0 && N == a.length/4) resize(a.length/2); 
		return item; 
	} 
	
	public Iterator<Item> iterator() 
	{ return new ReverseArrayIterator(); }

	private class ReverseArrayIterator implements Iterator<Item>
	{   // Support LIFO iteration.
		private int i = N; 
		public boolean hasNext() { return i > 0;  } 
		public Item next()       { return a[--i]; } 
		public void remove()     {                } 
	} 
}
```
+ Linked-list implementation:
```java
public class Stack<Item> implements Iterable<Item> 
{
	private Node first;  // top of stack (most recently added node)
	private int N;       // number of items
	private class Node
	{   // nested class to define nodes
        Item item; 
        Node next; 
    }

	public boolean isEmpty() { return first == null; }    // Or: N == 0.
	public int size()        { return N;             } 
	public void push(Item item)
	{   // Add item to top of stack.
		Node oldfirst = first; 
		first = new Node(); 
		first.item = item; 
		first.next = oldfirst; 
		N++; 
	} 
	
	public Item pop()
	{   // Remove item from top of stack.
		Item item = first.item; 
		first = first.next; 
		N--; 
		return item; 
	}
}
```
# Sources
+ [Princeton University: Algorithms part | - Lecture 3 ](https://www.coursera.org/learn/algorithms-part1/lecture/jSxyD/stacks)
+ Algorithms 1.3