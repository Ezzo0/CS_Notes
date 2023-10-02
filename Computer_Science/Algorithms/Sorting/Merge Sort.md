# Explanation
+ Merge sort is based on merging which is combining two ordered arrays to make one larger ordered array. 
+ Merge sort operations are:
	1. Divide it into two halves. 
	2. Sort the two halves (recursively).
	3. Merge the results.

![[Mergesort overview.png]]
# Merge Sort Implementation
+ Top-down merge sort (Recursively Method):
```java
public class Merge 
{ 
	private static Comparable[] aux;      // auxiliary array for merges
	public static void sort(Comparable[] a) 
	{
		aux = new Comparable[a.length];   // Allocate space just once.
		sort(a, 0, a.length - 1);
	}
	
	public static void merge(Comparable[] a, int lo, int mid, int hi)
	{   // Merge a[lo..mid] with a[mid+1..hi].|
		int i = lo, j = mid+1; 
		for (int k = lo; k <= hi; k++) 
			aux[k] = a[k];               // Copy a[lo..hi] to aux[lo..hi].
		for (int k = lo; k <= hi; k++)   // Merge back to a[lo..hi].
		{
			if (i > mid)                    a[k] = aux[j++]; 
			else if (j > hi )               a[k] = aux[i++]; 
			else if (less(aux[j], aux[i]))  a[k] = aux[j++]; 
			else                            a[k] = aux[i++]; 
		}
	}
	
	private static void sort(Comparable[] a, int lo, int hi)
	{   // Sort a[lo..hi]. 
		if (hi <= lo)   return;
		int mid = lo + (hi - lo)/2;
		sort(a, lo, mid);        // Sort left half.
		sort(a, mid+1, hi);      // Sort right half.
		merge(a, lo, mid, hi);
	}
}
```
+ Bottom-up merge sort (Iterative Method):
```java
public class MergeBU 
{
	private static Comparable[] aux;   // auxiliary array for merges
	// See Recursively Method for merge() code.
	public static void sort(Comparable[] a)
	{   // Do lg N passes of pairwise merges.
		int N = a.length; 
		aux = new Comparable[N]; 
		for (int sz = 1; sz < N; sz = sz+sz)           // sz: subarray size 
			for (int lo = 0; lo < N-sz; lo += sz+sz)   // lo: subarray index 
				merge(a, lo, lo+sz-1, Math.min(lo+sz+sz-1, N-1)); 
	} 
}
```
# Analysis
+ It sorts any array of N items in ~ $N log N$.
+ It uses extra space proportional to N.
# Sources
+ [Princeton University: Algorithms part | - Lecture 5 ](https://www.coursera.org/learn/algorithms-part1/lecture/ARWDq/mergesort)
+ Algorithms 2.2