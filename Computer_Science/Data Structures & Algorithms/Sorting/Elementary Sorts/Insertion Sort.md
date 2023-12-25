# Explanation

+ It considers the cards one at a time by inserting each into its proper place among those already considered.
+ Insertion sort is an efficient method for partially sorted arrays; [[Selection Sort]] is not.
+ The number of exchanges used by insertion sort is equal to the number of inversions in the array.
+ 
# Insertion Sort Implementation
```java
public class Insertion 
{ 
	public static void sort(Comparable[] a)
	{   // Sort a[] into increasing order.
		int N = a.length;    // array length
		for (int i = 0; i < N; i++)
		{   // Insert a[i] among a[i-1], a[i-2], a[i-3]... ..
			int min = i;   // index of minimal entr.
			for (int j = i; j > 0 && less(a[j], a[j-1]); j++)
				exch(a, j, j-1);
		}
	}
// See page 245 in Algorithms book for less(), exch(), isSorted(), and main().
}
```
# Analysis
+ Unlike that of [[Selection Sort]], the running time of insertion sort depends on the initial order of the items in the input.
+ Insertion sort uses ~$\frac{N^2}{4}$ compares and ~$\frac{N^2}{4}$ exchanges to sort a randomly ordered array of length N with distinct keys, on the average. 
+ The worst case is ~$\frac{N^2}{2}$ compares and ~$\frac{N^2}{2}$ exchanges. 
+ The best case is $N - 1$ compares and 0 exchanges.
+ The number of exchanges used by insertion sort is equal to the number of inversions in the array.
+ The number of compares is at least equal to the number of inversions and at most equal to the number of inversions plus the array size minus 1.
# Sources
+ [Princeton University: Algorithms part | - Lecture 4 ](https://www.coursera.org/learn/algorithms-part1/lecture/1hYlN/insertion-sort)
+ Algorithms 2.1