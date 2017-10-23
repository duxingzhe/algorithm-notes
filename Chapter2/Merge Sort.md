### Merge Sort

The straightforward approach to implementing merging is to design a method that merges two disjoint ordered arrays of Comparable objects into a third array. This strategy is easy to implement: create an output array of the requisite size and then choose successively the smallest remaining item from the two input arrays to be the next item added to the output array.

#### Abstract in-place merge

The abstraction of an in-place merge is useful. Accordingly, we use the method signature merge(a, lo, mid, hi) to specify a merge method that puts the result of merging the subarrays a[lo..mid] with a[mid+1..hi] into a single ordered array, leaving the result in a[lo..hi]. The code on the next page implements this merge method in just a few lines by copying everything to an auxiliary array and then merging back to the original.

```
public static void merge(Comparable[] a, int low,int mid,int high){
    int i=low,j=mid+1;
    for(int k=low;k<=high;k++){
        aux[k]=a[k];
        for(int k=low;k<=high;k++){
            if(i>mid)
                a[k]=aux[j++];
            else if(j>high)
                a[k]=aux[i++];
            else if(less(aux[j],aux[i]))
                a[k]=aux[j++];
            else
                a[k]=aux[i++];
        }
    }
}
```

#### Top-down mergesort

This is a recursive mergesort implementation based on this abstract inplace merge. It is one of the best-known examples of the utility of the divide-and-conquer paradigm for efficient algorithm design. This recursive code is the basis for an inductive proof that the algorithm sorts the array: if it sorts the two subarrays, it sorts the whole array, by merging together the subarrays.

Proposition F

Top-down mergesort uses between ![](http://latex.codecogs.com/gif.latex?\frac{1}/{2}NlgN) and ![](http://latex.codecogs.com/gif.latex?NlgN) compares to sort any array of length N.

```
public class Merge {

    public static void sort(Comparable[] a){
        int N=a.length;
        sort(a,0,a.length-1);
    }

    private static void sort(Comparable[] a, int low, int high){
        if(high<=low) return;
        int mid=low+(high-low)/2;
        sort(a,low,mid);
        sort(a,mid+1,high);
        merge(a,low,mid,high);
    }
}
```

Proposition G

Top-down mergesort uses at most ![](http://latex.codecogs.com/gif.latex?6NlgN) array accesses to sort an array of length N.

#### Improvement

##### Use insertion sort for small subarrays

We can improve most recursive algorithms by handling small cases differently, because the recursion guarantees that the method will be used often for small cases, so improvements in handling them lead to improvements in the whole algorithm. In the case of sorting, we know that insertion sort (or selection sort) is simple and therefore likely to be faster than mergesort for tiny subarrays. As usual, a visual trace provides insight into the operation of mergesort. The visual trace on the facing page shows the operation of a mergesort implementation with a cutoff for small subarrays. Switching to insertion sort for small subarrays (length 15 or less, say) will improve the running time of a typical mergesort implementation by 10 to 15 percent.

#### Test whether the array is already in order

We can reduce the running time to be linear for arrays that are already in order by adding a test to skip the call to merge() if a[mid] is less than or equal to a[mid+1]. With this change, we still do all the recursive
calls, but the running time for any sorted subarray is linear.

#### Eliminate the copy to the auxiliary array

It is possible to eliminate the time (but not the space) taken to copy to the auxiliary array used for merging. To do so, we use two invocations of the sort method: one takes its input from the given array and puts the sorted output in the auxiliary array; the other takes its input from the auxiliary array and puts the sorted output in the given array. With this approach, in a bit of recursive trickery, we can arrange the recursive calls such that the computation switches the roles of the input array and the auxiliary array at each level.

#### Bottom-up mergesort
```
public class BottomUpMergeSort {

    private static Comparable[] aux;
    
    public static void sort(Comparable[] a){
        int N=a.length;
        aux=new Comparable[N];
        for(int size=1;size<N;size+=size){
            for(int low=0;low<N-size;low+=size+size)
                merge(a,low,low+size-1,Math.min(low+size+size-1,N-1));
        }
    }
}
```
Bottom-up mergesort consists of a sequence of passes over the whole array, doing sz-by-sz merges, starting with sz equal to 1 and doubling sz on each pass. The final subarray is of size sz only when the array size is an even multiple of sz (otherwise it is less than sz).

#### Proposition H

Bottom-up mergesort uses between ![](http://latex.codecogs.com/gif.latex?\frac{1}/{2}NlgN) and ![](http://latex.codecogs.com/gif.latex?NlgN) and ![](http://latex.codecogs.com/gif.latex?NlgN) compares and at most ![](http://latex.codecogs.com/gif.latex?6NlgN) array accesses to sort an array of length N.

#### The complexity of Sorting

#### Proposition I

No compare-based sorting algorithm can guarantee to sort N items
with fewer than ![](http://latex.codecogs.com/gif.latex?lg(N!) ~ ![](http://latex.codecogs.com/gif.latex?NlgN) compares.

#### Proposition J

Mergesort is an asymptotically optimal compare-based sorting algorithm.