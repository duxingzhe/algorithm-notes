### Application

#### Sorting various types of data

Our implementations sort arrays of Comparable objects. This Java convention allows us to use Java’s callback mechanism to sort arrays of objects of any type that implements the Comparable interface. As described in Section 2.1, implementing Comparable amounts to defining a compareTo() method that implements a natural ordering for the type. We can use our code immediately to sort arrays of type String, Integer, Double, and other types such as File and URL, because these data types all implement Comparable.

##### Transaction example

A prototypical breeding ground for sorting applications is commercial data processing.

##### Pointer sorting

The approach we are using is known in the classical literature as pointer sorting, so called because we process references to items and do not move the data itself. In programming languages such as C and C++, programmers explicitly decide whether to manipulate data or pointers to data; in Java, pointer manipulation is implicit.

##### Keys are immutable

It stands to reason that an array might not remain sorted if a client is allowed to change the values of keys after the sort.

##### Exchanges are inexpensive

Another advantage of using references is that we avoid the cost of moving full items.

##### Alternate orderings

There are many applications where we want to use different orders for the objects that we are sorting, depending on the situation. The Java Comparator interface allows us to build multiple orders within a single class.

##### Items with multiple keys

In typical applications, items have multiple instance variables that might need to serve as sort keys.

With this definition, a client can sort an array of Transaction objects by time with the call
```
Insertion.sort(a, new Transaction.WhenOrder());
```
or by amount with the call
```
Insertion.sort(a, new Transaction.HowMuchOrder());
```

To avoid the cost of making a new Comparator object for each sort, we could use public final instance variables to define the comparators (as Java does for CASE_INSENSITIVE_ORDER).

```
public static void sort(Object[] a, Comparator c){
    int N=a.length;
    for(int i=1;i<N;i++)
        for(int j=i;j>0&&less(c,a[j],a[j-1]);j--)
            exch(a,j,j-1);
}

private static boolean less(Comparator c, Object v, Object w){
    return c.compare(v,w)<0;
}

private static void exch(Object[] a, int i,int j){
    Object t=a[i];
    a[i]=a[j];
    a[j]=t;
}
```

##### Priority queues with comparators

The same flexibility to use comparators is also useful for priority queues. Extending our standard implementation in Algorithm 2.6 to support comparators involves the following steps:
* Import java.util.Comparator.
* Add to MaxPQ an instance variable comparator and a constructor that takes a
comparator as argument and initializes comparator to that value.
* Add code to less() that checks whether comparator is null (and uses it if it is
not null).

```
import java.util.Comparator;

public class Transaction {

    private final String who;
    private final Date when;
    private final double amount;
    
    public static class WhoOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            return v.who.compareTo(w.who);
        }
    }

    public static class WhenOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            return v.when.compareTo(w.when);
        }
    }

    public static class HowMuchOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            if(v.amount<w.amount)return -1;
            if(v.amount>w.amount)return 1;
            return 0;
        }
    }
}
```

##### Stability

A sorting method is stable if it preserves the relative order of equal keys in the array.

#### Which sorting algorithm should I use?

<table>
    <tr>
        <th rowspan="2">Client</th>
        <th rowspan="2">stable?</th>
        <th rowspan="2">in place?</th>
        <th colspan="2">order of growth to sort N items</th>
        <th >notes</th>
    </tr>
    <tr>
        <th>time</th>
        <th>space</th>
    </tr>
    <tr>
        <th>selection sort</th>
        <th>no</th>
        <th>yes</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th>1</th>
        <th rowspan="3">depends on order items</th>
    </tr>
    <tr>
        <th>insertion sort</th>
        <th>yes</th>
        <th>yes</th>
        <th>between N and <img src="http://latex.codecogs.com/gif.latex?N^2"></th>
        <th>1</th>
    </tr>
    <tr>
        <th>shellsort</th>
        <th>no</th>
        <th>yes</th>
        <th>
        <img src="http://latex.codecogs.com/gif.latex?NlogN">?
        <img src="http://latex.codecogs.com/gif.latex?N^{\frac{6}/5}">?</th>
        <th>1</th>
    </tr>
    <tr>
        <th>quicksort</th>
        <th>no</th>
        <th>yes</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th><img src="http://latex.codecogs.com/gif.latex?lgN"></th>
        <th>probabilistic guarantee</th>
    </tr>
    <tr>
        <th>selection sort</th>
        <th>no</th>
        <th>yes</th>
        <th>between N and <img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th><img src="http://latex.codecogs.com/gif.latex?lgN"></th>
        <th>probabilistic, also depends on distribution of input keys</th>
    </tr>
    <tr>
        <th>mergesort</th>
        <th>yes</th>
        <th>no</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th>N</th>
    </tr>
    <tr>
        <th>heapsort</th>
        <th>no</th>
        <th>yes</th>
        <th><img src="http://latex.codecogs.com/gif.latex?NlogN"></th>
        <th>1</th>
    </tr>
</table>
Performance characteristics of sorting algorithms

##### Property T

Quicksort is the fastest general-purpose sort.

#### Sorting primitive types

In some performance-critical applications, the focus may be on sorting numbers, so it is reasonable to avoid the costs of using references and sort primitive types instead.

#### Java system sort

As an example of applying the information given in the table on page 342, consider Java’s primary system sort method,  java.util.Arrays.sort(). With overloading of argument types, this name actually represents a collection of methods:

* A different method for each primitive type
* A method for data types that implement Comparable
* A method that uses a Comparator
