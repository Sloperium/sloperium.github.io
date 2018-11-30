Title: Calculating the last digits of large fibonacci numbers
Date: 2018-09-08
Category: Python
Tags: python, math
Author: Christian Sloper


Fibonacci numbers occur many places in science, in nature and especially in programming puzzles.  They are easy to understand, and easy to implement, poorly. In this blog post I will show the naive way, the "standard" way, and in the end a sub-linear algorithm for calculating the nth Fibonacci number that allows for calculating huge Fibonacci numbers very quickly.

# Definition of Fibonacci
Each Fibonacci number is defined as the sum of the two previous Fibonacci numbers.  Formally $ F_n = F_{n-1} + F_{n-2}$ where $ F_0 = 0$ and $ F_1 = 1$.

The sequence is then : $ 0,1,1,2,3,5,8,13,21, ...$

Each number very roughly doubles in size from the previous one and to avoid storing numbers with tens of thousands of digits, we will look at the problem of calculating the last 16 digits of each Fibonacci number.

# The naive way:
Since the definition of Fibonacci numbers is recursive it is a natural first step to use a recursive algorithm to create a first attempt at calculating the sequence:

```python
def fib1(n,mod):
    if n <= 1:
        return n
    else:
        return (fib1(n-1) + fib1(n-2))%mod
```

While this function produces the correct results, we will quickly discover it is extremely slow. The reason for this is that there is a lot of duplicated function calls, both sub trees call the fib1 function with the same values. It runs in time $ \mathcal{O}(2^n)$, so even calculating Fibonacci for $ n = 50$ takes a long time.

# Memoization, the easy fix
When you notice that you keep recalculating the same values, it makes sense to store them and make a lookup instead of calculating them again.  This is known as memoization and in python is extremely simple with the decorator lru_cache.

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib2(n,mod):
    if n <= 1:
        return n
    else:
        return (fib2(n-1,mod) + fib2(n-2,mod))%mod
```

This allows for calculating much larger fibonacci numbers like 
$ fib2(1000,10^{16}) = 7795166849228875$
(the full number is )
fib(1000) = 43466557686937456435688527675040625802564660517371780402481729089536555417949051890403879840079255169295922593080322634775209689623239873322471161642996440906533187938298969649928516003704476137795166849228875

but we (in python) will run into maximum recursion depth.  Even without recursion depth it runs in $ \mathcal{O}(n)$ time, but also in $ \mathcal{O}(n)$ space.

# Linear time, linear space and no recursion
We can improve to constant space by storing the only two previous values.  

```python
def fib3(n, mod):
    if n == 0:
        return 0
    f1 = 0
    f2 = 1

    for i in range(n-1):
        f1,f2 = f2, (f1+f2)%mod
    return f2
```

This allows for computing yet larger fibonacci, like$ fib3(10^7, 10^{16}) = 8673686380546875$. But it is still linear in n and computing higher than $ n= 10^7$ is slow.

# Sublinear
We can express the Fibonacci recursion in quite a cool way using matrices:
$$  \begin{pmatrix} F_{k+2} \\  F_{k+1} \end{pmatrix}   = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} \begin{pmatrix} F_{k+1} \\  F_{k} \end{pmatrix}$$

Multiplying the result again with $$ \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}$$ yields the next step in the recursion, so we can easily see that we can express the n'th fibonacci number as:

$$ \begin{pmatrix} F_{n+1} \\  F_{n} \end{pmatrix}  =  \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} ^n \begin{pmatrix} 1 \\  0 \end{pmatrix}$$

Some trickery and handwaving about matrix manipulation later, we can reach the following two very useful identities:

$$ F_{2n-1} = F_n^2 + F_{n-1}^2$$
$$ F_{2n} = (2*F_{n-1} + F_{n})F_n$$

This is very useful, because now we can  express a Fibonacci number using only Fibonacci numbers half way through the sequence (and not the previous two as earlier).

This lets us set up the following recursive function:
```python
@lru_cache(maxsize=None)
def fib4(n,mod):
    if n <= 1:
        return n

    if n%2:
        m = (n+1)//2
        return (fib4(m, mod) ** 2 + fib4(m - 1, mod) ** 2) % mod
    else:
        m = n//2
        return ((2*fib4(m - 1, mod) + fib4(m, mod)) * fib4(m, mod)) % mod
```

 This is now sublinear $ \mathcal{O}(\log n)$ and calculates the last 16 digits of a trillion, trillion ($10^{24}$) in about 200ns.








