### Context

##### Commercial applications

The emergence of the internet has underscored the central role of algorithms in commercial applications. All of the applications that you use regularly benefit from the classic algorithms that we have studied:

* Infrastructure (operating systems, databases, communications)
* Applications (email, document processing, digital photography)
* Publishing (books, magazines, web content)
* Networks (wireless networks, social networks, the internet)
* Transaction processing (fi nancial, retail, web search)

##### Scientific computing

Since von Neumann developed mergesort in 1950, algorithms have played a central role in scientific computing. Today’s scientists are awash in experimental data and are using both mathematical and computational models to understand the natural world for:

* Mathematical calculations (polynomials, matrices, differential equations)
* Data processing (experimental results and observations, especially genomics)
* Computational models and simulation All of these can require complex and extensive computing with huge amounts of data.

##### Engineering

Almost by definition, modern engineering is based on technology. Modern technology is computer-based, so algorithms play a central role for

* Mathematical calculations and data processing
* Computer-aided design and manufacturing
* Algorithm-based engineering (networks, control systems)
* Imaging and other medical systems

##### Operations research

Researchers and practitioners in OR develop and apply mathematical models for problem solving, including

* Scheduling
* Decision making
* Assignment of resources

#### Event-driven simulation

Our first example is a fundamental scientific application: simulate the motion of a system of moving particles that behave according to the laws of elastic collision. Scientists use such systems to understand and predict properties of physical systems.

##### Hard-disc model

We begin with an idealized model of the motion of atoms or molecules in a container that has the following salient features:

* Moving particles interact via elastic collisions with each other and with walls.
* Each particle is a disc with known position, velocity, mass, and radius.
* No other forces are exerted.

This simple model plays a central role in statistical mechanics, a field that relates macroscopic observables (such as temperature and pressure) to microscopic dynamics (such as the motion of individual atoms and molecules).

##### Time-driven simulation

Our primary goal is simply to maintain the model: that is, we want to be able to keep track of the positions and velocities of all the particles as time passes. The basic calculation that we have to do is the following: given the positions and velocities for a specific time t, update them to reflect the situation at a future time t+dt for a specifi c amount of time dt.

##### Event-driven simulation

We pursue an alternative approach that focuses only on those times at which collisions occur. In particular, we are always interested in the next collision (because the simple update of all of the particle positions using their velocities is valid until that time). Therefore, we maintain a priority queue of events, where an event is a potential collision sometime in the future, either between two particles or between a particle and a wall.

##### Collision prediction

##### Collision resolution

##### Invalidated events

When we remove an event from the priority queue for processing, we check whether the counts corresponding to its particle(s) have changed since the event was created. This approach to handling invalidated collisions is the so-called lazy approach: when a particle is involved in a collision, we leave the now-invalid events associated with it on the priority queue and essentially ignore them when they come off. An alternative approach, the so-called eager approach, is to remove from the priority queue all events involving any colliding particle before calculating all of the new potential collisions for that particle. This approach requires a more sophisticated priority queue (that implements the remove operation).

##### Particles

Exercise 6.1 outlines the implementation of a data type particles, based on a direct application of Newton’s laws of motion.

##### Events

We encapsulate in a private class the description of the objects to be placed on the priority queue (events). The instance variable time holds the time when the event is predicted to happen, and the instance variables a and b hold the particles associated with the event.

A slight twist in the implementation of Event is that we use the fact that particle values may be null to encode these four different types of events, as follows:

* Neither a nor b null: particle-particle collision
* a not null and b null: collision between a and a vertical wall
* a null and b not null: collision between b and a horizontal wall
* Both a and b null: redraw event (draw all particles)

```
private class Event implements Comparable<Event> {
    private final double time;
    private final Particle a,b;
    private final int countA,countB;
    
    public Event(double t, Particle a, Particle b) {
        this.time=t;
        this.a=a;
        this.b=b;
        if(a!=null)
            countA=a.count();
        else
            countA=-1;
        if(b!=null)
            countB=b.count();
        else
            countB=-1;
    }
    
    public int compareTo(Event that) {
        if(this.time<that.time)
            return -1;
        else if(this.time>that.time)
            return +1;
        else
            return 0;
    }
    
    public boolean isValid() {
        if(a!=null&&a.count()!=countA)
            return false;
        if(b!=null^b.count()!=countB)
            return false;
        return true;
    }
}
```

##### Simulation code

With the computational details encapsulated in Particle and Event, the simulation itself requires remarkably little code, as you can see in the implementation in the class CollisionSystem (see page 863 and page 864).

```
private void predictCollisions(Particle a, double limit) {
    if(a==null)
        return;
    for(int i=0;i<particles.length;i++) {
        double dt=a.timeToHit(particles[i]);
        if(t+dt<=limit)
            pq.insert(new Event(t+dt,a,particles[i]));
    }
    
    double dtX=a.timeToHitVerticalWall();
    if(t+dtX<=limit)
        pq.insert(new Event(t+dtX,a,null));
    double dtY=a.timeToHitHorizontalWall();
    if(t+dtY<=limit)
        pq.insert(new Event(t+dtY,null,a));
}
```

The heart of the simulation is the simulate() method shown on page 864. We initialize by calling predictCollisions() for each particle to fill the priority queue with the potential collisions involving all particle-wall and all particle-particle pairs. Then we enter the main event-driven simulation loop, which works as follows:

* Delete the impending event (the one with minimum priority t).
* If the event is invalid, ignore it.
* Advance all particles to time t on a straight-line trajectory.
* Update the velocities of the colliding particle(s).
* Use predictCollisions() to predict future collisions involving the colliding particle(s) and insert onto the priority queue an event corresponding to each.

```
public class CollisionSystem {
	
	private class Event implements Comparable<Event> {
		private final double time;
		private final Particle a,b;
		private final int countA,countB;
		
		public Event(double t, Particle a, Particle b) {
			this.time=t;
			this.a=a;
			this.b=b;
			if(a!=null)
				countA=a.count();
			else
				countA=-1;
			if(b!=null)
				countB=b.count();
			else
				countB=-1;
		}
		
		public int compareTo(Event that) {
			if(this.time<that.time)
				return -1;
			else if(this.time>that.time)
				return +1;
			else
				return 0;
		}
		
		public boolean isValid() {
			if(a!=null&&a.count()!=countA)
				return false;
			if(b!=null^b.count()!=countB)
				return false;
			return true;
		}
	}
	
	private MinPQ<Event> pq;
	private double t=0.0;
	private Particle[] particles;
	
	public CollisionSystem(Particle[] particles) {
		this.particles=particles;
	}
	
	private void predictCollisions(Particle a, double limit) {
		if(a==null)
			return;
		for(int i=0;i<particles.length;i++) {
			double dt=a.timeToHit(particles[i]);
			if(t+dt<=limit)
				pq.insert(new Event(t+dt,a,particles[i]));
		}
		
		double dtX=a.timeToHitVerticalWall();
		if(t+dtX<=limit)
			pq.insert(new Event(t+dtX,a,null));
		double dtY=a.timeToHitHorizontalWall();
		if(t+dtY<=limit)
			pq.insert(new Event(t+dtY,null,a));
	}
	
	public void redraw(double limit, double Hz) {
		StdDraw.clear();
		for(int i=0;i<particles.length;i++) {
			particles[i].draw();
		}
		StdDraw.show(20);
		if(t<limit)
			pq.insert(new Event(t+1.0/Hz,null,null));
	}
	
	public void simulate(double limit, double Hz) {
		pq=new MinPQ<Event>();
		for(int i=0;i<particles.length;i++) {
			predictCollisions(particles[i],limit);
		}
		pq.insert(new Event(0,null,null));
		
		while(!pq.isEmpty()) {
			Event event=pq.delMin();
			for(int i=0;i<particles.length;i++) {
				particles[i].move(event.time-t);
			}
			t=event.time;
			Particle a=event.a, b=event.b;
			if(a!=null&&b!=null) {
				a.bounceOff(b);
			}else if(a!=null&b==null) {
				a.bounceOffHorizontalWall();
			}else if(a==null&&b!=null) {
				b.bounceOffVerticalWall();
			}else if(a==null&&b==null) {
				redraw(limit,Hz);
			}
			predictCollisions(a,limit);
			predictcollisions(b,limit);
		}
	}
	
	public static void main(String[] args) {
		StdDraw.show(0);
		int N=Integer.parseInt(arg0);
		Particle[] particles=new Particle[N];
		for(int i=0;i<N;i++) {
			particles[i]=new Particle();
		}
		CollisionSystem system=new CollisionSystem(particles);
		system.simulate(10000, 0.5);
	}

}
```

```
public void simulate(double limit, double Hz) {
    pq=new MinPQ<Event>();
    for(int i=0;i<particles.length;i++) {
        predictCollisions(particles[i],limit);
    }
    pq.insert(new Event(0,null,null));
    
    while(!pq.isEmpty()) {
        Event event=pq.delMin();
        for(int i=0;i<particles.length;i++) {
            particles[i].move(event.time-t);
        }
        t=event.time;
        Particle a=event.a, b=event.b;
        if(a!=null&&b!=null) {
            a.bounceOff(b);
        }else if(a!=null&b==null) {
            a.bounceOffHorizontalWall();
        }else if(a==null&&b!=null) {
            b.bounceOffVerticalWall();
        }else if(a==null&&b==null) {
            redraw(limit,Hz);
        }
        predictCollisions(a,limit);
        predictcollisions(b,limit);
    }
}
```

##### Performance

As described at the outset, our interest in event-driven simulation is to avoid the computationally intensive inner loop intrinsic in time-driven simulation.

##### Proposition A

An event-based simulation of N colliding particles requires at most N 2 priority queue operations for initialization, and at most N priority queue operations per collision (with one extra priority queue operation for each invalid collision).

#### B-trees

Searching is a fundamental operation on huge data sets, and such searching consumes a significant fraction of the resources used in many computing environments. With the advent of the web, we have the ability to access a vast amount of information that might be relevant to a task—our challenge is to be able to search through it efficiently. In this section, we describe a further extension of the balanced-tree algorithms from Section 3.3 that can support external search in symbol tables that are kept on a disk or on the web and are thus potentially far larger than those we have been considering (which have to fit in addressable memory).

##### Cost model

Data storage mechanisms vary widely and continue to evolve, so we use a simple model to capture the essentials. We use the term page to refer to a contiguous block of data and the term probe to refer to the first access to a page. We assume that accessing a page involves reading its contents into local memory, so that subsequent accesses are relatively inexpensive.

##### B-tree cost model

When studying algorithms for external searching, we count page accesses (the number of times a page is accessed, for read or write).

##### B-trees

The approach is to extend the 2-3 tree data structure described in Section 3.3, with a crucial difference: rather than store the data in the tree, we build a tree with copies of the keys, each key copy associated with a link. This approach enables us to more easily separate the index from the table itself, much like the index in a book.

##### Conventions

To illustrate the basic mechanisms, we consider an (ordered) SET implementation (with keys and no values). Extending to provide an ordered ST to associate keys with values is an instructive exercise (see Exercise 6.16). Our goal is to support add() and contains() for a set of keys that could be huge.

In external searching applications, it is common to keep the index separate from the data. For B-trees, we do so by using two different kinds of nodes:

* Internal nodes, which associate copies of keys with pages
* External nodes, which have references to the actual data

##### Search and insert

Search in a B-tree is based on recursively searching in the unique subtree that could contain the search key.

As with 2-3 trees, we can use recursive code to insert a new key at the bottom of the tree.

##### Definition

A B-tree of order M (where M is an even positive integer) is a tree that either is an external k-node (with k keys and associated information) or comprises internal k-nodes (each with k keys and k links to B-trees representing each of the k intervals delimited by the keys), having the following structural properties: every path from the root to an external node must be the same length ( perfect balance); and k must be between 2 and M-1 at the root and between M/2 and M-1 at every other node.

##### Representation

As just discussed, we have a great deal of freedom in choosing concrete representations for nodes in B-trees.We encapsulate these choices in a Page API that associates keys with links to Page objects and supports the operations that we need to test for overfull pages, split them, and distinguish between internal and external pages. You can think of a Page as a symbol table, kept externally (in a file on your computer or on the web). The terms open and close in the API refer to the process of bringing an external page into internal memory and writing its contents back out (if necessary).

With these preparations, the code for BTreeSET on page 872 is remarkably simple. For contains(), we use a recursive method that takes a Page as argument and handles three cases:

* If the page is external and the key is in the page, return true.
* If the page is external and the key is not in the page, return false.
* Otherwise, do a recursive call for the subtree that could contain the key.

##### Performance

The most important property of B-trees is that for reasonable values of the parameter M the search cost is constant, for all practical purposes:

##### Proposition B

A search or an insertion in a B-tree of order M with N items requires between ![](http://latex.codecogs.com/gif.latex?log_{M}N) and ![](http://latex.codecogs.com/gif.latex?log_{\frac{M}{2}}N) probes—a constant number, for practical purposes.

##### Space

The space usage of B-trees is also of interest in practical applications. By construction, the pages are at least half full, so, in the worst case, B-trees use about double the space that is absolutely necessary for keys, plus extra space for links.

```
public class BTreeSET<Key extends Comparable<Key>> {
	private Page root=new Page(true);
	
	public BTreeSET(Key sentinel) {
		put(sentinel);
	}
	
	public boolean contains(Key key) {
		return contains(root,key);
	}
	
	private boolean contains(Page h, Key key) {
		if(h.isExternal())
			return h.contains(key);
		return contains(h.next(key),key);
	}
	
	public void add(Key key) {
		put(root, key);
		if(root.isFull()) {
			Page lefthalf=root;
			Page righthalf=root.split();
			root=new Page(false);
			root.put(lefthalf);
			root.put(righthalf);
		}
	}
	
	public void add(Page h,Key key) {
		if(h.isExternal()) {
			h.put(key);
			return;
		}
		
		Page next=h.next(key);
		put(next,key);
		if(next.isFull()) {
			h.put(next.split());
		}
		next.close();
	}

}
```

#### Suffix arrays

Efficient algorithms for string processing play a critical role in commercial applications and in scientific computing. From the countless strings that define web pages that are searched by billions of users to the extensive genomic databases that scientists are studying to unlock the secret of life, computing applications of the 21st century are increasingly string-based. As usual, some classic algorithms are effective, but remarkable new algorithms are being developed.

##### Longest repeated substring

What is the longest substring that appears at least twice in a given string? For example, the longest repeated substring in the string "to be or not to be" is the string "to be". Think briefly about how you might solve it. Could you find the longest repeated substring in a string that has millions of characters? This problem is simple to state and has many important applications, including data compression, cryptography, and computer-assisted music analysis.

##### Brute-force solution

As a warmup, consider the following simple task: given two strings, find their longest common prefix (the longest substring that is a prefix of both strings).

```
private static int lcp(String s, String t)
{
	int N=Math.min(s.length(), t.length());
	for(int i=0;i<N;i++)
		if(s.charAt(i)!=t.charAt(i))
			return i;
	return N;
}
```

##### Suffix sort solution

The following clever approach, which takes advantage of sorting in an unexpected way, is an effective way to find the longest repeated substring, even in a huge string: we use Java’s substring() method to make an array of strings that consists of the suffixes of s (the substrings starting at each position and going to the end), and then we sort this array. The key to the algorithm is that every substring appears somewhere as a prefix of one of the suffixes in the array.

##### Indexing a string

When you are trying to find a particular substring within a large text—for example, while working in a text editor or within a page you are viewing with a browser—you are doing a substring search, the problem we considered in Section 5.3. For that problem, we assume the text to be relatively large and focus on preprocessing the substring, with the goal of being able to efficiently find that substring in any given text. When you type search keys into your web browser, you are doing a search with string keys, the subject of Section 5.2. Your search engine must precompute an index, since it cannot afford to scan all the pages in the web for your keys. As we discussed in Section 3.5 (see FileIndex on page 501), this would ideally be an inverted index associating each possible search string with all web pages that contain it—a symbol table where each entry is a string key and each value is a set of pointers (each pointer giving the information necessary to locate an occurrence of the key on the web—perhaps a URL that names a web page and an integer offset within that page).

##### API and client code

```
public class LRS {
	
	public static void main(String[] args) {
		String text=StdIn.readAll();
		int N=text.length();
		SuffixArray sa=new SuffixArray(text);
		String lrs="";
		for(int i=1;i<N;i++) {
			int length=sa.lcp(i);
			if(length>substring.length())
				lrs=sa.select(i).substring(0,length);
		}
		StdOut.println(lrs);
	}

}
```

```
public class KWIC {
	
	public static void main(String[] args) {
		In in=new In(args[0]);
		int context=Integer.parseInt(args[1]);
		
		String text=in.readAll().repaceAll("\\s+"," ");
		int N=text.length();
		SuffixArray sa=new SuffixArray(text);
		
		while(StdIn.hasNextLine()) {
			String q=StdIn.readLine();
			for(int i=sa.randk(q);i<N&&sa.select(i).startsWith(q);i++) {
				int from=Math.max(0,sa.index(i)-context);
				int to=Math.min(N-1, from+q.length()+2*context);
				StdOut.println(text.substring(from,to));
			}
			StdOut.println(text.substring(from,to));
		}
	}

}
```

##### Implementation

The code on the facing page is a straightforward implementation of the SuffixArray API. Its instance variables are an array of strings and (for economy in code) a variable N that holds the length of the array (the length of the string and its number of suffixes).

##### Performance 

The efficiency of suffix sorting depends on the fact that Java substring extraction uses a constant amount of space—each substring is composed of standard object overhead, a pointer into the original, and a length. Thus, the size of the index is linear in the size of the string. This point is a bit counterintuitive because the total number of characters in the suffixes is ~![](http://latex.codecogs.com/gif.latex?\frac{N^2}{2}), a quadratic function of the size of the string. Moreover, that quadratic factor gives one pause when considering the cost of sorting the suffix array.

##### Proposition C

Using 3-way string quicksort, we can build a suffix array from a random string of length N with space proportional to N and ~ ![](http://latex.codecogs.com/gif.latex?2NlnN) character compares, on the average.

Improved implementations

Our elementary implementation of SuffixArray has poor worst-case performance. For example, if all the characters are equal, the sort examines every character in each substring and thus takes quadratic time. For strings of the type we have been using as examples, such as genomic sequences or natural-language text, this is not likely to be problematic, but the algorithm can be slow for texts with long runs of identical characters. Another way of looking at the problem is to observe that the cost of finding the longest repeated substring is quadratic in the length of the substring because all of the prefixes of the repeat need to be checked (see the diagram at right).

With a bit more work, the Manber-Myers implementation can also support a two-argument lcp() that finds the longest common prefix of two given suffixes that are not necessarily adjacent in guaranteed constant time, again a remarkable improvement over the straightforard implementation. These results are quite surprising, as they achieve efficiencies quite beyond what you might have expected.

##### Proposition D

With suffix arrays, we can solve both the suffix sorting and longest repeated substring problems in linear time.

#### Network-flow algorithms

The study of network-flow algorithms is fascinating because it brings us tantalizingly close to compact and elegant implementations that achieve both goals. As you will see, we have straightforward implementations that are guaranteed to run in time proportional to a polynomial in the size of the network.

##### A physical model

We begin with an idealized physical model in which several of the basic concepts are intuitive. Specifically, imagine a collection of interconnected oil pipes of varying sizes, with switches controlling the direction of flow at junctions, as in the example illustrated at right. Suppose further that the network has a single source (say, an oil field) and a single sink (say, a large refinery) to which all the pipes ultimately connect.

##### Definitions

Because of this broad applicability, it is worthwhile to consider precise statements of the terms and concepts that we have just informally introduced:

##### Definition

A flow network is an edge-weighted digraph with positive edge weights (which we refer to as capacities). An st-flow network has two identified vertices, a source s and a sink t.

##### Definition

An an st-flow network is a set of nonnegative values associated with each edge, which we refer to as edge flows. We say that a flow is feasible if it satisfies the condition that no edge’s flow is greater than that edge’s capacity and the local equilibrium condition that the every vertex’s netflow is zero (except s and t).

With these definitions, the formal statement of our basic problem is straightforward:

Maximum st-flow. Given an st-flow network, find an st-flow such that no other flow from s to t has a larger value.

##### APIs

```
private boolean localEq(FlowNetwork G, int v) {
	double EPSILON=1E-11;
	double netflow=0.0;
	for(FlowEdge e: G.adj(v)) {
		if(v==e.from())
			netflow-=e.flow();
		else
			netflow+=e.flow();
	}
	return Math.abs(netflow)<EPSILON;
}

private boolean isFeasible(FlowNetwork G) {
	for(int v=0;v<G.V();v++) {
		flow(FlowEdge e: G.adj(v)){
			if(e.flow()<0||e.flow()>e.cap()) {
				return false;
			}
		}
	}
	
	for(int v=0;v<G.V();v++) {
		if(v!=s&&v!=t&&!localEq(v))
			return false;
	}
	
	return true;
}
```

##### Ford-Fulkerson algorithm

It is a generic method for increasing flows incrementally along paths from source to sink that serves as the basis for a family of algorithms. It is known as the Ford-Fulkerson algorithm in the classical literature; the more descriptive term augmenting-path algorithm is also widely used. Consider any directed path from source to sink through an st-flow network.

##### Ford-Fulkerson maxflow algorithm

Start with zero flow everywhere. Increase the flow along any augmenting path from source to sink (with no full forward edges or empty backward edges), continuing until there are no such paths in the network.

##### Maxflow-mincut theorem

To show that any flow computed by any implementation of the Ford-Fulkerson algorithm is indeed a maxflow, we prove a key fact known as the maxflow-mincut theorem.

##### Definition

An s in one of its sets and vertex t in the other.

If we view the capacity of the cut as the cost of doing so, to stop the flow in the most economical manner is to solve the following problem:

Minimum st-cut. Given an st-network, find an st-cut such that the capacity of no other cut is smaller. For brevity, we refer to such a cut as a mincut and to the problem of finding one in a network as the mincut problem.

##### Proposition E

For any st-flow, the flow across each st-cut is equal to the value of the flow.

##### Corollary

The outflow from s is equal to the inflow to t (the value of the st-flow).

##### Corollary

No st-fl ow’s value can exceed the capacity of any st-cut.

##### Proposition F ( Maxflow-mincut theorem)

Let f be an st-flow. The following three conditions are equivalent:

i. There exists an st-cut whose capacity equals the value of the flow f.
ii. f is a maxflow.
iii. There is no augmenting path with respect to f.

##### Corollary ( Integrality property)

When capacities are integers, there exists an integer-valued maxflow, and the Ford-Fulkerson algorithm finds it.

##### Definition

Given a st-flow network and an st-flow, the residual network for the flow has the same vertices as the original and one or two edges in the residual network for each edge in the original, defined as follows: For each edge e from v to w in the original, let ![](http://latex.codecogs.com/gif.latex?f_e) be its flow and ce its capacity. If ![](http://latex.codecogs.com/gif.latex?f_e) is positive, include an edge w->v in the residual with capacity ![](http://latex.codecogs.com/gif.latex?f_e) ; and if ![](http://latex.codecogs.com/gif.latex?f_e) is less than ![](http://latex.codecogs.com/gif.latex?c_e), include an edge v->w in the residual with capacity ![](http://latex.codecogs.com/gif.latex?c_e)-![](http://latex.codecogs.com/gif.latex?f_e).

```
public class FlowEdge {

	private final int v;
	private final int w;
	private final double capacity;
	private double flow;
	
	public FlowEdge(int v,int w, double capacity) {
		this.v=v;
		this.w=w;
		this.capacity=capacity;
		this.flow=0.0;
	}
	
	public int from() {
		return v;
	}
	
	public int to() {
		return w;
	}
	
	public double capacity() {
		return capacity;
	}
	
	public double flow() {
		return flow;
	}
	
	public int other(int vertex)
	
	public double residualCapacityTo(int vertex) {
		if(vertex==v)
			return flow;
		else if(vertex==w)
			return cap-flow;
		else
			throw new RuntimeException("Inconsistent edge");
	}
	
	public void addResidualFlowTo(int vertex, double delta) {
		if(vertex==v) {
			flow-=delta;
		}else if(vertex==w) {
			flow+=delta;
		}else
			throw new RuntimeException("Inconsistent edge");
	}
	
	public String toString() {
		return String.format("%d->%d %.2f %.2f", v,w,capacity,flow);
	}
}
```

##### Shortest-augmenting-path method

Perhaps the simplest Ford-Fulkerson implementation is to use a shortest augmenting path (as measured by the number of edges on the path, not flow or capacity).

In this case, the search for an augmenting path amounts to breadth-first search (BFS) in the residual network, precisely as described in Section 4.1, as you can see by comparing the hasAugmentingPath() implementation below to our breadth-first search implemention in Algorithm 4.2 on page 540 (the residual graph is a digraph, and this is fundamentally a digraph processing algorithm, as mentioned on page 685).

```
private boolean hasAugmentingPath(FlowNetwork G,int s, int t) {
	marked=new boolean[G.V()];
	edgeTo=new FlowEdge[G.V()];
	Queue<Integer> q=new Queue<Integer>();
	
	marked[s]=true;
	q.enqueue(s);
	while(!q.isEmpty()) {
		int v=q.dequeue();
		for(FlowEdge e:G.adj(v)) {
			int w=e.other(v);
			if(e.residualCapacityTo(w)>0&&!marked[w]) {
				edgeTo[w]=e;
				marked[w]=true;
				q.enqueue(w);
			}
		}
	}
}
```

```
public class FordFulkerson {

	private boolean[] marked;
	private FlowEdge[] edgeTo;
	private double value;
	
	public FordFulkerson(FlowNetwork G, int s, int t) {
		while(hasAugmentingPath(G,s,t)) {
			double bottle=Double.POSITIVE_INFINITY;
			for(int v=t;v!=s;v=edgeTo[v].other(v))
				bottle=Math.min(bottle, edgeTo[v].residualCapacityTo(v));
			
			for(int v=t;v!=s;v=edgeTo[v].other(v))
				edgeTo[v].addResidualFlowTo(v, bottle);
			
			value+=bottle;
		}
	}
	
	public double value() {
		return value;
	}
	
	public boolean inCut(int v) {
		return marked[v];
	}
	
	public static void main(String[] args) {
		FlowNetwork G=new FlowNetwork(new In(args[0]));
		int s=0,t=G.V()-1;
		FordFulkerson maxflow=new FordFulkerson(G,s,t);
		for(int v=0;v<G.V();v++) {
			for(FlowEdge e: G.adj(v)) {
				if((v==e.from())&&e.flow()>0)
					StdOut.println("   "+e);
			}
		}
		StdOut.println("Max flow value=" + maxflow.value());
	}
}
```

##### Performance

A larger example is shown in the figure above. As is evident from the figure, the lengths of the augmenting paths form a nondecreasing sequence. This fact is a first key to analyzing the performance of the algorithm.

##### Proposition G

The number of augmenting paths needed in the shortest-augmenting-path implementation of the Ford-Fulkerson maxflow algorithm for a flow network with V vertices and E edges is at most ![](http://latex.codecogs.com/gif.latex?\frac{EV}{2}).

###### Corollary

The shortest-augmenting-path implementation of the Ford-Fulkerson maxflow algorithm takes time proportional to VE 2/2 in the worst case.

##### Other implementations

Another Ford-Fulkerson implementation, suggested by Edmonds and Karp, is the following: Augment along the path that increases the flow by the largest amount.

A complete analysis establishing which method is best is a complex task, because their running times depend on

* The number of augmenting paths needed to fi nd a maxflow
* The time needed to fi nd each augmenting path

<table>
	<tr>
		<th>algorithm</th>
		<th>worst-case order of growth of running time for V vertices and E edges with integral capacities (max C)</th>
	</tr>
	<tr>
		<th>Ford-Fulkerson shortest augmenting path</th>
		<th><img src=http://latex.codecogs.com/gif.latex?VE^2></img></th>
	</tr>
	<tr>
		<th>Ford-Fulkerson maximal augmenting path</th>
		<th><img src=http://latex.codecogs.com/gif.latex?E^2logC</th>
	</tr>
	<tr>
		<th>preflow-push</th>
		<th><img src=http://latex.codecogs.com/gif.latex?EVlog{\frac{E}{V^2}}></img></th>
	</tr>
	<tr>
		<th>possible?</th>
		<th>V+E?</img></th>
	</tr>
</table>
Performance characteristics of maxflow algorithms

#### Reduction

In several cases (many of which are listed below), we have found it convenient to solve a problem by formulating it as an instance of another problem that we have already solved.

##### Definition

We say that a problem A reduces to another problem B if we can use an algorithm that solves B to develop an algorithm that solves A.

##### Sorting reductions

We first encountered reduction in Chapter 2, to express the idea that an efficient sorting algorithm is useful for efficiently solving many other problems, that may not seem to be at all related to sorting.

Finding the median. Given a set of numbers, find the median value.

Distinct values. Determine the number of distinct values in a set of numbers.

Scheduling to minimize average completion time. Given a set of jobs of specified duration to be completed, how can we schedule the jobs on a single processor so as to minimize their average completion time?

##### Proposition H

The following problems reduce to sorting:

* Finding the median
* Counting distinct values
* Scheduling to minimize average completion time

Sorting is valuable for three reasons:

* It is useful in its own right.
* We have an effi cient algorithms for solving it.
* Many problems reduce to it.

##### Shortest-paths reductions

In Section 4.4, we revisited the idea of reduction in the context of shortest-paths algorithms. We considered the following problems, among many others:

Single-source shortest paths in undirected graphs. Given an edge-weighted undirected graph with nonnegative weights and a source vertex s, support queries of the form Is there a path from s to a given target vertex v? If so, find a shortest such path (one whose total weight is minimal).

Parallel precedence-constrained scheduling. Given a set of jobs of specified duration to be completed, with precedence constraints that specify that certain jobs have to be completed before certain other jobs are begun, how can we schedule the jobs on identical processors (as many as needed) such that they are all completed in the minimum amount of time while still respecting the constraints?

Arbitrage. Find an arbitrage opportunity in a given table of currency-conversion rates.

##### Proposition I

The following problems reduce to shortest paths in weighted digraphs:

* Single-source shortest paths in undirected graphs with nonnegative weights
* Parallel precedence-constrained scheduling
* Arbitrage
* [many other problems]

##### Maxflow reductions

Maxflow algorithms are also important in a broad context. We can remove various restrictions on the flow network and solve related flow problems; we can solve other network- and graph-processing problems; and we can solve problems that are not network problems at all.

Job placement. A college’s job-placement office arranges interviews for a set of students with a set of companies; these interviews result in a set of job offers. Assuming that an interview followed by a job offer represents mutual interest in the student taking a job at the company, it is in everyone’s best interests to maximize the number of job placements. Is it possible to match every student with a job? What is the maximum number of jobs that can be filled?

Product distribution. A company that manufactures a single product has factories, where the product is produced; distribution centers, where the product is stored temporarily; and retail outlets, where the product is sold. The company must distribute the product from factories through distribution centers to retail outlets on a regular basis, using distribution channels that have varying capacities. Is it possible to get the product from the warehouses to the retail outlets such that supply meets demand everywhere?

Network reliability. A simplified model considers a computer network as consisting of a set of trunk lines that connect computers through switches such that there is the possibility of a switched path through trunk lines connecting any two given computers. What is the minimum number of trunk lines that can be cut to disconnect some pair of computers?

##### Proposition J

The following problems reduce to the maxflow problem:

* Job placement
* Product distribution
* Network reliability
* [many other problems]

Shortest paths and maxflow are important problem-solving models because they have the same properties that we articulated for sorting:

* They are useful in their own right.
* We have effi cient algorithms for solving them.
* Many problems reduce to them.

This short discussion serves only to introduce the idea.

##### Linear programming

One of the cornerstones of operations research is linear programming (LP). It refers to the idea of reducing a given problem to the following mathematical formulation:

Linear programming. Given a set of M linear inequalities and linear equations involving N variables, and a linear objective function of the N variables, find an assignment of values to the variables that maximizes the objective function, or report that no feasible assignment exists.

Linear programming is an extremely important problem-solving model because

* A great many important problems reduce to linear programming
* We have effi cient algorithms for solving linear-programming problems

##### Proposition K

The following problems reduce to linear programming

* Maxflow
* Shortest paths
* [many, many other problems]

What sorts of problems do not reduce to linear programming? Here is an example of such a problem:

Load balancing. Given a set of jobs of specified duration to be completed, how can we schedule the jobs on two identical processors so as to minimize the completion time of all the jobs?

#### Intractability

The practical utility of most of the algorithms is obvious, and for many problems we have the luxury of several efficient algorithms to choose from. Unfortunately, many other problems arise in practice that do not admit such efficient solutions. What’s worse, for a large class of such problems we cannot even tell whether or not an efficient solution exists. This state of affairs has been a source of extreme frustration for programmers and algorithm designers, who cannot find any efficient algorithm for a wide range of practical problems, and for theoreticians, who have been unable to find any proof that these problems are difficult.

##### Groundwork

One of the most beautiful and intriguing intellectual discoveries of the 20th century, developed by A. Turing in the 1930s, is the Turing machine, a simple model of computation that is general enough to embody any computer program or computing device.

Turing machines form the foundation of theoretical computer science, starting with the following two ideas:

* Universality . All physically realizable computing devices can be simulated by a Turing machine. This idea is known as the Church-Turing thesis. This is a statement about the natural world and cannot be proven (but it can be falsified). The evidence in favor of the thesis is that mathematicians and computer scientists have developed numerous models of computation, but they all have been proven equivalent to the Turing machine.
* Computability . There exist problems that cannot be solved by a Turing machine (or by any other computing device, by universality). This is a mathematical truth. The halting problem (no program can guarantee to determine whether a given program will halt) is a famous example of such a problem.

In the present context, we are interested in a third idea, which speaks to the efficiency of computing devices:

* Extended Church-Turing thesis. The order of growth of the running time of a program to solve a problem on any computing device is within a polynomial factor of some program to solve the problem on a Turing machine (or any other computing device).

##### Exponential running time

The purpose of the theory of intractability is to separate problems that can be solved in polynomial time from problems that (probably) require exponential time to solve in the worst case. It is useful to think of an exponential-time algorithm as one that, for some input of size N, takes time proportional to 2N (at least). The substance of the argument does not change if we replace 2 by any number  > 1.

Shortest-path length. What is the length of the shortest path from a given vertex s to a given vertex t in a given graph?

Longest-path length. What is the length of the longest simple path from a given vertex s to a given vertex t in a given graph?

```
public class LongestPath {
	private boolean[] marked;
	private int max;
	
	public LongestPath(Graph G, int s, int t) {
		marked=new boolean[G.V()];
		dfs(G,s,t,0);
	}
	
	private void dfs(Graph G, int v,int t,int i) {
		if(v==t&&i>max)
			max=i;
		if(v==t)
			return;
		marked[v]=true;
		for(int w: G.adj(v))
			if(!marked[w])
				dfs(G,w,t,i+1);
		marked[v]=false;
	}
	
	public int maxLength() {
		return max;
	}
}
```

##### Search problems

The great disparity between problems that can be solved with “efficient” algorithms of the type we have been studying in this book and problems where we need to look for a solution among a potentially huge number of possibilities makes it possible to study the interface between them with a simple formal model. The first step is to characterize the type of problem that we study:

##### Definition

A search problem is a problem having solutions with the property that the time needed to certify that any solution is correct is bounded by a polynomial in the size of the input. We say that an algorithm solves a search problem if, given any input, it either produces a solution or reports that none exists.

The name NP is commonly used to describe search problems—we will describe the reason for the name on page 914:

##### Definition

NP is the set of all search problems.

Linear equation satisfiability. Given a set of M linear equations involving N variables, find an assignment of values to the variables that satisfies all of the equations, or report that none exists.
Linear inequality satisfiability (search formulation of linear programming). Given a set of M linear inequalities involving N variables, find an assignment of values to the variables that satisfies all of the inequalities, or report that none exists.
0-1 integer linear inequality satisfiability (search formulation of 0-1 integer linear programming). Given a set of M linear inequalities involving N integer variables, find an assignment of the values 0 or 1 to the variables that satisfies all of the inequalities, or report that none exists.
Boolean satisfiability. Given a set of M equations involving and and or operations on N boolean variables, find an assignment of values to the variables that satisfies all of the equations, or report that none exists.

Selected search problems

##### Other types of problems

The concept of search problems is one of many ways to characterize the set of problems that form the basis of the study of intractability. Other possibilities are decision problems (does a solution exist?) and optimization problems (what is the best solution)?

##### Easy search problems

The definition of NP says nothing about the difficulty of finding the solution, just certifying that it is a solution. The second of the two sets of problems that form the basis of the study of intractability, which is known as P, is concerned with the difficulty of finding the solution.

##### Definition

P is the set of all search problems that can be solved in polynomial time.

##### Nondeterminism

The N in NP stands for nondeterminism. It represents the idea that one way (in theory) to extend the power of a computer is to endow it with the power of nondeterminism: to assert that when an algorithm is faced with a choice of several options, it has the power to “guess” the right one.

##### The main question

Nondeterminism is such a powerful notion that it seems almost absurd to consider it seriously.

Does N=NP?

Other ways of posing the question shed light on its fundamental nature:

* Are there any hard-to-solve search problems?
* Would we be able to solve some search problems more efficiently if we could build a nondeterministic computing device?

Poly-time reductions. Recall from page 903 that we show that a problem A reduces to another problem B by demonstrating that we can solve any instance of A in three steps:

* Transform it to an instance of B.
* Solve that instance of B.
* Transform the solution of B to be a solution of A.

##### Proposition L

Boolean satisfi ability poly-time reduces to 0-1 integer linear inequality satisfiability.

##### Corollary

If satisfi ability is hard to solve, then so is integer linear programming.

##### NP-completeness

Remarkably, all of these many, many problems have an additional property that provides convincing evidence that P ![](http://latex.codecogs.com/gif.latex?\neq) NP :

##### Definition

A search problem A is said to be NP-complete if all problems in NP polytime reduce to A.

##### Cook-Levin theorem

Reduction uses the NP-completeness of one problem to imply the NP-completeness of another. But reduction cannot be used in one case: how was the first problem proven to be NP-complete?

##### Proposition M (Cook-Levin theorem)

Boolean satisfiability is NP-complete.

##### Classifying problems

Classifying problems as being easy to solve (in P) or hard to solve (NP-complete) can be:

* Straightforward. For example, the venerable Gaussian elimination algorithm proves that linear equation satisfiability is in P.
* Tricky but not difficult. For example, developing a proof like the proof of Proposition A takes some experience and practice, but it is easy to understand.
* Extremely challenging. For example, linear programming was long unclassified, but Khachian’s ellipsoid algorithm proves that linear programming is in P.
* Open. For example, graph isomorphism (given two graphs, find a way to rename the vertices of one to make it identical to the other) and factor (given an integer, find a nontrivial factor) are still unclassified.

Boolean satisfiability. Given a set of M equations involving N boolean variables, find an assignment of values to the variables that satisfies all of the equations, or report that none exists.
Integer linear programming. Given a set of M linear inequalities involving N integer variables, find an assignment of values to the variables that satisfies all of the inequalities, or report that none exists.
Load balancing. Given a set of jobs of specified duration to be completed and a time bound T, how can we schedule the jobs on two identical processors so as to complete them all by time T?
Vertex cover. Given a graph and a integer C, find a set of C vertices such that each edge of the graph is incident to at least one vertex of the set.
Hamiltonian path. Given a graph, find a simple path that visits each vertex exactly once, or report that none exists.
Protein folding. Given energy level M, find a folded three-dimensional conformation of a protein having potential energy less than M.
Ising model. Given an Ising model on a lattice of dimension three and an energy threshhold E, is there a subgraph with free energy less than E ?
Risk portfolio of a given return. Given an investment portfolio with a given total cost, a given return, risk values assigned to each investment, and a threshold M, find a way to allocate the investments such that the risk is less than M.

Some famous NP-complete problems

##### Coping with NP-completeness

Some sort of solution to this vast panopoly of problems must be found in practice, so there is intense interest in finding ways to address them. It is impossible to do justice to this vast field of study in one paragraph, but we can briefly describe various approaches that have been tried. One approach is to change the problem and find an “approximation” algorithm that finds not the best solution but a solution guaranteed to be close to the best.