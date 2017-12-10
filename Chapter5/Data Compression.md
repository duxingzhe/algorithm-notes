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

