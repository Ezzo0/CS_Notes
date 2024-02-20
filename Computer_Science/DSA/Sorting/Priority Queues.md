# Explanation
+ Priority Queues is similar to [[Queues]].
+ Binary heap is a data structure that can efficiently support the basic priority-queue operations. 
+ Binary heap is a collection of keys arranged in a complete heap-ordered binary tree, represented in level order in an array.
+ In a binary heap, the keys are stored in an array such that each key (node) is larger than (or equal to) the keys (node’s two children) at two other specific positions.

![[Heap representations.png|500]]
# Priority Queues Implementation
```java
public class MaxPQ<Key extends Comparable<Key>> 
{
	private Key[] pq;                    // heap-ordered complete binary tree
	private int N = 0;                   // in pq[1..N] with pq[0] unused 
	public MaxPQ(int maxN) 
	{ 
		pq = (Key[]) new Comparable[maxN+1]; 
	} 
	
	public boolean isEmpty() 
	{  return N == 0;  } 
	public int size() 
	{  return N;  } 
	
	public void insert(Key v) 
	{ 
		pq[++N] = v; 
		swim(N); 
	} 
	
	public Key delMax() 
	{
		Key max = pq[1];                // Retrieve max key from top.
		exch(1, N--);                   // Exchange with last item.
		pq[N+1] = null;                 // Avoid loitering.
		sink(1);                        // Restore heap property.
		return max;
	}

	private boolean less(int i, int j) 
	{ return pq[i].compareTo(pq[j]) < 0; } 
	private void exch(int i, int j) 
	{ Key t = pq[i]; pq[i] = pq[j]; pq[j] = t; }
	
	private void swim(int k) 
	{ 
		while (k > 1 && less(k/2, k)) 
		{ 
			exch(k/2, k); 
			k = k/2; 
		} 
	}
	
	private void sink(int k) 
	{ 
		while (2*k <= N) 
		{ 
			int j = 2*k; 
			if (j < N && less(j, j+1)) j++; 
			if (!less(k, j)) break; 
			exch(k, j); k = j; 
		} 
	}
}
```

# Heap Sort Implementation
```java
public static void sort(Comparable[] a) 
{ 
	int N = a.length; 
	for (int k = N/2; k >= 1; k--) 
		sink(a, k, N); 
	while (N > 1) 
	{ 
		exch(a, 1, N--); 
		sink(a, 1, N); 
	} 
}
```
# Analysis
+ In an N-key priority queue, the heap algorithms require no more than $1 + lgN$ compares for insert and no more than $2 lgN$ compares for remove the maximum.
+ Heap Sort use ~$2N lgN$ compares and constant extra space in the worst case.
# Sources
+ [Princeton University: Algorithms part | - Lecture 7 ](https://www.coursera.org/learn/algorithms-part1/lecture/A3kA3/apis-and-elementary-implementations)
+ Algorithms 2.4