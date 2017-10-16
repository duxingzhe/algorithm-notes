### Quick Sort

#### The baisc algorithm

Quicksort is a divide-and-conquer method for sorting. It works by partitioning an array into two subarrays, then sorting the subarrays independently. 

##### Compare with Merge Sort

Quicksort is complementary to mergesort: for mergesort, we break the array into two subarrays to be sorted and then combine the ordered subarrays to make the whole ordered array; for quicksort, we rearrange the array such that, when the two subarrays are sorted, the whole array is ordered. In the first instance, we do the two recursive calls before working on the whole array; in the second instance, we do the two recursive calls after working on the whole array. For mergesort, the array is divided in half; for quicksort, the position of the partition depends on the contents of the array.

```
public class Quick {

    public static void sort(Comparable[] a){
        sort(a,0,a.length-1);
    }
    
    private static void sort(Comparable[] a,int low,int high){
        if(high<=low)
            return;
        int j=partition(a,low,high);
        sort(a,low,lt-1);
        sort(a,gt+1,high);
    }
    
}
```

The crux of the method is the partitioning process, which rearranges the array to
make the following three conditions hold:
* The entry a[j] is in its fi nal place in the array, for some j.
* No entry in a[lo] through a[j-1] is greater than a[j].
* No entry in a[j+1] through a[hi] is less than a[j].

```
private static int partition(Comparable[] a, int low,int high){
    int i=low,j=high+1;
    Comparable v=a[low];
    while(true){
        while(less(a[++i],v))
            if(i=high)
                break;
        while(less(v,a[--j]))
            if(j=low)
                break;
        if(i>=j)
            break;
        exch(a,j,j);
    }
    exch(a,low,j);
    return j;
}
```

#### Improve Proformance:

##### Partitioning in place.

If we use an extra array, partitioning is easy to implement, but not so much easier that it is worth the extra cost of copying the partitioned version back into the original. A novice Java programmer might even create a new spare array within the recursive method, for each partition, which would drastically slow down the sort.

##### Staying in bounds

If the smallest item or the largest item in the array is the partitioning item, we have to take care that the pointers do not run off the left or right ends of the array, respectively. Our partition() implementation has explicit tests to guard against this circumstance. The test (j == lo) is redundant, since the partitioning item is at a[lo] and not less than itself. With a similar technique on the right it is not difficult to eliminate both tests.

##### Preserving randomness

The random shuffle puts the array in random order. Since it treats all items in the subarrays uniformly, Algorithm 2.5 has the property that its two subarrays are also in random order. This fact is crucial to the predictability of the algorithm’s running time. An alternate way to preserve randomness is to choose a random item for partitioning within partition().

##### Terminating the loop

Experienced programmers know to take special care to ensure that any loop must always terminate, and the partitioning loop for quicksort is no exception. Properly testing whether the pointers have crossed is a bit trickier than it might seem at first glance. A common error is to fail to take into account that the array might contain other items with the same key value as the partitioning item. Handling items with keys equal to the partitioning item’s key. It is best to stop the left scan for items with keys greater than or equal to the partitioning item’s key and the right scan for items with key less than or equal to the partitioning item’s key, as in Algorithm 2.5. Even though this policy might seem to create unnecessary exchanges involving items with keys equal to the partitioning item’s key, it is crucial to avoiding quadratic running time in certain typical applications. Later, we discuss a better strategy for the case when the array contains a large number of items with equal keys.

##### Terminating the recursion

Experienced programmers also know to take special care to ensure that any recursive method must always terminate, and quicksort is again no exception. For instance, a common mistake in implementing quicksort involves not ensuring that one item is always put into position, then falling into an infinite recursive loop when the partitioning item happens to be the largest or smallest item in the array.

#### Proposition K

Quicksort uses ~ 2N ln N compares (and one-sixth that many exchanges) on the average to sort an array of length N with distinct keys.

#### Proposition L

Quicksort uses ~ N 2/2 compares in the worst case, but random shuffling protects against this case.

#### Algorithm Improvement

##### Cutoff to insertion sort

As with most recursive algorithms, an easy way to improve the performance of quicksort is based on the following two observations:

* Quicksort is slower than insertion sort for tiny subarrays.
* Being recursive, quicksort’s sort() is certain to call itself for tiny subarrays.

##### Median-of-three partitioning

A second easy way to improve the performance of
quicksort is to use the median of a small sample of items taken from the subarray as the
partitioning item. Doing so will give a slightly better partition, but at the cost of computing
the median. It turns out that most of the available improvement comes from choosing a sample of size 3 and then partitioning on the middle item. As a bonus, we can use the sample items as sentinels at the ends of the array and remove both array bounds tests in partition().

##### Entropy-optimal sorting

Arrays with large numbers of duplicate keys arise frequently in applications. For example, we might wish to sort a large personnel file by year of birth, or perhaps to separate females from males. In such situations, the quicksort implementation that we have considered has acceptable performance, but it can be substantially improved. For example, a subarray that consists solely of items that are equal (just one key value) does not need to be processed further, but our implementation keeps partitioning down to small subarrays. In a situation where there are large numbers of duplicate keys in the input array, the recursive nature of quicksort ensures that subarrays consisting solely of items with keys that are equal will occur often. There is potential for significant improvement, from the linearithmic-time performance of the implementations seen so far to linear-time performance.

```
public class Quick3way {
    
    private static void sort(Comparable[] a,int low,int high){
        if(high<=low)
            return;
        int lt=low,i=low+1,gt=high;
        Comparable v=a[low];
        while(i<=gt){
            int cmp=a[i].compareTo(v);
            if(cmp<0)
                exch(a,lt++,i++);
            else if(cmp>0)
                exch(a,i,gt--);
            else
                i++;
        }
        sort(a,low,lt-1);
        sort(a,gt+1,high);
    }
    
}
```

#### Proposition M

No compare-based sorting algorithm can guarantee to sort N items with fewer than NH ~ N compares, where H is the Shannon entropy, defined from
the frequencies of key values.

#### Proposition N

Quicksort with 3-way partitioning uses ~ (2ln 2) N H compares to sort N items, where H is the Shannon entropy, defined from the frequencies of key values.