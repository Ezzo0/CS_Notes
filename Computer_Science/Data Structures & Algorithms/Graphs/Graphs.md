# Topics
- [[Undirected Graphs]]
- [[Directed Graphs]]
- [[Minimum Spanning Trees]]
# Explanation
+ A graph is a set of vertices and a collection of edges that each connect a pair of vertices.
 ![[Two drawings of the same graph.png|200]]
+ When there is an edge connecting two vertices, we say that the vertices are adjacent to one another and that the edge is incident to both vertices. 
+ The degree of a vertex is the number of edges incident to it.
+ A path in a graph is a sequence of vertices connected by edges. 
+ A simple path is one with no repeated vertices. 
+ A cycle is a path with at least one edge whose first and last vertices are the same. 
+ A simple cycle is a cycle with no repeated edges or vertices (except the requisite repetition of the first and last vertices). 
+ The length of a path or a cycle is its number of edges.
+ A graph is connected if there is a path from every vertex to every other vertex in the graph.
![[Anatomy of a graph.png|250]]
+ An acyclic graph is a graph with no cycles.
+ A tree is an acyclic connected graph.
+ A spanning tree of a connected graph is a subgraph that contains all of that graph’s vertices and is a single tree.
![[A tree.png|300]]![[A spanning forest.png|200]]
+ A bipartite graph is a graph whose vertices we can divide into two sets such that all edges connect a vertex in one set with a vertex in the other set.
![[A bipartite graph.png|200]]
# Sources
+ [Princeton University: Algorithms part | | - Lecture 1 ](https://www.coursera.org/learn/algorithms-part2/lecture/dKTI4/introduction-to-graphs)
+ Algorithms 4.1