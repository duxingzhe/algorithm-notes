### Regular Expressions

IN many applications, we need to do substring searching with somewhat less than complete information about the pattern to be found. A user of a text editor may wish to specify only part of a pattern, or to specify a pattern that could match a few different words, or to specify that any one of a number of patterns would do. A biologist might search for a genomic sequence satisfying certain conditions. In this section we will consider how pattern matching of this type can be done efficiently.

The algorithms in the previous section fundamentally depend on complete specification of the pattern, so we have to consider different methods. The basic mechanisms we will consider make possible a very powerful string-searching facility that can match complicated M-character patterns in N-character text strings in time proportional to MN in the worst case, and much faster for typical applications.

First, we need a way to describe the patterns: a rigorous way to specify the kinds of partial-substring-searching problems suggested above. This specification needs to involve more powerful primitive operations than the “check if the i th character of the text string matches the j th character of the pattern’’ operation used in the previous section. For this purpose, we use regular expressions, which describe patterns in combinations of three natural, basic, and powerful operations.

#### Describing patterns with regular expressions

We focus on pattern descriptions made up of characters that serve as operands for three fundamental operations. In this context, we use the word language specifically to refer to a set of strings (possibly infinite) and the word pattern to refer to a language specification.

##### Concatenation

The first fundamental operation is the one used in the last section. When we write AB , we are specifying the language { AB } that has one two-character string, formed by concatenating A and B.

##### Or

The second fundamental operation allows us to specify alternatives in the pattern. If we have an or between two alternatives, then both are in the language. We will use the vertical bar symbol | to denote this operation.

##### Closure

The third fundamental operation allows parts of the pattern to be repeated arbitrarily. The closure of a pattern is the language of strings formed by concatenating the pattern with itself any number of times (including zero). We denote closure by placing a * after the pattern to be repeated. Closure has higher precedence than concatenation, so AB* specifies the language consisting of strings with an A followed by 0 or more Bs, while A * B specifies the language consisting of strings with 0 or more As followed by a B. The empty string, which we denote by , is found in every text string (and in A*).

##### Parentheses

We use parentheses to override the default precedence rules. For example, C(AC|B)D specifies the language { CACD , CBD }; (A|C)((B|C)D) specifies the language { ABD , CBD , ACD , CCD }; and (AB)* specifies the language of strings formed by concatenating any number of occurrences of AB , including no occurrences: { , AB , ABAB , . . .}.

##### Definition

A regular expression (RE) is either

* Empty
* A single character
* A regular expression enclosed in parentheses
* Two or more concatenated regular expressions
* Two or more regular expressions separated by the or operator (|)
* A regular expression followed by the closure operator (*)

This definition describes the syntax of regular expressions, telling us what constitutes a legal regular expression. The semantics that tells us the meaning of a given regular expression is the point of the informal descriptions that we have given in this section.

##### Definition (continued)

Each RE represents a set of strings, defi ned as follows:

* The empty RE represents the empty set of strings, with 0 elements.
* A character represents the set of strings with one element, itself.
* An RE enclosed in parentheses represents the same set of strings as the RE without the parentheses.
* The RE consisting of two concatenated REs represents the cross product of the sets of strings represented by the individual components (all possible strings that can be formed by taking one string from each and concatenating them, in the same order as the REs).
* The RE consisting of the or of two REs represents the union of the sets represented by the individual components.
* The RE consisting of the closure of an RE represents  (the empty string) or the union of the sets represented by the concatenation of any number of copies of the RE.

#### Shortcuts

From a theoretical standpoint, these are each simply a shortcut for a sequence of operations involving many operands; from a practical standpoint, they are a quite useful extention to the basic operations that enable us to develop compact patterns.


<table>
    <tr>
        <th>name</th>
		<th>notation</th>
		<th>example</th>
    </tr>
    <tr>
        <th>wildcard</th>
        <th>.</th>
		<th>A.B</th>
    </tr>
    <tr>
        <th>specified set</th>
        <th>enclosed in []</th>
		<th>[AEIOU]*</th>
    </tr>
	<tr>
        <th rowspan="2">range</th>
        <th>enclosed in []</th>
		<th>[A-Z]</th>
    </tr>
    <tr>
        <th>separated by -</th>
		<th>[0-9]</th>
    </tr>
	<tr>
        <th rowspan="2">complement</th>
        <th>enclosed in []</th>
		<th rowspan="2">[^AEIOU]</th>
    </tr>
    <tr>
        <th>preceded by ^</th>
    </tr>
</table>
Set-of-characters descriptors

##### Set-of-characters descriptors

It is often convenient to be able to use a single character or a short sequence to directly specify sets of characters. The dot character (.) is a wildcard that represents any single character. A sequence of characters within square brackets represents any one of those characters. The sequence may also be specified as a range of characters. If preceded by a ^, a sequence within square brackets represents any character but one of those characters. These notations are simply shortcuts for a sequence of or operations.

##### Closure shortcuts

The closure operator specifies any number of copies of its operand. In practice, we want the flexibility to specify the number of copies, or a range on the number. In particular, we use the plus sign (+) to specify at least one copy, the question mark (?) to specify zero or one copy, and a count or a range within braces ({}) to specify a given number of copies. Again, these notations are shortcuts for a sequence of the basic concatenation, or, and closure operations.

##### Escape sequences

Some characters, such as \, ., |, *, (, and ), are metacharacters that we use to form regular expressions. We use escape sequences that begin with a backslash character \ separating metacharacters from characters in the alphabet. An escape sequence may be a \ followed by a single metacharacter (which represents that character). For example, \\\\ represents \\. Other escape sequences represent special characters and whitespace. For example, \t represents a tab character, \n represents a newline, and \s represents any whitespace character.

#### REs in applications

REs have proven to be remarkably versatile in describing languages that are relevant in practical applications. Accordingly, REs are heavily used and have been heavily studied. To familiarize you with regular expressions while at the same time giving you some appreciation for their utility, we consider a number of practical applications before addressing the RE pattern-matching algorithm.

##### Substring search

Our general goal is to develop an algorithm that determines whether a given text string is in the set of strings described by a given regular expression. If a text is in the language described by a pattern, we say that the text matches the pattern. Pattern matching with REs vastly generalizes the substring search problem of Section 5.3. Precisely, to search for a substring pat in a text string txt is to check whether txt is in the language described by the pattern . * pat. * or not.

##### Validity checking

You frequently encounter RE matching when you use the web. When you type in a date or an account number on a commercial website, the inputprocessing program has to check that your response is in the right format. One approach to performing such a check is to write code that checks all the cases: if you were to type in a dollar amount, the code might check that the first symbol is a $, that the $ is followed by a set of digits, and so forth. A better approach is to define an RE that describes the set of all legal inputs. Then, checking whether your input is legal is precisely the pattern-matching problem: is your input in the language described by the RE? Libraries of REs for common checks have sprung up on the web as this type of checking has come into widespread use. Typically, an RE is a much more precise and concise expression of the set of all valid strings than would be a program that checks all the cases.

<table>
    <tr>
        <th>context</th>
		<th>regular expression</th>
		<th>matches</th>
    </tr>
    <tr>
        <th>substring search</th>
        <th>.*NEEDLE.*</th>
		<th>A HAYSTACK NEEDLE IN</th>
    </tr>
    <tr>
        <th>phone number</th>
        <th>\([0-9]{3}\)\ [0-9]{3}-[0-9]{4}</th>
		<th>(800) 867-5309</th>
    </tr>
	<tr>
        <th>Java identifier</th>
        <th>[$_A-Za-z][$_A-Za-z0-9]*</th>
		<th>Pattern_Matcher</th>
    </tr>
	<tr>
        <th>genome marker</th>
        <th>gcg(cgg|agg)*ctg</th>
		<th>gcgaggaggcggcggctg</th>
    </tr>
	<tr>
        <th>email address</th>
        <th>[a-z]+@([a-z]+\.)+(edu|com)</th>
		<th>rs@cs.princeton.edu</th>
    </tr>
</table>
Typical regular expressions in applications (simplified versions)

##### Programmer’s toolbox

The origin of regular expression pattern matching is the Unix
command grep, which prints all lines matching a given RE. This capability has proven
invaluable for generations of programmers, and REs are built into many modern programming
systems, from awk and emacs to Perl, Python, and JavaScript. For example,
suppose that you have a directory with dozens of .java files, and you want to know
which of them has code that uses StdIn. The command

% grep StdIn *.java

will immediately give the answer. It prints all lines that match .*StdIn.* for each file.

##### Genomics

Biologists use REs to help address important scientific problems. For example, the human gene sequence has a region that can be described with the RE gcg(cgg)*ctg, where the number of repeats of the cgg pattern is highly variable among individuals, and a certain genetic disease that can cause mental retardation and
other symptoms is known to be associated with a high number of repeats.

##### Search

Web search engines support REs, though not always in their full glory. Typically, if you want to specify alternatives with | or repetition with \*, you can do so. Possibilities. A first introduction to theoretical computer science is to think about the set of languages that can be specified with an RE. For example, you might be surprised to know that you can implement the modulus operation with an RE: for example, (0 | 1(01*0)*1)\* describes all strings of 0s and 1s that are the binary representatons of numbers that are multiples of three (!): 11 , 110 , 1001 , and 1100 are in the language, but 10 , 1011 , and 10000 are not. Limitations. Not all languages can be specified with REs. A thought-provoking example is that no RE can describe the set of all strings that specify legal REs. Simpler versions of this example are that we cannot use REs to check whether parentheses are balanced or to check whether a string has an equal number of As and Bs.