I can learn more in the Chapter 3 of Introduction to Algorithm.

At Page 181 (Algorithm 4th edition) , it give us an example to illustrate the method to calculate the length of running time.

For many programs, developing a mathematical model of running time
reduces to the following steps:
* Develop an input model, including a defi nition of the problem size.
* Identify the inner loop.
* Define a cost model that includes operations in the inner loop.
* Determine the frequency of execution of those operations for the given input.
Doing so might require mathematical analysis—we will consider some examples in the context of specific fundamental algorithms later in the book.

### Order-of-growth classifications

*Constant*

A program whose running time’s order of growth is constant executes a fixed number of operations to finish its job; consequently its running time does not depend on N. Most Java operations take constant time.

*Logarithmic*

A program whose running time’s order of growth is logarithmic is barely slower than a constant-time program. The classic example of a program whose running time is logarithmic in the problem size is binary search (see BinarySearch on page 47).
The base of the logarithm is not relevant with respect to the order of growth (since all logarithms with a constant base are related by a constant factor), so we use log N when referring to order of growth.

*Linear*

Programs that spend a constant amount of time processing each piece of input data, or that are based on a single for loop, are quite common. The order of growth of such a program is said to be linear —its running time is proportional to N.

*Linearithmic*

We use the term linearithmic to describe programs whose running time for a problem of size N has order of growth N log N. Again, the base of the logarithm is not relevant with respect to the order of growth. The prototypical examples of lin-earithmic algorithms are Merge.sort() (see Algorithm 2.4) and Quick.sort() (see Algorithm 2.5).

*Quadratic*

A typical program whose running time has order of growth N 2 has two nested for loops, used for some calculation involving all pairs of N elements. The elementary sorting algorithms Selection.sort() (see Algorithm 2.1) and Insertion.sort() (see Algorithm 2.2) are prototypes of the programs in this classiﬁcation. 

*Cubic*

 A typical program whose running time has order of growth N 3 has three nested for loops, used for some calculation involving all triples of N elements. Our example for this section, ThreeSum, is a prototype. 

*Exponential*

In ChAPter 6 (but not until then!) we will consider programs whose running times are proportional to ![](http://latex.codecogs.com/gif.latex?2^n) or higher. Generally, we use the term exponential to refer to algorithms whose order of growth is ![](http://latex.codecogs.com/gif.latex?b^n) for any constant b > 1, even though different values of b lead to vastly different running times. Exponential algorithms are extremely slow — you will never run one of them to completion for a large problem. Still, exponential algorithms play a critical role in the theory of algorithms because there exists a large class of problems for which it seems that an exponential algorithm is the best possible choice. 

We need to try to prevent exponential time.