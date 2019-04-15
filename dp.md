### What is dynamic programming?

Dynamic programming is a programming principle where a very complex problem can be solved by dividing it into smaller subproblems. This principle is very similar to recursion, but with a key difference, every distinct subproblem has to be solved only ONCE. To understand what this means, we first have to understand the problem of solving recurrence relations. Every single complex problem can be divided into very similar subproblems, this means we can construct a _recurrence relation_ between them. Let's take a look at an example we all are familiar with, the Fibonacci sequence! Well, that sequence is defined with the following `recurrence relation`:

$$
fibonacci(n)=fibonacci(n-1)+fibonacci(n-2)
$$

So, if we want to find the `n`-th number in the Fibonacci sequence, we have to know the two numbers preceding the `n`-th in the sequence. However, every single time we want to calculate a different element of the Fibonacci sequence, we have have certain `duplicate` calls in our recursive calls, as can be seen in following image, where we calculate Fibonacci(5):

![Drag Racing](E:\Programming\GitHub\dynamic_programming\graphics\fibonacci1.png)

As you can plainly see, this unmodified recursive solution repeats the same calculations quite a bit,which severely slows down this algorithm. For example, if we want to calculate F(4), we need to calculate F(3) and F(2), however we also need F(2) for F(3). The solution to this problem is `dynamic programming`, where we model a solution as if we were to solve it recursively, but we solve it from the ground up, and **memorize** subproblem solutions. Ergo, for the Fibonacci sequence, we **first** solve and memorize F(1), then F(2), then F(3) using F(2) and F(1) and so on. This means that the calculation of every individual element of the sequence is O(1), because we already know the former two.



When solving a problem using dynamic programming, we have to follow these steps:

* Determine the recurrence relation that applies to said problem
* Initialize the memory/array/matrix's starting values 
* Make sure that when we make a "recursive call" (access the memorized solution of a subproblem) it's always solved



### The Simplified Knapsack Problem

The Simplified Knapsack problem is a problem of optimization, which there is no ``one`` solution, but the question is, does one even exist? The problem is as follows:

``Given a set of items, each with a weight w1,w2... determine the number of each item to put in a knapsack so that the total weight is less than or equal to a given limit K.``

So let's take a step back, how will we represent the solutions to this problem? First, let's store the weights of all the items in an array W. Next, lets say that there are n items and lets enumerate them with numbers from 1 to n, so the weight of the i-th item is `W[i]`. We'll form a matrix M of (n+1)x(K+1) dimensions. M\[x\]\[y\] corresponding to the solution of the knapsack problem, but only with the first `x` items of the beginning array, and with a maximum capacity of `y`.

There are 2 things to note when filling up the matrix, first, does a solution exist for the given subproblem( M\[x\]\[y\].exists ), AND does the given solution include the newest item added to the array( M\[x\]\[y\].includes ). Therefore, initialization of the matrix is quite easy, M\[0\]\[k\].exists where k>0 is always `false`, because we can't fill a knapsack with no items, however M\[0\]\[0\].exists = true, because the knapsack **should** be empty to begin with. In addition, we can say that M\[k\]\[0\].exists = true and M\[k\]\[0\].includes = false for every `k`.

Next, let's construct the recurrence relation for ``M[i][k]`` with the following code:

```python

if(M[i-1][k].exists == True):
    M[i][k].exists = True
    M[i][k].includes = False
elif(k-W[i]>=0):
    if(M[i-1][k-W[i]].exists == true):
    	M[i][k].exists = True
    	M[i][k].includes = True
else:
    M[i][k].exists = False
```



So the gist of the solution is dividing the subproblem into two cases:

1. When a solution exists for the first `i-1` elements, for capacity `k`
2. When a solution exists for the first `i-1` elements, but for capacity `k-W[i]`

The first case is self explanatory, we already have a solution to the problem, why fix what isn't broken. The second case refers to knowing the solution for the first `i-1` elements, but the capacity is with exactly one `i`-th element short of being full, which means we can just add one i-th element, and we have a new solution!

In this implementation, to make things easier, we'll make the class `Elem` for storing elements:

```java
public class Elem
{
    private boolean exists;
    private boolean includes;

    public Elem (boolean exists, boolean includes) {
        this.exists=exists;
        this.includes=includes;
    }

    public Elem (boolean exists) {
        this.exists=exists;
        this.includes=false;
    }

    public boolean isExists () {
        return exists;
    }

    public void setExists (boolean exists) {
        this.exists=exists;
    }

    public boolean isIncludes () {
        return includes;
    }

    public void setIncludes (boolean includes) {
        this.includes=includes;
    }
}

```

Now we can dive into the main class:

```java
import java.util.Scanner;

public class knapsack
{
    public static void main (String[] args) {

        Scanner sc=new Scanner (System.in);

        System.out.println ("Insert knapsack capacity:");
        int K=sc.nextInt ();

        System.out.println ("Insert number of items:");
        int n=sc.nextInt ();

        System.out.println ("Insert weights: ");
        int[] W=new int[n + 1];

        for (int i=1 ; i <= n ; i++) {
            W[i]=sc.nextInt ();
        }

        Elem[][] M=new Elem[n + 1][K + 1];

        M[0][0]=new Elem (true);

        for (int i=1 ; i <= K ; i++) {
            M[0][i]=new Elem (false);
        }


        for (int i=1 ; i <= n ; i++) {
            for (int j=0 ; j <= K ; j++) {
                M[i][j]=new Elem (false);
                if (M[i - 1][j].isExists ()) {
                    M[i][j].setExists (true);
                    M[i][j].setIncludes (false);
                } else if (j >= W[i]) {
                    if (M[i - 1][j - W[i]].isExists ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                    }
                }
            }
        }

        System.out.println (M[n][K].isExists ());


```

The only thing that's left is reconstruction of the solution, in the class above, we know that a solution *EXISTS*, however we don't know what it is. For reconstruction we use the following code:

```java
import java.util.ArrayList;
import java.util.List;

List <Integer> solution=new ArrayList <> (n);

if (M[n][K].isExists ()) {
    int i=n;
    int j=K;
    while (j > 0 && i > 0) {
        if (M[i][j].isIncludes ()) {
            solution.add (i);
            j=j - W[i];
        }
        i=i - 1;

    }
}
System.out.println ("The elements with the following indexes are in the solution:\n"+(solution.toString ()));
```



```

```



