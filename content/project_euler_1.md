Title: Project Euler #1
Date: 2018-09-07
Category: Python
Tags: python, Project Euler
Author: Christian Sloper
Summary: Solutions to project Euler #1


The math/programming puzzles page "<a href="https://projecteuler.net/about">Project Euler</a>" have for many years been a hobby of mine. I want to share some solutions for simpler and harder problems and in the process highlight some techniques applicable in algorithm design.  Hopefully these posts will also teach me how to a) blog and b) use wordpress.

To get things started we will look at "<a href="https://projecteuler.net/problem=1">Project Euler , problem 1</a>".
<blockquote>If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.

Find the sum of all the multiples of 3 or 5 below 1000.</blockquote>
## Brute force programming

With any sort of programming background this task is very easy, in python we could write:

```python
s = 0
for i in range(1000):
   if i%3 == 0 or i%5 == 0:
   s +=i
print(s)
```

For a python user there is a more "pythonic" way using list comprehension:

```python
print( sum(i for i in range(1000) if i%3== 0 or i%5== 0 ))
```

both yielding the result 233168.

## The math
The above solution works very well for puzzle 1, as it is only 2000 or so "if" tests, but what if we dont want the sum of numbers below 1000 but numbers below 1 trillion? ( 1 000 000 000 000 ) 2 trillion if tests would take an enormous amount of time to calculate, but using some analysis we can avoid the for loop all together.

We will view the numbers divisible by 3 as one set, and the numbers divisible by 5 as another set. The sum of the first set $3, 6, 9, 12, 15, ..., \left \lfloor{n/3} \right \rfloor$  can be expressed like this $\sum_{i=1}^{\left \lfloor{n/3} \right \rfloor } 3*i$

The second set $5, 10, 15, 20, 25, ..., \left \lfloor{n/5} \right \rfloor$ can be expressed similarly $\sum_{i=1}^{\left \lfloor{n/5} \right \rfloor } 5*i$

We can add the sum of two sets together if there is no overlap, but we notice that every 15th number is divisble by both 3 and 5. To account for this we must subtract the sum: $\sum_{i=1}^{\left \lfloor{n/15} \right \rfloor } 15*i$.  This is known as <a href="https://en.wikipedia.org/wiki/Inclusion%E2%80%93exclusion_principle">the inclusion/exclusion principle</a>.

Using the identity $\sum_{i=1}^{n} i = \frac{n*(n+1)}{2}$ we can write a function to calculate each term:

```python
def SumDivisibleByN(n,m):
    return n * (m // n) * (m // n + 1) // 2

print(SumdivisbleByN(3, 999) + SumdivisbleByN(5, 999) - SumdivisbleByN(15, 999))
```
which yields the same result, and also allows for much higher input numbers, one trillion f.ex. :

```python
print(SumdivisbleByN(3, 10**12-1) + SumdivisbleByN(5, 10**12-1) - SumdivisbleByN(15, 10**12-1))
```

yields 233333333333166666666668 in an instant.