# Explanation
+ If keys are small integers, we can use an array to implement an unordered [[Symbol Tables]], by interpreting the key as an array index.
+ Hashing is an extension of this simple method that handles more complicated types of keys. 
+ Search algorithms that use hashing consist of two separate parts: 
	+ The first part is to compute a hash function that transforms the search key into an array index (there is a possibility that two or more different keys may hash to the same array index).
	+ The second part of a hashing search is a collision-resolution process that deals with this situation.
		+ Separate chaining.
		+ Linear probing.

+ ### Hash function
	+ If we have an array that can hold M key-value pairs, then we need a hash function that can transform any given key into an index into that array. 
	+ The hash function depends on the key type.
	+ three primary requirements in implementing a good hash function for a given data type: 
		+ It should be consistent—equal keys must produce the same hash value. 
		+ It should be efficient to compute. 
		+ It should uniformly distribute the keys.
+ ### collision resolution
	+ ###### Hashing with separate chaining:
		+ The idea is to build, for each of the M array indices, a linked list of the key-value pairs whose keys hash to that index.
		+ The basic idea is to choose M to be sufficiently large that the lists are sufficiently short to enable efficient search.
			![[Hashing with separate chaining for standard indexing client.png|400]]
	+ ###### Hashing with linear probing:
		+ The idea is to store N key-value pairs in a hash table of size M > N.
		+ Linear probing is characterized by identifying three possible outcomes:
			+ Key equal to search key: search hit
			+ Empty position (null key at indexed position): search miss
			+ Key not equal to search key: try next entry.
		![[Trace of linear-probing ST implementation for standard indexing client.png|600]]
# Hash Tables Implementation
+ Hashing with separate chaining:
```java
public class SeparateChainingHashST<Key, Value>
{
	private int N;                           // number of key-value pairs
	private int M;                           // hash table size|
	private SequentialSearchST<Key, Value>[] st;  // array of ST objects
	public SeparateChainingHashST() 
	{ this(997); } 
	public SeparateChainingHashST(int M)
	{   // Create M linked lists.
		this.M = M; 
		st = (SequentialSearchST<Key, Value>[]) new SequentialSearchST[M]; 
		for (int i = 0; i < M; i++) st[i] = new SequentialSearchST(); 
	} 
	
	private int hash(Key key)
	{ return (key.hashCode() & 0x7fffffff) % M; } 
	
	public Value get(Key key)
	{ return (Value) st[hash(key)].get(key); } 
	
	public void put(Key key, Value val)
	{   st[hash(key)].put(key, val);  }
}
```

+ Hashing with linear probing:
```java
public class LinearProbingHashST<Key, Value>
{
	private int N;                  // number of key-value pairs in the table
	private int M = 16;             // size of linear-probing table
	private Key[] keys;             // the keys
	private Value[] vals;           // the values
	
	public LinearProbingHashST()
	{
		keys = (Key[]) new Object[M];
		vals = (Value[]) new Object[M];
	}
	
	private int hash(Key key)
	{ return (key.hashCode() & 0x7fffffff) % M; }
	
	private void resize()             // See page 474.
	
	public void put(Key key, Value val)
	{
		if (N >= M/2) resize(2*M);     // double M (see text)
		int i;
		for (i = hash(key); keys[i] != null; i = (i + 1) % M)
			if (keys[i].equals(key)) { vals[i] = val; return; }
		keys[i] = key;
		vals[i] = val;
		N++;
	}

	public Value get(Key key)
	{
		for (int i = hash(key); keys[i] != null; i = (i + 1) % M)
			if (keys[i].equals(key))
				return vals[i];
			return null;
	}
	public void delete(Key key) 
	{ 
		if (!contains(key)) return; 
		int i = hash(key); 
		while (!key.equals(keys[i])) 
			i = (i + 1) % M; 
		keys[i] = null; 
		vals[i] = null; 
		i = (i + 1) % M; 
		while (keys[i] != null) 
		{ 
			Key keyToRedo = keys[i]; 
			Value valToRedo = vals[i]; 
			keys[i] = null; 
			vals[i] = null; 
			N--; 
			put(keyToRedo, valToRedo); 
			i = (i + 1) % M; 
		} 
		N--; 
		if (N > 0 N == M/8) resize(M/2); 
	}
}
```
# Analysis
+ With hashing, you can implement search and insert for symbol tables that require constant (amortized) time per operation.
+ In a separate-chaining hash table with M lists and N keys, the probability (under Assumption J) that the number of keys in a list is within a small constant factor of $N/M$ is extremely close to 1.
+ In a separate-chaining hash table with M lists and N keys, the number of compares (equality tests) for search miss and insert is ~$N/M$.
+ The average cost of linear probing depends on the way in which the entries clump together into contiguous groups of occupied table entries, called clusters, when they are inserted.
# Sources
+ [Princeton University: Algorithms part | - Lecture 11 ](https://www.coursera.org/learn/algorithms-part1/lecture/CMLqa/hash-tables)
+ Algorithms 3.4