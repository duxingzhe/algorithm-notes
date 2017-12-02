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