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

<table>
    <tr>
        <th>option</th>
		<th>notation</th>
		<th>example</th>
        <th>shortcut for</th>
        <th>in language</th>
        <th>not in language</th>
    </tr>
    <tr>
        <th>at least 1</th>
        <th>+</th>
		<th>(AB)+</th>
        <th>(AB)(AB)*</th>
        <th>AB ABABAB</th>
        <th> BBBAAA</th>
    </tr>
    <tr>
        <th>0 or 1</th>
        <th>?</th>
		<th>(AB)?</th>
        <th>| AB</th>
        <th> AB</th>
        <th>any other string</th>
    </tr>
	<tr>
        <th>specific</th>
        <th>count in{3}</th>
		<th>(AB){3}</th>
        <th>(AB)(AB)(AB)</th>
        <th>ABABAB</th>
        <th>any other string</th>
    </tr>
	<tr>
        <th>range</th>
        <th>range in {}</th>
		<th>(AB){1-2}</th>
        <th>(AB)|(AB)(AB)</th>
        <th>AB ABAB</th>
        <th>any other string</th>
    </tr>
</table>
Closure shortcuts (for specifying the number of copies of the operand)

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

Web search engines support REs, though not always in their full glory. Typically, if you want to specify alternatives with | or repetition with \*, you can do so. 

##### Possibilities

A first introduction to theoretical computer science is to think about the set of languages that can be specified with an RE. For example, you might be surprised to know that you can implement the modulus operation with an RE: for example, (0 | 1(01*0)*1)\* describes all strings of 0s and 1s that are the binary representatons of numbers that are multiples of three (!): 11 , 110 , 1001 , and 1100 are in the language, but 10 , 1011 , and 10000 are not.

##### Limitations

Not all languages can be specified with REs. A thought-provoking example is that no RE can describe the set of all strings that specify legal REs. Simpler versions of this example are that we cannot use REs to check whether parentheses are balanced or to check whether a string has an equal number of As and Bs.

##### Nondeterministic finite-state automata

Recall that we can view the Knuth-Morris-Pratt algorithm as a finite-state machine constructed from the search pattern that scans the text. For regular expression pattern matching, we will generalize this idea.

The overview of our RE pattern matching algorithm is the nearly the same as for KMP:

* Build the NFA corresponding to the given RE.
* Simulate the operation of that NFA on the given text.

Kleene’s Theorem, a fundamental result of theoretical computer science, asserts that there is an NFA corresponding to any given RE (and vice versa). We will consider a constructive proof of this fact that will demonstrate how to transform any RE into an NFA; then we simulate the operation of the NFA to complete the job.

As illustrated in this example, the NFAs that we define have the following characteristics:

* The NFA corresponding to an RE of length M has exactly one state per pattern character, starts at state 0, and has a (virtual) accept state M.
* States corresponding to a character from the alphabet have an outgoing edge that goes to the state corresponding to the next character in the pattern (black edges in the diagram).
* States corresponding to the metacharacters (, ), |, and * have at least one outgoing edge (red edges in the diagram), which may go to any other state.
* Some states have multiple outgoing edges, but no state has more than one outgoing black edge.

As with the DFAs of the previous section, we start the NFA at state 0, reading the first character of a text. The NFA moves from state to state, sometimes reading text characters, one at a time, from left to right. However, there are some basic differences from DFAs:

* Characters appear in the nodes, not the edges, in the diagrams.
* Our NFA recognizes a text string only after explicitly reading all its characters, whereas our DFA recognizes a pattern in a text without necessarily reading all the text characters.

The rules for moving from one state to another are also different than for DFAs—an NFA can do so in one of two ways:

* If the current state corresponds to a character in the alphabet and the current character in the text string matches the character, the automaton can scan past the character in the text string and take the (black) transition to the next state. We refer to such a transition as a match transition.
* The automaton can follow any red edge to another state without scanning any text character. We refer to such a transition as an -transition, referring to the idea that it corresponds to “matching” the empty string .

These examples illustrate the key difference between NFAs and DFAs: since an NFA may have multiple edges leaving a given state, the transition from such a state is not deterministic—it might take one transition at one point in time and a different transition at a different point in time, without scanning past any text character. To make some sense of the operation of such an automaton, imagine that an NFA has the power to guess which transition (if any) will lead to the accept state for the given text string. In other words, we say that an NFA recognizes a text string if and only if there is some sequence of transitions that scans all the text characters and ends in the accept state when started at the beginning of the text in state 0. Conversely, an NFA does not recognize a text string if and only if there is no sequence of match transitions and -transitions that can scan all the text characters and lead to the accept state for that string.

As with DFAs, we have been tracing the operation of the NFA on a text string simply by listing the sequence of state changes, ending in the final state. Any such sequence is a proof that the machine recognizes the text string (there may be other proofs). But how do we find such a sequence for a given text string? And how do we prove that there is no such sequence for another given text string? The answers to these questions are easier than you might think: we systematically try all possibilities.

##### Simulating an NFA

The idea of an automaton that can guess the state transitions it needs to get to the accept state is like writing a program that can guess the right answer to a problem: it seems ridiculous. On reflection, you will see that the task is conceptually not at all difficult: we make sure that we check all possible sequences of state transitions, so if there is one that gets to the accept state, we will find it.

##### Representation

To begin, we need an NFA representation. The choice is clear: the RE itself gives the state names (the integers between 0 and M, where M is the number of characters in the RE).

##### NFA simulation and reachability

To simulate an NFA, we keep track of the set of states that could possibly be encountered while the automaton is examining the current input character. The key computation is the familiar multiple-source reachability computation that we addressed in Algorithm 4.4 (page 571).

Iterating this process until all text characters are exhausted leads to one of two outcomes:

* The set of possible states contains the accept state.
* The set of possible states does not contain the accept state.

The first of these outcomes indicates that there is some sequence of transitions that takes the NFA to the accept state, so we report success. The second of these outcomes indicates that the NFA always stalls on that input, so we report failure.

##### Proposition Q

Determining whether an N-character text string is recognized by the NFA corresponding to an M-character RE takes time proportional to NM in the worst case.

```
public boolean recognizes(String txt) {
    Bag<Integer> pc=new Bag<Integer>();
    DirectedDFS dfs=new DirectedDFS(G,0);
    for(int v=0;v<G.V();v++) {
        if(dfs.marked(v))
            pc.add(v);
    }
    for(int i=0;i<txt.length();i++) {
        Bag<Integer> match=new Bag<Integer>();
        for(int v: pc) {
            if(v<M) {
                if(re[v]==txt.charAt(i)||re[v]=='.') {
                    match.add(v+1);
                }
            }
        }
        pc=new Bag<Integer>();
        dfs=new DirectedDFS(G,match);
        for(int v=0;v<G.V();v++) {
            if(dfs.marked(v))
                pc.add(v);
        }
    }
    for(int v:pc)
        if(v==M)
            return true;
    return false;
}
```

#### Building an NFA corresponding to an RE

From the similarity between regular expressions and familiar arithmetic expressions, you may not be surprised to find that translating an RE to an NFA is somewhat similar to the process of evaluating an arithmetic expression using Dijkstra’s two-stack algorithm, which we considered in Section 1.3. The process is a bit different because

* REs do not have an explicit operator for concatenation
* REs have a unary operator, for closure (*)
* REs have only one binary operator, for or (|)

Taking a cue from Dijkstra’s algorithm, we will use a stack to keep track of the positions of left parentheses and or operators.

##### Concatenation

In terms of the NFA, the concatenation operation is the simplest to implement. Match transitions for states corresponding to characters in the alphabet explicitly implement concatenation.

##### Parentheses

We push the RE index of each left parenthesis on the stack. Each time we encounter a right parenthesis, we eventually pop the corresponding left parentheses from the stack in the manner described below. As in Dijkstra’s algorithm, the stack enables us to handle nested parentheses in a natural manner.

##### Closure

A closure (*) operator must occur either (i ) after a single character, when we add -transitions to and from the character, or (ii ) after a right parenthesis, when we add -transitions to and from the corresponding left parenthesis, the one at the top of the stack.

##### Or expression

We process an RE of the form (A | B) where A and B are both REs by adding two -transitions: one from the state corresponding to the left parenthesis to the state corresponding to the first character of B and one from the state corresponding to the | operator to the state corresponding to the right parenthesis. We push the RE index corresponding the | operator onto the stack (as well as the index corresponding to the left parenthesis, as described above) so that the information we need is at the top of the stack when needed, at the time we reach the right parenthesis. These -transitions allow the NFA to choose one of the two alternatives. We do not add an -transition from the state corresponding to the | operator to the state with the next higher index, as we havefor all other states—the only way for the NFA to leave such a state is to take a transition to the state corresponding to the right parenthesis.

```
public class NFA {

	private char[] re;
	private Digraph G;
	private int M;
	
	public NFA(String regexp) {
		Stack<Integer> ops=new Stack<Integer>();
		re=regexp.toCharArray();
		M=re.length;
		G=new Digraph(M+1);
		
		for(int i=0;i<M;i++) {
			int lp=i;
			if(re[i]='('||re[i]=='|')
				ops.push(i);
			else if(re[i]==')') {
				int or=ops.pop();
				if(re[or]=='|') {
					lp=ops.pop();
					G.addEdge(lp, or+1);;
					G.addEdge(or, i);
				}
				else
					lp=or;
			}
			if(i<M-1&&re[i+1]=='*') {
				G.addEdge(lp, i+1);
				G.addEdge(i+1, lp);
			}
			if(re[i]=='('||re[i]=='*'||re[i]==')')
				G.addEdge(i, i+1);
		}
	}
	
	public boolean recognizes(String txt)	
	
}
```

##### Proposition R

Building the NFA corresponding to an M-character RE takes time and space proportional to M in the worst case.

```
public class GREP {

	public static void main(String[] args) {
		String regexp="(.*"+args[0]+".*)";
		NFA nfa=new NFA(regexp);
		while(StdIn.hasNextLine()) {
			String txt=StdIn.hasNextLine();
			if(nfa.recongizes(txt))
				StdOut.println(txt);
		}
	}
}
```