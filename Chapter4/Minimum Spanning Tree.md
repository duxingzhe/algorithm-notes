### Minimum Spanning Tree

In this section, we consider undirected edge-weighted graph models and examine algorithms for one such problem:

Minimum spanning tree. Given an undirected edgeweighted graph, find an MST.

Definition. Recall that a spanning tree of a graph is a connected subgraph with no cycles that includes all the vertices. A minimum spanning tree (MST) of an edge-weighted graph is a spanning tree whose weight (the sum of the weights of its edges) is no larger than the weight of any other spanning tree.

##### Assumptions

Various anomalous situations, which are generally easy to handle, can arise when computing minimum spanning trees. To streamline the presentation, we adopt the following conventions:

* The graph is connected. The spanning-tree condition in our definition implies that the graph must be connected for an MST to exist. Another way to pose the problem, recalling basic properties of trees from Section 4.1, is to find a minimal-weight set of V-1 edges that connect the graph. If a graph is not connected, we can adapt our algorithms to compute the MSTs of each of its connected components, collectively known as a minimum spanning forest.
* The edge weights are not necessarily distances. Geometric intuition is sometimes beneficial in understanding algorithms, so we use examples where vertices are points in the plane and weights are distances, such as the graph on the facing page. But it is important to remember that the weights might represent time or cost or an entirely different variable and do not need to be proportional to a distance at all.
* The edge weights may be zero or negative. If the edge weights are all positive, it suffices to define the MST as the subgraph with minimal total weight that connects all the vertices, as such a subgraph must form a spanning tree. The spanning-tree condition in the definition is included so that it applies for graphs that may have zero negative edge weights.
* The edge weights are all different. If edges can have equal weights, the minimum spanning tree may not be unique. The possibility of multiple MSTs complicates the correctness proofs of some of our algorithms, so we rule out that possibility in the presentation. It turns out that this assumption is not restrictive because our algorithms work without modification in the presence of equal weights.

#### Underlying principles

To begin, we recall from Section 4.1 two of the defining properties of a tree:

* Adding an edge that connects two vertices in a tree creates a unique cycle.
* Removing an edge from a tree breaks it into two separate subtrees.

##### Cut property

This property, which we refer to as the cut property, has to do with identifying edges that must be in the MST of a given edge-weighted graph, by dividing vertices into two sets and examining edges that cross the division.

##### Definition

A c ut of a graph is a partition of its vertices into two nonempty disjoint sets. A crossing edge of a cut is an edge that connects a vertex in one set with a vertex in the other.

##### Proposition J

( Cut property) Given any cut in an edgeweighted graph, the crossing edge of minimum weight is in the MST of the graph.

##### Greedy algorithm

The cut property is the basis for the algorithms that we consider for the MST problem. Specifically, they are special cases of a general paradigm known as the greedy algorithm: apply the cut property to accept an edge as an MST edge, continuing until finding all of the MST edges.

##### Proposition K

(Greedy MST algorithm) The following method colors black all edges in the the MST of any connected edgeweighted graph with V vertices: starting with all edges colored gray, find a cut with no black edges, color its minimum-weight edge black, and continue until V-1 edges have been colored black.

#### Edge-weighted graph data type

```
public Iterable<Edge> edges(){
    Bag<Edge> b=new Bag<Edge>();
    for(int v=0;v<V;v++)
        for(Edge e: adj[v])
        if(e.other(v)>v)
            b.add(e);
    return b;
}
```

```
public class Edge implements Comparable<Edge> {
	private final int v;
	private final int w;
	private final double weight;
	
	public Edge(int v, int w, double weight) {
		this.v=v;
		this.w=w;
		this.weight=weight;
	}
	
	public double weight() {
		return weight;
	}
	
	public int either() {
		return v;
	}
	
	public int other(int vertex) {
		if(vertex==v)
			return w;
		else if(vertex==w)
			return v;
		else
			throw new RuntimeException("Inconsistent edge");
	}
	
	public int compareTo(Edge that) {
		if(this.weight()<that.weight())
			return -1;
		else if(this.weight>that.weight())
			return 1;
		else
			return 0;
	}
	
	public String toString() {
		return String.format("%d-%d %.2f", v,w,weight);
	}
}
```

```
public class EdgeWeightedGraph {
	
	private final int V;
	private int E;
	private Bag<Edge>[] adj;
	
	public EdgeWeightedGraph(int V) {
		this.V=V;
		this.E=0;
		adj=(Bag<Edge>[])new Bag[V];
		for(int v=0;v<V;v++) {
			adj[v]=new Bag<Edge>();
		}
	}
	
	public EdgeWeightedGraph(In in) {
		
	}
	
	public int V() {
		return V;
	}
	
	public int E() {
		return E;
	}
	
	public void addEdge(Edge e) {
		int v=e.either(),w=e.other(v);
		adj[v].add(e);
		adj[w].add(e);
		E++;
	}
	
	public Iterable<Edge> adj(int v){
		return adj[v];
	}
	
	public Iterable<Edge> edges()

}
```

##### Comparing edges by weight

The natural ordering
for edges in an edge-weighted graph is by weight. Accordingly, the implementation
of compareTo() is straightforward.

##### Parallel edges

As with our undirected-graph implementations, we allow parallel
edges. Alternatively, we could develop a more complicated implementation of
EdgeWeightedGraph that disallows them, perhaps keeping the minimum-weight edge
from a set of parallel edges.

##### Self-loops

We allow self-loops. However, our edges() implementation in EdgeWeightedGraph does not include self-loops even though they might be present in the input or in the data structure. This omission has no effect on our MST algorithms because no MST contains a self-loop.

#### MST API and test client

The MST of a graph G is a subgraph of G that is also a tree, so we have numerous options. Chief among them are

* A list of edges
* An edge-weighted graph
* A vertex-indexed array with parent links

##### Test client

A sample client is shown below. It reads edges from the input stream, builds an edge-weighted graph, computes the MST of that graph, prints the MST edges, and prints the total weight of the MST.

#### Prim’s algorithm

Prim’s algorithm is to attach a new edge to a single growing tree at each step. Start with any vertex as a single-vertex tree; then add V-1 edges to it, always taking next (coloring black) the minimumweight edge that connects a vertex on the tree to a vertex not yet on the tree (a crossing edge for the cut defined by tree vertices).

##### Proposition L

Prim’s algorithm computes the MST of any connected edge-weighted graph.

##### Data structures

We implement Prim’s algorithm with the aid of a few simple and familiar data structures. In particular, we represent the vertices on the tree, the edges on the tree, and the crossing edges, as follows:

* Vertices on the tree : We use a vertex-indexed boolean array marked[], where marked[v] is true if v is on the tree.
* Edges on the tree : We use one of two data structures: a queue mst to collect MST edges or a vertex-indexed array edgeTo[] of Edge objects, where edgeTo[v] is the Edge that connects v to the tree.
* Crossing edges : We use a MinPQ<Edge> priority queue that compares edges by weight.

##### Maintaining the set of crossing edges

Each time that we add an edge to the tree, we also add a vertex to the tree. To maintain the set of crossing edges, we need to add to the priority queue all edges from that vertex to any non-tree vertex (using marked[] to identify such edges). But we must do more: any edge connecting the vertex just added to a tree vertex that is already on the priority queue now becomes ineligible (it is no longer a crossing edge because it connects two tree vertices). An eager implementation of Prim’s algorithm would remove such edges from the priority queue; we first consider a simpler lazy implementation of the algorithm where we leave such edges on the priority queue, deferring the eligibility test to when we remove them.

##### Implementation

With these preparations, implementing Prim’s algorithm is straightforward, as shown in the implementation LazyPrimMST on the facing page. As with our depth-first search and breadth-first search implementations in the previous two sections, it computes the MST in the constructor so that client methods can learn properties of the MST with query methods.

##### Running time

##### Proposition M

The lazy version of Prim’s algorithm uses space proportional to E and time proportional to ![](http://latex.codecogs.com/gif.latex?ElongE) (in the worst case) to compute the MST of a connected edge-weighted graph with E edges and V vertices.

```
public class LazyPrimMST {

	private boolean[] marked;
	private Queue<Edge> mst;
	private MinPQ<Edge> pq;
	
	public LazyPrimMST(EdgeWeightedGraph G) {
		pq=new MinPQ<Edge>();
		marked=new boolean[G.V()];
		mst=new Queue<Edge>();
		
		visit(G,0);
		while(!pq.isEmpty()) {
			Edge e=pq.delMin();
			int v=e.either(), w=e.other(v);
			if(marked[v]&&marked[w])
				continue;
			mst.enqueue(e);
			if(!marked[v])
				visit(G,v);
			if(!marked[w])
				visit(G,w);
		}
	}
	
	private void visit(EdgeWeightedGrap G) {
		marked[v]=true;
		for(Edge e: G.adj(v)) {
			if(!marked[e.other(v)])
				pq.insert(e);
		}
	}
	
	public Iterable<Edge> edges(){
		return mst;
	}
	
	public double weight()
}
```

#### Eager version of Prim’s algorithm

To improve the LazyPrimMST, we might try to delete ineligible edges from the priority queue, so that the priority queue contains only the crossing edges between tree vertices and non-tree vertices. But we can eliminate even more edges. The key is to note that our only interest is in the minimal edge from each non-tree vertex to a tree vertex. When we add a vertex v to the tree, the only possible change with respect to each nontree vertex w is that adding v brings w closer than before to the tree. In short, we do not need to keep on the priority queue all of the edges from w to tree vertices—we just need to keep track of the minimum-weight edge and check whether the addition of v to the tree necessitates that we update that minimum (because of an edge v-w that has lower weight), which we can do as we process each edge in v’s adjacency list.

PrimMST (Algorithm 4.7 on page 622) implements Prim’s algorithm using our index priority queue data type from Section 2.4 (see page 320). It replaces the data structures marked[] and mst[] in LazyPrimMST by two vertex-indexed arrays edgeTo[] and distTo[], which have the following properties:

* If v is not on the tree but has at least one edge connecting it to the tree, then edgeTo[v] is the shortest edge connecting v to the tree, and distTo[v] is the weight of that edge.
* All such vertices v are maintained on the index priority queue, as an index v associated with the weight of edgeTo[v].

The key implications of these properties is that the minimum key on the priority queue is the weight of the minimal-weight crossing edge, and its associated vertex v is the next to add to the tree.

```
public class PrimMST {
	private Edge[] edgeTo;
	private double[] distTo;
	private boolean[] marked;
	private IndexMinPQ<Double> pq;
	
	public PrimMST(EdgeWeightedGraph G) {
		edgeTo=new Edge[G.V()];
		distTo=new double[G.V()];
		marked=new boolean[G.V()];
		for(int v=0;v<G.V();v++) {
			distTo[v]=Double.POSITIVE_INFINITY;
		}
		pq=new IndexMinPQ<Double>(G.V());
		
		distTo[0]=0.0;
		pq.insert(0,0.0);
		while(!pq.isEmpty()) {
			visit(G, pq.delMin());
		}
	}
	
	private void visit(EdgeWeightedGraph G) {
		marked[v]=true;
		for(Edge e:G.adj(v)) {
			int w=e.other(v);
			if(marked[w])
				continue;
			if(e.weight()<distTo[w]) {
				edgeTo[w]=e;
				distTo[w]=e.weight();
				if(pq.contains(w))
					pq.change(w,distTo[w]);
				else
					pq.insert(w,distTo[w]);
			}
		}
	}
	
	public Iterable<Edge> edges()
	public double weight()
}
```

##### Proposition N

The eager version of P rim’s algorithm uses extra space proportional to V and time proportional to E log V (in the worst case) to compute the MST of a connected edge-weighted graph with E edges and V vertices.

#### Kruskal’s algorithm

The second MST algorithm that we consider in detail is to process the edges in order of their weight values (smallest to largest), taking for the MST (coloring black) each edge that does not form a cycle with edges previously added, stopping after adding V-1 edges have been taken. The black edges form a forest of trees that evolves gradually into a single tree, the MST.

##### Proposition O

Kruskal’s algorithm computes the MST of any edge-weighted connected graph.

##### Proposition N (continued)

Kruskal’s algorithm uses space proportional to E and time proportional to ![](http://latex.codecogs.com/gif.latex?ElongE) (in the worst case) to compute the MST of an edgeweighted connected graph with E edges and V vertices.

```
public class KruskalMST {
	private Queue<Edge> mst;
	
	public KruskalMST(EdgeWeightedGraph G) {
		mst=new Queue<Edge>();
		MinPQ<Edge> pq=new MinPQ<Edge>(G.edges());
		UF uf=new UF(G.V());
		
		while(!pq.isEmpty()&&mst.size()<G.V()-1) {
			Edge e=pq.delMin();
			int v=e.either(),w=e.other(v);
			if(uf.connected(v,w))
				continue;
			uf.union(v,w);
			mst.enqueue(e);
		}
	}
	
	public Iterable<Edge> edges(){
		return mst;
	}
	
	public double weight()
}
```

#### Perspective

The MST problem is one of the most heavily studied problems that we encounter in this book. Basic approaches to solving it were invented long before the development of modern data structures and modern techniques for analyzing the performance of algorithms, at a time when finding the MST of a graph that contained, say, thousands of edges was a daunting task.

<table>
    <tr>
        <th rowspan="2">algorithm</th>
        <th colspan="2">worst-case cost(after N inserts)</th></th>
    </tr>
    <tr>
        <th>space</th>
        <th>time</th>
    </tr>
    <tr>
        <th>lazyPrim</th>
        <th>E</th>
		<th><img src=http://latex.codecogs.com/gif.latex?ElogE></img>
    </tr>
    <tr>
        <th>eager Prim</th>
        <th>V</th>
        <th><img src=http://latex.codecogs.com/gif.latex?ElogV></th>
    </tr>
	<tr>
        <th>Kruskal</th>
        <th>E</th>
        <th><img src=http://latex.codecogs.com/gif.latex?ElogE></th>
    </tr>
	<tr>
        <th>Fredman-Tarjan</th>
        <th>V</th>
        <th><img src=http://latex.codecogs.com/gif.latex?E+VlogV></th>
    </tr>
	<tr>
        <th>Chazelle</th>
        <th>V</th>
        <th>very, very nearly, but not quite E</th>
    </tr>
	<tr>
        <th>impossible?</th>
        <th>V</th>
        <th>E?</th>
    </tr>
</table>
Performance characteristics of MST algorithms

##### A linear-time algorithm?

On the one hand, no theoretical results have been developed that deny the existence of an MST algorithm that is guaranteed to run in linear time for all graphs. On the other hand, the goal of developing algorithms for computing the MST of sparse graphs in linear time remains elusive.