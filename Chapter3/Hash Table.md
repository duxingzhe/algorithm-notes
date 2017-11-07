### Hash Table

If keys are small integers, we can use an array to implement an unordered symbol table, by interpreting the key as an array index so that we can store the value associated with key i in array entry i, ready for immediate access. In this section, we consider hashing, an extension of this simple method that handles more complicated types of keys.

#### Hash functions

The first problem that we face is the computation of the hash function, which transforms keys into array indices. If we have an array that can hold M key-value pairs, then we need a hash function that can transform any given key into an index into that array: an integer in the range [0, M – 1]. We seek a hash function that both is easy to compute and uniformly distributes the keys: for each key, every integer between 0 and M – 1 should be equally likely (independently for every key).

###### Typical example

Suppose that we have an application where the keys are U.S. social security numbers.

###### Positive integers

The most commonly used method for hashing integers is called modular hashing : we choose the array size M to be prime and, for any positive integer key k, compute the remainder when dividing k by M.

###### Floating-point numbers

If the keys are real numbers between 0 and 1, we might just multiply by M and round off to the nearest integer to get an index between 0 and M – 1.

###### Strings

Modular hashing works for long keys such as strings, too: we simply treat them as huge integers.

###### Compound keys

If the key type has multiple integer fields, we can typically mix them together in the way just described for String values.

###### Java conventions

Java helps us address the basic problem that every type of data needs a hash function by ensuring that every data type inherits a method called hashCode() that returns a 32-bit integer.

###### Converting a hashCode() to an array index

Since our goal is an array index, not a 32-bit integer, we combine hashCode() with modular hashing in our implementations to produce integers between 0 and M – 1, as follows:

```
private int hash(Key x){
    return (x.hashCode()&0x7fffffff)%M;
}
```

##### User-defined hashCode()

Client code expects that hashCode() disperses the keys uniformly among the possible 32-bit result values.

##### Assumption J ( uniform hashing assumption)

The hash functions that we use uniformly and independently distribute keys among the integer values between 0 and M–1.

#### Hashing with separate chaining

A hash function converts keys into array indices. The second component of a hashing algorithm is collision resolution: a strategy for handling the case when two or more keys to be inserted hash to the same index. A straightforward and general approach to collision resolution is to build, for each of the M array indices, a linked list of the key-value pairs whose keys hash to that index. This method is known as separate chaining because items that collide are chained together in separate linked lists.

```
public class SeparateChainingHashST<Key, Value>{

    private int N;
    private int M;
    private SequentialSearchST<Key,Value>[]st;

    public SeparateChainingHashST(){
        this(997);
    }

    public SeparateChainingHashST(int M){
        this.M=M;
        st(SequentialSearchST<Key,Value>[])new SequentialSearchST[M];
        for(int i=0;i<M;i++){
            st[i]=new SequentialSearchST();
        }
    }

    private int hash(Key key){
        return (key.hashCode()&0x7fffffff)%M;
    }

    public Value get(Key key){
        return (Value)st[hash(key)];
    }
    
    public void put(Key key, Value value){
        st[hash(key)].put(key,value);
    }
    
    public Iterable<Key> keys();
}
```

##### Proposition K

In a separate-chaining hash table with M lists and N keys, the probability (under Assumption J) that the number of keys in a list is within a small constant factor of ![](http://latex.codecogs.com/gif.latex?\frac{N}{M}) is extremely close to 1.

##### Property L

In a separate-chaining hash table with M lists and N keys, the number of compares (equality tests) for search miss and insert is ~![](http://latex.codecogs.com/gif.latex?\frac{N}{M}).

##### Table size

In a separate-chaining implementation, our goal is to choose the table size M to be sufficiently small that we do not waste a huge area of contiguous memory with empty chains but sufficiently large that we do not waste time searching through long chains.

##### Deletion

To delete a key-value pair, simply hash to find the SequentialSearchST containing the key, then invoke the delete() method for that table. Reusing code in this way is preferable to reimplementing this basic operation on linked lists.

##### Ordered operations

The whole point of hashing is to uniformly disperse the keys, so any order in the keys is lost when hashing. If you need to quickly find the maximum or minimum key, find keys in a given range, or implement any of the other operations in the ordered symbol-table API on page 366, then hashing is not appropriate, since these operations will all take linear time.

#### Hashing with linear probing

Another approach to implementing hashing is to store N key-value pairs in a hash table of size M > N, relying on empty entries in the table to help with collision resolution. Such methods are called open-addressing hashing methods.

The simplest open-addressing method is called linear probing: when there is a collision (when we hash to a table index that is already occupied with a key different from the search key), then we just check the next entry in the table (by incrementing the index). Linear probing is characterized by identifying three possible outcomes:

* Key equal to search key: search hit
* Empty position (null key at indexed position): search miss
* Key not equal to search key: try next entry

```
public class LinearProbingHashST<Key, Value>{

    private int N;
    private int M=16;
    private Key[] keys;
    private Value[] values;

    public LinearProbingHashST(){
        keys=(Key[])new Object[M];
        values=(Value[])new Object[M];
    }

    private int hash(Key key){
        return (key.hashCode()&0x7fffffff)%M;
    }

    private void resize();

    public void put(Key key, Value value){
        if(N>=M/2)
            resize(2*M);
        int i;
        for(i=hash(key);keys[i]!=null;i=(i+1)%M){
            if(keys[i].equals(key)){
                values[i]=value;
                return;
            }
        }
        keys[i]=key;
        values[i]=value;
        N++;
    }

    public Value get(Key key){
        for(int i=hash(key);keys[i]!=null;i=(i+1)%M){
            if(keys[i].equals(key)){
                return values[i];
            }
        }
        return null;
    }
}
```

##### Deletion

How do we delete a key-value pair from a linear-probing table? If you think about the situation for a moment, you will see that setting the key’s table position to null will not work, because that might prematurely terminate the search for a key that was inserted into the table later.

```
public void delete(Key key){
    if(!contains(key))
        return;
    int i=hash(key);
    while(!key.equals(keys[i])){
        i=(i+1)%M;
    }
    keys[i]=null;
    values[i]=null;
    i=(i+1)%M;
    while(keys[i]!=null){
        Key keyToRedo=keys[i];
        Value valueToRedo=values[i];
        keys[i]=null;
        values[i]=null;
        N--;
        put(keyToRedo,valueToRedo);
        i=(i+1)%M;
    }
    N--;
    if(N>0&&N==M/8)
        resize(M/2);
}
```

##### Clustering

The average cost of linear probing depends on the way in which the entries clump together into contiguous groups of occupied table entries, called clusters, when they are inserted.

##### Analysis of linear probing

Despite the relatively simple form of the results, precise analysis of linear probing is a very challenging task. Knuth’s derivation of the following formulas in 1962 was a landmark in the analysis of algorithms:

Proposition M. In a linear-probing hash table with M lists and ![](http://latex.codecogs.com/gif.latex?N=\alpha{M}) keys, the
average number of probes (under Assumption J) required is ~![](http://latex.codecogs.com/gif.latex?\frac{1}{2}(1+\frac{1}{1-a})) and ![](http://latex.codecogs.com/gif.latex?\frac{1}{2}(1+\frac{1}{(1-a)^2}))
