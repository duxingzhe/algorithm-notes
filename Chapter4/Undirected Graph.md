To illustrate the diversity of applications that involve graph processing, we begin our exploration of algorithms in this fertile area by introducing several examples.

##### Maps

A person who is planning a trip may need to answer questions such as “What is the shortest route from Providence to Princeton?” A seasoned traveler who has experienced traffic delays on the shortest route may ask the question “What is the fastest way to get from Providence to Princeton?” To answer such questions, we process information about connections (roads) between items (intersections).

##### Web content

When we browse the web, we encounter pages that contain references (links) to other pages and we move from page to page by clicking on the links. The entire web is a graph, where the items are pages and the connections are links. Graphprocessing algorithms are essential components of the search engines that help us locate information on the web.

##### Circuits

An electric circuit comprises devices such as transistors, resistors, and capacitors that are intricately wired together. We use computers to control machines that make circuits and to check that the circuits perform desired functions. We need to answer simple questions such as “Is a short-circuit present?” as well as complicated questions such as “Can we lay out this circuit on a chip without making any wires cross?” The answer to the first question depends on only the properties of the connections (wires), whereas the answer to the second question requires detailed information about the wires, the devices that those wires connect, and the physical constraints of the chip.

##### Schedules

A manufacturing process requires a variety of jobs to be performed, under a set of constraints that specify that certain tasks cannot be started until certain other tasks have been completed. How do we schedule the tasks such that we both respect the given constraints and complete the whole process in the least amount of time?

##### Commerce

Retailers and financial instututions track buy/sell orders in a market. A connection in this situation represents the transfer of cash and goods between an institution and a customer. Knowledge of the nature of the connection structure in this instance may enhance our understanding of the nature of the market.

##### Matching

Students apply for positions in selective institutions such as social clubs, universities, or medical schools. Items correspond to the students and the institutions; connections correspond to the applications. We want to discover methods for matching interested students with available positions.

##### Computer networks

A computer network consists of interconnected sites that send, forward, and receive messages of various types. We are interested in knowing about the nature of the interconnection structure because we want to lay wires and build switches that can handle the traffic efficiently.

##### Software

A compiler builds graphs to represent relationships among modules in a large software system. The items are the various classes or modules that comprise the system; connections are associated either with the possibility that a method in one class might call another (static analysis) or with actual calls while the system is in operation (dynamic analysis). We need to analyze the graph to determine how best to allocate resources to the program most efficiently. 

##### Social networks

When you use a social network, you build explicit connections with your friends. Items correspond to people; connections are to friends or followers. Understanding the properties of these networks is a modern graph-processing applications of intense interest not just to compaines that support such networks, but also in politics, diplomacy, entertainment, education, marketing, and many other domains.

<table>
    <tr>
        <th>application</th>
        <th>item</th>
        <th>connection</th>
    </tr>
    <tr>
        <th>map</th>
        <th>intersection</th>
        <th>road</th>
    </tr>
    <tr>
        <th>web content</th>
        <th>page</th>
        <th>link</th>
    </tr>
    <tr>
        <th>circuit</th>
        <th>device</th>
        <th>wire</th>
    </tr>
    <tr>
        <th>schedule</th>
        <th>job</th>
        <th>constrain</th>
    </tr>
    <tr>
        <th>commerce</th>
        <th>customer</th>
        <th>transaction</th>
    </tr>
    <tr>
        <th>matching</th>
        <th>student</th>
        <th>application</th>
    </tr>
    <tr>
        <th>computer network</th>
        <th>site</th>
        <th>connection</th>
    </tr>
    <tr>
        <th>software</th>
        <th>method</th>
        <th>call</th>
    </tr>
    <tr>
        <th>social network</th>
        <th>person</th>
        <th>friendship</th>
    </tr>
</table>

### Undirected Graph

Definition. A graph is a set of vertices and a collection of edges that each connect a pair of vertices.

Anomalies. Our definition allows two simple anomalies:

* A self-loop is an edge that connects a vertex to itself.
* Two edges that connect the same pair of vertices are parallel.

#### Glossary

A substantial amount of nomenclature is associated with graphs. Most of the terms have straightforward definitions, and, for reference, we consider them in one place: here.

##### Definition

A path in a graph is a sequence of vertices connected by edges. A simple path is one with no repeated vertices. A cycle is a path with at least one edge whose first and last vertices are the same. A simple cycle is a cycle with no repeated edges or vertices (except the requisite repetition of the first and last vertices). The length of a path or a cycle is its number of edges.

##### Definition

A graph is connected if there is a path from every vertex to every other vertex in the graph. A graph that is not connected consists of a set of connected components, which are maximal connected subgraphs.

A graph G with V vertices is a tree if and only if it satisfies any of the following five conditions:

* G has V-1 edges and no cycles.
* G has V-1 edges and is connected.
* G is connected, but removing any edge disconnects it.
* G is acyclic, but adding any edge creates a cycle.
* Exactly one simple path connects each pair of vertices in G.

#### Undirected graph data type

Our starting point for developing graph-processing algorithms is an API that defines the fundamental graph operations. This scheme allows us to address graph-processing tasks ranging from elementary maintenance operations to sophisticated solutions of difficult problems.

Representation alternatives. The next decision that we face in graph processing is which graph representation (data structure) to use to implement this API. We have two basic requirements:

* We must have the space to accommodate the types of graphs that we are likely to encounter in applications.
* We want to develop time-effi cient implementations of Graph instance methods—the basic methods that we need to develop graph-processing clients.

These requirements are a bit vague, but they are still helpful in choosing among the three data structures that immediately suggest themselves for representing graphs:

* An adjacency matrix, where we maintain a V-by-V boolean array, with the entry in row v and column w defined
to be true if there is an edge adjacent to both vertex v and vertex w in the graph, and to be false otherwise. This representation fails on the first count—graphs with millions of vertices are common and the space cost for the ![](http://latex.codecogs.com/gif.latex?V^2) boolean values needed is prohibitive.
* An array of edges, using an Edge class with two instance variables of type int. This direct representation is simple, but it fails on the second count—implementing adj() would involve examining all the edges in the graph.
* An array of adjacency lists, where we maintain a vertex-indexed array of lists of the vertices adjacent to each vertex. This data structure satisfies both requirements for typical applications and is the one that we will use throughout this chapter.

##### Adjacency-lists data structure

The standard graph representation for graphs that are not dense is called the adjacency-lists data structure, where we keep track of all the vertices adjacent to each vertex on a linked list that is associated with that vertex. This Graph implementation achieves the following performance characteristics:

* Space usage proportional to V + E
* Constant time to add an edge
* Time proportional to the degree of v to iterate through vertices adjacent to v (constant time per adjacent vertex processed)

It is certainly reasonable to contemplate other operations that might be useful in applications, and to consider methods for

* Adding a vertex
* Deleting a vertex

One way to handle such operations is to expand the API to use a symbol table (ST) instead of a vertex-indexed array (with this change we also do not need our convention that vertex names be integer indices). We might also consider methods for

* Deleting an edge
* Checking whether the graph contains the edge v-w

To implement these methods (and disallow parallel edges) we might use a SET instead of a Bag for adjacency lists. We refer to this alternative as an adjacency set representation. We do not use these alternatives in this book for several reasons:

* Our clients do not need to add vertices, delete vertices and edges, or check whether an edge exists.
* When clients do need these operations, they typically are invoked infrequently or for short adjacency lists, so an easy option is to use a brute-force implementation that iterates through an adjacency list.
* The SET and ST representations slightly complicate algorithm implementation code, diverting attention from the algorithms themselves.
* A performance penalty of log V may be involved in some situations.

It is not difficult to adapt our algorithms to accommodate other designs (for example disallowing parallel edges or self-loops) without undue performance penalties. The table below summarizes performance characteristics of the alternatives that we have mentioned.

##### Design pattern for graph processing

Since we consider a large number of graph-processing algorithms, our initial design goal is to decouple our implementations from the graph representation.

Search API

the union-find algorithms of Chapter 1. The constructor can build a UF object, do a union() operation for each of the graph’s edges, and implement marked(v) by calling connected(s, v). Implementing count() requires using a weighted UF implementation and extending its API to use a count() method that returns wt[find(v)]. This implementation is simple and efficient, but the implementation that we consider next is even simpler and more efficient. It is based on depth-first search, a fundamental recursive method that follows the graph’s edges to find the vertices connected to the source.

```
public class TestSearch {

	public static void main(String[] args) {
		Graph G=new Graph(new In(args[0]));
		int s=Integer.parseInt(args[1]);
		
		for(int v=0;v<G.V();v++) {
			if(search.marked(v)) {
				StdOut.print(v+" ");
			}
		}
		StdOut.println();
		
		if(search.count()!=G.V()) {
			StdOut.print("NOT ");
		}
		StdOut.println("connected");
	}
}
```

#### Depth-first search

We often learn properties of a graph by systematically examining each of its vertices and each of its edges. Determining some simple graph properties—for example, computing the degrees of all the vertices—is easy if we just examine each edge (in any order whatever). But many other graph properties are related to paths, so a natural way to learn them is to move from vertex to vertex along the graph’s edges.

###### Searching in a maze

It is instructive to think about the process of searching through a graph in terms of an equivalent problem that has a long and distinguished history—finding our way through a maze that consists of passages connected by intersections. Some mazes can be handled with a simple rule, but most mazes require a more sophisticated strategy. Using the terminology maze instead of graph, passage instead of edge, and intersection instead of vertex is making mere semantic distinctions, but, for the moment, doing so will help to give us an intuitive feel for the problem.

To explore all passages in a maze:

* Take any unmarked passage, unrolling a string behind you.
* Mark all intersections and passages when you first visit them.
* Retrace steps (using the string) when approaching a marked intersection.
* Retrace steps when no unvisited options remain at an intersection encountered while retracing steps.

The string guarantees that you can always find a way out and the marks guarantee that you avoid visiting any passage or intersection twice.

##### Warmup

The classic recursive method for searching in a connected graph (visiting all of its vertices and edges) mimics Tremaux maze exploration but is even simpler to describe. To search a graph, invoke a recursive method that visits vertices. To visit a vertex:

* Mark it as having been visited.
* Visit (recursively) all the vertices that are adjacent to it and that have not yet been marked.
This method is called depth-first search (DFS).

##### Proposition A

DFS marks all the vertices connected to a given source in time proportional to the sum of their degrees.