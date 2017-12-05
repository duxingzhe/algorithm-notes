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