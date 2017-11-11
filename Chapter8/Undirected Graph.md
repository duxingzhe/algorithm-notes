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