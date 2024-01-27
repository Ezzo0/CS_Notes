# Explanation
+ In Directed [[Graphs]], edges are one-way.
+ A directed edge points from the first vertex in the pair and points to the second vertex in the pair.
+ The outdegree of a vertex in a digraph is the number of edges going from it; the indegree of a vertex is the number of edges going to it.
+ The first vertex in a directed edge is called its head ; the second vertex is called its tail.
	 ![[Anatomy of a digraph.png|350]]
+ We use the adjacency-lists representation, where an edge v->w is represented as a list node containing w in the linked list corresponding to v.
+ In the adjacency-lists representation of an  [[Undirected Graphs]], we know that if v is on w’s list, then w will be on v’s list; the adjacency-lists representation of a digraph has no such symmetry.
+ #### Cycles and DAGs
	+ A directed acyclic graph (DAG) is a digraph with no directed cycles.
	+ A digraph has a topological order if and only if it is a DAG.
	 ![[Topological sort.png|250]]
+ #### Strong connectivity in digraphs
	+ Two vertices v and w are strongly connected if they are mutually reachable: that is, if there is a directed path from v to w and a directed path from w to v. 
	+ A digraph is strongly connected if all its vertices are strongly connected to one another.
	 ![[Strongly connected digraphs.png|300]]
	+ The equivalence classes are maximal subsets of vertices that are strongly connected to one another, with each vertex in exactly one subset.
	 ![[A digraph and its strong components.png|300]]
	+ A strongly connected digraph has 1 strong component. 
	+ A DAG has V strong components. 
	+ The strong components are defined in terms of the vertices, not the edges.
	+ In a DFS of a digraph G where marked vertices are considered in reverse postorder given by a DFS of the digraph’s reverse $G^R$ (Kosaraju’s algorithm), the vertices reached in each call of the recursive method from the constructor are in a strong component.
# Implementation
- Digraph adjacency-list implementation:
```java
public class Digraph 
{ 
	private final int V; 
	private int E; 
	private Bag<Integer>[] adj;
	 
	public Digraph(int V) 
	{ 
		this.V = V; 
		this.E = 0; 
		adj = (Bag<Integer>[]) new Bag[V]; 
		for (int v = 0; v < V; v++) 
		adj[v] = new Bag<Integer>(); 
	}
	
	public int V() { return V; } 
	public int E() { return E; } 
	
	public void addEdge(int v, int w) 
	{ 
		adj[v].add(w); 
		E++; 
	} 
	
	public Iterable<Integer> adj(int v) 
	{ return adj[v]; } 
	
	public Digraph reverse() 
	{ 
		Digraph R = new Digraph(V); 
		for (int v = 0; v < V; v++) 
			for (int w : adj(v)) 
				R.addEdge(w, v); 
		return R; 
	} 
}
```
- Digraph DFS:
```java
public class DirectedDFS 
{ 
	private boolean[] marked;
	
	public DirectedDFS(Digraph G, int s) 
	{ 
		marked = new boolean[G.V()]; 
		dfs(G, s); 
	} 
	
	private void dfs(Digraph G, int v) 
	{ 
		marked[v] = true; 
		for (int w : G.adj(v)) 
			if (!marked[w]) dfs(G, w); 
	}
	
	public boolean marked(int v) 
	{    return marked[v];     } 

	// DirectedBFS code is the same as UndirectedBFS
}
```
- Find a directed cycle:
```java
public class DirectedCycle 
{ 
	private boolean[] marked; 
	private int[] edgeTo; 
	private Stack<Integer> cycle;      // vertices on a cycle (if one exists)
	private boolean[] onStack;         // vertices on recursive call stack
	
	public DirectedCycle(Digraph G) 
	{ 
		onStack = new boolean[G.V()]; 
		edgeTo = new int[G.V()]; 
		marked = new boolean[G.V()]; 
		for (int v = 0; v < G.V(); v++) 
			if (!marked[v]) dfs(G, v); 
	} 
	
	private void dfs(Digraph G, int v) 
	{ 
		onStack[v] = true; 
		marked[v] = true; 
		for (int w : G.adj(v)) 
			if (this.hasCycle()) return; 
			else if (!marked[w]) 
			{ edgeTo[w] = v; dfs(G, w); } 
			else if (onStack[w]) 
			{ 
				cycle = new Stack<Integer>(); 
				for (int x = v; x != w; x = edgeTo[x]) 
					cycle.push(x); 
				cycle.push(w); 
				cycle.push(v); 
			} 
		onStack[v] = false; 
	} 
	
	public boolean hasCycle() 
	{ return cycle != null; } 
	public Iterable<Integer> cycle() 
	{  return cycle;  }
}
```
+ Ordering in Digraph:
```java
public class DepthFirstOrder 
{ 
	private boolean[] marked;
	private Queue<Integer> pre;           // vertices in preorder
	private Queue<Integer> post;          // vertices in postorder
	private Stack<Integer> reversePost;   // vertices in reverse postorder
	
	public DepthFirstOrder(Digraph G) 
	{ 
		pre = new Queue<Integer>(); 
		post = new Queue<Integer>(); 
		reversePost = new Stack<Integer>(); 
		marked = new boolean[G.V()]; 
		for (int v = 0; v < G.V(); v++) 
			if (!marked[v]) dfs(G, v); 
	} 
	
	private void dfs(Digraph G, int v) 
	{ 
		pre.enqueue(v); marked[v] = true; 
		for (int w : G.adj(v)) 
			if (!marked[w]) dfs(G, w); 
		post.enqueue(v); 
		reversePost.push(v); 
	} 
	
	public Iterable<Integer> pre() 
	{  return pre;  } 
	public Iterable<Integer> post() 
	{  return post; } 
	public Iterable<Integer> reversePost() 
	{  return reversePost;  } 
}
```
- Kosaraju’s algorithm:
```java
public class KosarajuSCC 
{ 
	private boolean[] marked;    // reached vertices 
	private int[] id;            // component identifiers
	private int count;           // number of strong components

	public KosarajuSCC(Digraph G) 
	{ 
		marked = new boolean[G.V()]; 
		id = new int[G.V()]; 
		DepthFirstOrder order = new DepthFirstOrder(G.reverse()); 
		for (int s : order.reversePost()) 
			if (!marked[s]) 
			{ dfs(G, s); count++; } 
	} 
	private void dfs(Digraph G, int v) 
	{ 
		marked[v] = true; 
		id[v] = count; 
		for (int w : G.adj(v)) 
			if (!marked[w]) dfs(G, w); 
	} 
	
	public boolean stronglyConnected(int v, int w) 
	{ return id[v] == id[w]; } 
	public int id(int v) 
	{ return id[v]; } 
	public int count() 
	{ return count; } 
}
```
# Analysis
+ DFS marks all the vertices in a digraph reachable from a given set of sources in time proportional to the sum of the outdegrees of the vertices marked.
+ Reverse postorder in a DAG is a topological sort.
+ With DFS, we can topologically sort a DAG in time proportional to $V + E$.
+ Topological sorting and cycle detection go hand in hand, with cycle detection playing the role of a debugging tool.
+ Kosaraju’s algorithm uses preprocessing time and space proportional to $V +E$ to support constant-time strong connectivity queries in a digraph.
# Sources
+ [Princeton University: Algorithms part | | - Lecture 2 ](https://www.coursera.org/learn/algorithms-part2/lecture/QKBnh/introduction-to-digraphs)
+ Algorithms 4.2