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

##### Array representation
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