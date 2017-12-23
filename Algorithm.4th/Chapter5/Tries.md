### Tries

Specifically, the methods that we consider in this section achieve the following performance characteristics in typical applications, even for huge tables:

* Search hits take time proportional to the length of the search key.
* Search misses involve examining only a few characters.

This API differs from the symbol-table API introduced in Chapter 3 in the following aspects:

* We replace the generic type Key with the concrete type String.
* We add three new methods, longestPrefixOf(), keysWithPrefix() and keysThatMatch().

We retain the basic conventions of our symbol-table implementations in Chapter 3 (no duplicate or null keys and no null values).

The following descriptions of the three new methods use the keys she sells sea shells by the sea shore to give examples:

* longestPrefixOf() takes a string as argument and returns the longest key in the symbol table that is a prefix of that string. For the keys above, longestPrefixOf ("shell") is she and longestPrefixOf("shellsort") is shells.
* keysWithPrefix() takes a string as argument and returns all the keys in the symbol table having that string as prefix. For the keys above, keysWithPrefix("she") is she and shells, and keysWithPrefix ("se") is sells and sea.
* keysThatMatch() takes a string as argument and returns all the keys in the symbol table that match that string, in the sense that a period (.) in the argument string matches any character. For the keys above, keysThatMatch (".he") returns she and the, and keysThatMatch("s..") returns she and sea.

#### Tries

In this section, we consider a search tree known as a trie, a data structure built from the characters of the string keys that allows us to use the characters of the search key to guide the search. The name “trie” is a bit of wordplay introduced by E. Fredkin in 1960 because the data structure is used for retrieval, but we pronounce it “try” to avoid confusion with “tree.”

##### Basic properties

As with search trees, tries are data structures composed of nodes that contain links that are either null or references to other nodes.

##### Search in a trie

Finding the value associated with a given string key in a trie is a simple process, guided by the characters in the search key.

We start at the root, then follow the link associated with the first character in the key; from that node we follow the link associated with the second character in the key; from that node we follow the link associated with the third character in the key and so forth, until reaching the last character of the key or a null link. At this point, one of the following three conditions holds (refer to the figure above for examples):

* The value at the node corresponding to the last character in the key is not null (as in the searches for shells and she depicted at left above). This result is a search hit—the value associated with the key is the value in the node corresponding to its last character.
* The value in the node corresponding to the last character in the key is null (as in the search for shell depicted at top right above). This result is a search miss: the key is not in the table.
* The search terminated with a null link (as in the search for shore depicted at bottom right above). This result is also a search miss.

In all cases, the search is accomplished just by examining nodes along a path from the root to another node in the trie.

Insertion into a trie. As with binary search trees, we insert by first doing a search: in a trie that means using the characters of the key to guide us down the trie until reaching the last character of the key or a null link. At this point, one of the following two conditions holds:

* We encountered a null link before reaching the last character of the key. In this case, there is no trie node corresponding to the last character in the key, so we need to create nodes for each of the characters in the key not yet encountered and set the value in the last one to the value to be associated with the key.
* We encountered the last character of the key before reaching a null link. In this case, we set that node’s value to the value to be associated with the key (whether or not that value is null), as usual with our associative array convention.

In all cases, we examine or create a node in the trie for each key character.

##### Node representation

As mentioned at the outset, our trie diagrams do not quite correspond to the data structures our programs will build, because we do not draw null links. Taking null links into account emphasizes the following important characteristics of tries:

* Every node has R links, one for each possible character.
* Characters and keys are implicitly stored in the data structure.

##### Size

As for the binary search trees of Chapter 3, three straightforward options are available for implementing size():

* An eager implementation where we maintain the number of keys in an instance variable N.
* A very eager implementation where we maintain the number of keys in a subtrie as a node instance variable that we update after the recursive calls in put() and delete().
* A lazy recursive implementation like the one at right. It traverses all of the nodes in the trie, counting the number having a non-null value.

As with binary search trees, the lazy implementation is instructive but should be avoided because it can lead to performance problems for clients. The eager implementations are explored in the exercises.

```
public class TrieST<Value> {
	
	private static int R=256;
	private Node root;
	
	private static class Node {
		private Object val;
		private Node[] next=new Node[R];
	}
	
	public Value get(String key) {
		Node x=get(root, key, 0);
		if(x==null)
			return null;
		return (Value)x.val;
	}
	
	private Node get(Node x, String key, int d) {
		if(x==null)
			return null;
		if(d==key.length())
			return x;
		char c=key.charAt(d);
		return get(x.next[c],key,d+1);
	}
	
	public void put(String key, Value val) {
		root=put(root,key,val,0);
	}
	
	private Node put(Node x, String key, Value val, int d) {
		if(x==null)
			x=new Node();
		if(d==key.length()) {
			x.val=val;
			return x;
		}
		char c=key.charAt(d);
		x.next[c]=put(x.next[c],key,val,d+1);
		return x;
	}

}
```

##### Collecting keys

Because characters and keys are represented implicitly in tries, providing clients with the ability to iterate through the keys presents a challenge. As with binary search trees, we accumulate the string keys in a Queue, but for tries we need to create explicit representations of all of the string keys, not just find them in the data structure. We do so with a recursive private method collect() that is similar to size() but also maintains a string with the sequence of characters on the path from the root.

```
public Iterable<String> keys(){
    return keysWithPrefix("");
}

public Iterable<String> keysWithPrefix(String pre){
    Queue<String> q=new Queue<String>();
    collect(get(root,pre,0),pre,q);
    return q;
}

private void collect(Node x, String pre, Queue<String> q) {
    if(x==null)
        return;
    if(x.val!=null)
        q.enqueue(pre);
    for(char c=0;c<R;c++) {
        collect(x.next[c],pre+c,q);
    }
}
```

##### Wildcard match

To implement keysThatMatch(), we use a similar process, but add an argument specifying the pattern to collect() and add a test to make a recursive call for all links when the pattern character is a wildcard or only for the link corresponding to the pattern character otherwise, as in the code below. Note also that we do not need to consider keys longer than the pattern.

##### Longest prefix

To find the longest key that is a prefix of a given string, we use a recursive method like get() that keeps track of the length of the longest key found on the search path (by passing it as a parameter to the recursive method, updating the value whenever a node with a non-null value is encountered). The search ends when the end of the string or a null link is encountered, whichever comes first.


```
public Iterable<String> keysThatMatch(String pat){
	Queue<String> q=new Queue<String>();
	collect(root,"",pat,q);
	return q;
}

public void collect(Node x,String pre, String pat, Queue<String> q) {
	int d=pre.length();
	if(x==null)
		return;
	if(d==pat.length()&&x.val!=null)
		q.enqueue(pre);
	if(d==pat.length())
		return;
	char next=pat.charAt(d);
	for(char c=0;c<R;c++) {
		if(next=='.'||next==c) {
			collect(x.next[c],pre+c,pat,q);
		}
	}
}
```

```
public String longestPrefixOf(String s) {
	int length=search(root, s,0,0);
	return s.substring(0,length);
}

private int search(Node x, String s, int d, int length) {
	if(x==null)
		return length;
	if(x.val!=null)
		length=d;
	if(d==s.length())
		return length;
	char c=s.charAt(d);
	return search(x.next[c],s,d+1,length);
}
```

##### Deletion

The first step needed to delete a key-value pair from a trie is to use a normal search to find the node corresponding to the key and set the corresponding value to null. If that node has a non-null link to a child, then no more work is required; if all the links are null, we need to remove the node from the data structure. If doing so leaves all the links null in its parent, we need to remove that node, and so forth.

Alphabet

As usual, Algorithm 5.4 is coded for Java String keys, but it is a simple matter to modify the implementation
to handle keys taken from any alphabet, as follows:

* Implement a constructor that takes an Alphabet as argument, which sets an Alphabet instance variable to that argument value and the instance variable R to the number of characters in the alphabet.
* Use the toIndex() method from Alphabet in get() and put() to convert string characters to indices between 0 and R-1.
* Use the toChar() method from Alphabet to convert indices between 0 and R-1 to char values. This operation is not needed in get() and put() but is important in the implementations of keys(), keysWithPrefix(), and keysThatMatch().

#### Properties of tries

As usual, we are interested in knowing the amount of time and space required to use tries in typical applications. Tries have been extensively studied and analyzed, and their basic properties are relatively easy to understand and to apply.

##### Proposition F

The linked structure (shape) of a trie is independent of the key insertion/deletion order: there is a unique trie for any given set of keys.

##### Worst-case time bound for search and insert

For BSTs, hashing, and other methods in Chapter 4, we needed mathematical analysis to study this question, but for tries it is very easy to answer:

##### Proposition G

The number of array accesses when searching in a trie or inserting a key into a trie is at most 1 plus the length of the key.

##### Expected time bound for search miss

Suppose that we are searching for a key in a trie and find that the link in the root node that corresponds to its first character is null.

##### Proposition H

The average number of nodes examined for search miss in a trie built from N random keys over an alphabet of size R is ~![](http://latex.codecogs.com/gif.latex?log_{R}N) .

##### Space

How much space is needed for a trie? Addressing this question (and understanding how much space is available) is critical to using tries effectively.

##### Proposition I

The number of links in a trie is between RN and RNw, where w is the average key length.

The table on the facing page shows the costs for some typical applications that we have considered. It illustrates the following rules of thumb for tries:

* When keys are short, the number of links is close to RN.
* When keys are long, the number of links is close to RNw.
* Therefore, decreasing R can save a huge amount of space.

##### One-way branching

The primary reason that trie space is excessive for long keys is that long keys tend to have long tails in the trie, with each node having a single link to the next node (and, therefore, R-1 null links).

#### Ternary search tries (TSTs)

To help us avoid the excessive space cost associated with R-way tries, we now consider an alternative representation: the ternary search trie (TST). In a TST, each node has a character, three links, and a value. The three links correspond to keys whose current characters are less than, equal to, or greater than the node’s character. In the R-way tries of Algorithm 5.4, trie nodes are represented by R links, with the character corresponding to each non-null link implictly represented by its index.

##### Search and insert

The search and insert code for implementing our symbol-table API with TSTs writes itself. To search, we compare the first character in the key with the character at the root. If it is less, we take the left link; if it is greater, we take the right link; and if it is equal, we take the middle link and move to the next search key character.

```
public class TST<Value> {
	
	private Node root;
	
	private class Node {
		char c;
		Node left, mid, right;
		Value val;
	}
	
	public Value get(String key)
	
	private Node get(Node x, String key, int d) {
		char c=key.charAt(d);
		if(c<x.c)
			return get(x.left,key,d);
		else if(c>x.c)
			return get(x.right,key,d);
		else if(d<key.length()-1)
			return get(x.mid,key,d+1);
		else
			return x;
	}
	
	public void put(String key, Value val) {
		root=put(root,key,val,0);
	}
	
	private Node put(Node x,String key, Value val, int d) {
		char c=key.charAt(d);
		if(x==null) {
			x=new Node();
			x.c=c;
		}
		if(c<x.c)
			x.left=put(x.left,key,val,d);
		else if(c>x.c)
			x.right=put(x.right,key,val,d);
		else if(d<key.length()-1)
			x.mid=put(x.mid,key,val,d+1);
		else
			x.val=val;
		return x;
	}

}
```

##### Properties of TSTs

A TST is a compact representation of an R-way trie, but the two data structures have remarkably different properties. Perhaps the most important difference is that Property A does not hold for TSTs: the BST representations of each trie node depend on the order of key insertion, as with any other BST.

##### Space

The most important property of TSTs is that they have just three links in each node, so a TST requires far less space than the corresponding trie.

##### Proposition J

The number of links in a TST built from N string keys of average length w is between 3N and 3Nw.

##### Search cost

To determine the cost of search (and insert) in a TST, we multiply the cost for the corresponding trie by the cost of traversing the BST representation of each trie node..

##### Proposition K

A search miss in a TST built from N random string keys requires ~![](http://latex.codecogs.com/gif.latex?lnN) character compares, on the average. A search hit or an insertion in a TST uses a character compare for each character in the search key.

##### Alphabet

The prime virtue of using TSTs is that they adapt gracefully to irregularities in search keys that are likely to appear in practical applications. In particular, note that there is no reason to allow for strings to be built from a client-supplied alphabet, as was crucial for tries. There are two main effects. First, keys in practical applications come from large alphabets, and usage of particular characters in the character sets is far from uniform. With TSTs, we can use a 256-character ASCII encoding or a 65,536-character Unicode encoding without having to worry about the excessive costs of nodes with 256- or 65,536-way branching, and without having to determine which sets of characters are relevant. Unicode strings in non-Roman alphabets can have thousands of characters—TSTs are especially appropriate for standard Java String keys that consist of such characters. Second, keys in practical applications often have a structured format, differing from application to application, perhaps using only letters in one part of the key, only digits in another part of the key.

##### Prefix match, collecting keys, and wildcard match

Since a TST represents a trie, implementations of longestPrefixOf(), keys(), keysWithPrefix(), and keysThatMatch() are easily adapted from the corresponding code for tries in the previous section, and a worthwhile exercise for you to cement your understanding of both tries and TSTs (see Exercise 5.2.9).

##### Deletion

The delete() method for TSTs requires more work. Essentially, each character in the key to be deleted belongs to a BST. In a trie, we could remove the link corresponding to a character by setting the corresponding entry in the array of links to null; in a TST, we have to use BST node deletion to remove the node corresponding to
the character.

##### Hybrid TSTs

An easy improvement to TST-based search is to use a large explicit multiway node at the root. The simplest way to proceed is to keep a table of R TSTs: one for each possible value of the first character in the keys. If R is not large, we might use the first two letters of the keys (and a table of size ![](http://latex.codecogs.com/gif.latex?R^2)).

##### One-way branching

Just as with tries, we can make TSTs more efficient in their use of space by putting keys in leaves at the point where they are distinguished and by eliminating one-way branching between internal nodes.

##### Proposition L

A search or an insertion in a TST built from N random string keys with no external one-way branching and R t-way branching at the root requires roughly ![](http://latex.codecogs.com/gif.latex?lnN-tlnR) character compares, on the average.

<table>
    <tr>
        <th rowspan="2">algorithm(data structure)</th>
        <th colspan="2">typical growth rate for N strings from an R-character alphabet (average length w)</th>
		<th rowspan="2">sweet spot</th>
    </tr>
    <tr>
        <th>characters examined for search miss</th>
        <th>memory usage</th>
    </tr>
    <tr>
        <th>binary tree search(BST)</th>
		<th><img src=http://latex.codecogs.com/gif.latex?c_{1}(lgN)^2></img></th>
		<th>64N</th>
		<th>randomly ordered keys</th>
    </tr>
    <tr>
        <th>2-3 tree search(red-black BST)</th>
		<th><img src=http://latex.codecogs.com/gif.latex?c_{2}(lgN)^2></img></th>
		<th>64</th>
		<th>guaranteed performance</th>
    </tr>
	<tr>
        <th>linear probing(parallel arrays</th>
		<th>w</th>
		<th>32N to 128N</th>
		<th>built-in types cached hash values</th>
    </tr>
	<tr>
        <th>trie search(R-way trie)</th>
		<th><img src=http://latex.codecogs.com/gif.latex?log_{R}N></img></th>
		<th>(8R+56)N to (8R+56)Nw</th>
		<th>short keys small alphabets</th>
    </tr>
	<tr>
        <th>trie search(TST)</th>
		<th><img src=http://latex.codecogs.com/gif.latex?1.39lgN></img></th>
		<th>64N to 64Nw</th>
		<th>nonrandom keys</th>
    </tr>
</table>
Performance characteristics of string-search algorithms