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

We allow self-loops. However, our edges() implementation in
EdgeWeightedGraph does not include self-loops even though they might be present
in the input or in the data structure. This omission has no effect on our MST algorithms
because no MST contains a self-loop.