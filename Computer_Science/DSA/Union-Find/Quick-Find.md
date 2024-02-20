# Explanation
+ p and q are connected if and only if id[p] is equal to id[q]. In other words, all sites in a component must have the same value in id[].
+ This method is called quick-find because find() just returns id[p], which immediately implies that connected() reduces to just the test (id[p] == id[q]).
+ To combine two components into one, we have to make all of the id[] entries corresponding to both sets of sites the same value by go through the array, changing all the entries with values equal to id[p] to the value id[q] or vice versa. 

![[Quick-find overview.png]]
# Implementation
```java
public int find(int p) 
{ return id[p]; } 

public void union(int p, int q)
{    // Put p and q into the same component.
	int pID = find(p);
	int qID = find(q);

	// Nothing to do if p and q are already in the same component.
	if (pID == qID) return;

	// Rename p’s component to q’s name.
	for (int i = 0; i < id.length; i++) 
		if (id[i] == pID) 
			id[i] = qID; 
	count--; 
}
```
# Analysis
+ The `find()` operation is certainly quick, as it only accesses the id[] array
+ Quick-find is typically slow because `union()` needs to scan through the whole id[] array for each input pair.
+ The quick-find algorithm uses one array access for each call to find() and between N + 3 and 2N + 1 array accesses for each call to union().
+ Quick-find is quadratic for typical applications where we end up with a small number of components. (i.e. suppose that we use quick-find for the dynamic connectivity problem and wind up with a single component. This requires at least N-1 calls to union(), and, consequently, at least (N+3)(N-1) ~ $N^2$  array accesses) 

![[order of growth of number of array accesses.png]]
# Sources
+ [Princeton University: Algorithms part | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part1/lecture/EcF3P/quick-find)
+ Algorithms 1.5