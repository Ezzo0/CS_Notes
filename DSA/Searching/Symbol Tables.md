# Explanation
+ A symbol table is a data structure for key-value pairs.
+ Symbol tables supports two operations: 
	+ Insert (put) a new pair into the table.
	+ Search for (get) the value associated with a given key.
+ Only one value is associated with each key (no duplicate keys in a table).
+ When a client puts a key-value pair into a table already containing that key (and an associated value), the new value replaces the old one.
+ Keys must not be null.
# Symbol Tables Implementation
+ Linked-List implementation:
```java
public class SequentialSearchST<Key, Value> 
{
	private Node first; // first node in the linked list

	private class Node 
	{   // linked-list node 
		Key key; 
		Value val; 
		Node next; 
		public Node(Key key, Value val, Node next) 
		{ 
			this.key = key; 
			this.val = val; 
			this.next = next; 
		} 
	} 
	
	public Value get(Key key) 
	{   // Search for key, return associated value. 
		for (Node x = first; x != null; x = x.next) 
			if (key.equals(x.key)) 
				return x.val;  // search hit 
			return null;       // search miss 
	} 
	
	public void put(Key key, Value val) 
	{   // Search for key. Update value if found; grow table if new. 
		for (Node x = first; x != null; x = x.next) 
			if (key.equals(x.key)) 
			{ x.val = val; return; }       // Search hit: update val.
		first = new Node(key, val, first); // Search miss: add new node. 
	}

    public void delete(Key key) 
    {
        if (key == null) 
	        throw new IllegalArgumentException("argument to delete() is null");
        first = delete(first, key);
    }
    private Node delete(Node x, Key key) 
    {
        if (x == null) return null;
        if (key.equals(x.key))
        {
            n--;
            return x.next;
        }
        x.next = delete(x.next, key);
        return x;
    }
    public Iterable<Key> keys()  {
        Queue<Key> queue = new Queue<Key>();
        for (Node x = first; x != null; x = x.next)
            queue.enqueue(x.key);
        return queue;
    }
}
```

+ Binary Search implementation:
```java

public class BinarySearchST<Key extends Comparable<Key>, Value> 
{
    private Key[] keys;
    private Value[] vals;
    private int n = 0;

     //Initializes an empty symbol table with the specified initial capacity.
    public BinarySearchST(int capacity)
    {
        keys = (Key[]) new Comparable[capacity];
        vals = (Value[]) new Object[capacity];
    }
    
    public int size() 
    {   return n;   }

    public boolean isEmpty()
    {  return size() == 0; }
    
    public Value get(Key key) 
    {
        if (key == null) 
	        throw new IllegalArgumentException("argument to get() is null");
        if (isEmpty()) return null;
        int i = rank(key);
        if (i < n && keys[i].compareTo(key) == 0) return vals[i];
        else                                      return null;
    }

    public int rank(Key key) 
    {
        if (key == null) 
	        throw new IllegalArgumentException("argument to rank() is null");
        int lo = 0, hi = n-1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            int cmp = key.compareTo(keys[mid]);
            if      (cmp < 0) hi = mid - 1;
            else if (cmp > 0) lo = mid + 1;
            else return mid;
        }
        return lo;
    }

    public void put(Key key, Value val)  
    {
        int i = rank(key); 
        if (i < N && keys[i].compareTo(key) == 0) 
        {        vals[i] = val; return;         } 
        for (int j = N; j > i; j--) 
        { 
	        keys[j] = keys[j-1]; 
	        vals[j] = vals[j-1]; 
	    } 
	    keys[i] = key; 
	    vals[i] = val;
	    N++;
    }
    
    public void delete(Key key) 
    {
        if (key == null) 
	        throw new IllegalArgumentException("argument to delete() is null");
        if (isEmpty()) return;
        // compute rank
        int i = rank(key);
        
        // key not in table
        if (i == n || keys[i].compareTo(key) != 0) {
            return;
        }
        for (int j = i; j < n-1; j++)
        {
            keys[j] = keys[j+1];
            vals[j] = vals[j+1];
        }
        n--;
        keys[n] = null;  // to avoid loitering
        vals[n] = null;
        // resize if 1/4 full
        if (n > 0 && n == keys.length/4) resize(keys.length/2);
    }
}
```
# Analysis
+ Unsuccessful search ,Successful search, and insertions in an (unordered) linked-list symbol table having N key-value pairs require N compares in the worst case.
+ Binary search in an ordered array with N keys uses no more than $lg N + 1$ compares for a search (successful or unsuccessful).
+ Inserting a new key into an ordered array of size N uses ~ $2N$ array accesses in the worst case.
# Sources
+ [Princeton University: Algorithms part | - Lecture 8 ](https://www.coursera.org/learn/algorithms-part1/lecture/7WFvG/symbol-table-api)
+ Algorithms 3.1