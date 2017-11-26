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

#### Theoretical basis for shortest-paths algorithms

Edge relaxation is an easy-to-implement fundamental operation that provides a practical basis for our shortestpaths implementations. It also provides a theoretical basis for understanding the algorithms and an opportunity for us to do our algorithm correctness proofs at the outset.

##### Optimality conditions

##### Proposition P ( Shortest-paths optimality conditions)

Let G be an edge-weighted digraph, with s a source vertex in G and distTo[] a vertex-indexed array of path lengths in G such that, for all v reachable from s, the value of distTo[v] is the length of some path from s to v with distTo[v] equal to infinity for all v not reachable from s. These values are the lengths of shortest paths if and only if they satisfy distTo[w] <= distTo[v] + e.weight() for each edge e from v to w (or, in other words, no edge is eligible).

##### Certification

An important practical consequence of Proposition P is its applicability to certification.

##### Generic algorithm

The optimality conditions lead immediately to a generic algorithm that encompasses all of the shortest-paths algorithms that we consider. For the moment, we restrict attention to nonnegative weights.

##### Proposition Q ( Generic shortest-paths algorithm)

Initialize distTo[s] to 0 and all other distTo[] values to infinity, and proceed as follows:

Relax any edge in G, continuing until no edge is eligible.

For all vertices w reachable from s, the value of distTo[w] after this computation
is the length of a shortest path from s to w (and the value of edgeTo[] is the last
edge on that path).

#### Dijkstra’s algorithm

Dijkstra’s algorithm is an analogous scheme to compute an SPT. We begin by initializing dist[s] to 0 and all other distTo[] entries to positive infinity, then we relax and add to the tree a non-tree vertex with the lowest distTo[] value, continuing until all vertices are on the tree or no non-tree vertex has a finite distTo[] value.

##### Proposition R

Dijkstra’s algorithm solves the single-source shortest-paths problem in edge-weighted digraphs with nonnegative weights.

##### Data structures

To implement Dijkstra’s algorithm we add to our distTo[] and edgeTo[] data structures an index priority queue pq to keep track of vertices that are candidates for being the next to be relaxed. Recall that an IndexMinPQ allows us to associate indices with keys (priorities) and to remove and return the index corresponding to the lowest key.

##### Alternative viewpoint

Another way to understand the dynamics of the algorithm derives from the proof, diagrammed at left: we have the invariant that distTo[] entries for tree vertices are shortest-paths distances and for each vertex w on the priority queue, distTo[w] is the weight of a shortest path from s to w that uses only intermediate vertices in the tree and ends in the crossing edge edgeTo[w]. The distTo[] entry for the vertex with the smallest priority is a shortestpath weight, not smaller than the shortest-path weight to any vertex already relaxed, and not larger than the shortest-path weight to any vertex not yet relaxed. That vertex is next to be relaxed. Reachable vertices are relaxed in order of the weight of their shortest path from s.

##### Variants

Our implementation of Dijkstra’s algorithm, with suitable modifications, is effective for solving other versions of the problem, such as the following:

Single-source shortest paths in undirected graphs. Given an edge-weighted undirected graph and a source vertex s, support queries of the form Is there a path from s to a given target vertex v? If so, find a shortest such path (one whose total weight is minimal).

```
public class DijkstraSP {

	private DirectedEdge[] edgeTo;
	private double[] distTo;
	private IndexMinPQ<Double> pq;
	
	public DijkstraSP(EdgeWeightedDigraph G,int s) {
		edgeTo=new DirectedEdge[G.V()];
		distTo=new double[G.V()];
		pq=new IndexMinPQ<Double>(G.V());
		
		for(int v=0;v<G.V();v++) {
			distTo[v]=Double.POSITIVE_INFINITY;
		}
		distTo[s]=0.0;
		while(!pq.isEmpty()) {
			relax(G,pq.delMin());
		}
	}
	
	private void relax(EdgeWeightedDigraph G, int v) {
		for(DirectedEdge e: G.adj(v)) {
			int w=e.to();
			if(distTo[w]>distTo[v]+e.weight()) {
				distTo[w]=distTo[v]+e.weight();
				edgeTo[w]=e;
				if(pq.contains(w))
					pq.change(w,distTo[w]);
				else
					pq.insert(w,distTo[w]);
			}
		}
		
	}
	
	public double distTo(int v)
	public boolean hasPathTo(int v)
	public Iterable<Edge> pathTo(int v)
}
```

Source-sink shortest paths. Given an edge-weighted digraph, a source vertex s, and a target vertex t, find the shortest path from s to t.

All-pairs shortest paths. Given an edge-weighted digraph, support queries of the form Given a source vertex s and a target vertex t, is there a path from s to t? If so, find a shortest such path (one whose total weight is minimal).

Shortest paths in Euclidean graphs. Solve the single-source, source-sink, and all-pairs shortest-paths problems in graphs where vertices are points in the plane and edge weights are proportional to Euclidean distances between vertices.

#### Acyclic edge-weighted digraphs

For many natural applications, edge-weighted digraphs are known to have no directed cycles. For economy, we use the equivalent term edge-weighted DAG to refer to an acyclic edge-weighted digraph. We now consider an algorithm for finding shortest paths that is simpler and faster than Dijkstra’s algorithm for edge-weighted DAGs. Specifically, it

* Solves the single-source problem in linear time
* Handles negative edge weights
* Solves related problems, such as finding longest paths.

Proposition S

By relaxing vertices in t opological order, we can solve the single-source shortest-paths problem for edge-weighted DAGs in time proportional to E + V.

```
public class AcyclicSP {

	private DirectedEdge[] edgeTo;
	private double[] distTo;
	
	public AcyclicSP(EdgeWeightedDigraph G, int s) {
		edgeTo=new DirectedEdge[G.v()];
		distTo-new double[G.V()];
		
		for(int v=0;v<G.V();v++) {
			distTo[v]=Double.POSITIVE_INFINITY;
		}
		distTo[s]=0.0;
		
		Topological top=new Topological(G);
		
		for(int v:top.order()) {
			relax(G,v);
		}
		
	}
	
	private void relax(EdgeWeightedDigraph G, int v)
	
	public double disTo(int v)
	public boolean distTo(int v)
	public Iterable<Edge> pathTo(int v)
}
```

##### Longest paths

Consider the problem of finding the longest path in an edge-weighted DAG with edge weights that may be positive or negative.

Single-source longest paths in edge-weighted DAGs. Given an edge-weighted DAG (with negative weights allowed) and a source vertex s, support queries of the form: Is there a directed path from s to a given target vertex v? If so, find a longest such path (one whose total weight is maximal).

##### Proposition T

We can solve the longest-paths problem in edge-weighted DAGs in time proportional to E + V.

##### Parallel job scheduling

As an example application, we revisit the class of scheduling problems that we first considered in Section 4.2 (page 574). Specifically, consider the following scheduling problem (differences from the problem on page 575 are italicized):

Parallel precedence-constrained scheduling. Given a set of jobs of specified duration to be completed, with precedence constraints that specify that certain jobs have to be completed before certain other jobs are begun, how can we schedule the jobs on identical processors (as many as needed ) such that they are all completed in the minimum amount of time while still respecting the constraints?

##### Definition

The critical path method for parallel scheduling is to proceed as follows: Create an edge-weighted DAG with a source s, a sink t, and two vertices for each job (a start vertex and an end vertex). For each job, add an edge from its start vertex to its end vertex with weight equal to its duration. For each precedence constraint v->w, add a zero-weight edge from the end vertex corresponding tovs to the beginning vertex corresponding to w. Also add zero-weight edges from the source to each job’s start vertex and from each job’s end vertex to the sink. Now, schedule each job at the time given by the length of its longest path from the source.

```
public class CPM {

	public static void main(String[] args) {
		int N=StdIn.readInt();
		StdIn.readLine();
		G=new EdgeWeightedDigraph(2*N+2);
		
		int s=2*N, t=2*N+1;
		for(int i=0;i<N;i++) {
			String[] a=StdIn.readLine().split("\\s+");
			double duration=Double.parseDouble(a[0]);
			G.addEdge(new DirectedEdge(i,i+N,duration));
			G.addEdge(new DirectedEdge(s,i,0.0));
			G.addEdge(new DirectedEdge(i+N,t,0.0));
			for(int j=1;j<a.length;j++) {
				int successor=Integer.parseInt(a[j]);
				G.addEdge(new DirectedEdge(i+N,successor,0.0));
			}
			
			AcyclicLP lp=new AcyclicLP(G, s);
			
			StdOut.println("Start times: ");
			for(int i=0;i<N;i++) {
				StdOut.printf("%4d: %5.1f\n",i,lp.distTo(i));
			}
			StdOut.printf("Finish time: %5.1f\n",lp.distTo(t));
		}
	}
}
```

##### Proposition U

The critical path method solves the parallel precedence-constrained scheduling problem in linear time.

##### Parallel job scheduling with relative deadlines

Conventional deadlines are relative to the start time of the first job. Suppose that we allow an additional type of constraint in the job-scheduling problem to specify that a job must begin before a specified amount of time has elapsed, relative to the start time of another job. Such constraints are commonly needed in time-critical manufacturing processes and in many other applications, but they can make the job-scheduling problem considerably more difficult to solve.

##### Proposition V

Parallel job scheduling with relative deadlines is a shortest-paths problem in edge-weighted digraphs (with cycles and negative weights allowed).

#### Shortest paths in general edge-weighted digraphs

Our job-schedulingwith-deadlines example just discussed demonstrates that negative weights are not merely a mathematical curiosity; on the contrary, they significantly extend the applicability of the shortest-paths problem as a problem-solving model. Accordingly we now consider algorithms for edge-weighted digraphs that may have both cycles and negative weights.

##### Strawman I

The first idea that suggests itself is to find the smallest (most negative) edge weight, then to add the absolute value of that number to all the edge weights to transform the digraph into one with no negative weights. This naive approach does not work at all, because shortest paths in the new digraph bear little relation to shortest paths in the old one. The more edges a path has, the more it is penalized by this transformation.

##### Strawman II

The second idea that suggests itself is to try to adapt Dijkstra’s algorithm in some way. The fundamental difficulty with this approach is that the algorithm depends on examining paths in increasing order of their distance from the source. The proof in Proposition R that the algorithm is correct assumes that adding an edge to a path makes that path longer. But any edge with negative weight makes the path shorter, so that assumption is unfounded.

##### Negative cycles

When we consider digraphs that could have negative edge weights, the concept of a shortest path is meaningless if there is a cycle has negative weight.

##### Definition

A negative cycle in an edgeweighted digraph is a directed cycle whose total weight (sum of the weights of its edges) is negative.

##### Proposition W

There exists a shortest path from s to v in an edge-weighted digraph if and only if there exists at least one directed path from s to v and no vertex on any directed path from s to v is on a negative cycle.

##### Strawman III

Whether or not there are negative cycles, there exists a shortest simple path connecting the source to each vertex reachable from the source. Why not define shortest paths so that we seek such paths? Unfortunately, the best known algorithm for solving this problem takes exponential time in the worst case (see Chapter 6).

A well-posed and tractable version of the shortest paths problem in edgeweighted digraphs is to require algorithms to

* Assign a shortest-path weight of ![](http://latex.codecogs.com/gif.latex?+\infty) to vertices that are not reachable from the
source
* Assign a shortest-path weight of ![](http://latex.codecogs.com/gif.latex?-\infty) to vertices that are on a path from the source that has a vertex that is on a negative cycle
* Compute the shortest-path weight (and tree) for all other vertices

We now adopt these less stringent restrictions and focus on the following problems in general digraphs:

Negative cycle detection. Does a given edge-weighted digraph have a negative cycle? If it does, find one such cycle.
Single-source shortest paths when negative cycles are not reachable. Given an edge-weighted digraph and a source s with no negative cycles reachable from s, support queries of the form Is there a directed path from s to a given target vertex v? If so, find a shortest such path (one whose total weight is minimal).

To summarize: while shortest paths in digraphs with directed cycles is an ill-posed problem and we cannot efficiently solve the problem of finding simple shortest paths in such digraphs, we can identify negative cycles in practical situations.

##### Proposition X (Bellman-Ford algorithm)

The following method solves the singlesource shortest-paths problem from a given source s for any edge-weighted digraph with V vertices and no negative cycles reachable from s: Initialize distTo[s] to 0 and all other distTo[] values to infinity. Then, considering the digraph’s edges in any order, relax all edges. Make V such passes.

##### Proposition W (continued)

The Bellman-Ford algorithm takes time proportional to EV and extra space proportional to V.

##### Queue-based Bellman-Ford

Specifically, we can easily determine a priori that numerous edges are not going to lead to a successful relaxation in any given pass: the only edges that could lead to a change in distTo[] are those leaving a vertex whose distTo[] value changed in the previous pass. To keep track of such vertices, we use a FIFO queue.

##### Implementation

Implementing the Bellman-Ford algorithm along these lines requires remarkably little code, as shown in Algorithm 4.11. It is based on two additional data structures:

* A queue q of vertices to be relaxed
* A vertex-indexed boolean array onQ[] that indicates which vertices are on the queue, to avoid duplicates

To add vertices to the queue, we augement our relax() implementation from page 646 to put the vertex pointed to by any edge that successfully relaxes onto the queue, as shown in the code at right. The data structures ensure that

* Only one copy of each vertex appears on the queue
* Every vertex whose edgeTo[] and distTo[] values change in some pass is processed in the next pass

```
private void relax(EdgeWeightedDigraph G, int v) {
	for(DirectedEdge e:G.adj(v)) {
		int w=e.to();
		if(distTo[w]>distTo[v]+e.weight()) {
			distTo[w]=distTo[v]+e.weight();
			edgeTo[w]=e;
			if(!onQ[w]) {
				q.enqueue(w);
				onQ[w]=true;
			}
		}
		if(cost++%G.V()==0) {
			findNegativeCycle();
		}
	}
}
```

##### Proposition Y

The queue-based implementation of the Bellman-Ford algorithm solves the shortest-paths problem from a given source s (or finds a negative cycle reachable from s) for any edge-weighted digraph with V vertices, in time proportional to EV and extra space proportional to V, in the worst case.

```
public class BellmanFordSP {

	private double[] distTo;
	private DirectedEdge[] edgeTo;
	private boolean[] onQ;
	private Queue<Integer> queue;
	private int cost;
	private Iterable<DirectedEdge> cycle;
	
	public BellmanFordSP(EdgeWeightedDigraph G, int s) {
		distTo=new double[G.V()];
		edgeTo=new DirectedEdge[G.V()];
		onQ=new boolean[G.V()];
		queue=new Queue<Integer>();
		for(int v=0;v<G.V();v++) {
			distTo[v]=Double.POSITIVE_INFINITY;
		}
		distTo[s]=0.0;
		queue.enqueue(s);
		onQ[s]=true;
		while(!queue.isEmpty()&&!this.hasNegativeCycle()) {
			int v=queue.dequeue();
			onQ[v]=false;
			relax(v);
		}
	}
	
	private void relax(int v)
	
	public double distTo(int v)
	public boolean hasPathTo(int v)
	public Iterable<Edge> pathTo(int v)
	
	private void findNegativeCycle()
	public boolean hasNegativeCycle()
	public Iterable<Edge> negativeCycle()
}
```