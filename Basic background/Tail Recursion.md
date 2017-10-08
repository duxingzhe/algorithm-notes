Tail Recursion

The problem is that we need to use tail recurrsion to solve the summation from 1 to 100 which the step is 1.

Question:[用递归的方法完成下面的操作](https://github.com/zhanwen/AlgorithmDiagram/blob/master/chapter3/%E9%80%92%E5%BD%92%E7%BB%83%E4%B9%A0%E9%A2%98.md)

There is the code:

```
public class Series {

    public static void main(String args[]){
        int sum=tailAdd(100,0);
        System.out.println(sum);
    }

    private static int tailAdd(int number, int sum){
        if(number==0)return sum;
        else return tailAdd(number-1,sum+number);
    }
}
```

About the recursion and tail recursion:

Definition:

Function for itself is recursion. When the recursion is at the end of the function, it is called tail recursion.

As we know, recursion occupied so much memory, which leads the error of stack overflow. Tail recursion, however, has only one recursion call. The error of stack overflow will never happen.


    function factorial(n) {
      if (n === 1) return 1;
      return n * factorial(n - 1);
    }

    factorial(5) // 120

There is a factor function above. It caculate the result of factor of n, which the complexity is O(n) for storing n record of call.

If we change to tail recursion, we only store one record of call in memory, whose the complexity will reduce into O(1).

    function factorial(n, total) {
      if (n === 1) return total;
      return factorial(n - 1, n * total);
    }

    factorial(5, 1) // 120

Therefore, the optimization of tail recursion plays a significant role in the operation of recursion. In some functional programming language, it becomes as the standard. So is ES6. At first time, it regulate the realization of ECMAscript should deploy the optimzation of tail recursion. Put another way, once we use tail recursion in ES6, never did we encounter the problem of stack overflow to save the usage of the memory.