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