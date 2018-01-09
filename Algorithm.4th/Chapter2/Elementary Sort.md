### Selection

sort One of the simplest sorting algorithms works as follows: First, find the smallest item in the array and exchange it with the first entry (itself if the first entry is already the smallest). Then, find the next smallest item and exchange it with the second entry. Continue in this way until the entire array is sorted. This method is called selection sort because it works by repeatedly selecting the smallest remaining item.

#### Proposition A.
Selection sort uses ![](http://latex.codecogs.com/gif.latex?\frac{N^2}{2}) compares and N exchanges to sort an array of length N.

```
public class Selection {
    
    public static void sort(Comparable[] a){
        int N=a.length;
        for(int i=0;i<N;i++){
            int min=i;
            for(int j=i+1;j<N;j++)
                if(less(a[j],a[min]))min=j;
            exch(a,i,min);
        }
    }
    
    private static boolean less(Comparable v, Comparable w){
        return v.compareTo(w)<0;
    }
    
    private static void exch(Comparable[] a, int i, int j){
        Comparable t=a[i];
        a[i]=a[j];
        a[j]=t;
    }
    
    private static void show(Comparable[] a){
        for(int i=0;i<a.length;i++)
            System.out.print(a[i]+" ");
        System.out.println();
    }
    
    public static boolean isSorted(Comparable[] a){
        for(int i=1;i<a.length;i++)
            if(less(a[i],a[i-1])) return false;
        return true;
    }
}
```

### Insert Sort

The algorithm that people often use to sort ridge hands is to consider the cards one at a time, inserting each into its proper place among those already considered (keeping them sorted). In a computer implementation, we need to make space to insert the current item by moving larger items one position to the right, before inserting the current item into the vacated position. Algorithm 2.2 is an implementation of this method, which is called insertion sort.

#### Proposition B.

Insertion sort uses ![](http://latex.codecogs.com/gif.latex?\frac{N^2}4) compares and ![](http://latex.codecogs.com/gif.latex?\frac{N^2}4) exchanges to sort
a randomly ordered array of length N with distinct keys, on the average. The worst
case is ![](http://latex.codecogs.com/gif.latex?\frac{N^2}2) compares and ![](http://latex.codecogs.com/gif.latex?\frac{N^2}2) exchanges and the best case is N  1 compares and 0 exchanges.

```
public class Insertion {

    public static void sort(Comparable[] a){
        int N=a.length;
        for(int i=1;i<N;i++){
            for(int j=i;j>0&&less(a[j],a[j-1);j--)
            exch(a,j,j-1);
        }
    }
    
    private static boolean less(Comparable v, Comparable w){
        return v.compareTo(w)<0;
    }

    private static void exch(Comparable[] a, int i, int j){
        Comparable t=a[i];
        a[i]=a[j];
        a[j]=t;
    }

    private static void show(Comparable[] a){
        for(int i=0;i<a.length;i++)
            System.out.print(a[i]+" ");
        System.out.println();
    }

    public static boolean isSorted(Comparable[] a){
        for(int i=1;i<a.length;i++)
            if(less(a[i],a[i-1])) return false;
        return true;
    }
}
```

#### Proposition C

The number of exchanges used by insertion sort is equal to the number of inversions in the array, and the number of compares is at least equal to the number of inversions and at most equal to the number of inversions plus the array size minus 1.

#### Property D

The running times of insertion sort and selection sort are quadratic and within a small constant factor of one another for randomly ordered arrays of distinct values.

### Shell Sort

Shellsort is a simple extension of insertion sort that gains
speed by allowing exchanges of array entries that are far apart, to produce partially sorted arrays that can be efficiently sorted, eventually by insertion sort.

The idea is to rearrange the array to give it the property that taking every hth entry (starting anywhere) yields a sorted subsequence. Such an array is said to be h-sorted. Put another way, an h-sorted array is h independent sorted subsequences, interleaved together. By h-sorting for some large values
of h, we can move items in the array long distances and thus make it easier to h-sort for smaller values of h. Using such 
a procedure for any sequence of values of h that ends in 1 will produce a sorted array: that is shellsort. The implementation in Algorithm 2.3 on the facing page uses the sequence of decreasing values ![](http://latex.codecogs.com/gif.latex?\frac{1}{3}(3^k-1)), starting at the largest increment less than N/3 and decreasing to 1. We refer to such a sequence as an increment sequence. Algorithm 2.3 computes its increment sequence; another alternative is to store an increment sequence in an array.

One way to implement shellsort would be, for each h, to use insertion sort independently on each of the h subsequences. Because the subsequences are independent, we can use an even simpler approach: when h-sorting the array, we insert each item among the previous items in its h-subsequence by exchanging it with those that have larger keys (moving them each one position to the right in the subsequence). We accomplish this task by using the insertion-sort code, but modified to decrement by h instead of 1 when moving through the array. This observation reduces the shellsort implementation to an insertion-sort-like pass through the array for each increment.

```
public class Insertion {

    public static void sort(Comparable[] a){
        int N=a.length;
        int h=1;
        while(h<N/3)
            h=3*h+1;
        while(h>=1){
            for(int i=h;i<N;i++){
                for(int j=i;j>=h&&less(a[j],a[j-h]);j-=h)
                exch(a,j,j-h);
            }
            h=h/3;
        }
    }
    
    private static boolean less(Comparable v, Comparable w){
        return v.compareTo(w)<0;
    }

    private static void exch(Comparable[] a, int i, int j){
        Comparable t=a[i];
        a[i]=a[j];
        a[j]=t;
    }

    private static void show(Comparable[] a){
        for(int i=0;i<a.length;i++)
            System.out.print(a[i]+" ");
        System.out.println();
    }

    public static boolean isSorted(Comparable[] a){
        for(int i=1;i<a.length;i++)
            if(less(a[i],a[i-1])) return false;
        return true;
    }
}
```

#### Property E

The number of compares used by shellsort with the increments 1, 4, 13, 40, 121, 364, . . . is bounded by a small multiple of N times the number of increments used.