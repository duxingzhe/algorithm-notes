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
    
}
```

Analysis:
Initialization: O(N)
find: O(N)
union: O(N) (worse case)

Quick Union