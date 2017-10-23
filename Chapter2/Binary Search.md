The complexity of Binary Search is O(logn).

The normal version:
```
public int binarySearch(int[] array,int key){
    int low=0;
    int high=array.length-1;
    while(low<=high){
        int middle=(low+high)/2;
        if(key==array[middle]){
            return middle+1;
        }else if(key<array[middle]){
            high=middle-1;
        }else if(key>array[middle]){
            low=middle+1;
        }
    }
    return -1;
}
```

The recursion version
```
public int binarySearchRecursion(int[] array,int low,int high,int key){
    if(low<=high){
        int middle=(low+high)/2;
        if(array[middle]==key){
            return middle+1;
        }else if(array[middle]>key){
            return binarySearchRecursion(array,low,middle-1,key);
        } else if (array[middle]<key){
            return binarySearchRecursion(array,middle+1,high,key);
        }
    }
    return -1;
}
```