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