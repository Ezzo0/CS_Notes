# Explanation
- An edge-weighted [[Graphs]] are graph models where we associate weights or costs with each edge.
- Spanning tree of a graph is a connected subgraph with no cycles that includes all the vertices.
- A minimum spanning tree (MST ) of an edge-weighted graph is a spanning tree whose weight (the sum of the weights of its edges) is no larger than the weight of any other spanning tree.
 ![[An edge-weighted graph and its MST.png|300]]
- A cut of a graph is a partition of its vertices into two nonempty disjoint sets. 
- A crossing edge of a cut is an edge that connects a vertex in one set with a vertex in the other.
 ![[Cut property.png|300]]
- #### Greedy Algorithm
	- The greedy method colors black all edges in the the MST of any connected edge weighted graph with V vertices: 
		1. Starting with all edges colored gray. 
		2. Find a cut with no black edges.
		3. Color its minimum-weight edge black.
		4. Continue until $V - 1$ edges have been colored black.
	- The following diagram is a typical trace of the greedy algorithm. 
	- Each drawing depicts a cut and identifies the minimum-weight edge in the cut (thick red) that is added to the MST by the algorithm.
	 ![[Greedy MST algorithm_1.png|200]] ![[Greedy MST algorithm_2.png| 200]] ![[Greedy MST algorithm_3.png|200]]
- #### Prim’s algorithm
	- First MST method, known as Prim’s algorithm, is to attach a new edge to a single growing tree at each step. 
	- Start with any vertex as a single-vertex tree; then add $V - 1$ edges to it, always taking next (coloring black) the minimumweight edge that connects a vertex on the tree to a vertex not yet on the tree (a crossing edge for the cut defined by tree vertices).
	 ![[Prim’s MST algorithm.png|250]]
- #### Eager version of Prim’s algorithm
	- To improve the LazyPrimMST, we might try to delete ineligible edges from the priority queue, so that the priority queue contains only the crossing edges between tree vertices and non-tree vertices.
	- In short, we do not need to keep on the priority queue all of the edges from w to tree vertices—we just need to keep track of the minimum-weight edge and check whether the addition of v to the tree necessitates that we update that minimum (because of an edge v-w that has lower weight), which we can do as we process each edge in v’s adjacency list.
	 ![[Eager version of Prim’s algorithm.png|250]]
- #### Kruskal’s algorithm
	- Kruskal’s algorithm process the edges in order of their weight values (smallest to largest), taking for the MST each edge that does not form a cycle with edges previously added, stopping after adding $V - 1$ edges have been taken.
	 ![[Trace of Kruskal’s algorithm_1.png|250]] ![[Trace of Kruskal’s algorithm_2.png|250]]
	- [[Priority Queues]] are used to consider the edges in order by weight.
	- A [[Union-Find]] data structure is used to identify those that cause cycles.
	- [[Queues]] are used to collect the MST edges.
# Implementation
- **_Weighted edge data type_**
```java
public class Edge implements Comparable<Edge>
{
	private final int v;                    // one vertex
	private final int w;                    // the other vertex
	private final double weight;            // edge weight
	
	public Edge(int v, int w, double weight)
	{
		this.v = v;
		this.w = w;
		this.weight = weight;
	}

	public double weight()
	{ return weight; }
	
	public int either()
	{ return v; }
	
	public int other(int vertex)
	{
		if (vertex == v) return w;
		else if (vertex == w) return v;
		else throw new RuntimeException("Inconsistent edge");
	}
	
	public int compareTo(Edge that)
	{
		if (this.weight() < that.weight()) return -1;
		else if (this.weight() > that.weight()) return +1;
		else return 0;
	}
	
	public String toString()
	{ return String.format("%d-%d %.2f", v, w, weight); }
}
```
- **_Edge-weighted graph data type_**
```java
public class EdgeWeightedGraph
{
	private final int V;               // number of vertices
	private int E;                     // number of edges
	private Bag<Edge>[] adj;           // adjacency lists
	
	public EdgeWeightedGraph(int V)
	{
		this.V = V;
		this.E = 0;
		adj = (Bag<Edge>[]) new Bag[V];
		for (int v = 0; v < V; v++)
			adj[v] = new Bag<Edge>();
	}
	
	public int V() { return V; }
	public int E() { return E; }
	
	public void addEdge(Edge e)
	{
		int v = e.either(), w = e.other(v);
		adj[v].add(e);
		adj[w].add(e);
		E++;
	}
	
	public Iterable<Edge> adj(int v)
	{ return adj[v]; }
	
	public Iterable<Edge> edges() 
	{ 
		Bag<Edge> b = new Bag<Edge>(); 
		for (int v = 0; v < V; v++) 
			for (Edge e : adj[v]) 
				if (e.other(v) > v) b.add(e); 
		return b; 
	}
}
```
- **_Lazy version of Prim’s MST algorithm_**
```java
public class LazyPrimMST 
{ 
	private boolean[] marked;    // MST vertices 
	private Queue<Edge> mst;     // MST edges 
	private MinPQ<Edge> pq;      // crossing (and ineligible) edges 
	
	public LazyPrimMST(EdgeWeightedGraph G) 
	{ 
		pq = new MinPQ<Edge>(); 
		marked = new boolean[G.V()]; 
		mst = new Queue<Edge>();
		visit(G, 0);            // assumes G is connected (see Exercise 4.3.22)
		while (!pq.isEmpty())
		{
			Edge e = pq.delMin();                          // Get lowest-weight
			int v = e.either(), w = e.other(v);            //edge from pq.
			if (marked[v] && marked[w]) continue;          // Skip if ineligible.
			mst.enqueue(e);                                // Add edge to tree.
			if (!marked[v]) visit(G, v);                   // Add vertex to tree 
			if (!marked[w]) visit(G, w);                   // (either v or w).
		}
	}
	
	private void visit(EdgeWeightedGraph G, int v)
	{   // Mark v and add to pq all edges from v to unmarked vertices.
		marked[v] = true; 
		for (Edge e : G.adj(v)) 
			if (!marked[e.other(v)]) pq.insert(e); 
	} 
	
	public Iterable<Edge> edges() 
	{ return mst; }
	
}
```
- **_Eager version of Prim’s MST algorithm_**
```java
public class PrimMST 
{
	private Edge[] edgeTo;              // shortest edge from tree vertex
	private double[] distTo;            // distTo[w] = edgeTo[w].weight()
	private boolean[] marked;           // true if v on tree
	private IndexMinPQ<Double> pq;      // eligible crossing edges
	
	public PrimMST(EdgeWeightedGraph G) 
	{ 
		edgeTo = new Edge[G.V()]; 
		distTo = new double[G.V()]; 
		marked = new boolean[G.V()]; 
		for (int v = 0; v < G.V(); v++) 
			distTo[v] = Double.POSITIVE_INFINITY; 
		pq = new IndexMinPQ<Double>(G.V()); 
		distTo[0] = 0.0;
		pq.insert(0, 0.0);              // Initialize pq with 0, weight 0.
		while (!pq.isEmpty()) 
			visit(G, pq.delMin());      // Add closest vertex to tree.
	}
	
	private void visit(EdgeWeightedGraph G, int v)
	{   // Add v to tree; update data structures.
		marked[v] = true; 
		for (Edge e : G.adj(v)) 
		{ 
			int w = e.other(v);
			if (marked[w]) continue; 
			if (e.weight() < distTo[w])  // v-w is ineligible.
			{   // Edge e is new best connection from tree to w.|
				edgeTo[w] = e; 
				distTo[w] = e.weight(); 
				if (pq.contains(w)) pq.change(w, distTo[w]); 
				else                pq.insert(w, distTo[w]); 
			} 
		}
	}
}
```
- **_Kruskal’s MST algorithm_**
```java
public class KruskalMST
{
	private Queue<Edge> mst;
	
	public KruskalMST(EdgeWeightedGraph G)
	{
		mst = new Queue<Edge>();
		MinPQ<Edge> pq = new MinPQ<Edge>(G.edges());
		UF uf = new UF(G.V());
		while (!pq.isEmpty() && mst.size() < G.V()-1)
		{
			Edge e = pq.delMin();                   // Get min weight edge on pq
			int v = e.either(), w = e.other(v);     // and its vertices.
			if (uf.connected(v, w)) continue;       // Ignore ineligible edges.
			uf.union(v, w);                         // Merge components.
			mst.enqueue(e);                         // Add edge to mst.
		}
	}
	
	public Iterable<Edge> edges()
	{ return mst; }
}
```
# Analysis
- ###### Lazy version of Prim’s algorithm
	- The lazy version of Prim’s algorithm uses space proportional to $E$ and time proportional to $E log E$(in the worst case) to compute the MST of a connected edge-weighted graph with E edges and V vertices.
- ###### Eager version of Prim’s algorithm
	- The eager version of Prim’s algorithm uses extra space proportional to $V$ and time proportional to $E log V$ (in the worst case) to compute the MST of a connected edge weighted graph with $E$ edges and $V$ vertices.
- ###### Kruskal’s algorithm
	- Kruskal’s algorithm uses space proportional to $E$ and time proportional to $E log E$ (in the worst case) to compute the MST of an edge weighted connected graph with $E$ edges and $V$ vertices.
	- As with Prim’s algorithm the cost bound is conservative, since the algorithm terminates after finding the $V - 1$ MST edges. 
	- The order of growth of the actual cost is $E + E_0 log E$, where $E_0$ is the number of edges whose weight is less than the weight of the MST edge with the highest weight. 
	- Despite this advantage, Kruskal’s algorithm is generally slower than Prim’s algorithm because it has to do a `connected()` operation for each edge, in addition to the priority-queue operations that both algorithms do for each edge processed.
# Sources
+ [Princeton University: Algorithms part | | - Lecture 3 ](https://www.coursera.org/learn/algorithms-part2/lecture/lEPxc/introduction-to-msts)
+ Algorithms 4.3
