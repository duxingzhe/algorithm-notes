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