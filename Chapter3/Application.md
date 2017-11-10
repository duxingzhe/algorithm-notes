### Application

We will consider several representative examples in this section:

* A dictionary client and an indexing client that enable fast and flexible access to information in comma-separated-value files (and similar formats), which are widely used to store data on the web
* An indexing client for building an inverted index of a set of files
* A sparse-matrix data type that uses a symbol table to address problem sizes far beyond what is possible with the standard implementation

#### Which symbol-table implementation should I use?

<table>
    <tr>
        <th rowspan="2">algorithm(data structure)</th>
        <th colspan="2">worst-case cost(after N inserts)</th>
        <th colspan="2">average-case cost(after N random inserts)</th>
        <th rowspan="2">efficiently support ordered operations?</th>
        <th rowspan="2">key inferface</th>
        <th rowspan="2">memory(bytes)</th>
    </tr>
    <tr>
        <th>search</th>
        <th>insert</th>
        <th>search hit</th>
        <th>insert</th>
    </tr>
    <tr>
        <th>sequential search(unordered linked list)</th>
        <th>N</th>
        <th>N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{2}></img></th>
        <th>N</th>
        <th>equals</th>
        <th>48N</th>
    </tr>
    <tr>
        <th>binary search(ordered array)</th>
        <th><img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th>N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{2}></img></th>
        <th>compareTo()</th>
        <th>16N</th>
    </tr>
    <tr>
        <th>binary tree search(BST)</th>
        <th>N</th>
        <th>N</th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.39lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.39lgN></th>
        <th>compareTo()</th>
        <th>64N</th>
    </tr>
    <tr>
        <th>2-3 tree search(red-black BST)</th>
        <th><img src=http://latex.codecogs.com/gif.latex?2lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?2lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.00lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?1.00lgN></img></th>
        <th>compareTo()</th>
        <th>64N</th>
    </tr>
    <tr>
        <th>separate chaining(array of lists)</th>
        <th><<img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th><<img src=http://latex.codecogs.com/gif.latex?lgN></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{2M}></img></th>
        <th><img src=http://latex.codecogs.com/gif.latex?\frac{N}{M}></img></th>
        <th>equals()hashCode()</th>
        <th>48N+64N</th>
    </tr>
    <tr>
        <th>linear probing(parallel arrays)</th>
        <th><<img src=http://latex.codecogs.com/gif.latex?clgN></img></th>
        <th><<img src=http://latex.codecogs.com/gif.latex?clgN></img></th>
        <th><1.50</th>
        <th><2.50</th>
        <th>equals()hashCode()</th>
        <th>between 32N and 128N</th>
    </tr>
</table>

##### Primitive types

Suppose that we have a symbol table with integer keys and associated floating-point numbers. When we use our standard setup, the keys and values are stored as Integer and Double wrapper-type values, so we need two extra memory references to access each key-value pair. These references may be no problem in an application that involves thousands of searches on thousands of keys but may represent excessive cost in an application that involves billions of searches on millions of keys. Using a primitive type instead of Key would save one reference per key-value pair. When the associated value is also primitive, we can eliminate another reference.

##### Duplicate keys

The possibility of duplicate keys sometimes needs special consideration in symboltable implementations. In many applications, it is desirable to associate multiple values with the same key.

##### Java libraries

Java’s java.util.TreeMap and java.util.HashMap libraries are symbol-table implementations based on red-black BSTs and hashing with separate chaining respectively.

#### Set APIs

Some symbol-table clients do not need the values, just the ability to insert keys into a table and to test whether a key is in the table.

##### Dedup

The prototypical filter example is a SET or HashSET client that removes duplicates in the input stream. It is customary to refer to this operation as dedup. We maintain a set of the string keys seen so far. If the next key is in the set, ignore it; if it is not in the set, add it to the set and print it. The keys appear on standard output in the order they appear on standard input, with duplicates removed.

##### Whitelist and blacklist

Another classic filter uses keys in a separate file to decide which keys from the input stream are passed to the output stream. This general process has many natural applications. The simplest example is a whitelist, where any key that is in the file is identified as “good.”

```
public class WhiteFilter{
    
    private WhiteFilter(){

    }

    public static void main(String[] args){
        SET<String> set=new SET<String>();

        In in=new In(args[0]);
        while(!in.isEmpty()){
            String word=in.readString();
            set.add(word);
        }

        while(!StdIn.isEmpty()){
            String word=StdIn.readString();
            if(set.contains(word))
                StdOut.println(word);
        }
    }
}
```

#### Dictionary clients

The most basic kind of symbol-table client builds a symbol table with successive put operations in order to support get requests. Many applications also take advantage of the idea that a symbol table is a dynamic dictionary, where it is easy to look up information and to update the information in the table. The following list of familiar examples illustrates the utility of this approach:

* Phone book
* Dictionary
* Account infromation
* Genomics
* Experimental data
* Compilers
* File Systems
* Internet DNS

<table>
    <tr>
        <th>domain</th>
        <th>key</th>
        <th>value</th>
    </tr>
    <tr>
        <th>phone book</th>
        <th>name</th>
        <th>phone number</th>
    </tr>
    <tr>
        <th>dictionary</th>
        <th>word</th>
        <th>definition</th>
    </tr>
    <tr>
        <th>account</th>
        <th>account number</th>
        <th>balance</th>
    </tr>
    <tr>
        <th>genomics</th>
        <th>codon</th>
        <th>amino acid</th>
    </tr>
    <tr>
        <th>data</th>
        <th>data/time</th>
        <th>results</th>
    </tr>
    <tr>
        <th>compiler</th>
        <th>variable name</th>
        <th>memory location</th>
    </tr>
    <tr>
        <th rowspan="2">file share</th>
        <th>song name</th>
        <th>machine</th>
    </tr>
    <tr>
        <th>website</th>
        <th>IP address</th>
    </tr>
</table>

```
public class LookupCSV {
	
	private LookupCSV() {
		
	}
	
	public static void main(String[] args) {
		int keyField=Integer.parseInt(args[1]);
		int valueField=Integer.parseInt(args[2]);
		
		In in=new In(args[0]);
		while(in.hasNextLine()) {
			String line=in.readLine();
			String[] tokens=line.split(",");
			String key=tokens[keyField];
			String value=tokens[valueField];
			st.put(key,value);
		}
		
		while(!StdIn.isEmpty()) {
			String s=StdIn.readString();
			if(st.contains(s))
				StdOut.println(st.get(s));
			else
				StdOut.println("Not found");
		}
	}
}
```

#### Indexing clients

Dictionaries are characterized by the idea that there is one value associated with each key, so the direct use of our ST data type, which is based on the associative-array abstraction that assigns one value to each key, is appropriate. We use the term index to describe symbol tables that associate multiple values with
each key. Here are some more examples:

* Commercial transactions. One way for a company that maintains customer accounts to keep track of a day’s transactions is to keep an index of the day’s transactions. The key is the account number; the value is the list of occurrences of that account number in the transaction list.
* Web search. When you type a keyword and get a list of websites containing that keyword, you are using an index created by your web search engine. There is one value (the set of pages) associated with each key (the query), although the reality is a bit more complicated because we often specify multiple keys.
* Movies and performers. The file movies.txt on the booksite (excerpted below) is taken from the Internet Movie Database (IMDB). Each line has a movie name (the key), followed by a list of performers in that movie (the value), separated by slashes.

##### Inverted index

The term inverted index is normally applied to a situation where values are used to locate keys.

* Internet Movie DataBase (IMDB). In the example just considered, the input is an index that associates each movie with a list of performers. The inverted index associates each performer with a list of movies.
* Book index. Every textbook has an index where you look up a term and get the page numbers containing that term. While creating a good index generally involves work by the book author to eliminate common and irrelevant words, a document preparation system will certainly use a symbol table to help automate the process. An interesting special case is known as a concordance, which associates each word in a text with the set of positions in the text where that word occurs.
* Compiler. In a large program that uses a large number of symbols, it is useful to know where each name is used. Historically, an explicit printed symbol table was one of the most important tools used by programmers to keep track of where symbols are used in their programs. In modern systems, symbol tables are the basis of software tools that programmers use to manage names.
* File search. Modern operating systems provide you with the ability to type a term and to learn the names of files containing that term. The key is the term; the value is the set of files containing that term.
* Genomics. In a typical (if oversimplified) scenario in genomics research, a scientist wants to know the positions of a given genetic sequence in an existing genome or set of genomes. Existence or proximity of certain sequences may be of scientific significance. The starting point for such research is an index like a concordance, but modified to take into account the fact that genomes are not separated into words.

```
public class LookupIndex {
	
	public static void main(String args[]) {
		In in=new In(args[0]);
		String sp=args[1];
		
		ST<String, Queue<String>> st=new ST<String, Queue<String>>();
		ST<String, Queue<String>> ts=new ST<String, Queue<String>>();
		
		while(in.hashNextLine()) {
			String[] a=in.readLine().split(sp);
			String key=a[0];
			for(int i=1;i<a.length;i++) {
				String value=a[i];
				if(!st.contains(key)) {
					st.put(key,new Queue<String>());
				}
				if(!ts.contains(value)) {
					ts.put(value,new Queue<String>());
				}
				st.get(key).enqueue(value);
				ts.get(value).enqueue(key);
			}
		}
		
		while(!StdIn.isEmpty()) {
			String query=StdIn.readLine();
			if(st.contains(query))
				for(String s: st.get(query))
					StdOut.println("  "+s);
			if(ts.contains(query))
				for(String s:ts.get(query))
					StdOut.println("  "+s);
		}
	}

}
```

FileIndex (on the facing page) takes file names from the command line and uses a symbol table to build an inverted index associating every word in any of the files with a SET of file names where the word can be found, then takes keyword queries from standard input, and produces its associated list of files. Developers of such tools typically embellish the process by paying careful attention to

* The form of the query
* The set of fi les/pages that are indexed
* The order in which fi les are listed in the response

```
import java.io.File;

public class FileIndex {
	
	public static void main(String[] args) {
		ST<String, SET<File>> st=new ST<String,SET<File>>();
		
		for(String filename: args) {
			File file=new File(filename);
			In in=new In(file);
			while(!in.isEmpty()) {
				String word=in.readString();
				if(!st.contains(word))
					st.put(word,new SET<File>());
				SET<File> set=st.get(word);
				set.add(file);
			}
		}
		
		while(!StdIn.isEmpty()) {
			String query=StdIn.readString();
			if(st.contains(query))
				for(File file:st.get(query)) {
					StdOut.println("  "+file.getName());
				}
		}
	}
```

Sparse vectors

Our next example illustrates the importance of symbol tables in scientific and mathematical calculations. The basic calculation that we consider is matrix-vector multiplication: given a matrix and a vector, compute a result vector whose ith entry is the dot product of the given vector and the ith row of the matrix.

```
public class SparseVector {
	
	private HashST<Integer, Double> st;
	
	public SparseVector() {
		st=new HashST<Integer, Double>();
	}
	
	public int size() {
		return st.size();
	}
	
	public void put(int i, double x) {
		st.put(i,x);
	}
	
	public double get(int i) {
		if(!st.contains(i))
			return 0;
		else
			return st.get(i);
	}
	
	public double dot(double[] that) {
		double sum=0.0;
		for(int i: st.keys()) {
			sum+=that[i]*this.get(i);
		}
		return sum;
	}

}
```