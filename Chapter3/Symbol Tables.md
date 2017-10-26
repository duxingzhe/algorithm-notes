### Symbol Tables

The primary purpose of a symbol table is to associate a value with a key. The client can insert key-value pairs into the symbol table with the expectation of later being able to search for the value associated with a given key, from among all of the key-value pairs that have been put into the table.

Search is so important to so many computer applications that symbol tables are available as high-level abstractions in many programming environments, including Java—we shall discuss Java’s symbol-table implementations in Section 3.5.

#### Definition

A symbol table is a data structure for key-value pairs that supports two operations: insert (put) a new pair into the table and search for (get) the value associated with a given key.

<table>
    <tr>
        <th >application</th>
        <th >purpose of search</th>
        <th >key</th>
        <th >value</th>
    </tr>
    <tr>
        <th>dictionary</th>
        <th>find definition</th>
        <th>word</th>
        <th>definition</th>
    </tr>
    <tr>
        <th>book index</th>
        <th>find relevant pages</th>
        <th>term</th>
        <th>list of page numbers</th>
    </tr>
    <tr>
        <th>file share</th>
        <th>find song to download</th>
        <th>name of song</th>
        <th>computer ID</th>
    </tr>
    <tr>
        <th>account management</th>
        <th>process transactions</th>
        <th>account number</th>
        <th>transaction details</th>
    </tr>
    <tr>
        <th>web search</th>
        <th>find relevant web pages</th>
        <th>keyword</th>
        <th>list of page names</th>
    </tr>
    <tr>
        <th>compiler</th>
        <th>find type and value</th>
        <th>variable name</th>
        <th>type and value</th>
    </tr>
</table>

Typical symbol-table applications