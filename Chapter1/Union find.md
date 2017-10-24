Case Study: Union Find

Dynamic connectivity
* Reflexive
* Symmetric
* Transitive

Network

Variable-name equvianlence

Mathematical sets

Code:
```
public class UF {

    private int[] id;
    private int count;

    public UF(int N){
        count=N;
        id=new int[N];
        for(int i=0;i<N;i++)
            id[i]=i;
    }

    public int count() {
        return count;
    }


    public boolean connected(int p, int q){
        return find(p)==find(q);
    }

    // See page 222 (quick-find), page 224 (quick-union) and page 228 (weighted).
    public int find(int p){
        return 0;
    }
    
    public void union(int p, int q){}
    
}
```

Analysis:
Initialization: O(N)
find: O(N)
union: O(N) (worse case)

Quick find

```
public int find(int p){
    return id[p];
}

public void union(int p, int q){
    int pid=id[p];
    int qid=id[q];

    for(int i=0;i<id.length;i++){
        if(id[i]==pid)
            id[i]=qid;
    }
    count--;
}
```

Quick union

```
public int find(int p){
    return id[p];
}

public void union(int p, int q){
    int pid=id[p];
    int qid=id[q];

    for(int i=0;i<id.length;i++){
        if(id[i]==pid)
            id[i]=qid;
    }
}
```

Definition in a tree.

The size of a tree is its number of nodes.

 The depth of a node in a tree is the number of links on the path from it to the root. 
 
 The height of a tree is the maximum depth among its nodes.

 Weighted Quick Union

 ```
 public class WeightedQuickUF {

    private int[] id;
    private int[] size;
    private int count;

    public WeightedQuickUF(int N){
        count=N;
        id=new int[N];
        for(int i=0;i<N;i++)
            id[i]=i;
        size=new int[N];
        for(int i=0;i<N;i++)
            size[i]=1;
    }

    public int count() {
        return count;
    }

    public boolean connected(int p, int q){
        return find(p)==find(q);
    }

    private int find(int p){
        while(p!=id[p])
            p=id[p];
        return p;
    }

    private void union(int p, int q){
        int i=find(p);
        int j=find(q);

        if(size[i]<size[j]){
            id[i]=j;
            size[j]+=size[i];
        }else{
            id[j]=1;
            size[i]+=size[j];
        }
        count--;
    }
}
 ```

| algorithm | constructor | union  | find|
| :-------: |:-----------:|:------:|:---:|
| quick-find | N | N | 1|
| quick-union| N |  tree height |  tree height |
| weighted quick-union| N |![](http://latex.codecogs.com/gif.latex?lgN)| ![](http://latex.codecogs.com/gif.latex?lgN)|
| weighted quick-union with path compresson| N |  very, very nearly, but not quite 1 (amortized)|very, very nearly, but not quite 1 (amortized)|
| impossible | N |1|1|
