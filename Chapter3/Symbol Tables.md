### Symbol Tables

The primary purpose of a symbol table is to associate a value with a key. The client can insert key-value pairs into the symbol table with the expectation of later being able to search for the value associated with a given key, from among all of the key-value pairs that have been put into the table.

Search is so important to so many computer applications that symbol tables are available as high-level abstractions in many programming environments, including Java—we shall discuss Java’s symbol-table implementations in Section 3.5.

#### Definition

A symbol table is a data structure for key-value pairs that supports two operations: insert (put) a new pair into the table and search for (get) the value associated with a given key.

<table>
    <tr>
        <th >application</th>
        <th >purpose of search</th>
        <th >key</th>
        <th >value</th>
    </tr>
    <tr>
        <th>dictionary</th>
        <th>find definition</th>
        <th>word</th>
        <th>definition</th>
    </tr>
    <tr>
        <th>book index</th>
        <th>find relevant pages</th>
        <th>term</th>
        <th>list of page numbers</th>
    </tr>
    <tr>
        <th>file share</th>
        <th>find song to download</th>
        <th>name of song</th>
        <th>computer ID</th>
    </tr>
    <tr>
        <th>account management</th>
        <th>process transactions</th>
        <th>account number</th>
        <th>transaction details</th>
    </tr>
    <tr>
        <th>web search</th>
        <th>find relevant web pages</th>
        <th>keyword</th>
        <th>list of page names</th>
    </tr>
    <tr>
        <th>compiler</th>
        <th>find type and value</th>
        <th>variable name</th>
        <th>type and value</th>
    </tr>
</table>

Typical symbol-table applications

##### Generics

As we did with sorting, we will consider the methods without specifying the types of the items being processed, using generics. For symbol tables, we emphasize the separate roles played by keys and values in search by specifying the key and value types explicitly instead of viewing keys as implicit in items as we did for priority queues in Section 2.4.

##### Duplicate keys
We adopt the following conventions in all of our implementations:

* Only one value is associated with each key (no duplicate keys in a table).
* When a client puts a key-value pair into a table already containing that key (and an associated value), the new value replaces the old one.

##### Null keys

Keys must not be null. As with many mechanisms in Java, use of a null key results in an exception at runtime (see the third Q&A on page 387).

##### Null values

We also adopt the convention that no key can be associated with the value null. This convention is directly tied to our specification in the API that get() should return null for keys not in the table, effectively associating the value null with every key not in the table.

##### Deletion

Deletion in symbol tables generally involves one of two strategies: lazy deletion, where we associate keys in the table with null, then perhaps remove all such keys at some later time; and eager deletion, where we remove the key from the table immediately.

#### Shorthand methods

For clarity in client code, we include the methods contains() and isEmpty() in the API, with the default one-line implementations shown here.

##### Iteration

To enable clients to process all the keys and values in the table, we might add the phrase implements Iterable<Key> to the first line of the API to specify that every implementation must implement an iterator() method that returns an iterator having appropriate implementations of hasNext() and next(), as described for stacks and queues in Section 1.3.

##### Key equality

Determining whether or not a given key is in a symbol table is based on the concept of object equality, which we discussed at length in Section 1.2 (see page 102).

#### Ordered symbol tables

In typical applications, keys are Comparable objects, so the option exists of using the code a.compareTo(b) to compare two keys a and b. Several symbol-table implementations take advantage of order among the keys that is implied by Comparable to provide efficient implementations of the put() and get() operations.

##### Minimum and maximum

Perhaps the most natural queries for a set of ordered keys are to ask for the smallest and largest keys. We have already encountered these operations, in our discussion of priority queues in Section 2.4. In ordered symbol tables, we also have methods to delete the maximum and minimum keys (and their associated values).

##### Floor and ceiling

Given a key, it is often useful to be able to perform the floor operation (find the largest key that is less than or equal to the given key) and the ceiling operation (find the smallest key that is greater than or equal to the given key).

##### Rank and selection

The basic operations for determining where a new key fits in the order are the rank operation (find the number of keys less than a given key) and the select operation (find the key with a given rank).

##### Range queries

How many keys fall within a given range (between two given keys)? Which keys fall in a given range? The two-argument size() and keys() methods that answer these questions are useful in many applications, particularly in large databases.

##### Exceptional cases

When a method is to return a key and there is no key fitting the description in the table, our convention is to throw an exception (an alternate approach, which is also reasonable, would be to return null in such cases).

##### Shorthand methods

As we have already seen with isEmpty() and contains() in our basic API, we keep some redundant methods in the API for clarity in client code.

##### Key equality (revisited)

The best practice in Java is to make compareTo() consistent with equals() in all Comparable types.

##### Cost model

Whether we use equals() (for symbol tables where keys are not Comparable) or compareTo() (for ordered symbol tables with Comparable keys), we use the term compare to refer to the operation of comparing a symboltable entry against a search key.

#### Sample clients

While we defer detailed consideration of applications to Section 3.5, it is worthwhile to consider some client code before considering implementations.

##### Test client

For tracing our algorithms on small inputs we assume that all of our implementations use the test client below, which takes a sequence of strings from standard input, builds a symbol table that associates the value i with the ith string in the input, and then prints the table.

#### Performance client

FrequencyCounter (on the next page) is a symbol-table client that finds the number of occurrences of each string (having at least as many characters as a given threshold length) in a sequence of strings from standard input, then iterates through the keys to find the one that occurs the most frequently. This client is an example of a dictionary client, an application that we discuss in more detail in Section 3.5.

FrequencyCounter is surrogate for a very common situation. Specifically, it has the following characteristics, which are shared by many other symbol-table clients:

* Search and insert operations are intermixed.
* The number of distinct keys is not small.
* Substantially more searches than inserts are likely.
* Search and insert patterns, though unpredictable, are not random.

##### Sequential search in an unordered linked list

One straightforward option for the underlying data structure for a symbol table is a linked list of nodes that contain keys and values, as in the code on the facing page.

##### Proposition A

Search misses and insertions in an (unordered) linked-list symbol table having N key-value pairs both require N compares, and search hits N compares in the worst case. In particular, inserting N distinct keys into an initially empty linked-list symbol table uses ~![](http://latex.codecogs.com/gif.latex?\frac{N^2}2) compares.

##### Corollary

Inserting N distinct keys into an initially empty linked-list symbol table uses ~![](http://latex.codecogs.com/gif.latex?\frac{N^2}2) compares.

#### Binary search in an ordered array

Next, we consider a full implementation of our ordered symbol-table API. The underlying data structure is a pair of parallel arrays, one for the keys and one for the values.

##### Binary search

The reason that we keep keys in an ordered array is so that we can use array indexing to dramatically reduce the number of compares required for each search, using the classic and venerable binary search algorithm that we used as an example in Chapter 1.

This recursive rank() preserves the following properties:

* If key is in the table, rank() returns its index in the table, which is the same as
the number of keys in the table that are smaller than key.
* If key is not in the table, rank() also returns the number of keys in the table
that are smaller than key.

##### Other operations

Since the keys are kept in an ordered array, most of the order-based operations are compact and straightforward, as you can see from the code on page 382. For example, a call to select(k) just returns keys[k].

##### Analysis of binary search

The recursive implementation of rank() also leads to an immediate argument that binary search guarantees fast search, because it corresponds to a recurrence relation that describes an upper bound on the number of compares.

##### Proposition B

Binary search in an ordered array with N keys uses no more than lg N  1 compares for a search (successful or unsuccessful).

##### Proposition B (continued)

Inserting a new key into an ordered array of size N uses ~ 2N array accesses in the worst
case, so inserting N keys into an initially empty table uses ~![](http://latex.codecogs.com/gif.latex?N^2) array accesses in the worst case.

#### Preview

Binary search is typically far better than sequential search and is the method of choice in numerous practical applications.

#### Full implementation of Binary Search in Symbol Table

```
public class BinarySearch<Key implements Comparable<Key>, Value> {

    private Key[] keys;
    private Value[] values;

    public BinarySearchST(int capacity){
        keys=(Key[])new Comparable[capacity];
        values=(Value[])new Object[capacity];
    }

    public int size(){
        return N;
    }

    public Value get(Key key){
        if(isEmpty())
            return null;
        int i=rank(key);
        if(i<N&&keys[i].compareTo(key)==0)
            return values[i];
        else
            return null;
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

    public void put(Key key, Value value){
        int i=rank(key);
        if(i<N&&keys[i].copareTo(key)==0){
            values[i]=value;
            return;
        }
        for(int j=N;j>i;j--){
            keys[j]=keys[j-1];
            values[j]=values[j-1];
        }
        keys[i]=key;
        values[i]=value;
        N++;
    }

    public void delete(Key key);

    public Key min(){
        return keys[0];
    }

    public Key max(){
        return keys[N-1];
    }

    public Key select(int k){
        return keys[k];
    }
    
    public Key ceiling(Key key){
        int i=rank(key);
        return keys[i];
    }
    
    public Key floor(Key key);
    
    public Key delete(Key key);
    
    public Iterable<Key> keys(Key low, Key high){
        Queue<Key> q=new Queue<Key>();
        for(int i=rank(low);i<rank(high);i++){
            q.enqueue(keys[i]);
        }
        if(contains(high)){
            q.enqueue(keys[rank(high)]);
        }
        return q;
    }
}
```

BinarySearchST costs

Cost summary for basic symbol-table implementations