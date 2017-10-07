Tail Recursion

The problem is that we need to use tail recurrsion to solve the summation from 1 to 100 which the step is 1.
[用递归的方法完成下面的操作](https://github.com/zhanwen/AlgorithmDiagram/blob/master/chapter3/%E9%80%92%E5%BD%92%E7%BB%83%E4%B9%A0%E9%A2%98.md)

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