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

Function called for itself is recursion. When the recursion is at the end of the function, it is definited as tail recursion.

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

Therefore, the optimization of tail recursion plays a significant role in the operation of recursion. In some functional programming language, it becomes the standard. So is ES6. At first time, the tail recursion is regulated. The realization of ECMAscript should deploy the optimization of tail recursion. Put another way, once we use tail recursion in ES6, never did we encounter the problem of stack overflow. We save the usage of the memory.

The tail recursion will rewrite the recursion function to ensure that we can call for itself. To achieve this goal, we need to change the internal variable into the parameter of function. As mentioned above, factorial function factorial needs a variable total, so we need to change it into a parameter of function. The disadvantages is that it dosen't directly call when we use it. Then we may ask, "Why when we count the factorial of 5, we need to set two parameter as 5 and 1 respectively?

Offering a normal function which is out side of tail recursion will solve this problem.

    function tailFactorial(n, total) {
      if (n === 1) return total;
      return tailFactorial(n - 1, n * total);
    }

    function factorial(n) {
      return tailFactorial(n, 1);
    }

    factorial(5) // 120

After calling for tailFactorial function, it looks better than normal factorial function.

There is a concept called currying in functional programming, which is convert the function of multi parameters into single parameter. Therefore, we can use currying here.


    function currying(fn, n) {
      return function (m) {
        return fn.call(this, m, n);
      };
    }

    function tailFactorial(n, total) {
      if (n === 1) return total;
      return tailFactorial(n - 1, n * total);
    }

    const factorial = currying(tailFactorial, 1);

    factorial(5) // 120

After currying, tail recursion tailFactorial become a function of single parameter factorial.

To summarize, recursion is a kind operation of loop. There is no loop gramma in functional programming language and all the loops are taken place by recursion. That's why tail recursion plays a significant role in these languages. In the language which support the tail recursion, such as Lua, ES6, we can use recursion. Instead of loop, we had better use recursion and the tail recursion is recommanded.