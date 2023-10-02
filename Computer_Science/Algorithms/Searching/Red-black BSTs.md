# Explanation
+ The basic idea behind red-black BSTs is to encode [[2-3 Search Trees]] by starting with standard [[Binary Search Trees (BST)]] and adding extra information to encode 3-nodes.
+ The links is consisting of two different types: red links, which bind together two 2-nodes to represent 3-nodes, and black links, which bind together the 2-3 tree.
+ No node has two red links connected to it.

![[Encoding a 3-node with two 2-nodes.png|300]]
+ If we draw the red links horizontally in a red-black BST, all of the null links are the same distance from the root, and if we then collapse together the nodes connected by red links, the result is a 2-3 tree.

![[1-1 correspondence between red-black BSTs and 2-3 trees.png|400]]
+ #### Rotations
	+ Suppose that we have a right-leaning red link that needs to be rotated to lean to the left. This operation is called a left rotation.

![[Left rotate (right link of h)_1.png|200]]
![[Left rotate (right link of h)_2.png|200]]

+ #### Insert into a single 2-node
	+ If the new key is smaller than the key in the tree, we just make a new (red) node with the new key and we are done: we have a red-black BST that is equivalent to a single 3-node. 
	+ If the new key is larger than the key in the tree, then attaching a new (red) node gives a right-leaning red link, and the code `root = rotateLeft(root);` completes the insertion by rotating the red link to the left and updating the tree root link. 
	+ The result in both cases is the red-black representation of a single 3-node, with two keys, one left-leaning red link, and black height 1.
	![[Insert into a single 2-node (two cases) 1.png| 150]]
+ #### Insert into a tree with two keys (in a 3-node).
	+ This case reduces to three subcases: The new key is either less than both keys in the tree, between them, or greater than both of them.
		1. The simplest of the three cases is when the new key is larger than the two in the tree: 
			+ Therefore the node is attached on the rightmost link of the 3-node, making a balanced tree with the middle key at the root, connected with red links to nodes containing a smaller and a larger key. 
			+ If we flip the colors of those two links from red to black, then we have a balanced tree of height 2 with three nodes.
		2. If the new key is smaller than the two keys in the tree and goes on the left link: 
			+ Then the tree has two red links in a row, both leaning to the left, which can be reduce to the previous case by rotating the top link to the right.
		3. If the new key goes between the two keys in the tree: 
			+ The tree has two red links in a row, a right-leaning one below a left-leaning one, which  can be reduce to the previous case by rotating left the bottom link.

![[Insert into a single 3-node (three cases).png|500]]
+ ==Study deletion process in details==
# Implementation

```java
public class RedBlackBST<Key extends Comparable<Key>, Value> 
{
    private static final boolean RED   = true;
    private static final boolean BLACK = false;
    
    private Node root;     // root of the BST
    
    // BST helper node data type
    private class Node {
        private Key key;           // key
        private Value val;         // associated data
        private Node left, right;  // links to left and right subtrees
        private boolean color;     // color of parent link
        private int size;          // subtree count
        
        public Node(Key key, Value val, boolean color, int size)
        {
            this.key = key;
            this.val = val;
            this.color = color;
            this.size = size;
        }
    }
    
    public RedBlackBST() { }
    
    // is node x red; false if x is null ?
    private boolean isRed(Node x)
    {
        if (x == null) return false;
        return x.color == RED;
    }
    
    // number of node in subtree rooted at x; 0 if x is null
    private int size(Node x) {
        if (x == null) return 0;
        return x.size;
    }
    
    public int size()
    { return size(root); }
    
    public boolean isEmpty()
    { return root == null; }
    
    public void put(Key key, Value val) 
    {
        if (key == null) throw new IllegalArgumentException("first argument is null");
        if (val == null) 
        {
            delete(key);
            return;
        }
        root = put(root, key, val);
        root.color = BLACK;
    }
    
    // insert the key-value pair in the subtree rooted at h
    private Node put(Node h, Key key, Value val) 
    {
        if (h == null) return new Node(key, val, RED, 1);
        
        int cmp = key.compareTo(h.key);
        if      (cmp < 0) h.left  = put(h.left,  key, val);
        else if (cmp > 0) h.right = put(h.right, key, val);
        else              h.val   = val;
        
        // fix-up any right-leaning links
        if (isRed(h.right) && !isRed(h.left))      h = rotateLeft(h);
        if (isRed(h.left)  &&  isRed(h.left.left)) h = rotateRight(h);
        if (isRed(h.left)  &&  isRed(h.right))     flipColors(h);
        h.size = size(h.left) + size(h.right) + 1;
        
        return h;
    }
```
# Analysis
+ All [[Symbol Tables]] operations in red-black BSTs are guaranteed to be logarithmic in the size of the tree (except for range search, which additionally costs time proportional to the number of keys returned).
+ The height of a red-black BST with N nodes is no more than $2 lg N$.
+ The average length of a path from the root to a node in a red-black BST with N nodes is ~$1.00 lg N$.
# Sources
+ [Princeton University: Algorithms part | - Lecture 9 ](https://www.coursera.org/learn/algorithms-part1/lecture/GZe13/red-black-bsts)
+ Algorithms 3.3