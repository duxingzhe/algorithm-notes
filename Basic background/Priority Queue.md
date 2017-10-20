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

##### Array representation (unordered)
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

#### Algorithms on heaps

There are two cases. When the priority of some node is increased (or a new node is added at the bottom of a heap), we have to travel up the heap to restore the heap order. When the priority of some node is decreased (for example, if we replace the node at the root with a new node that has a smaller key), we have to travel down the heap to restore the heap order. First, we will consider how to implement these two basic auxiliary operations; then, we shall see how to use them to implement insert and remove the maximum.

Compare and exchange methods for heap implementations
```
private boolean less(int i, int j){
    return pq[i].compareTo(pq[j]) < 0;
}
private void exch(int i, int j){
    Key t = pq[i];
    pq[i] = pq[j];
    pq[j] = t;
}
```

##### Bottom-up reheapify (swim)

If the heap order is violated because a node’s key becomes larger than that node’s parent’s key, then we can make progress toward fixing the violation by exchanging the node with its parent. After the exchange, the node is larger than both its children (one is the old parent, and the other is smaller than the old parent because it was a child of that node) but the node may still be larger than its parent. We can fix that violation in the same way, and so forth, moving up the heap until we reach a node with a larger key, or the root. Coding this process is straightforward when you keep in mind that the parent of the node at position k in a heap is at position k/2. The loop in swim() preserves the invariant that the only place the heap order could be violated is when the node at position k might be larger than its parent. Therefore, when we get to a place where that node is not larger than its parent, we know that the heap order is satisfied throughout the heap. To justify the method’s name, we think of the new node, having too large a key, as having to swim to a higher level in the heap.

```
private void swim(int k)
{
    while (k > 1 && less(k/2, k))
    {
        exch(k/2, k);
        k = k/2;
    }
}
```

##### Top-down reheapify (sink)

If the heap order is violated because a node’s key becomes smaller than one or both of that node’s children’s keys, then we can make progress toward fixing the violation by exchanging the node with the larger of its two children. This switch may cause a violation at the child; we fix that violation in the same way, and so forth, moving down the heap until we reach a node with both children smaller (or equal), or the bottom. The code again follows directly from the fact that the children of the node at position k in a heap are at positions 2k and 2k+1. To justify the method’s name, we think about the node, having too small a key, as having to sink to a lower level in the heap.

```
private void sink(int k)
{
    while (2*k <= N)
    {
        int j = 2*k;
        if (j < N && less(j, j+1)) j++;
        if (!less(k, j)) break;
        exch(k, j);
        k = j;
    }
}
```

###### Insert

We add the new key at the end of the array, increment the size of the heap, and then swim up through the heap with that key to restore the heap condition. 

###### Remove the maximum

We take the largest key off the top, put the item from the end of the heap at the top, decrement the size of the heap, and then sink down through the heap with that key to restore the heap condition.