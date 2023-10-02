# Explanation
+ Quicksort is a divide-and-conquer method for sorting.
+ It works by partitioning an array into two subarrays, then sorting the subarrays independently. 
+ Quicksort is complementary to [[Merge Sort]].
+ In quicksort, the two recursive calls are done after working on the whole array. 
+ For merge sort, the array is divided in half; for quicksort, the position of the partition depends on the contents of the array.
+ The partitioning process makes the following three conditions hold:
	+ The entry a[ j ] is in its final place in the array, for some j.
	+ No entry in a[ lo ] through a[ j - 1 ] is greater than a[ j ].
	+ No entry in a[ j + 1 ] through a[ hi ] is less than a[ j ].

![[Quicksort overview.png]]
# Quick Sort Implementation
```java
public class Quick 
{ 
	public static void sort(Comparable[] a) 
	{
		StdRandom.shuffle(a);            // Eliminate dependence on input.
		sort(a, 0, a.length - 1); 
	} 
	private static void sort(Comparable[] a, int lo, int hi) 
	{ 
		if (hi <= lo) return;
		int j = partition(a, lo, hi); 
		sort(a, lo, j-1);                // Sort left part a[lo .. j-1]. 
		sort(a, j+1, hi);                // Sort right part a[j+1 .. hi].

	}
	/* 
	 * partition without caring about other items with the same key value as the
	 * partitioning item
	 */
	private static int partition(Comparable[] a, int lo, int hi)
	{   // Partition into a[lo..i-1], a[i], a[i+1..hi].
		int i = lo, j = hi+1;           // left and right scan indices
		Comparable v = a[lo];          // partitioning item 
		while (true) 
		{   // Scan right, scan left, check for scan complete, and exchange.
			while (less(a[++i], v)) if (i == hi) break; 
			while (less(v, a[--j])) if (j == lo) break; 
			if (i >= j) break; 
			exch(a, i, j); 
		} 
		exch(a, lo, j);                // Put v = a[j] into position 
		return j;                     // with a[lo..j-1] <= a[j] <= a[j+1..hi]. }
}

public class Quick3way 
{ 
	private static void sort(Comparable[] a, int lo, int hi)
	{   // See page 289 for public sort() that calls this method.
		if (hi <= lo) return; 
		int lt = lo, i = lo+1, gt = hi; 
		Comparable v = a[lo]; 
		while (i <= gt) 
		{ 
			int cmp = a[i].compareTo(v); 
			if (cmp < 0) exch(a, lt++, i++); 
			else if (cmp > 0) exch(a, i, gt--); 
			else i++; 
		}   // Now a[lo..lt-1] < v = a[lt..gt] < a[gt+1..hi]. 
		sort(a, lo, lt - 1); 
		sort(a, gt + 1, hi); 
	} 
}
```

![[Partitioning trace.png]]

![[3-way partitioning trace.png]]
# Analysis
+ The quicksort requires time proportional to ~$Nlog N$ on the average.
+ The efficiency of the sort depends on how well the partitioning divides the array, which in turn depends on the value of the partitioning item’s key. 
+ Partitioning divides a large randomly ordered array into two smaller randomly ordered subarrays, but the actual split is equally likely (for distinct keys) to be anywhere in the array.
+ The best case for quicksort is when each partitioning stage divides the array exactly in half.
+ Quicksort uses ~$2NlnN$compares (and one-sixth that many exchanges) on the average to sort an array of length N with distinct keys.
+ Quicksort uses ~ $N^2/2$ compares in the worst case, but random shuffling protects against this case.
+ [[Merge Sort]] is linearithmic for a randomly ordered array that has only a constant number of distinct key values, but quicksort with 3-way partitioning is linear for such an array.
# Sources
+ [Princeton University: Algorithms part | - Lecture 6 ](https://www.coursera.org/learn/algorithms-part1/lecture/1hYlN/insertion-sort)
+ [HackerRank - Algorithms: Quicksort](https://www.youtube.com/watch?v=SLauY6PpjW4)
+ Algorithms 2.3