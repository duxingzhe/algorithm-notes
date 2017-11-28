In this chapter, we consider classic algorithms for addressing the underlying computational challenges surrounding applications such as the following:

##### Information processing

When you search for web pages containing a given keyword, you are using a string-processing application. In the modern world, virtually all information is encoded as a sequence of strings, and the applications that process it are string-processing applications of crucial importance.

##### Genomics

Computational biologists work with a genetic code that reduces DNA to (very long) strings formed from four characters (A, C, T, and G). Vast databases giving codes describing all manner of living organisms have been developed in recent years, so that string processing is a cornerstone of modern research in computational biology.

##### Communications systems

When you send a text message or an email or download an ebook, you are transmitting a string from one place to another. Applications that process strings for this purpose were an original motivation for the development of string-processing algorithms.

##### Programming systems

Programs are strings. Compilers, interpreters, and other applications
that convert programs into machine instructions are critical applications that use sophisticated string-processing techniques. Indeed, all written languages are expressed as strings, and another motivation for the development of string-processing algorithms was the theory of formal languages, the study of describing sets of strings.

#### Rules of the game

For clarity and efficiency, our implementations are expressed in terms of the Java String class, but we intentionally use as few operations as possible from that class to make it easier to adapt our algorithms for use on other string-like types of data and to other programming languages.

##### Characters

A String is a sequence of characters. Characters are of type char and can have one of 216 possible values. For many decades, programmers restricted attention to characters encoded in 7-bit ASCII (see page 815 for a conversion table) or 8-bit extended ASCII, but many modern applications call for 16-bit Unicode.

##### Immutability

String objects are immutable, so that we can use them in assignment
statements and as arguments and return values from methods without having to worry about their values changing.

##### Indexing

The operation that we perform most often is extract a specified character from a string that the charAt() method in Java’s String class provides. We expect charAt() to complete its work in constant time, as if the string were stored in a char[] array. As discussed in Chapter 1, this expectation is quite reasonable.

##### Length

In Java, the find the length of a string operation is implemented in the length() method in String. Again, we expect length() to complete its work in constant time, and again, this expectation is reasonable, although some care is needed in some programming environments.

##### Substring

Java’s substring() method implements the extract a specified substring operation. Again, we expect a constant-time implementation of this method, as in Java’s standard implementation. If you are not familiar with substring() and the reason that it is constant-time, be sure to reread our discussion of Java’s standard string implementation in Section 1.2 (see page 80 and page 204).

##### Concatenation

In Java, the create a new string formed by appending one string to another operation is a built-in operation (using the + operator) that takes time proportional to the length of the result. For example, we avoid forming a string by appending one character at a time because that is a quadratic process in Java. (Java has a StringBuilder class for that use.)

##### Character arrays

The Java String is decidedly not a primitive type. The standard implementation provides the operations just described to facilitate client programming. By contrast, many of the algorithms that we consider can work with a low-level representation such as an array of char values, and many clients might prefer such a representation, because it consumes less space and takes less time. For several of the algorithms that we consider, the cost of converting from one representation to the other would be higher than the cost of running the algorithm. As indicated in the table below, the differences in code that processes the two representations are minor (substring() is more complicated and is omitted), so use of one representation or the other is no barrier to understanding the algorithm.

<table>
    <tr>
        <th>operation</th>
        <th>array of characters</th>
        <th>Java string</th>
    </tr>
    <tr>
        <th>declare</th>
        <th>char[] a</th>
        <th>String s</th>
    </tr>
    <tr>
        <th>indexed character access</th>
        <th>a[i]</th>
        <th>s.charAt(i)</th>
    </tr>
    <tr>
        <th>length</th>
        <th>a.length</th>
        <th>s.length</th>
    </tr>
    <tr>
        <th>convert</th>
        <th>a=s.toCharArray();</th>
        <th>s=new String(a)</th>
    </tr>
</table>
Two ways to represent strings in Java

#### Alphabets

Some applications involve strings taken from a restricted alphabet.

##### Character-indexed arrays

One of the most important reasons to use Alphabet is that many algorithms gain efficiency through the use of character-indexed arrays, where we associate information with each character that we can retrieve with a single array access. With a Java String, we have to use an array of size 65,536; with Alphabet, we just need an array with one entry for each alphabet character. Some of the algorithms that we consider can produce huge numbers of such arrays, and in such cases, the space for arrays of size 65,536 can be prohibitive.

##### Numbers

As you can see from several of the standard Alphabet examples, we often represent numbers as strings. The method toIndices() converts any String over a given Alphabet into a base-R number represented as an int[] array with all values between 0 and R-1. In some situations, doing this conversion at the start leads to compact code, because any digit can be used as an index in a character-indexed array.

Despite the advantages of using a data type such as Alphabet in string-processing algorithms (particularly for small alphabets), we do not develop our implementations in the book for strings taken from a general Alphabet because

* The preponderance of clients just use String
* Conversion to and from indices tends to fall in the inner loop and slow down implementations considerably
* The code is more complicated, and therefore more diffi cult to understand

