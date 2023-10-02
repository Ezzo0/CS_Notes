# Explanation

+ A queue is a collection that is based on the first-in-first-out (FIFO) policy.
# Implementation
+ Linked-list implementation:
```java
public class Queue<Item> implements Iterable<Item> 
{
	private Node first;  // link to least recently added node
	private Node last;   // link to most recently added node
	private int N;       // number of items on the queue
	private class Node
	{   // nested class to define nodes
        Item item; 
        Node next; 
    }

	public boolean isEmpty() { return first == null; }    // Or: N == 0.
	public int size()        { return N;             } 
	public void enqueue(Item item)
	{   // Add item to the end of the list.
		Node oldlast = last; 
		last = new Node(); 
		last.item = item; 
		last.next = null;
		if (isEmpty()) first = last; 
		else           oldlast.next = last; 
		N++; 
	} 
	
	public Item dequeue()
	{   // Remove item from the beginning of the list.
		Item item = first.item; 
		first = first.next; 
		N--;
		if (isEmpty()) last = null;
		return item; 
	}
}
```
# Sources
+ [Princeton University: Algorithms part | - Lecture 3 ](https://www.coursera.org/learn/algorithms-part1/lecture/5vgrm/queues)
+ Algorithms 1.3