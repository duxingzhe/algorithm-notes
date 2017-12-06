### Substring Search

A fundamental operation on strings is substring search: given a text string of length N and a pattern string of length M, find an occurrence of the pattern within the text.

#### A short history

The algorithms that we examine have an interesting history; we summarize it here to help place the various methods io perspective. There is a simple brute-force algorithm for substring search that is in widespread use. While it has a worst-case running time proportional to MN, the strings that arise in many applications lead to a running time that is (except in pathological cases) proportional to M  N. Furthermore, it is well-suited to standard architectural features on most computer systems, so an optimized version provides a standard benchmark that is difficult to beat, even with a clever algorithm.

Knuth, Morris, and Pratt didn’t get around to publishing their algorithm until 1976, and in the meantime R. S. Boyer and J. S. Moore (and, independently, R. W. Gosper) discovered an algorithm that is much faster in many applications, since it often examines only a fraction of the characters in the text string. Many text editors use this algorithm to achieve a noticeable decrease in response time for substring search. Both the Knuth-Morris-Pratt (KMP) and the Boyer-Moore algorithms require some complicated preprocessing on the pattern that is difficult to understand and has limited the extent to which they are used. (In fact, the story goes that an unknown systems programmer found Morris’s algorithm too difficult to understand and replaced it with a brute-force implementation.)

Brute-force substring search An obvious method for substring search is to check, for each possible position in the text at which the pattern could match, whether it does in fact match.

```
public int search(String pat, String txt) {
    int M=pat.length();
    int N=txt.length();
    for(int i=0;i<=N-M;i++){
        int j;
        for(j=0;j<M;j++)
            if(txt.charAt(i+j)!=pat.charAt(j))
                break;
        if(j==M)
            return i;
    }
    return N;
}
```

##### Proposition M

Brute-force substring search requires ~NM character compares to search for a pattern of length M in a text of length N, in the worst case.

```
public int search(String pat, String txt) {
    int j,M=pat.length();
    int i,N=txt.length();
    for(i=0;i<=N-M;i++){
        if(txt.charAt(i)!=pat.charAt(j))
            j++
        else {
            i-=j;
            j=0;
        }
    }
    return N;
}
```

##### Knuth-Morris-Pratt substring search

The basic idea behind the algorithm discovered by Knuth, Morris, and Pratt is this: whenever we detect a mismatch, we already know some of the characters in the text (since they matched the pattern characters prior to the mismatch).

##### KMP search method

Once we have computed the dfa[][] array, we have the substring search method at the top of the next page: when i and j point to mismatching characters (testing for a pattern match beginning at position i-j+1 in the text string), then the next possible position for a pattern match is beginning at position i-dfa[txt.charAt(i)][j]. But by construction, the first dfa[txt.charAt(i)][j] characters at that position match the first dfa[txt.charAt(i)][j] characters of the pattern, so there is no need to back up the i pointer: we can simply set j to dfa[txt.charAt(i)][j] and increment i, which is precisely what we do when i and j point to matching characters.

##### DFA simulation

A useful way to describe this process is in terms of a deterministic finite-state automaton (DFA). Indeed, as indicated by its name, our dfa[][] array precisely defines a DFA. The graphical DFA represention shown at the bottom of this page consists of states (indicated by circled numbers) and transitions (indicated by labeled lines).

##### Constructing the DFA

When we have a mismatch at pat.charAt(j), our interest is in knowing in what state the DFA would be if we were to back up the text index and rescan the text characters that we just saw after shifting to the right one position. We do not want to actually do the backup, just restart the DFA as if we had done the backup. The key observation is that the characters in the text that would need to be rescanned are precisely pat.charAt(1) through pat.charAt(j-1): we drop the first character to shift right one position and the last character because of the mismatch. These are pattern characters that we know, so we can figure out ahead of time, for each possible mismatch position, the state where we need to restart the DFA. The figure at left shows the possibilities for our example.

##### DFA is in next chapter.

The discussion above leads to the remarkably compact code below for constructing
the DFA corresponding to a given pattern. For each j, it

* Copies dfa[][X] to dfa[][j] (for mismatch cases)
* Sets dfa[pat.charAt(j)][j] to j+1 (for the match case)
* Updates X

```
public class KMP {

	private Strng pat;
	private int[][] dfa;
	
	public KMP(String pat) {
		this.pat=pat;
		int M=pat.length();
		int R=256;
		dfa=new int[R][M];
		dfa[pat.charAt(0)][0]=1;
		for(int X=0,j=1;j<M;j++) {
			for(int c=0;c<R;c++) {
				dfa[c][j]=dfa[c][X];
			}
			dfa[pat.charAt(j)][j]=j+1;
			X=dfa[pat.charAt(j)][X];
		}
	}
	
	public int search(String txt) {
		int i,j,N=txt.length(),M=pat.length();
		for(i=0,j=0;i<N&&j<M;i++) {
			j=dfa[txt.charAt(i)][j];
		}
		if(j==M)
			return i-M;
		else
			return N;
	}
}
```

##### Proposition N

Knuth-Morris-Pratt substring search accesses no more than M+N characters to search for a pattern of length M in a text of length N.

#### Boyer-Moore substring

search When backup in the text string is not a problem, we can develop a significantly faster substring-searching method by scanning the pattern from right to left when trying to match it against the text.

##### Mismatched character heuristic

Starting point. To implement the mismatched character heuristic, we use an array right[] that gives, for each character in the alphabet, the index of its rightmost occurrence in the pattern (or -1 if the character is not in the pattern).

##### Substring search

With the right[] array precomputed, the implementation in Algorithm 5.7 is straightforward. We have an index i moving from left to right through the text and an index j moving from right to left through the pattern. The inner loop tests whether the pattern aligns with the text at position i. If txt.charAt(i+j) is equal to pat.charAt(j) for all j from M-1 down to 0, then there is a match. Otherwise, there is a character mismatch, and we have one of the following three cases:

* If the character causing the mismatch is not found in the pattern, we can slide the pattern j+1 positions to the right (increment i by j+1). Anything less would align that character with some pattern character. Actually, this move aligns some known characters at the beginning of the pattern with known characters at the end of the pattern so that we could further increase i by precomputing a KMP-like table (see example at right).
* If the character c causing the mismatch is found in the pattern, we use the right[] array to line up the pattern with the text so that character will match its rightmost occurrence in the pattern. To do so, we increment i by j minus right[c]. Again, anything less would align that text character with a pattern character it could not match (one to the right of its rightmost occurrence). Again, there is a possibility that we could do better with a KMP-like table, as indicated in the top example in the figure on page 773.
* If this computation would not increase i, we just increment i instead, to make sure that the pattern always slides at least one position to the right. The bottom example in the figure at right illustrates this situation.

Algorithm 5.7 is a straightforward implementation of this process.

```
public class BoyerMoore {
	
	private int[] right;
	private String pat;
	
	BoyerMoore(String pat){
		this.pat=pat;
		int M=pat.length();
		int R=256;
		right=new int[R];
		for(int c=0;c<R;c++) {
			right[c]=-1;
		}
		for(int j=0;j<M;j++) {
			right[pat.charAt(j)]=j;
		}
	}
	
	public int search(String txt) {
		int N=txt.length();
		int M=pat.length();
		int skip;
		for(int i=0;i<N-M;i+=skip) {
			skip=0;
			for(int j=M-1;j>=0;j--) {
				if(pat.charAt(j)!=txt.charAt(i+j)) {
					skip=j-right[txt.charAt(i+j)];
					if(skip<1)
						skip=1;
					break;
				}
			}
			if(skip==0)
				return i;
		}
		return N;		
	}

}
```

##### Property O

On typical inputs, substring search with the Boyer-Moore mismatched character heuristic uses ~N/M character compares to search for a pattern of length M in a text of length N.

#### Rabin-Karp fingerprint search

The method developed by M. O. Rabin and R. A. Karp is a completely different approach to substring search that is based on hashing. We compute a hash function for the pattern and then look for a match by using the same hash function for each possible M-character substring of the text. If we find a text substring with the same hash value as the pattern, we can check for a match. This process is equivalent to storing the pattern in a hash table, then doing a search for each substring of the text, but we do not need to reserve the memory for the hash table because it would have just one entry.

##### Basic plan

A string of length M corresponds to an M-digit base-R number. To use a hash table of size Q for keys of this type, we need a hash function to convert an M-digit base-R number to an int value between 0 and Q-1. Modular hashing (see Section 3.4) provides an answer: take the remainder when dividing the number by Q. In practice, we use a random prime Q, taking as large a value as possible while avoiding overflow (because we do not actually need to store a hash table).

##### Computing the hash function

With five-digit values, we could just do all the necessary calculations with int values, but what do we do when M is 100 or 1,000? A simple application of Horner’s method, precisely like the method that we examined in Section 3.4 for strings and other types of keys with multiple values, leads to the code shown at right, which computes the hash function for an M-digit base-R number represented as a char array in time proportional to M. (We pass M as an argument so that we can use the method for both the pattern and the text, as you will see.)

```
public class RabinKarp {
	private String pat;
	private long patHash;
	private int M;
	private long Q;
	private int R=256;
	private long RM;
	
	public RabinKarp(String pat) {
		this.pat=pat;
		this.M=pat.length();
		Q=longRandomPrime();
		RM=1;
		for(int i=1;i<=M-1;i++) {
			RM=(R*RM)%Q;
		}
		patHash=hash(pat,M);
	}
	
	public boolean check(int i) {
		return true;
	}
	
	private long hash(String key, int M){
		
	}
	
	private int search(String txt) {
		int N=txt.length();
		long txtHash=hash(txt,M);
		if(patHash==txtHash)
			return 0;
		for(int i=M;i<N;i++) {
			txtHash=(txtHash+Q-RM*txt.charAt(i-M)%Q)%Q;
			txtHash=(txtHash*R+txt.charAt(i))%Q;
			if(patHash==txtHash)
				if(check(i-M+1))
					return i-M+1;
		}
		return N;
	}
	
}
```

##### Key idea

The Rabin-Karp method is based on efficiently computing the hash function for position i+1 in the text, given its value for position i. It follows directly from a simple mathematical formulation. Using the notation ti for txt.charAt(i), the number corresponding to the M-character substring of txt that starts at position i is

![](http://latex.codecogs.com/gif.latex?x_i=t_iR^{M-2}+...+t_{i+M-1}R^0)

and we can assume that we know the value of h(xi) = xi mod Q . Shifting one position right in the text corresponds to replacing xi by

![](http://latex.codecogs.com/gif.latex?x_{i+1}=(x_i-t_iR^{M-1})R+t_{i+M})

We subtract off the leading digit, multiply by R, then add the trailing digit.

##### Implementation

The constructor computes a hash value patHash for the pattern; it also computes the value of RM-1mod Q in the variable RM. The hashSearch() method begins by computing the hash function for the first M characters of the text and comparing that value against the hash value for the pattern. If that is not a match, it proceeds through the text string, using the technique above to maintain the hash function for the M characters starting at position i for each i in a variable txtHash and comparing each new hash value to patHash. (An extra Q is added during the txtHash calculation to make sure that everything stays positive so that the remainder operation works as it should.)

##### A trick: Monte Carlo correctness

After finding a hash value for an M-character substring of txt that matches the pattern hash value, you might expect to see code to compare those characters with the pattern to ensure that we have a true match, not just a hash collision. We do not do that test because using it requires backup in the text string. Instead, we make the hash table “size” Q as large as we wish, since we are not actually building a hash table, just testing for a collision with one key, our pattern.

```
```

##### Property P

The Monte Carlo version of Rabin-Karp substring search is linear-time and extremely likely to be correct, and the Las Vegas version of Rabin-Karp substring search is correct and extremely likely to be linear-time.

#### Summary

<table>
    <tr>
        <th rowspan="2">algorithm</th>
		<th rowspan="2">stable?</th>
		<th rowspan="2">inplace?</th>
        <th colspan="2">order of growth of typical number calls to charAt() to sort N strings from an R-character alphabet (average length w, max length W)</th>
		<th rowspan="2">sweet spot</th>
    </tr>
    <tr>
        <th>running time</th>
        <th>extra space</th>
    </tr>
    <tr>
        <th>insertion sort for strings</th>
        <th>yes</th>
		<th>yes</th>
		<th>between N and <img src=http://latex.codecogs.com/gif.latex?N^2></img></th>
		<th>1</th>
		<th>small arrays, arrays in order</th>
    </tr>
    <tr>
        <th>quicksort</th>
        <th>no</th>
		<th>yes</th>
		<th><img src=http://latex.codecogs.com/gif.latex?Nlog^{2}N></img></th>
		<th><img src=http://latex.codecogs.com/gif.latex?logN></img></th>
		<th>gernal-purpose when space is tight</th>
    </tr>
	<tr>
        <th>mergesort</th>
        <th>yes</th>
		<th>no</th>
		<th><img src=http://latex.codecogs.com/gif.latex?Nlog^{2}N></img></th>
		<th><img src=http://latex.codecogs.com/gif.latex?logN></img></th>
		<th>gernal-purpose stable sort</th>
    </tr>
	<tr>
        <th>3-way quicksort</th>
        <th>no</th>
		<th>yes</th>
		<th>between N and <img src=http://latex.codecogs.com/gif.latex?NlogN></img></th>
		<th><img src=http://latex.codecogs.com/gif.latex?logN></img></th>
		<th>large numbers of equal keys</th>
    </tr>
	<tr>
        <th>LSD string sort</th>
        <th>yes</th>
		<th>no</th>
		<th>NW</th>
		<th>N</th>
		<th>short fixed-length strings</th>
    </tr>
	<tr>
        <th>MSD string sort</th>
        <th>yes</th>
		<th>no</th>
		<th>between N and Nw</th>
		<th>N+WR</th>
		<th>random strings</th>
    </tr>
	<tr>
        <th>3-way string quicksort</th>
        <th>no</th>
		<th>yes</th>
		<th>between N and Nw</th>
		<th>W+<img src=http://latex.codecogs.com/gif.latex?logN></img></th>
		<th>general-purpose, strings with long prefix matches</th>
    </tr>
</table>
Performance characteristics of string-sorting algorithms