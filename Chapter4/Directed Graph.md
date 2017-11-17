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