# Explanation

+ A bag is a collection where removing items is not supported.
+ Its purpose is to provide the ability to collect items and then to iterate through the collected items.
+ In Bags the order in which items are processed is immaterial.
![[Bag.png| 200]]
# Implementation
```java
import java.util.Iterator; 
public class Bag<Item> implements Iterable<Item> 
{
	private Node first; // first node in list
	
	private class Node 
	{ 
		Item item; 
		Node next; 
	} 
	
	public void add(Item item)

	{   // same as push() in Stack
		Node oldfirst = first; 
		first = new Node(); 
		first.item = item; 
		first.next = oldfirst; 
	} 
	
	public Iterator<Item> iterator() 
	{ return new ListIterator();   }
	
	private class ListIterator implements Iterator<Item>
	{ 
		private Node current = first; 
		public boolean hasNext() 
		{ return current != null; } 
		
		public void remove() { } 
		
		public Item next() 
		{ 
			Item item = current.item; 
			current = current.next; 
			return item; 
		} 
	} 
}
```
# Sources
+ [Princeton University: Algorithms part | - Lecture 3 ](https://www.coursera.org/learn/algorithms-part1/lecture/zGZE0/iterators)
+ Algorithms 1.3