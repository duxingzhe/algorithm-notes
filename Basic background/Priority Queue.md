### Piriority Queue

Costs of finding the largest M in a stream of N items
<table>
    <tr>
        <th rowspan="2">Client</th>
        <th colspan="2">order of growth</th>
    </tr>
    <tr>
        <th>time</th>
        <th>space</th>
    </tr>
    <tr>
        <th>sort client</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th>N</th>
    </tr>
    <tr>
        <th>PQ client using elementary implementation</th>
        <th>NM</th>
        <th>M</th>
    </tr>
    <tr>
        <th>PQ client using heap-based implementation</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogM"></th>
        <th>M</th>
    </tr>
</table>

#### Elementary Implementations

##### Array representation
```
public class TopM {

    public static void main(String[] args){
        int M=Integer.parseInt(args[0]);
        MinPQ<Transaction>pq=new MinPQ<Transaction>(M+1);
        while(StdIn.hasNextline()){
            pq.insert(new Transaction(StdIn.readLine()));
            if(pq.size()>M)
                pq.delMin();
        }
        Stack<Transaction> stack=new Stack<Transaction>();
        while(!pq.isEmpty()) stack.push(pq.delMin());
        for(Transaction t: stack)
            StdOut.println(t);
    }

}
```

##### Array representation (ordered)

Another approach is to add code for insert to move larger entries one position to the right, thus keeping the keys in the array in order (as in insertion sort). Thus, the largest element is always at the end, and the code for remove the maximum in the priority queue is the same as for pop in the stack.

##### Linked-list representations

Similarly, we can start with our linked-list code for pushdown stacks, modifying either the code for pop() to find and return the maximum or the code for push() to keep keys in reverse order and the code for pop() to unlink and return the first (maximum) item on the list.

#### Heap definitions

The binary heap is a data structure that can efficiently support the basic priority-queue operations. In a binary heap, the keys are stored in an array such that each key is guaranteed to be larger than (or equal to) the keys at two other specific positions. In turn, each of those keys must be larger than (or equal to) two additional keys, and so forth. This ordering is easy to see if we view the keys as being in a binary tree structure with edges from each key to the two keys known to be smaller.

##### Definition

A binary tree is heap-ordered if the key in each node is larger than or equal to the keys in that node’s two children (if any).

##### Proposition O

The largest key in a heap-ordered binary tree is found at the root.

##### Binary heap representation

If we use a linked representation for heap-ordered binary trees, we would need to have three links associated with each key to allow travel up and down the tree (each node would have one pointer to its parent and one to each child). It is particularly convenient, instead, to use a complete binary tree like the one drawn at right. We draw such a structure by placing the root node and then proceeding down the page and from left to right, drawing and connecting two nodes beneath each node on the previous level until we have drawn N nodes.

##### Definition
A binary heap is a collection of keys arranged in a complete heap-ordered binary tree, represented in level order in an array (not using the first entry).

##### Proposition P

The height of a complete binary tree of size N is ![](http://latex.codecogs.com/gif.latex?\lfloor(lgN)\rfloor) .