### Binary Search Trees

We are working with data structures made up of nodes that contain links that are either null or references to other nodes. In a binary tree, we have the restriction that every node is pointed to by just one other node, which is called its parent (except for one node, the root, which has no nodes pointing to it), and that each node has exactly two links, which are called its left and right links, that point to nodes called its left child and right child, respectively. Although links point to nodes, we can view each link as pointing to a binary tree, the tree whose root is the referenced node. Thus, we can define a binary tree as a either a null link or a node with a left link and a right link, each references to (disjoint) subtrees that are themselves binary trees. In a binary search tree, each node also has a key and a value, with an ordering restriction to support efficient search.

##### Definition

A binary search tree (BST) is a binary tree where each node has a Comparable key (and an associated value) and satisfies the restriction that the key in any node is larger than the keys in all nodes in that node’s left subtree and smaller than the keys in all nodes in that node’s right subtree.

#### Basic implementation

Algorithm 3.3 defines the BST data structure that we use throughout this section to implement the ordered symbol-table API.

##### Representation

We define a private nested class to define nodes in BSTs, just as we did for linked lists. Each node contains a key, a value, a left link, a right link, and a node count (when relevant, we include node counts in red above the node in our drawings). The left link points to a BST for items with smaller keys, and the right link points to a BST for items with larger keys. The instance variable N gives the node count in the subtree rooted at the node.

size(x) = size(x.left) + size(x.right) + 1

If we project the keys in a BST such that all keys in each node’s left subtree appear to the left of the key in the node and all keys in each node’s right subtree appear to the right of the key in the node, then we always get the keys in sorted order.

##### Search

If a node containing the key is in the table, we have a search hit, so we return the associated value. Otherwise, we have a search miss (and return null). A recursive algorithm to search for a key in a BST follows immediately from the recursive structure: if the tree is empty, we have a search miss; if the search key is equal to the key at the root, we have a search hit. Otherwise, we search (recursively) in the appropriate subtree, moving left if the search key is smaller, right if it is larger.

##### Insert

The search code in Algorithm 3.3 is almost as simple as binary search; that simplicity is an essential feature of BSTs. A more important essential feature of BSTs is that insert is not much more difficult to implement than search. Indeed, a search for a key not in the tree ends at a null link, and all that we need to do is replace that link with a new node containing the key (see the diagram on the next page).

##### Recursion

It is worthwhile to take the time to understand the dynamics of these recursive implementations. You can think of the code before the recursive calls as happening on the way down the tree: it compares the given key against the key at each node and moves right or left accordingly.

##### Implementation
```
public class BST<Key extends Comparable<Key>,Value {

    private Node root;

    private class Node{
        private Key key;
        private Value value;
        private Node left, right;
        private int N;
        public Node(Key key, Value value){
            this.key=key;
            this.value=value;
        }
    }

    public int size(){
        return size(root);
    }

    private int size(Node x){
        if(x==null)
            return 0;
        else
            return x.N;
    }

    public int size(){
        return N;
    }

    public Value get(Key key){
        return get(root,key);
    }

    public Value get(Node x,Key key){
        if(x==null)
            return null;
        int cmp=key.compareTo(x.key);
        if(cmp<0)
            return get(x.left,key);
        else if(cmp>0)
            return get(x.right,key);
        else
            return x.value;
    }

    public int rand(Key key){
        int low=0,high=N-1;
        while(low<=high<N-1){
            int mid=low+(high-low)/2;
            int cmp=key.compareTo(keys[mid]);
            if(cmp<0)
                high=mid-1;
            else if(cmp>0)
                low=mid+1;
            else
                return mid;
        }
    }

    public void put(Key key,Value value){
        root=put(root,key,value);
    }

    private Node put(Node x,Key key, Value value){
        if(x==null)
            return new Node(key,value,1);
        int cmp=key.compareTo(x.key);
        if(cmp<0)
            x.left=put(x.left,key,value);
        else if(cmp>0)
            x.right=put(x.right,key,value);
        else
            x.value=value;
        x.N=size(x.left)+size(x.right)+1;
        return x;
    }

}
```

#### Analysis

The running times of algorithms on binary search trees depend on the shapes of the trees, which, in turn, depend on the order in which keys are inserted. In the best case, a tree with N nodes could be perfectly balanced, with ~![](http://latex.codecogs.com/gif.latex?NlgN) nodes between the root and each null link. In the worst case there could be N nodes on the search path. The balance in typical trees turns out to be much closer to the best case than the worst case.

##### Proposition C

Search hits in a BST built from N random keys require ~![](http://latex.codecogs.com/gif.latex?2lnN) (about ![](http://latex.codecogs.com/gif.latex?1.39lgN)) compares, on the average.

##### Proposition D

Insertions and search misses in a BST built from N random keys require ~![](http://latex.codecogs.com/gif.latex?2lnN) (about ![](http://latex.codecogs.com/gif.latex?1.39lgN)) compares, on the average.

This prediction has at least thefollowing inherent inaccuracies:

* Many operations are for smaller tables.
* The keys are not random.
* The table size may be too small for the approximation 2 ln N to be accurate.

#### Order-based methods and deletion

An important reason that BSTs are widely used is that they allow us to keep the keys in order.

##### Minimum and maximum

If the left link of the root is null, the smallest key in a BST is the key at the root; if the left link is not null, the smallest key in the BST is the smallest key in the subtree rooted at the node referenced by the left link.

##### Floor and ceiling

If a given key key is less than the key at the root of a BST, then the floor of key (the largest key in the BST less than or equal to key) must be in the left subtree. If key is greater than the key at the root, then the floor of key could be in the right subtree, but only if there is a key smaller than or equal to key in the right subtree; if not (or if key is equal to the key at the root), then the key at the root is the floor of key.

##### Selection

Selection in a BST works in a manner analogous to the partition-based method of selection in an array that we studied in Section 2.5. We maintain in BST nodes the variable N that counts the number of keys in the subtree rooted at that node precisely to support this operation.

##### Rank

The inverse method rank() that returns the rank of a given key is similar: if the given key is equal to the key at the root, we return the number of keys t in the left subtree; if the given key is less than the key at the root, we return the rank of the key in the left subtree (recursively computed); and if the given key is larger than the key at the root, we return t plus one (to count the key at the root) plus the rank of the key in the right subtree (recursively computed).

##### Delete the minimum/maximum

The most difficult BST operation to implement is the delete() method that removes a key-value pair from the symbol table. As a warmup, consider deleteMin() (remove the key-value pair with the smallest key). For deleteMin() we go left until finding a Node that has a null left link and then replace the link to that node by its right link (simply by returning the right link in the recursive method). The deleted node, with no link now pointing to it, is available for garbage collection.

##### Delete

The replacement preserves order in the tree because there are no keys between x.key and the successor’s key. We can accomplish the task of replacing x by its successor in four (!) easy steps:

* Save a link to the node to be deleted in t.
* Set x to point to its successor min(t.right).
* Set the right link of x (which is supposed to point to the BST containing all the keys larger than x.key) to deleteMin(t.right), the link to the BST containing all the keys that are larger than x.key after the deletion.
* Set the left link of x (which was null) to t.left
(all the keys that are less than both the deleted
key and its successor).

##### Range queries

To implement the keys() method that returns the keys in a given range, we begin with a basic recursive BST traversal method, known as inorder traversal. To illustrate the method, we consider the task of printing all the keys in a BST in order.

###### Proposition E

In a BST, all operations take time proportional to the height of the tree, in the worst case.

<table>
    <tr>
        <th rowspan="2">algorithm(data structure)</th>
        <th colspan="2">worst-case cost(after N inserts)</th>
        <th colspan="2">average-case cost(after N random inserts)</th>
        <th rowspan="2">efficiently support ordered operations?</th>
    </tr>
    <tr>
        <th>search</th>
        <th>insert</th>
        <th>search hit</th>
        <th>insert</th>
    </tr>
    <tr>
        <th>sequential search(unordered linked list)</th>
        <th>N</th>
        <th>N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{2}></img></th>
        <th>N</th>
        <th>No</th>
    </tr>
    <tr>
        <th>binary search(ordered array)</th>
        <th><img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th>2N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{2}></img></th>
        <th>yes</th>
    </tr>
    <tr>
        <th>binary tree search(BST)</th>
        <th>N</th>
        <th>N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.39lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.39lgN></img></th>
        <th>yes</th>
    </tr>
</table>