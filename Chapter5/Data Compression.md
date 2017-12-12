### Data Compression

The world is awash with data, and algorithms designed to represent data efficiently play an important role in the modern computational infrastructure. There are two primary reasons to compress data: to save storage when saving information and to save time when communicating information.

#### Rules of the game

All of the types of data that we process with modern computer systems have something in common: they are ultimately represented in binary. We can consider each of them to be simply a sequence of bits (or bytes).

##### Basic model

Accordingly, our basic model for data compression is quite simple, having two primary components, each a black box that reads and writes bitstreams:

* A compress box that transforms a bitstream B into a compressed version C (B)
* An expand box that transforms C (B) back into B

Using the notation |B| to denote the number of bits in a bitstream, we are interested in minimizing the quantity |C(B)|/|B|, which is known as the compression ratio.

#### Reading and writing binary data

A full description of how information is encoded on your computer is system-dependent and is beyond our scope, but with a few basic assumptions and two simple APIs, we can separate our implementations from these details.

##### Binary input and output

Most systems nowadays, including Java, base their I/O on 8-bit bytestreams, so we might decide to read and write bytestreams to match I/O formats with the internal representations of primitive types, encoding an 8-bit char with 1 byte, a 16-bit short with 2 bytes, a 32-bit int with 4 bytes, and so forth. Since bitstreams are the primary abstraction for data compression, we go a bit further to allow clients to read and write individual bits, intermixed with data of primitive types. The goal is to minimize the necessity for type conversion in client programs and also to take care of operating system conventions for representing data.

##### Example

As a simple example, suppose that you have a data type where a date is represented as three int values (month, day, year). Using StdOut to write those values in the format 12/31/1999 requires 10 characters, or 80 bits. If you write the values directly with BinaryStdOut, you would produce 96 bits (32 bits for each of the 3 int values); if you use a more economical representation that uses byte values for the month and day and a short value for the year, you would produce 32 bits. With BinaryStdOut you could also write a 4-bit field, a 5-bit field, and a 12-bit field, for a total of 21 bits (24 bits, actually, because files must be an integral number of 8-bit bytes, so close() adds three 0 bits at the end).

##### Binary dumps

How can we examine the contents of a bitstream or a bytestream while debugging? This question faced early programmers when the only way to find a bug was to examine each of the bits in memory, and the term dump has been used since the early days of computing to describe a human-readable view of a bitstream. If you try to open a file with an editor or view it in the manner in which you view text files (or just run a program that uses BinaryStdOut), you are likely to see gibberish, depending on the system you use.

```
public class BinaryDump {
	
	public static void main(String[] args) {
		int width=Integer.parseInt(args[0]);
		int count;
		for(count=0;!BinaryStdIn.isEmpty();count++) {
			if(width==0)
				continue;
			if(count!=0&&count%width==0) {
				StdOut.println();
			}
			if(BinaryStdIn.readBoolean()) {
				StdOut.print("1");
			} else {
				StdOut.print("0");
			}
		}
		StdOut.println();
		StdOut.println(count+" bits");
	}

}
```

##### ASCII encoding

When you HexDump a bitstream that contains ASCII-encoded characters, the table at right is useful for reference. Given a two digit hex number, use the first hex digit as a row index and the second hex digit as a column index to find the character that it encodes.

#### Limitations

To appreciate data-compression algorithms, you need to understand fundamental limitations. Researchers have developed a thorough and important theoretical basis for this purpose, which we will consider briefly at the end of this section, but a few ideas will help us get started.

##### Universal data compression

Armed with the algorithmic tools that have proven so useful for so many problems, you might think that our goal should be universal data compression: an algorithm that can make any bitstream smaller. Quite to the contrary, we have to adopt more modest goals because universal data compression is impossible.

##### Proposition S

No algorithm can compress every bitstream.

##### Undecidability

Consider the million-bit string pictured at the top of this page. This string appears to be random, so you are not likely to find a lossless compression algorithm that will compress it. But there is a way to represent that string with just a few thousand bits, because it was produced by the program below.

```
public class RandomBits {
	
	public static void main(String[] args) {
		int x=11111;
		for(int i=0;i<100000;i++) {
			x=x*314159+218281;
			BinaryStdOut.write(x>0);
		}
		BinaryStdOut.close();
	}

}
```

The four methods that we consider exploit, in turn, the following structural characteristics:

* Small alphabets
* Long sequences of identical bits/characters
* Frequently used characters
* Long reused bit/character sequences

#### Warmup: genomics

As preparation for more complicated data-compression algorithms, we now consider an elementary (but very important) data-compression task.

##### Genomic data

As a first example of data compression, consider this string:

ATAGATGCATAGCGCATAGCTAGATGTGCTAGCAT

Using standard ASCII encoding (1 byte, or 8 bits per character), this string is a bitstream of length 835 = 280. Strings of this sort are extremely important in modern biology, because biologists use the letters A, C, T, and G to represent the four nucleotides in the DNA of living organisms. A genome is a sequence of nucleotides.

##### 2-bit code compression

One simple property of genomes is that they contain only four different characters, so each can be encoded with just 2 bits per character, as in the compress() method shown at right.

```
public static void compress() {
	Alphabet DNA=new Alphabet("ACTG");
	String s=BinaryStdIn.readString();
	int N=s.length();
	BinaryStdOut.write(N);
	for(int i=0;i<N;i++) {
		int d=DNA.toIndex(s.charAt(i));
		BinaryStdOut.write(d,DNA.lgR());
	}
	BinaryStdOut.close();
}
```

##### 2-bit code expansion

The expand() method at the top of the next page expands a bitstream produced by this compress() method. As with compression, this method reads a bitstream and writes a bitstream, in accordance with the basic data-compression model. The bitstream that we produce as output is the original input.

```
public static void expand() {
	Alphabet DNA=new Alphabet("ACTG");
	int w=DNA.lgR();
	int N=BinaryStdIn.readInt();
	for(int i=0;i<N;i++) {
		char c=BinaryStdIn.readChar(w);
		BinaryStdOut.write(DNA.toChar(c));
	}
	BinaryStdOut.close();
}
```

```
public class Genome {

	public static void compress()
	
	public static void expand()
	
	public static void main(String[] args) {
		if(args[0].equals("-"))
			compress();
		if(args[0].equals("+"))
			expand();
	}
}
```

#### Run-length encoding

The simplest type of redundancy in a bitstream is long runs of repeated bits. Next, we consider a classic method known as run-length encoding for taking advantage of this redundancy to compress data.

In order to turn this description into an effective data compression method, we have to consider the following issues:

* How many bits do we use to store the counts?
* What do we do when encountering a run that is longer than the maximum count implied by this choice?
* What do we do about runs that are shorter than the number of bits needed to store their length?
We are primarily interested in long bitstreams with relatively few short runs, so we address these questions by making the following choices:

* Counts are between 0 and 255, all encoded with 8 bits.
* We make all run lengths less than 256 by including runs of length 0 if needed.
* We encode short runs, even though doing so might lengthen the output.

##### Bitmaps

As an example of the effectiveness of run-length encoding, we consider bitmaps, which are widely use to represent pictures and scanned documents. For brevity and simplicity, we consider binary-valued bitmaps organized as bitstreams formed by taking the pixels in row-major order.

##### Implementation

The informal description just given leads immediately to the compress() and expand() implementations on the next page.

The compress() method is not much more difficult, consisting of the following steps while there are bits in the input stream:

* Read a bit.
* If it differs from the last bit read, write the current count and reset the count to 0.
* If it is the same as the last bit read, and the count is a maximum, write the count, write a 0 count, and reset the count to 0.
* Increment the count.

When the input stream empties, writing the count (length of the last run) completes the process.

##### Increasing resolution in bitmaps

Then the following facts are evident:

* The number of bits increases by a factor of 4.
* The number of runs increases by about a factor of 2.
* The run lengths increase by about a factor of 2.
* The number of bits in the compressed version increases by about a factor of 2.
* Therefore, the compression ratio is halved!

```
public static void compress() {
	char count=0;
	boolean b,old=false;
	while(!BinaryStdIn.isEmpty()) {
		b=BinaryStdIn.readBoolean();
		if(b!=old) {
			BinaryStdOut.write(count);
			count=0;
			old=!old;
		} else {
			if(count==255) {
				BinaryStdOut.write(count);
				count=0;
				BinaryStdOut.write(count);
				
			}
		}
		count++;
	}
	BinaryStdOut.write(count);
	BinaryStdOut.close();
}

public static void expand() {
	boolean b=false;
	while(!BinaryStdIn.isEmpty()) {
		char count=BinaryStdIn.readChar();
		for(int i=0;i<count;i++) {
			BinaryStdOut.write(b);
		}
		b=!b;
	}
	BinaryStdOut.close();
}
```

#### Huffman compression

We now examine a data-compression technique that can save a substantial amount of space in natural language files (and many other kinds of files). The idea is to abandon the way in which text files are usually stored: instead of using the usual 7 or 8 bits for each character, we use fewer bits for characters that appear often than for those that appear rarely.

##### Variable-length prefix-free codes

A code associates each character with a bitstring: a symbol table with characters as keys and bitstrings as values.

##### Trie representation for prefix-free codes

One convenient way to represent a prefix-free code is with a trie (see Section 5.2).

##### Overview

Using a prefix-free code for data compression involves five major steps. We view the bitstream to be encoded as a bytestream and use a prefix-free code for the characters as follows:

* Build an encoding trie.
* Write the trie (encoded as a bitstream) for use in expansion.
* Use the trie to encode the bytestream as a bitstream.

Then expansion requires that we

* Read the trie (encoded at the beginning of the bitstream)
* Use the trie to decode the bitstream

##### Trie nodes

We begin with the Node class at left. It is similar to the nested classes that we have used before to construct binary trees and tries: each Node has left and right references to Nodes, which define the trie structure. Each Node also has an instance variable freq that is used in construction, and an instance variable ch, which is used in leaves to represent characters to be encoded.

```
public class Node implements Comparable<Node> {
	
	private char ch;
	private int freq;
	private final Node left,right;
	
	Node(char ch, int freq, Node left, Node right){
		this.ch=ch;
		this.freq=freq;
		this.left=left;
		this.right=right;
	}
	
	public boolean isLeaf() {
		return left==null&&right==null;
	}
	
	public int compareTo(Node that) {
		return this.freq-that.freq;
	}

}
```

##### Expansion for prefix-free codes

Expanding a bitstream that was encoded with a prefixfree code is simple, given the trie that defines the code.

```
public static void expand() {
	Node root=readTrie();
	int N=BinaryStdIn.readInt();
	for(int i=0;i<N;i++) {
		Node x=root;
		while(!x.isLeft()) {
			if(BinaryStdIn.readBoolean()) {
				x=x.right;
			}else {
				x=x.left;
			}
		}
		BinaryStdOut.write(x.ch);
	}
	BinaryStdOut.close();
}
```

```
private static void buildCode(String[] st, Node x, String s) {
	if(x.isLeaf()) {
		st[x.ch]=s;
		return;
	}
	buildCode(st,x.left,s+'0');
	buildCode(st,x.right,s+'1');
}
```

##### Compression for prefix-free codes

For compression, we use the trie that defines the code to build the code table, as shown in the buildCode() method at the top of this page. This method is compact and elegant, but a bit tricky, so it deserves careful study. For any trie, it produces a table giving the bitstring associated with each character in the trie (represented as a String of 0s and 1s).

```
for(int i=0;i<input.length;i++) {
	String code=st[input[i]];
	for(int j=0;j<code.length();j++) {
		if(code.charAt(j)=='1')
			BinaryStdOut.write(true);
		else
			BinaryStdOut.write(false);
	}
}
```

##### Trie construction

The process works as follows: we find the two nodes with the smallest frequencies and then create a new node with those two nodes as children (and with frequency value set to the sum of the values of the children). This operation reduces the number of tries in the forest by one. Then we iterate the process: find the two nodes with smallest frequency in that forest and a create a new node created in the same way. Implementing the process is straightforward with a priority queue, as shown in the buildTrie() method on page 830. (For clarity, the tries in the figure are kept in sorted order.)

```
private static Node buildTrie(int[] freq) {
	MinPQ<Node> pq=new MinPQ<Node>();
	for(char c=0;c<R;c++) {
		if(freq[c]>0) {
			pq.insert(new Node(c,freq[c],null,null));
		}
	}
	while(pq.size()>1) {
		Node x=pq.delMin();
		Node y=pq.delMin();
		Node parent=new Node('\0',x.freq+y.freq,x,y);
		pq.insert(parent);
	}
	return pq.delMin();
}
```

##### Optimality

We have observed that high-frequency characters are nearer the root of the tree than lower-frequency characters and are therefore encoded with fewer bits, so this is a good code, but why is it an optimal prefix-free code? To answer this question, we begin by defining the weighted external path length of a tree to be the sum of the weight (associated frequency count) times depth (see page 226) of all of the leaves.

##### Proposition T

For any prefi x-free code, the length of the encoded bitstring is equal to the weighted external path length of the corresponding trie.

##### Proposition U

Given a set of r symbols and frequencies, the Huffman algorithm builds an optimal prefix-free code.

```
```

```
```

##### Huffman compression implementation

Along with the methods buildCode(), buildTrie(), readTrie() and writeTrie() that we have just considered (and the expand() method that we considered first), Algorithm 5.10 is a complete implementation of Huffman compression. To expand the overview that we considered several pages earlier, we view the bitstream to be encoded as a stream of 8-bit char values and compress it as follows:

* Read the input.
* Tabulate the frequency of occurrence of each char value in the input.
* Build the Huffman encoding trie corresponding to those frequencies.
* Build the corresponding codeword table, to associate a bitstring with each char value in the input.
* Write the trie, encoded as a bitstring.
* Write the count of characters in the input, encoded as a bitstring.
* Use the codeword table to write the codeword for each input character.

To expand a bitstream encoded in this way, we

* Read the trie (encoded at the beginning of the bitstream)
* Read the count of characters to be decoded
* Use the trie to decode the bitstream

```
```

##### LZW compression

To fix ideas, we will consider a compression example where we read the input as a stream of 7-bit ASCII characters and write the output as a stream of 8-bit bytes. (In practice, we typically use larger values for these parameters—our implementations use 8-bit inputs and 12-bit outputs.)

##### The LZW compression algorithm

is based on maintaining a symbol table that associates string keys with (fixedlength) codeword values. We initialize the symbol table with the 128 possible singlecharacter string keys and associate them with 8-bit codewords obtained by prepending 0 to the 7-bit value defining each character.

To compress, we perform the following steps as long as there are unscanned input characters:

* Find the longest string s in the symbol table that is a prefi x of the unscanned
input.
* Write the 8-bit value (codeword) associated with s.
* Scan one character past s in the input.
* Associate the next codeword value with s + c (c appended to s) in the symbol table, where c is the next character in the input.

##### LZW trie representation

LZW compression involves two symbol-table operations:

* Find a longest-prefi x match of the input with a symbol-table key.
* Add an entry associating the next codeword

with the key formed by appending the lookahead character to that key.

##### LZW expansion

The input for LZW expansion in our example is a sequence of 8-bit codewords; the output is a string of 7-bit ASCII characters. To implement expansion, we maintain a symbol table that associates strings of characters with codeword values (the inverse of the table used for compression). We fill the table entries from 00 to 7F with one-character strings, one for each ASCII character, set the first unassigned codeword value to 81 (reserving 80 for end of file), set the current string val to the onecharacter string consisting of the first character, and perform the following steps until reading codeword 80 (end of file):

* Write the current string val.
* Read a codeword x from the input.
* Set s to the value associated with x in the symbol table.
* Associate the next unasssigned codeword value to val + c in the symbol table, where c is the first character of s.
* Set the current string val to s.

```
```

##### Tricky situation

There is a subtle bug in the process just described, one that is often discovered by students (and experienced programmers!) only after developing an implementation based on the description above.

##### Implementation

With these descriptions, implementing LZW encoding is straightforward, given in Algorithm 5.11 on the facing page (the implementation of expand() is on the next page).

```
```