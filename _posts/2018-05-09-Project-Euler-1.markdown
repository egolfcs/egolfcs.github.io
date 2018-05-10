---
layout: default
title:  Project Euler - Problem 1
date:   2018-05-09 16:00:00 -0400
categories: ProjectEuler
permalink: "/pe/1"
---

# Project Euler Problem 1

>
If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.<br/><br/>
Find the sum of all the multiples of 3 or 5 below 1000.


## Naive Approach

The naive approach is to iterate over the numbers from 1 to 1000, check if each number is divisible by 3 or 5, and add that number to a running total if it is. That would look something like this:

{% highlight python %}
def sol1(n):
    s = 0
    for i in range(n):
        if (i % 3 == 0 or i % 5 == 0):
            s += i

    return s
{% endhighlight %}

This works, but it isn't the most efficient solution. In order to come to a solution this way, we must iterate over each number less than 1000. This solution is therefore O(n). This is perhaps the best we can do without using math, but we absolutely can do better.

## Some Math
It is know that the sum of every number from 1 to n is equal to `n*(1+n)*0.5.` This is not hard to prove. Consider the list:

`1,2,3... 499, 501,... 997, 998, 999`

Take the sum of the pairs of the ends and move inwards such that:

`1 + 999 = 2 + 998 = 3 + 997 = ...499+501 = 1000`

There are 499 pairs of numbers in the list which each sum to 1 + n with n = 999. We can map (n+1)/2, in this case 500, to each number in the pair. That is, if there are 499 pairs that add to 1000, we can "average out" this 1000 by assigning 500 to 998 numbers. Finally, observe that 500 was left by itself. This is the 999th 500. This is not a perfectly rigorous proof for the general formula, but hopefully it gives an intuition if you've never seen this before or is a sufficient refresher if you have.

## A Faster Approach

Using this formula we can find `3 + 6 + 9... + ~n` and `5 + 10 + 15... + ~n` with ~n being the closest number to n which is a multiple of 3 or 5. By factoring out 3 and 5 we get

`3(1 + 2 + 3... + ~n/3)` and `5(1 + 2 + 3... + ~n/5)`

Conveniently, ~n/k is simply n/k by integer division, so we need not actually find ~n. Therefore, we can find n/k, plug it into our sum equation, and multiply the result by k to find the sum of integers less than n which are divisible by k.

Using this method, we might sum all of the numbers divisible by 3 and then sum all of the numbers divisible by 5. We can then take those two sums and add them together. Notice though that numbers like 15, 30, 45... are divisible by both 3 and 5. Simply adding the two sums together will double count the numbers divisible by 15. This can be remedied by finding a third sum, that being the sum of the numbers divisible by 15. We can then subtract this sum from the double counting sum in order to adjust.

Implementing these ideas might look something like this:

{% highlight python %}
def linsum(k,n):
    u = n / k
    return k*(u * (1 + u) * 0.5)

def sol2(n):
    return ((linsum(3,n) + linsum(5,n)) - linsum(15,n))
{% endhighlight %}

This solution will be O(1) with respect to n as each calculation of `linsum()` is constant. There is no need to populate or iterate over a list using this approach.
