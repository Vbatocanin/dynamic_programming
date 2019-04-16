### What is dynamic programming?

Dynamic programming is a programming principle where a very complex problem can be solved by dividing it into smaller subproblems. This principle is very similar to recursion, but with a key difference, every distinct subproblem has to be solved only ONCE. To understand what this means, we first have to understand the problem of solving recurrence relations. Every single complex problem can be divided into very similar subproblems, this means we can construct a _recurrence relation_ between them. Let's take a look at an example we all are familiar with, the Fibonacci sequence! The Fibonacci sequence is defined with the following `recurrence relation`:

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

Following these rules, let's take a look at some examples of algorithms that use dynamic programming:

### The Simplified Knapsack Problem

The Simplified Knapsack problem is a problem of optimization, which there is no ``one`` solution, but the question is, does one even exist? The problem is as follows:

> Given a set of items, each with a weight w1,w2... determine the number of each item to put in a knapsack so that the total weight is less than or equal to a given limit K.

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



### The Traditional Knapsack Problem and Variations



Let's now take a look at the traditional knapsack problem and see how it differs from the simplified variation:

> Given a set of items, each with a weight w1,w2...  and a value v1,v2... determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit `K` and the total value is as large as possible.

In the simplified version, every single solution was equally as good. However, now we have a criteria for finding an **optimal** solution (aka the largest value possible). 

In the implementation we'll be deriving a class from `Elem`, with an added private field `value` for storing the   largest possible value for a given subproblem:

```java
public class Elem_with_value extends Elem
{
    private int value;
	
	//appropriate constructors, getters and setters
    
}
```

The implementation is very similar, with the only difference being that now we have to chose the optimal solution judging by the resulting value:

```java
for (int i=1 ; i <= n ; i++) {
            for (int j=0 ; j <= K ; j++) {
                int tmp_max=0;
                M[i][j]=new Elem_with_value (false, 0);
                if (M[i - 1][j].isExists ()) {
                    M[i][j].setExists (true);
                    M[i][j].setIncludes (false);
                    tmp_max=M[i - 1][j].getValue ();
                }
                if (j >= W[i]) {
                    if (M[i - 1][j - W[i]].isExists () && tmp_max < M[i - 1][j - W[i]].getValue ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                        M[i][j].setValue (M[i - 1][j - W[i]].getValue () + V[i]);
                    }
                }

            }
}
```

Another variation of the knapsack problem is filling a knapsack with/without value optimization, but now with unlimited amounts of every individual item. This variation can be solved by making a simple adjustment to our existing code:

```java
//old code for simplified knapsack problem
else if (j >= W[i]) {
                    if (M[i - 1][j - W[i]].isExists ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                    }
}
//new code, note that we're searching for a solution in the same row (i-th row), which means we're looking for a solution that already has some number of i-th elements in it's solution
else if (j >= W[i]) {
                    if (M[i][j - W[i]].isExists ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                    }
}
```



### Short lookback concerning Levenshtein distance

Another very good example of using dynamic programming is edit distance or Levenshtein distance. Levenshtein distance for given 2 strings `A` and `B` is the number of atomic operations we need to use to transform `A` into `B`. Atomic operations are:

1. character deletion
2. character insertion
3. character substitution (technically it's more than one operation, but for the sake of simplicity let's call it an atomic operation)

This problem is handled by methodically solving the problem for substrings of the beginning strings, gradually increasing the size of the substrings until they're equal to the beginning strings.

The recurrence relation we use for this problem is as follows:
$$
lev_{a,b}(i,j)=min\begin{cases}
lev_{a,b}(i-1,j)+1\\lev_{a,b}(i,j-1)+1\\lev_{a,b}(i-1,j-1)+c(a_i,b_j)\end{cases}
$$
`c(a,b)` being 1 if a==b, and 0 if a!=b. 

For a more detailed article about Levenshtein distance, click [here](<https://stackabuse.com/levenshtein-distance-and-text-similarity-in-python/>).



### LCS algorithm