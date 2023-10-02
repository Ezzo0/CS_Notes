# Explanation
+ It is a complementary method to [[Quick-Find]] that concentrates on speeding up the union() operation.
+ It is based on the same data structure (the site-indexed array).
+ To implement find(), we start at the given site, follow its link (id[p]) until reaching a root, a site that has a link to itself. 
+ Two sites are in the same component if and only if this process leads them to the same root.
+ To implement union(p, q), we follow links to find the roots associated with p and q, then rename one of the components by linking one of these roots to the other. 
+ The resulting structures of Quick-Union operations are trees.

![[Quick-union overview.png]]
# Implementation
```java
private int find(int p)
{   // Find component name.
	while (p != id[p]) p = id[p]; 
	return p; 
} 

public void union(int p, int q)
{   // Give p and q the same root.
	int pRoot = find(p); 
	int qRoot = find(q);
	if (pRoot == qRoot) return; 
	id[pRoot] = qRoot; 
	count--; 
}
```
# Analysis
+ The number of array accesses used by find() in quick-union is 1 + 2 x depth of the node corresponding to the given site. 
+ The number of array accesses used by union() and connected() is the cost of the two find() operations.

![[order of growth of number of array accesses with quick union.png]]
# Sources
+ [Princeton University: Algorithms part | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part1/lecture/ZgecU/quick-union)
+ Algorithms 1.5