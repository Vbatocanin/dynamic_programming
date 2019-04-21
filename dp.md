To understand the concepts of dynamic programming we need to get acquainted with the following subjects:

- [What is Dynamic Programming?](#what-is-dynamic-programming)
- [Rod Cutting Algorithm](#rod-cutting-algorithm)
- [The Simplified Knapsack Problem](#the-simplified-knapsack-problem)
- [The Traditional Knapsack Problem](#the-traditional-knapsack-problem)
- [Short Description of Levenshtein Distance](#short-description-of-levenshtein-distance)
- [LCS - Longest Common Subsequence](#lcs-longest-common-subsequence)
- [Other Problems That Utilize Dynamic Programming](#other-problems-that-utilize-dynamic-programming)
- [Conclusion](#conclusion)

### What is Dynamic Programming?

Dynamic programming is a programming principle where a very complex problem can be solved by dividing it into smaller subproblems. This principle is very similar to recursion, but with a key difference, every distinct subproblem has to be solved only **once**. To understand what this means, we first have to understand the problem of solving recurrence relations. Every single complex problem can be divided into very similar subproblems, this means we can construct a _recurrence relation_ between them. Let's take a look at an example we all are familiar with, the Fibonacci sequence! The Fibonacci sequence is defined with the following `recurrence relation`:
$$
fibonacci(n)=fibonacci(n-1)+fibonacci(n-2)
$$
So, if we want to find the `n`-th number in the Fibonacci sequence, we have to know the two numbers preceding the `n`-th in the sequence. However, every single time we want to calculate a different element of the Fibonacci sequence, we have have certain `duplicate` calls in our recursive calls, as can be seen in following image, where we calculate Fibonacci(5):

![Drag Racing](E:\Programming\GitHub\dynamic_programming\graphics\fibonacci1.png)

This unmodified recursive solution repeats the same calculations quite a bit,which severely slows down this algorithm. For example, if we want to calculate F(4), we need to calculate F(3) and F(2), however we also need F(2) for F(3). The solution to this problem is `dynamic programming`, where we model a solution as if we were to solve it recursively, but we solve it from the ground up, and **memorize** subproblem solutions. Ergo, for the Fibonacci sequence, we **first** solve and memorize F(1), then F(2), then F(3) using F(2) and F(1) and so on. This means that the calculation of every individual element of the sequence is O(1), because we already know the former two.



When solving a problem using dynamic programming, we have to follow these steps:

- Determine the recurrence relation that applies to said problem
- Initialize the memory/array/matrix's starting values 
- Make sure that when we make a "recursive call" (access the memorized solution of a subproblem) it's always solved

Following these rules, let's take a look at some examples of algorithms that use dynamic programming:



### Rod Cutting Algorithm

Let's start with something simple, the problem goes as follows:

> Given a rod of length n and an array of prices that contains prices of all pieces of size smaller than n. Determine the maximum value obtainable by cutting up the rod and selling the pieces. 

#### Naive Solution

This problem is like tailor-made for dynamic programming, but because this is our first real example, let's see how many fires we can start by letting this poor little code run:

```java
public class naive_solution
{
    static int getValue (int[] values, int length) {
        if (length <= 0)
            return 0;
        int tmp_max=-1;
        for (int i=0 ; i < length ; i++) {
            tmp_max=Math.max (tmp_max, values[i] + getValue (values, length - i - 1));
        }
        return tmp_max;
    }

    public static void main (String[] args) {
        int[] values=new int[]{3, 7, 1, 3, 9};
        int rod_length=values.length;

        System.out.println ("Max rod value: " + getValue (values, rod_length));

    }
}
```

**Output**:

```java
Max rod value: 17
```



This solution is highly inefficient, recursive calls aren't memorized so the poor guy has to solve the same subproblem every time there's a single overlapping solution.

#### DP Solution

Utilizing the same basic principle from above, but adding **memoization** and excluding recursive calls, we get the following implementation:

```java
public class dp_solution
{
    static int getValue (int[] values, int rod_length) {
        int[] sub_solutions=new int[rod_length + 1];

        for (int i=1 ; i <= rod_length ; i++) {
            int tmp_max=-1;
            for (int j=0 ; j < i ; j++)
                tmp_max=Math.max (tmp_max, values[j] + sub_solutions[i - j - 1]);
            sub_solutions[i]=tmp_max;
        }
        return sub_solutions[rod_length];
    }

    public static void main (String[] args) {
        int[] values=new int[]{3, 7, 1, 3, 9};
        int rod_length=values.length;

        System.out.println ("Max rod value: " + getValue (values, rod_length));

    }
}
```

**Output**:

```java
Max rod value: 17
```

As we can see, the resulting outputs are the same, only with different time/space complexity.

We eliminate the need for recursive calls by solving the subproblems from the ground-up, utilizing the fact that all previous subproblems to a given problem are already solved.

### The Simplified Knapsack Problem

The Simplified Knapsack problem is a problem of optimization, for which there is no ``one`` solution. The question for this problem would be, does a solution even exist? The problem is as follows:

> Given a set of items, each with a weight w1,w2... determine the number of each item to put in a knapsack so that the total weight is less than or equal to a given limit K.

So let's take a step back, how will we represent the solutions to this problem? First, lets store the weights of all the items in an array W. Next, lets say that there are n items and lets enumerate them with numbers from 1 to n, so the weight of the i-th item is `W[i]`. We'll form a matrix `M` of` (n+1)`x`(K+1)` dimensions. `M[x][y]` corresponding to the solution of the knapsack problem, but only with the first `x` items of the beginning array, and with a maximum capacity of `y`.

#### Example

Let's say we have 3 items, with the weights being `w1=2kg`, `w2=3kg` and `w3=4kg`, how do we optimally fill a knapsack with a capacity of `K=6kg`. Utilizing the method above, we can say for example that `M[1][2]`, this means that we are trying to fill a knapsack with a capacity of `2kg` with just the first item from the weight array (`w1`). While in `M[3][5] ` we are trying to fill up a knapsack with a capacity of `5 kg` using the first `3` items of the weight array (`w1,w2,w3`).

#### Matrix Initialization

There are 2 things to note when filling up the matrix, first, does a solution exist for the given subproblem( M\[x\]\[y\].exists ), AND does the given solution include the newest item added to the array( M\[x\]\[y\].includes ). Therefore, initialization of the matrix is quite easy, M\[0\]\[k\].exists, where k>0, is always `false`, because we can't fill a knapsack with no items, however M\[0\]\[0\].exists = true, because the knapsack **should** be empty to begin with. In addition, we can say that M\[k\]\[0\].exists = true and M\[k\]\[0\].includes = false for every `k`.

#### Algorithm Principle

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

#### Implementation

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
	}
}

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



**Output**:

```java
Insert knapsack capacity:
12
Insert number of items:
5
Insert weights: 
9 7 4 10 3
true
The elements with the following indexes are in the solution:
[5, 1]

```



A simple variation of the knapsack problem is filling a knapsack without value optimization, but now with unlimited amounts of every individual item. This variation can be solved by making a simple adjustment to our existing code:

```java
//old code for simplified knapsack problem
else if (j >= W[i]) {
                    if (M[i - 1][j - W[i]].isExists ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                    }
}
//new code, note that we're searching for a solution in the same row (i-th row), which means we're looking for a solution that already has some number of i-th elements (including 0) in it's solution
else if (j >= W[i]) {
                    if (M[i][j - W[i]].isExists ()) {
                        M[i][j].setExists (true);
                        M[i][j].setIncludes (true);
                    }
}


```



### The Traditional Knapsack Problem



Utilizing both previous variations, let's now take a look at the traditional knapsack problem and see how it differs from the simplified variation:

> Given a set of items, each with a weight w1,w2...  and a value v1,v2... determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit `K` and the total value is as large as possible.

In the simplified version, every single solution was equally as good. However, now we have a criteria for finding an **optimal** solution (aka the largest value possible). Keep in mind, this time we have an **infinite number of each item**, so items can occur multiple times in a solution.

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
                M[i][j]=new Elem_with_value (true, M[i - 1][j].getValue ());

                M[i][j].setIncludes (false);
                M[i][j].setValue (M[i - 1][j].getValue ());

                if (j >= W[i]) {
                    if ((M[i][j - W[i]].getValue () > 0 || j == W[i])) {
                        if (M[i][j - W[i]].getValue () + V[i] > M[i][j].getValue ()) {
                            M[i][j].setIncludes (true);
                            M[i][j].setValue (M[i][j - W[i]].getValue () + V[i]);
                        }
                    }
                }
                System.out.print (M[i][j].getValue () + "  ");

            }
            System.out.println ();
}

System.out.println ("Value: " + M[n][K].getValue ());

```



**Output**:

```java
Insert knapsack capacity:
12
Insert number of items:
5
Insert weights: 
9 7 4 10 3
Insert values: 
1 2 3 4 5
0  0  0  0  0  0  0  0  0  1  0  0  0  
0  0  0  0  0  0  0  2  0  1  0  0  0  
0  0  0  0  3  0  0  2  6  1  0  5  9  
0  0  0  0  3  0  0  2  6  1  4  5  9  
0  0  0  5  3  0  10  8  6  15  13  11  20  
Value: 20

```





### Short Description of Levenshtein Distance

Another very good example of using dynamic programming is Edit distance or Levenshtein distance. Levenshtein distance for given 2 strings `A` and `B` is the number of atomic operations we need to use to transform `A` into `B`. Atomic operations are:

1. character deletion
2. character insertion
3. character substitution (technically it's more than one operation, but for the sake of simplicity let's call it an atomic operation)

This problem is handled by methodically solving the problem for substrings of the beginning strings, gradually increasing the size of the substrings until they're equal to the beginning strings.

The recurrence relation we use for this problem is as follows:
$$
lev_{a,b}(i,j)=min\begin{cases}
lev_{a,b}(i-1,j)+1\\lev_{a,b}(i,j-1)+1\\lev_{a,b}(i-1,j-1)+c(a_i,b_j)\end{cases}
$$
`c(a,b)` being 0 if a==b, and 1 if a!=b. 

For a more detailed article about Levenshtein distance, click [here](<https://stackabuse.com/levenshtein-distance-and-text-similarity-in-python/>).



### LCS - Longest Common Subsequence

The problem goes as follows:

> Given two sequences, find the length of the longest subsequence present in both of them. A subsequence is a sequence that appears in the same relative order, but not necessarily contiguous.

#### Clarification

If we have two strings, `s1="MICE"` and `s2="MINCE"`, the longest common **substring** would be "MI" or "CE", however, the longest common **subsequence** would be "MICE" because the elements of the resulting subsequence don't have to be in consecutive order.

#### Recurrence Relation and General Logic

$$
lcs_{a,b}(i,j)=min\begin{cases}
lcs_{a,b}(i-1,j)\\lcs_{a,b}(i,j-1)\\lcs_{a,b}(i-1,j-1)+c(a_i,b_j)\end{cases}
$$

As we can plainly see, there is only a slight difference between Edit distance and LCS, specifically, in the cost of `left` and `up` moves. This means that in LCS, we have no cost for character insertion and character deletion, which means that we only count `diagonal` moves (character substitution), which have a cost of 1 if the two current string characters `a[i]` and `b[j]` are the same. The final cost of LCS is the length of the longest subsequence for the 2 string, which is exactly what we needed.



> Using this logic, we can boil down a lot of string comparison algorithms to simple recurrence relations which utilize the base formula of the Levenshtein distance. 



#### Implementation

```java
public class LCS
{
    public static void main (String[] args) {
        String s1=new String ("Hillfinger");
        String s2=new String ("Hilfiger");
        int n=s1.length ();
        int m=s2.length ();
        int[][] solution_matrix=new int[n + 1][m + 1];
        for (int i=0 ; i < n ; i++) {
            solution_matrix[i][0]=0;
        }
        for (int i=0 ; i < m ; i++) {
            solution_matrix[0][i]=0;
        }

        for (int i=1 ; i <= n ; i++) {
            for (int j=1 ; j <= m ; j++) {
                int max1, max2, max3;
                max1=solution_matrix[i - 1][j];
                max2=solution_matrix[i][j - 1];
                if (s1.charAt (i - 1) == s2.charAt (j - 1)) {
                    max3=solution_matrix[i - 1][j - 1] + 1;
                } else {
                    max3=solution_matrix[i - 1][j - 1];
                }
                int tmp=Math.max (max1, max2);
                solution_matrix[i][j]=Math.max (tmp, max3);

            }
        }
        System.out.println ("Length of longest subsequence: " + solution_matrix[n][m]);

    }
}



```



**Output**:

```java
Length of longest continuous subsequence: 8

```



### Other Problems That Utilize Dynamic Programming

There are a lot more problems that can be solved with dynamic programming, these are just a few of them:

- Partition Problem

  > Given a set of integers, find out if it can be divided into two subsets with equal sums

- Subset Sum Problem

  > Given a set of positive integers, and a value sum, determine if there is a subset of the given set with sum equal to given sum.

- Coin Change Problem (Total number of ways to get the denomination of coins)

  > Given an unlimited supply of coins of given denominations, find the total number of distinct ways to get a desired change.

- Total possible solutions to linear equation of k variables 

  > Given a linear equation of k variables, count total number of possible solutions of it.

- Find Probability that a Drunkard doesn't fall off a cliff (`KIDS, do not try this at home`)

  > Given a linear space representing the distance from a cliff, and providing you know the starting distance of the drunkard from the cliff, and his tendency to go towards the cliff `p` and away from the cliff `1-p`, calculate the probability of his survival.

- Many more...



### Conclusion

Dynamic programming is a tool that can save us a lot of `computational time` in exchange for a bigger `space complexity`, granted some of them only go halfway (a matrix is needed for memoization, but an ever-changing array is used). This highly on the type of system you're working on, if CPU time is precious, you opt for a `memory savy` solution, on the other hand, if your memory is limited, you opt for a more `time savy` solution for a better `time/space complexity` ratio.  