# Explanation
+ A 2-3 search tree is a tree that is either empty or 
	+ A 2-node, with one key (and associated value) and two links, a left link to a 2-3 search tree with smaller keys, and a right link to a 2-3 search tree with larger keys.
	+ A 3-node, with two keys (and associated values) and three links, a left link to a 2-3 search tree with smaller keys, a middle link to a 2-3 search tree with keys between the node’s keys, and a right link to a 2-3 search tree with larger keys As usual, we refer to a link to an empty tree as a null link.
+ A perfectly balanced 2-3 search tree is one whose null links are all the same distance from the root.
![[Anatomy of a 2-3 search tree.png|250]]
+ #### Search
	+ To determine whether a key is in the tree, we compare it against the keys at the root. 
	+ If it is equal to any of them, we have a successful search; otherwise, we follow the link from the root to the subtree corresponding to the interval of key values that could contain the search key. If that link is null, we have an unsuccessful search; otherwise we recursively search in that subtree.

	![[Search hit (left) and search miss (right) in a 2-3 tree.png]]
+ #### Insert into a 2-node
	+ 2-3 trees do insertions and still maintain perfect balance. 
	+ If the node at which the search terminates is a 2-node: we just replace the node with a 3-node containing its key and the new key to be inserted.

	![[Insert into a 2-node.png|350]]
+ #### Insert into a tree consisting of a single 3-node
	+ To be able to perform the insertion, we temporarily put the new key into a 4-node.
	+ It is easy to convert it into a 2-3 tree made up of three 2-nodes: 
		+ One with the middle key (at the root). 
		+ One with the smallest of the three keys (pointed to by the left link of the root).
		+ One with the largest of the three keys (pointed to by the right link of the root).

	![[Insert into a single 3-node.png|230]]
+ #### Insert into a 3-node whose parent is a 2-node
	+ Suppose that the search ends at a 3-node at the bottom whose parent is a 2-node. 
		+ In this case, we can still make room for the new key, by making a temporary 4-node. 
		+ Splitting the 4-node.
		+ Moving the middle key to the node’s parent.

	![[Insert into a 3-node whose parent is a 2-node 1.png|300]]
+ #### Insert into a 3-node whose parent is a 3-node
	+ Suppose that the search ends at a node whose parent is a 3-node. 
		+ We make a temporary 4-node, then split it and insert its middle key into the parent. 
		+ The parent was a 3-node, so we replace it with a temporary new 4-node containing the middle key from the 4-node split. 
		+ Then, we perform precisely the same transformation on that node and continue up the tree until reaching a 2-node, which we replace with a 3-node that does not need to be further split, or until reaching a 3-node at the root. 

	![[Insert into a 3-node whose parent is a 3-node.png|300]]

	![[Splitting the root.png|300]]

# Analysis
+ The basis of the 2-3 tree insertion algorithm is that all of these transformations are purely local: no part of the tree needs to be examined or modified other than the specified nodes and links.
+ The number of links changed for each transformation is bounded by a small constant.
+ The local transformations preserve the global properties that the tree is ordered and perfectly balanced (the number of links on the path from the root to any null link is the same).
+ Unlike standard [[Binary Search Trees (BST)]], which grow down from the top, 2-3 trees grow up from the bottom.
+ In a BST, the increasing-order sequence for 10 keys results in a worst-case tree of height 9. In the 2-3 trees, the height is 2.
+ Search and insert operations in a 2-3 tree with N keys are guaranteed to visit at most lg N nodes.
+ The height of an N-node 2-3 tree is between $⎣log3 N⎦ = ⎣(lg N)/(lg 3)⎦$ (if the tree is all 3-nodes) and $⎣lg N⎦$ (if the tree is all 2-nodes).
# Sources
+ [Princeton University: Algorithms part | - Lecture 9 ](https://www.coursera.org/learn/algorithms-part1/lecture/wIUNW/2-3-search-trees)
+ Algorithms 3.3