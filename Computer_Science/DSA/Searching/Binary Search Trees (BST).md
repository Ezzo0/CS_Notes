# Explanation
+ Binary trees is a data structures made up of nodes that contain links.
+ Links in Binary tress are either null or references to other nodes.
+ In a binary tree, every node is pointed to by just one other node, which is called its parent (except for one node, the root, which has no nodes pointing to it).
+ In a binary tree, each node has exactly two links, which are called its left and right links, that point to nodes called its left child and right child, respectively.
+ In a binary tree, each link references to (disjoint) subtrees that are themselves binary trees.
+ A binary search tree (BST) is a binary tree where each node has a Comparable key and an associated value.
+ BST satisfies the restriction that the key in any node is larger than the keys in all nodes in that node’s left subtree and smaller than the keys in all nodes in that node’s right subtree.

![[Anatomy of a binary search tree.png|400]]
# Ordered [[Symbol Tables]] using BST Implementation

```java
public class BST<Key extends Comparable<Key>, Value>
{
	private Node root;              // root of BST
	private class Node 
	{ 
		private Key key;            // key
		private Value val;          // associated value
		private Node left, right;   // links to subtrees
		private int N;              // # nodes in subtree rooted here
		
		public Node(Key key, Value val, int N) 
		{ this.key = key; this.val = val; this.N = N; } 
	} 
	
	public int size() 
	{ return size(root); } 
	private int size(Node x) 
	{ 
		if (x == null) return 0; 
		else return x.N; 
	} 
	
	public Value get(Key key) 
	{ return get(root, key); } 
	private Value get(Node x, Key key)
	{    // Return value associated with key in the subtree rooted at x; 
	     // return null if key not present in subtree rooted at x.|
		if (x == null) return null; 
		int cmp = key.compareTo(x.key); 
		if (cmp < 0) return get(x.left, key); 
		else if (cmp > 0) return get(x.right, key); 
		else return x.val; 
	}
	
	public void put(Key key, Value val) 
	{   // Search for key. Update value if found; grow table if new. 
		root = put(root, key, val); 
	} 
	private Node put(Node x, Key key, Value val)
	{   // Change key’s value to val if key in subtree rooted at x. 
	    // Otherwise, add new node to subtree associating key with val.
		if (x == null) return new Node(key, val, 1); 
		int cmp = key.compareTo(x.key); 
		if (cmp < 0) x.left = put(x.left, key, val); 
		else if (cmp > 0) x.right = put(x.right, key, val); 
		else x.val = val; 
		x.N = size(x.left) + size(x.right) + 1; 
		return x; 
	}
	
	public Key min() 
	{ return min(root).key; }
	private Node min(Node x) 
	{ 
		if (x.left == null) return x; 
		return min(x.left); 
	}
	
	public Key floor(Key key)
	{ 
		Node x = floor(root, key); 
		if (x == null) return null; 
		return x.key; 
	} 
	private Node floor(Node x, Key key)
	{ 
		if (x == null) return null; 
		int cmp = key.compareTo(x.key); 
		if (cmp == 0) return x; 
		if (cmp < 0) return floor(x.left, key); 
		Node t = floor(x.right, key); 
		if (t != null) return t; 
		else return x; 
	}
	
	public Key select(int k)
	{
		return select(root, k).key;
	}
	private Node select(Node x, int k)
	{   // Return Node containing key of rank k.
		if (x == null) return null;
		int t = size(x.left);
		if (t > k) return select(x.left, k);
		else if (t < k) return select(x.right, k-t-1);
		else return x;
	}
	
	public int rank(Key key)
	{ return rank(key, root); }
	private int rank(Key key, Node x)
	{   // Return number of keys less than x.key in the subtree rooted at x.
		if (x == null) return 0;
		int cmp = key.compareTo(x.key);
		if (cmp < 0) return rank(key, x.left);
		else if (cmp > 0) return 1 + size(x.left) + rank(key, x.right);
		else return size(x.left);
	}
	
	public void deleteMin() 
	{ root = deleteMin(root); } 
	private Node deleteMin(Node x) 
	{ 
		if (x.left == null) return x.right; 
		x.left = deleteMin(x.left); 
		x.N = size(x.left) + size(x.right) + 1; 
		return x; 
	} 
	
	public void delete(Key key) 
	{ root = delete(root, key); } 
	private Node delete(Node x, Key key) 
	{ 
		if (x == null) return null; 
		int cmp = key.compareTo(x.key); 
		if (cmp < 0) x.left = delete(x.left, key); 
		else if (cmp > 0) x.right = delete(x.right, key); 
		else 
		{ 
			if (x.right == null) return x.left; 
			if (x.left == null) return x.right; 
			Node t = x;
			x = min(t.right);
			x.right = deleteMin(t.right); 
			x.left = t.left;
		} 
		x.N = size(x.left) + size(x.right) + 1; 
		return x; 
	}
	public Iterable<Key> keys()
	{ return keys(min(), max()); }
	
	public Iterable<Key> keys(Key lo, Key hi)
	{
		Queue<Key> queue = new Queue<Key>();
		keys(root, queue, lo, hi);
		return queue;
	}
	private void keys(Node x, Queue<Key> queue, Key lo, Key hi)
	{
		if (x == null) return;
		int cmplo = lo.compareTo(x.key);
		int cmphi = hi.compareTo(x.key);
		if (cmplo < 0) keys(x.left, queue, lo, hi);
		if (cmplo <= 0 && cmphi >= 0) queue.enqueue(x.key);
		if (cmphi > 0) keys(x.right, queue, lo, hi);
	}
}
```
# Analysis
+ The running times of algorithms on binary search trees depend on the shapes of the trees, which, in turn, depend on the order in which keys are inserted. 
+ In the best case, a tree with N nodes could be perfectly balanced, with ~ $lg N$ nodes between the root and each null link. 
+ In the worst case there could be N nodes on the search path. The balance in typical trees turns out to be much closer to the best case than the worst case.
+ BSTs are dual to [[Quick Sort]]. 
+ The node at the root of the tree corresponds to the first partitioning item in [[Quick Sort]] (no keys to the left are larger, and no keys to the right are smaller).
+ Search hits in a BST built from N random keys require ~ $2 ln N$ (about $1.39 lg N$) compares, on the average.
+ Insertions and search misses in a BST built from N random keys require ~ $2 ln N$ (about $1.39 lg N$) compares, on the average.
+ In a BST, all operations take time proportional to the height of the tree, in the worst case.
+ After sufficiently long sequence of random inserts and deletes (using successor), the height of the tree becomes $\sqrt{N}$.
# Sources
+ [Princeton University: Algorithms part | - Lecture 8 ](https://www.coursera.org/learn/algorithms-part1/lecture/7An9B/binary-search-trees)
+ Algorithms 3.2