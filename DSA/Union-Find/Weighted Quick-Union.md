# Explanation
+ Rather than arbitrarily connection of the two trees in [[Quick-Union]] for `union()`, we keep track of the size of each tree and always connect the smaller tree to the larger.
# Implementation
```java
public class WeightedQuickUnionUF 
{
	private int[] id;    // parent link (site indexed)
	private int[] sz;    // size of component for roots (site indexed)
	private int count;   // number of components

	public WeightedQuickUnionUF(int N) 
	{ 
		count = N; 
		id = new int[N]; 
		for (int i = 0; i < N; i++) id[i] = i; 
		sz = new int[N]; 
		for (int i = 0; i < N; i++) sz[i] = 1; 
	} 
	
	private int find(int p)
	{   // Follow links to find a root.
		while (p != id[p]) p = id[p]; 
	    return p; 
	} 
	public void union(int p, int q) 
	{ 
		int i = find(p); 
		int j = find(q); 
		if (i == j) return;
		// Make smaller root point to larger one.
		
		if (sz[i] < sz[j]) { id[i] = j; sz[j] += sz[i]; } 
		else { id[j] = i; sz[i] += sz[j]; } 
		count--; 
	} 
}
```
# Analysis
+ The figure at right illustrates the worst case for weighted quick union, when the sizes of the trees to be merged by union() are always equal (and a power of 2). 
+ The tree structures have the simple property that the height of a tree of $2^n$ nodes is n.
+ When two trees of $2^n$ nodes are merged, a tree of $2^n +1$ nodes is got, and the height of the tree is increased to $n+1$. 
+ The weighted algorithm can guarantee logarithmic performance.

![[Weighted quick-union traces (forests of trees).png]]

![[Performance characteristics of union-find algorithms.PNG]]
# Sources
+ [Princeton University: Algorithms part | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part1/lecture/RZW72/quick-union-improvements)
+ Algorithms 1.5