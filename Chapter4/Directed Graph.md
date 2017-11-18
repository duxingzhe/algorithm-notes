### Directed Graph

In directed graphs, edges are one-way: the pair of vertices that defines each edge is an ordered pair that specifies a one-way adjacency. Many applications (for example, graphs that represent the web, scheduling constraints, or telephone calls) are naturally expressed in terms of directed graphs.

<table>
	<tr>
        <th>application</th>
        <th>vertex</th>
        <th>edge</th>
    </tr>
    <tr>
        <th>food web</th>
        <th>species</th>
        <th>predator-prey</th>
    </tr>
    <tr>
        <th>internet content</th>
        <th>page</th>
        <th>hyperlink</th>
    </tr>
    <tr>
        <th>program</th>
        <th>module</th>
        <th>external reference</th>
    </tr>
    <tr>
        <th>cellphone</th>
        <th>phone</th>
        <th>call</th>
    </tr>
    <tr>
        <th>scholarship</th>
        <th>paper</th>
        <th>citation</th>
    </tr>
    <tr>
        <th>financial</th>
        <th>stock</th>
        <th>transaction</th>
    </tr>
    <tr>
        <th>internet</th>
        <th>machine</th>
        <th>connection</th>
    </tr>
</table>
Typical digraph applications

#### Glossary

Our definitions for directed graphs are nearly identical to those for undirected graphs (as are some of the algorithms and programs that we use), but they are worth restating. The slight differences in the wording to account for edge directions imply structural properties that will be the focus of this section.

##### Definition

A directed graph (or digraph) is a set of vertices and a collection of directed edges. Each directed edge connects an ordered pair of vertices.

##### Definition

A directed path in a digraph is a sequence of vertices in which there is a (directed) edge pointing from each vertex in the sequence to its successor in the sequence. A directed cycle is a directed path with at least one edge whose first and last vertices are the same. A simple cycle is a cycle with no repeated edges or vertices (except the requisite repetition of the first and last vertices). The length of a path or a cycle is its number of edges.

#### Digraph data type

##### Representation

We use the a djacency-lists representation, where an edge v->w is represented as a list node containing w in the linked list corresponding to v. This representation is essentially the same as for undirected graphs but is even more straightforward because each edge occurs just once, as shown on the facing page.

##### Input format

The code for the constructor that takes a digraph from an input stream is identical to the corresponding constructor in Graph—the input format is the same, but all edges are interpreted to be directed edges. In the list-of-edges format, a pair v w is interpreted as an edge v->w.

##### Reversing a digraph

Digraph also adds to the API a method reverse() which returns a copy of the digraph, with all edges reversed. This method is sometimes needed in digraph processing because it allows clients to find the edges that point to each vertex, while adj() gives just vertices connected by edges that point from each vertex.

##### Symbolic names

It is also a simple matter to allow clients to use symbolic names in
digraph applications. To implement a class SymbolDigraph like  SymbolGraph on page 552, replace Graph by Digraph everywhere.

```
public class Digraph {
	private final int V;
	private int E;
	private Bag<Integer> adj;

	public Digraph(int V) {
		this.V=V;
		this.E=0;
		adj=(Bag<Integer>[])new Bag[V];
		for(int v=0;v<V;v++) {
			adj[v]=new Bag<Integer>();
		}
	}
	
	public int V() {
		return V;
	}
	
	public int E() {
		return E;
	}
	
	public void addEdge(int v,int w) {
		adj[v].add(w);
		E++;
	}
	
	public Iterable<Integer> adj(int v){
		return adj[v];
	}
	
	public Digraph reverse() {
		Digraph R=new Digraph(V);
		for(int v=0;v<V;v++) {
			for(int w:adj(v)) {
				R.addEdge(w, v);
			}
		}
		
		return R;
	}
}
```

#### Reachability in digraphs

Single-source reachability. Given a digraph and a source vertex s, support queries of the form Is there a directed path from s to a given target vertex v?

Multiple-source reachability. Given a digraph and a set of source vertices, support queries of the form Is there a directed path from any vertex in the set to a given target vertex v?

DirectedDFS uses our standard graph-processing paradigm and a standard recursive depth-first search to solve these problems. It calls the recursive dfs() for each source, which marks every vertex encountered. 

##### Proposition D.

DFS marks all the vertices in a digraph reachable from a given set of sources in time proportional to the sum of the outdegrees of the vertices marked.

```
public class DirectedDFS {

	private boolean[] marked;
	
	public DirectedDFS(Digraph G, int s) {
		marked=new boolean[G.V()];
		dfs(G,s);
	}
	
	public DirectedDFS(Digraph G, Iterable<Integer> sources) {
		marked=new boolean[G.V()];
		for(int s:sources) {
			if(!marked[s])
				dfs(G,s);
		}
	}
	
	private void dfs(Digraph G,int v) {
		marked[v]=true;
		for(int w:G.adj(v)) {
			if(!marked[w]) {
				dfs(G,w);
			}
		}
	}
	
	public boolean marked(int v) {
		return marked[v];
	}
	
	public static void main(String[] args) {
		Digraph G=new Digraph(new In(args));
		
		Bag<Integer> sources=new Bag<Integer>();
		for(int i=1;i<args.length;i++) {
			source.add(Integer.parseInt(args[i]));
		}
		DirectedDFS reachable=new DirectedDFS(G,sources);
		for(int v=0;v<G.V();v++) {
			if(reachable.marked(v)) {
				StdOut.print(v+" ");
			}
		}
		StdOut.println();
	}
}
```

##### Mark-and-sweep garbage collection

An important application of multiple-source reachability is found in typical memory-management systems, including many implementations of Java.

##### Finding paths in digraphs.

Single-source directed paths. Given a digraph and a source vertex s, support queries of the form Is there a directed path from s to a given target vertex v? If so, find such a path.
Single-source shortest directed paths. Given a digraph and a source vertex s, support queries of the form Is there a directed path from s to a given target vertex v? If so, find a shortest such path (one with a minimal number of edges).

#### Cycles and DAGs

Directed cycles are of particular importance in applications that involve processing digraphs. Identifying directed cycles in a typical digraph can be a challenge without the help of a computer, as shown at right.

##### Scheduling problems

A widely applicable problem-solving model has to do with arranging for the completion of a set of jobs, under a set of constraints, by specifying when and how the jobs are to be performed.

Precedence-constrained scheduling. Given a set of jobs to be completed, with precedence constraints that specify that certain jobs have to be completed before certain other jobs are begun, how can we schedule the jobs such that they are all completed while still respecting the constraints?

Topological sort. Given a digraph, put the vertices in order such that all its directed edges point from a vertex earlier in the order to a vertex later in the order (or report that doing so is not possible).

<table>
	<tr>
        <th>application</th>
        <th>vertex</th>
        <th>edge</th>
    </tr>
    <tr>
        <th>job schedule</th>
        <th>job</th>
        <th>precedence constraint</th>
    </tr>
    <tr>
        <th>course schedule</th>
        <th>course</th>
        <th>prerequisite</th>
    </tr>
    <tr>
        <th>inheritance</th>
        <th>Java class</th>
        <th>extends</th>
    </tr>
    <tr>
        <th>spreadsheet</th>
        <th>cell</th>
        <th>formula</th>
    </tr>
    <tr>
        <th>symbolic links</th>
        <th>file name</th>
        <th>link</th>
    </tr>
</table>
Typical topological-sort applications

##### Cycles in digraphs

If job x must be completed before job y, job y before job z, and job z before job x, then someone has made a mistake, because those three constraints cannot all be satisfied. In general, if a precedence-constrained scheduling problem has a directed cycle, then there is no feasible solution.

Directed cycle detection. Does a given digraph have a directed cycle? If so, find the vertices on some such cycle, in order from some vertex back to itself.

##### Definition

A directed acyclic graph (DAG) is a digraph with no directed cycles.

```
public class DirectedCycle {

	private boolean[] marked;
	private int[] edgeTo;
	private Stack<Integer> cycle;
	private boolean[] onStack;
	
	public DirectedCycle(Graph G) {
		onStack=new boolean[G.V()];
		edgeTo=new int[G.V()];
		marked=new boolean[G.V()];
		for(int v=0;v<G.V();v++){
			if(!marked[v])
				dfs(G,v);
		}
	}
	
	private void dfs(Digraph G, int v) {
		onStack[v]=true;
		marked[v]=true;
		for(int w:G.adj(v)) {
			if(this.hasCycle())
				return;
			else if(!marked[w]) {
				edgeTo[w]=v;
				dfs(G,w);
			}else if(onStack[w]) {
				cycle=new Stack<Integer>();
				for(int x=v;x!=w;x++) {
					cycle.push(x);
				}
				cycle.push(w);
				cycle.push(v);
			}
		}
		onStack[v]=false;
	}
	
	public boolean hasCycle() {
		return cycle!=null;
	}
	
	public Iterable<Integer> cycle(){
		return cycle;
	}
}
```