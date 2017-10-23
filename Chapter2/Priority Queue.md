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

```
public class MaxPQ<Key extends Comparable<Key>> {
    
    private Key[] pq;
    private int N=0;
    
    public MaxPQ(int maxN){
        pq=(Key[])new Comparable[maxN+1];
    }
    
    public boolean isEmpty(){
        return N==0;
    }
    
    public int size(){
        return N;
    }
    
    public void insert(Key v){
        pq[++N]=v;
        swim(N);
    }
    
    public Key delMax(){
        Key max=pq[1];
        exch(1,N--);
        sink(1);
        return max;
    }

    //See pages 145-147 for implementations of these helper methods.
    private boolean less(int i,int j);
    private void exch(int i,int j);
    private void swim(int k);
    private void sink(int k);
}
```

##### Proposition Q

In an N-key priority queue, the heap algorithms require no more than ![](http://latex.codecogs.com/gif.latex?1+logN)compares for insert and no more than ![](http://latex.codecogs.com/gif.latex?2logN) compares for remove the maximum.

##### Multiway heaps

It is not difficult to modify our code to build heaps based on an array representation of complete heap-ordered ternary trees, with an entry at position k larger than or equal to entries at positions 3k-1, 3k, and 3k+1 and smaller than or equal to entries at position ![](http://latex.codecogs.com/gif.latex?\lfloor(\frac{k+1}/{3})\rfloor), for all indices between 1 and N in an array of N items, and not much more difficult to use d-ary heaps for any given d. There is a tradeoff between the lower cost from the reduced tree height ![](http://latex.codecogs.com/gif.latex?log_{b}N) and the higher cost of finding the largest of the d children at each node. This tradeoff is dependent on details of the implementation and the expected relative frequency of operations.

##### Array resizing

We can add a no-argument constructor, code for array doubling in insert(), and code for array halving in delMax(), just as we did for stacks in Section 1.3. Thus, clients need not be concerned about arbitrary size restrictions. The logarithmic time bounds implied by PROPOSITION Q are amortized when the size of the priority queue is arbitrary and the arrays are resized.

##### Immutability of keys

The priority queue contains objects that are created by clients but assumes that client code does not change the keys (which might invalidate the heap-order invariant). It is possible to develop mechanisms to enforce this assumption, but programmers typically do not do so because they complicate the code and are likely to degrade performance.

##### Index priority queue

In many applications, it makes sense to allow clients to refer to items that are already on the priority queue. One easy way to do so is to associate a unique integer index with each item. Moreover, it is often the case that clients have a universe of items of a known size N and perhaps are using (parallel) arrays to store information about the items, so other unrelated client code might already be using an integer index to refer to items.

##### Proposition Q (continued)

In an index priority queue of size N, the number of compares required is proportional to at most ![](http://latex.codecogs.com/gif.latex?logN) for insert, change priority, delete, and remove the minimum.

##### Index priority-queue client

The IndexMinPQ client Multiway on page 322 solves the multiway merge problem: it merges together several sorted input streams into one sorted output stream.

```
public class Multiway {

    public static void merge(In[] streams){
        int N=streams.length;
        IndexMinPQ<String> pq=new IndexMinPQ<String>(N);
        
        for(int i=0;i<N;i++){
            if(!streams[i].isEmpty())
                pq.insert(i,streams[i].readString());
        }
        
        while(!pq.isEmpty()){
            StdOut.println(pq.min());
            int i=pq.delMin();
            if(!streams[i].isEmpty()){
                pq.insert(i,streams[i].readString());
            }
        }
    }
}
```

#### Heapsort

Heapsort breaks into two phases: heap construction, where we reorganize the original array into a heap, and the sortdown, where we pull the items out of the heap in decreasing order to build the sorted result. For consistency with the code we have studied, we use a maximum-oriented priority queue and repeatedly remove the maximum. Focusing on the task of sorting, we abandon the notion of hiding the representation of the priority queue and use swim() and sink() directly. Doing so allows us to sort an array without needing any extra space, by maintaining the heap within the array to be sorted.

##### Heap construction

As the first phase of a sort, heap construction is a bit counterintuitive, because its goal is to produce a
heap-ordered result, which has the largest item first in the array (and other larger items near the beginning), not at the end, where it is destined to finish.

#### Proposition R

Sink-based heap construction uses fewer than 2N compares and fewer than N exchanges to construct a heap from N items.

```
public static void sort(Comparable[] a){
    int N=a.length;
    for(int k=N/2;k>=1;k--){
        sink(a,k,N);
    }
    while(N>1){
        exch(a,1,N--);
        sink(a,1,N);
    }
}
```

#### Sortdown

Most of the work during heapsort is done during the second phase, where we remove the largest remaining item from the heap and put it into the array position vacated as the heap shrinks. This process is a bit like selection sort (taking the items in decreasing order instead of in increasing order), but it uses many fewer compares because the heap provides a much more efficient way to find the largest item in the unsorted part of the array.

#### Proposition S

Heapsort uses fewer than ![](http://latex.codecogs.com/gif.latex?2NlgN+2N) compares (and half that many exchanges) to sort N items.

#### Sink to the bottom, then swim

Most items reinserted into the heap during sortdown go all the way to the bottom. Floyd observed in 1964 that we can thus save time by avoiding the check for whether the item has reached its position, simply promoting the larger of the two children until the bottom is reached, then moving back up the heap to the proper position. This idea cuts the number of compares by a factor of 2 asymptotically—close to the number used by mergesort (for a randomly-ordered array). The method requires extra bookkeeping, and it is useful in practice only when the cost of compares is relatively high (for example, when we are sorting items with strings or other types of long keys).