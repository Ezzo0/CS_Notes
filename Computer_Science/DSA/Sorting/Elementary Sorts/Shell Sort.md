# Explanation

+ Shell sort is a simple extension of [[Insertion Sort]]. 
+ Shell sort allow exchanges of array entries that are far apart, to produce partially sorted arrays that can be efficiently sorted, eventually by [[Insertion Sort]].

![[Detailed trace of shellsort (insertions).png]]
# Shell Sort Implementation
```java
public class Shell 
{
	public static void sort(Comparable[] a)
	{   // Sort a[] into increasing order.
		int N = a.length; 
		int h = 1; 
		while (h < N/3) h = 3*h + 1; // 1, 4, 13, 40, 121, 364, 1093, ... 
		while (h >= 1)
		{   // h-sort the array.
			for (int i = h; i < N; i++) 
			{   // Insert a[i] among a[i-h], a[i-2*h], a[i-3*h]... . 
				for (int j = i; j >= h && less(a[j], a[j-h]); j -= h) 
					exch(a, j, j-h); 
			} 
			h = h/3; 
		} 
	}
// See page 245 for less(), exch(), isSorted(), and main().
}
```
# Analysis
+ The running time of shell sort is depending on the increment sequence. 
+ For the choosed sequence in the implementation, the worst-case number of compares for Algorithm 2.3 is ~$N^\frac{3}{2}$.
# Sources
+ [Princeton University: Algorithms part | - Lecture 4 ](https://www.coursera.org/learn/algorithms-part1/lecture/zPYhF/shellsort)
+ Algorithms 2.1