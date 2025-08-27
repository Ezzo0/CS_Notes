# Explanation

+ The input is a sequence of pairs of integers, where each integer represents an object of some type and we are to interpret the pair p q as meaning “p is connected to q.”
+ An equivalence relation means that:
	+ Reflexive : p is connected to p.
	+ Symmetric : If p is connected to q, then q is connected to p.
	+ Transitive : If p is connected to q and q is connected to r, then p is connected to r.
+ An equivalence relation partitions the objects into equivalence classes, which means that 2 objects is connected if and only if they are in the same equivalence class

![[dynamic connectivity.jpg]]
___
**Note:**
+ objects will be referred as sites, pairs as connections, and the equivalence classes as components.
___
+ The Data structure used to represent the components is site-indexed array. 
+ To solve Dynamic Connectivity problem there are 3 approaches:
	+ [[Quick-Find]]
	+ [[Quick-Union]]
	+ [[Weighted Quick-Union]]
+ APIs for these approaches are:
![[APIs.png]]
# Union-Find Implementation
```java
public class UF 
{
	private int[] id;     // access to component id (site indexed)
	private int count;    // number of components
	public UF(int N)
	{
		// Initialize component id array.
		count = N; 
		id = new int[N]; 
		for (int i = 0; i < N; i++) 
		id[i] = i; 
	} 
	public int count() 
	{ return count; } 
	public boolean connected(int p, int q) 
	{ return find(p) == find(q); } 
	public int find(int p) 
	public void union(int p, int q)

	/* 
	find & union are implemented in Quick-Find, Quick-Union
	 & Weighted Qucick-Union sections
	*/

	public static void main(String[] args)
	{
		// Solve dynamic connectivity problem on StdIn.
		int N = StdIn.readInt();               // Read number of sites.
		UF uf = new UF(N);                     // Initialize N components.
		while (!StdIn.isEmpty()) 
		{ 
			int p = StdIn.readInt();     
			int q = StdIn.readInt();           // Read pair to connect.
			if (uf.connected(p, q)) continue;  // Ignore if connected.
			uf.union(p, q);                    // Combine components
			StdOut.println(p + " " + q);       // and print connection.|
		}
		StdOut.println(uf.count() + " components"); 
	} 
}
```
# Sources
+ [Princeton University: Algorithms part | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part1/lecture/fjxHC/dynamic-connectivity)
+ Algorithms 1.5
