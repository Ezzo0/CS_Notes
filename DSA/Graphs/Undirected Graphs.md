# Explanation
+ Undirected [[Graphs]] have edges that do not have a direction.
+ #### Graph representation (data structure)
	+ An adjacency matrix, where we maintain a V-by-V boolean array, with the entry in row v and column w defined to be true if there is an edge adjacent to both vertex v and vertex w in the graph, and to be false otherwise. This representation fails on the first count— graphs with millions of vertices are common and the space cost for the $V^2$ boolean values needed is prohibitive.
	+ An array of edges, using an Edge class with two instance variables of type int. This direct representation is simple, but it fails on the second count— implementing adj() would involve examining all the edges in the graph.
	+ An array of adjacency lists, where we maintain a vertex-indexed array of lists of the vertices adjacent to each vertex. This data structure satisfies the requirements of space and time.
	+ ##### Adjacency-lists data structure
		+ The standard graph representation for graphs that are not dense is called the adjacency-lists data structure.
		+ In Adjacency-lists we keep track of all the vertices adjacent to each vertex on a linked list that is associated with that vertex. 
		+ An array of lists is maintained so that, given a vertex, we can immediately access its list. 
		+ To implement lists, we use our [[Bags]] ADT with a linked-list implementation, so that we can add new edges in constant time and iterate through adjacent vertices in constant time per adjacent vertex. 
		+ To add an edge connecting v and w, we add w to v’s adjacency list and v to w’s adjacency list. 
		![[Adjacency-lists representation (undirected graph).png|250]]
+ #### Searching
	+ ###### Depth-First-Search
		+ To search a graph, invoke a recursive method that visits vertices. To visit a vertex:
			+ Mark it as having been visited.
			+ Visit (recursively) all the vertices that are adjacent to it and that have not yet been marked.
		+ DFS marks all the vertices connected to a given source in time proportional to the sum of their degrees.
		 ![[Trace of depth-first search to find vertices connected to 0.png|300]]
	+ ###### Breadth-First-Search
		+ For any vertex v reachable from s, BFS computes a shortest path from s to v.
		+ It is based on maintaining a queue of all vertices that have been marked but whose adjacency lists have not been checked. We put the source vertex on the queue, then perform the following steps until the queue is empty:
			+ Take the next vertex v from the queue and mark it.
			+ Put onto the queue all unmarked vertices that are adjacent to v.
# Implementation
+ Adjacency-lists Graph:
```java
public class Graph 
{
	private final int V;                   // number of vertices
	private int E;                         // number of edges
	private Bag<Integer>[] adj;            // adjacency lists
	
	public Graph(int V)
	{
		this.V = V; 
		this.E = 0;
		adj = (Bag<Integer>[]) new Bag[V];               // Create array of lists.
		for (int v = 0; v < V; v++)                      // Initialize all lists
			adj[v] = new Bag<Integer>();                 // to empty.
	}
	
	public Graph(In in)
	{
		this(in.readInt());               // Read V and construct this graph.
		int E = in.readInt();             // Read E.
		for (int i = 0; i < E; i++)
		{   // Add an edge.
			int v = in.readInt();         // Read a vertex,
			int w = in.readInt();         // read another vertex,
			addEdge(v, w);                // and add edge connecting them.
		} 
	} 
	
	public int V() { return V; } 
	public int E() { return E; } 
	
	public void addEdge(int v, int w) 
	{
		adj[v].add(w);                  // Add w to v’s list. 
		adj[w].add(v);                  // Add v to w’s list.
		E++; 
	} 
	
	public Iterable<Integer> adj(int v)
	{ return adj[v]; } 
}
```
+ DFS (using [[Stack]] for iteration):
```java
public class DepthFirstSearch 
{ 
	private boolean[] marked;             // Has dfs() been called for this vertex?
	private int[] edgeTo;                 // last vertex on known path to this vertex
	private int s;                        // source
	
	public DepthFirstSearch(Graph G, int s) 
	{ 
		marked = new boolean[G.V()]; 
		edgeTo = new int[G.V()];
		this.s = s;
		dfs(G, s); 
	} 
	private void dfs(Graph G, int v) 
	{ 
		marked[v] = true; 
		for (int w : G.adj(v)) 
			if (!marked[w]) 
			{
				edgeTo[w] = v;
				dfs(G, w);
			} 
	} 
	
	public boolean hasPathTo(int v) { return marked[v]; } 
	
	public Iterable<Integer> pathTo(int v) 
	{ 
		if (!hasPathTo(v)) return null; 
		Stack<Integer> path = new Stack<Integer>(); 
		for (int x = v; x != s; x = edgeTo[x]) 
			path.push(x); 
		path.push(s); 
		return path; 
	} 
}
```
+ BFS:
```java
public class BreadthFirstPaths 
{
	private boolean[] marked;            // Is a shortest path to this vertex known?
	private int[] edgeTo;                // last vertex on known path to this vertex
	private final int s;                 // source 
	
	public BreadthFirstPaths(Graph G, int s) 
	{ 
		marked = new boolean[G.V()]; 
		edgeTo = new int[G.V()]; 
		this.s = s; 
		bfs(G, s); 
	} 
	private void bfs(Graph G, int s) 
	{ 
		Queue<Integer> queue = new Queue<Integer>();
		marked[s] = true;          // Mark the source
		queue.enqueue(s);          // and put it on the queue.    
		while (!q.isEmpty())
		{
			int v = queue.dequeue();      // Remove next vertex from the queue. 
			for (int w : G.adj(v))
				if (!marked[w])           // For every unmarked adjacent vertex,
				{ 
					edgeTo[w] = v;        // save last edge on a shortest path,
					marked[w] = true;     // mark it because path is known,
					queue.enqueue(w);     // and add it to the queue.
				}
		}
	} 
	
	public boolean hasPathTo(int v) 
	{ return marked[v]; } 
	
	public Iterable<Integer> pathTo(int v)
	// Same as for DFS.
}
```
+ Connected Components:
```java
public class CC 
{ 
	private boolean[] marked; 
	private int[] id; 
	private int count; 
	
	public CC(Graph G) 
	{ 
		marked = new boolean[G.V()]; 
		id = new int[G.V()]; 
		for (int s = 0; s < G.V(); s++) 
			if (!marked[s]) 
			{ 
				dfs(G, s); 
				count++; 
			} 
	} 
	
	private void dfs(Graph G, int v) 
	{ 
		marked[v] = true; 
		id[v] = count; 
		for (int w : G.adj(v)) 
			if (!marked[w]) 
				dfs(G, w); 
	} 
	
	public boolean connected(int v, int w) 
	{  return id[v] == id[w];  } 
	
	public int id(int v) 
	{  return id[v];  } 
	public int count() 
	{  return count;  } 
}
```
- Is acyclic:
```java
public class Cycle 
{ 
	private boolean[] marked; 
	private boolean hasCycle; 
	
	public Cycle(Graph G) 
	{ 
		marked = new boolean[G.V()]; 
		for (int s = 0; s < G.V(); s++) 
			if (!marked[s]) 
				dfs(G, s, s); 
	} 
	
	private void dfs(Graph G, int v, int u) 
	{ 
		marked[v] = true; 
		for (int w : G.adj(v)) 
			if (!marked[w]) 
				dfs(G, w, v); 
			else if (w != u)  hasCycle = true; 
	} 
	
	public boolean hasCycle() { return hasCycle; } 
}
```
- Is bipartite:
```java
public class TwoColor 
{ 
	private boolean[] marked; 
	private boolean[] color; 
	private boolean isTwoColorable = true; 
	
	public TwoColor(Graph G) 
	{ 
		marked = new boolean[G.V()]; 
		color = new boolean[G.V()]; 
		for (int s = 0; s < G.V(); s++) 
			if (!marked[s]) dfs(G, s); 
	} 
	
	private void dfs(Graph G, int v) 
	{ 
		marked[v] = true; 
		for (int w : G.adj(v)) 
			if (!marked[w]) 
			{ 
				color[w] = !color[v]; 
				dfs(G, w); 
			} 
			else if (color[w] == color[v]) isTwoColorable = false; 
	} 
	
	public boolean isBipartite() 
	{  return isTwoColorable;  } 
}
```
# Analysis
+ #### Graph representation (data structure)
	+ This Graph implementation achieves the following performance characteristics:
		+ Space usage proportional to $V + E$.
		+ Constant time to add an edge
		+ Time proportional to the degree of v to iterate through vertices adjacent to v (constant time per adjacent vertex processed).
+ #### Searching
	+ ###### Depth-First-Search
		+ DFS traverses each edge in the graph twice.
		+ DFS allows us to provide clients with a path from a given source to any marked vertex in time proportional its length.
		+ DFS paths tend to be long and winding.
	+ ###### Breadth-First-Search
		+ BFS takes time proportional to $V+E$ in the worst case.
		+ BFS marks all the vertices connected to **s** in time proportional to the sum of their degrees. 
		+ If the graph is connected, the sum is equal to the sum of the degrees of all the vertices, or 2E.
		+ BFS paths are short and direct.

# Sources
+ [Princeton University: Algorithms part | | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part2/lecture/4ZE6G/graph-api)
+ Algorithms 4.1