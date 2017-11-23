### Shortest Path

The possibility of one-way roads means that we will need to consider edge-weighted digraphs. In this model, the problem is easy to formulate:

Find the lowest-cost way to get from one vertex to another.

<table>
    <tr>
        <th >application</th>
        <th >vertex</th>
        <th >edge</th>
    </tr>
    <tr>
        <th>map</th>
        <th>intersection</th>
        <th>road</th>
    </tr>
    <tr>
        <th>network</th>
        <th>router</th>
        <th>connection</th>
    </tr>
    <tr>
        <th>schedule</th>
        <th>job</th>
        <th>precedence constraint</th>
    </tr>
    <tr>
        <th>arbitrage</th>
        <th>currency</th>
        <th>exchange rate</th>
    </tr>
</table>
Typical shortest-paths applications

##### Definition

A shortest path from vertex s to vertex t in an edge-weighted digraph is a directed path from s to t with the property that no other such path has a lower weight.

Thus, in this section, we consider classic algorithms for the following problem:

Single-source shortest paths. Given an edge-weighted digraph and a source vertex s, support queries of the form Is there a directed path from s to a given target vertex t? If so, find a shortest such path (one whose total weight is minimal).

Properties of shortest paths The basic definition of the shortest-paths problem is succinct, but its brevity masks several points worth examining before we begin to formulate algorithms and data structures for solving it:

* Paths are directed. A shortest path must respect the direction of its edges.
* The weights are not necessarily distances. Geometric intuition can be helpful in understanding algorithms, so we use examples where vertices are points in the plane and weights are Euclidean distances, such as the digraph on the facing page. But the weights might represent time or cost or an entirely different variable and do not need to be proportional to a distance at all. We are emphasizing this point by using mixed-metaphor terminology where we refer to a shortest path of minimal weight or cost.
* Not all vertices need be reachable. If t is not reachable from s, there is no path at all, and therefore there is no shortest path from s to t. For simplicity, our small running example is strongly connected (every vertex is reachable from every other vertex).
* Negative weights introduce complications. For the moment, we assume that edge weights are positive (or zero). The surprising impact of negative weights is a major focus of the last part of this section.
* Shortest paths are normally simple. Our algorithms ignore zero-weight edges that
form cycles, so that the shortest paths they find have no cycles.
* Shortest paths are not necessarily unique. There may be multiple paths of the low-est weight from one vertex to another; we are content to find any one of them.
* Parallel edges and self-loops may be present. Only the lowest-weight among a set of parallel edges will play a role, and no shortest path contains a self-loop (except possibly one of zero weight, which we ignore).

In the text, we implicitly assume that parallel edges are not present for convenience in using the notation v->w to refer unambiguously to the edge from v to w, but our code handles them without difficulty.

#### Shortest-paths tree

We focus on the single-source shortest-paths problem, where we are given a source vertex s. The result of the computation is a tree known as the shortest-paths tree (SPT), which gives a shortest path from s to every vertex reachable from s.

##### Definition

Given an edge-weighted digraph and a designated vertex s, a shortest-paths tree for a source s is a subgraph containing s and all the vertices reachable from s that forms a directed tree rooted at s such that every tree path is a shortest path in the digraph.

#### Edge-weighted digraph data types

```
public class DirectedEdge {

	private final int v;
	private final int w;
	private final double weight;
	
	public DirectedEdge(int v,int w, double weight) {
		this.v=v;
		this.w=w;
		this.weight=weight;
	}
	
	public double weight() {
		return weight;
	}
	
	public int from() {
		return v;
	}
	
	public int to() {
		return w;
	}
	
	@Override
	public String toString() {
		return String.format("%d->%d %.2f", v,w,weight);
	}
}
```

```
public class EdgeWeightedDigraph {

	private final int V;
	private int E;
	private Bag<DirectedEdge>[] adj;
	
	public EdgeWeightedDigraph(int V) {
		this.V=V;
		this.E=0;
		adj=(Bag<DirectedEdge>[])new Bag[V];
		for(int v=0;v<V;v++) {
			adj[v]=new Bag<DirectedEdge>();
		}
	}
	
	public EdgeWeightedDigraph(In in) {
		
	}
	
	public int V() {
		return V;
	}
	
	public int E() {
		return E;
	}
	
	public void addEdge(DirectedEdge e) {
		adj[e.from()].add(e);
		E++;
	}
	
	public Iterable<Edge> adj(int v){
		return adj[v];
	}
	
	public Iterable<DirectedEdge> edge(){
		Bag<DirectedEdge> bag=new Bag<DirectedEdge>();
		for(nt v=0;v<V;v++) {
			for(DirectedEdge e:adj[v])){
				bag.add(e);
			}
		}
		return bag;
	}
}
```

Data structures for shortest paths. The data structures that we need to represent shortest paths are straightforward:

* Edges on the shortest-paths tree : As for DFS, BFS, and Prim’s algorithm, we use a parent-edge representation in the form of a vertex-indexed array edgeTo[] of DirectedEdge objects, where edgeTo[v] is edge that connects v to its parent in the tree (the last edge on a shortest path from s to v).
* Distance to the source : We use a vertex-indexed array distTo[] such that distTo[v] is the length of the shortest known path from s to v.

##### Edge relaxation

Our shortest-paths implementations are based on a simple operation known as relaxation. We start knowing only the graph’s edges and weights, with the distTo[] entry for the source initialized to 0 and all of the other distTo[] entries initialized to Double.POSITIVE_INFINITY. As an algorithm proceeds, it gathers information about the shortest paths that connect the source to each vertex encountered in our edgeTo[] and distTo[] data structures. By updating this information when we encounter edges, we can make new inferences about shortest paths. Specifically, we use edge relaxation, defined as follows: to relax an edge v->w means to test whether the best known way from s to w is to go from s to v, then take the edge from v to w, and, if so, update our data structures to indicate that to be the case.

```
private void relax(DirectedEdge e) {
    int v=e.from(),w=.to();
    if(distTo[w]>distTo[v]+e.weight()) {
        distTo[w]=dist[v]+e.weight();
        edgeTo[w]=e;
    }
}
```

##### Vertex relaxation

All of our implementations actually relax all the edges pointing from a given vertex as shown in the (overloaded) implementation of relax() below. Note that any edge from a vertex whose distTo[v] entry is finite to a vertex whose distTo[] entry is infinite is eligible and will be added to edgeTo[] if relaxed. In particular, some edge leaving the source is the first to be added to edgeTo[].

```
private void relax(EdgeWeightedDigrpah G, int v) {
    for(DirectedEdge e:G.adj(v)) {
        int w=e.to();
        if(distTo[w]>distTo[v]+e.weight()) {
            distTo[w]=distTo[v]+e.weight();
            edgeTo[w]=e;
        }
    }
}
```

##### Client query methods

In a manner similar to our implementations for pathfinding APIs in Section 4.1, the edgeTo[] and distTo[] data structures directly support the pathTo(), hasPathTo(), and distTo() client query methods, as shown below. This code is included in all of our shortest-paths implementations.

```
public double distTo(int v) {
		return distTo[v];
}

public boolean hasPathTo(int v) {
    return distTo[v]<Dobule.POSITIVE_INFINITY;
}

public Iterable<DirectedEdge> pathTo(int v){
    if(!hasPathTo(v))
        return null;
    Stack<DirectedEdge> path=new Stack<DirectedEdge>();
    for(DirectedEdge e=edgeTo[v];e!=null;e=edgeTo[e.from()]) {
        path.push(e);
    }
    return path;
}
```