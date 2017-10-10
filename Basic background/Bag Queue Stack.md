Bag
```
/**
*
*Bag
*
*/
public class Bag<Item> {
    private Node node;

    private class Node{
        Item item;
        Node next;
    }
    
    private void add(Item item){
        Node oldFirst=node;
        node=new Node();
        node.item=item;
        node.next=oldFirst;
    }
}
```

Implements the iterator listener.
```
/**
*
*Bag
*
*/
import java.util.Iterator;

public class Stack<Item> implements Iterable<Item> {

    private Node first;
    private int N;
    private class Node{
        Item item;
        Node next;
    }

    public boolean isEmpty(){
        return first==null;
    }

    public int size(){
        return N;
    }

    public void push(Item item){
        Node oldFirst=first;
        first=new Node();
        first.item=item;
        first.next=oldFirst;
        N++;
    }

    public Item pop(){
        Item item=null;
        if(!isEmpty()){
            item=first.item;
            first=first.next;
            N--;
        }
        return item;
    }

    @Override
    public Iterator<Item> iterator()
    {
        return new ListIterator();
    }

    private class ListIterator implements Iterator<Item>{
        private Node current=first;

        @Override
        public boolean hasNext(){
            return current!=null;
        }

        @Override
        public void remove(){

        }

        @Override
        public Item next(){
            Item item=current.item;
            current=current.next;
            return item;
        }
    }
}
```

Queue
```
/**
 *
 *Queue
 *
 */
public class Queue<Item> {

    private Node first;
    private Node last;

    private int N;

    private class Node{
        Item item;
        Node next;
    }

    private boolean isEmpty(){
        return first==null;
    }

    private int size(){
        return N;
    }

    public void enqueue(Item item){
        Node oldFirst=last;
        last=new Node();
        last.item=item;
        last.next=null;
        if (isEmpty()) {
            first=last;
        }else{
            oldFirst.next=last;
        }
        N++;
    }

    public Item dequeue(){
        if(isEmpty()){
            return null;
        }else{
            Item item=first.item;
            first=first.next;
            N--;
            return item;
        }
    }
}
```

Stack

ArrayList Version:
```
/** 
 * Stack Change the size of array damatically
 * 
 * @param <Item> 
 */  
import java.util.Iterator;

public class ResizingArrayStack<Item> {

    private Item[] a=(Item[])new Object[1];
    private int N;

    public boolean isEmpty(){
        return N==0;
    }

    public int size(){
        return N;
    }

    private void resize(int max){
        Item[] temp=(Item[])new Object[max];
        for(int i=0;i<N;i++){
            temp[i]=a[i];
        }
        a=temp;
    }

    public void push(Item item){
        if(N==a.length){
            resize(2*a.length);
        }
        a[N++]=item;
    }

    public Item pop(){
        Item item=a[--N];
        a[N]=null;
        if(N>0&&N==a.length/4){
            resize(a.length/2);
        }
        return item;
    }

    public Iterator<Item> iterator(){
        return new ReverseArrayIterator();
    }

    private class ReverseArrayIterator implements Iterator<Item>{
        private int i=N;
        
        @Override
        public boolean hasNext(){
            return i>0;
        }
        
        @Override
        public Item next(){
            return a[--i];
        }
    }
}
```

LinkedList Version:
```
/** 
 * Stack by Linked List
 * 
 * @param <Item> 
 */  
public class Stack<Item> {
    
    private Node first;
    private int N;
    private class Node{
        Item item;
        Node next;
    }
    
    public boolean isEmpty(){
        return first==null;
    }
    
    public int size(){
        return N;
    }
    
    public void push(Item item){
        Node oldFirst=first;
        first=new Node();
        first.item=item;
        first.next=oldFirst;
        N++;
    }
    
    public Item pop(){
        Item item=null;
        if(!isEmpty()){
            item=first.item;
            first=first.next;
            N--;
        }
        return item;
    }
}
```