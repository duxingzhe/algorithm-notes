Tail Recursion

The problem is that we need to use tail recurrsion to solve the summation from 1 to 100 which the step is 1.

There is the code:

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