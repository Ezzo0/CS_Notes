# Explanation

+ Selection sort works as follows: 
	1. Find the smallest item in the array and exchange it with the first entry (itself if the first entry is already the smallest). 
	2. Find the next smallest item and exchange it with the second entry. 
	3. Continue in this way until the entire array is sorted.
# Selection Sort Implementation
```java
public class Selection 
{ 
	public static void sort(Comparable[] a)
	{   // Sort a[] into increasing order.
		int N = a.length;    // array length
		for (int i = 0; i < N; i++)
		{   // Exchange a[i] with smallest entry in a[i+1...N).
			int min = i;   // index of minimal entr.
			for (int j = i+1; j < N; j++)
				if (less(a[j], a[min])) min = j; 
			exch(a, i, min); 
		}
	}
// See page 245 in Algorithms book for less(), exch(), isSorted(), and main().
}
```
# Analysis
+ Selection sort uses ~$N^2/2$ compares and N exchanges to sort an array of length N.
+ Running time is insensitive to input: it takes about as long to run selection sort for an array that is already in order or for an array with all keys equal as it does for a randomly-ordered array.
+ The number of array accesses is a linear function of the array size.
# Sources
+ [Princeton University: Algorithms part | - Lecture 4 ](https://www.coursera.org/learn/algorithms-part1/lecture/VE0sv/selection-sort)
+ Algorithms 2.1