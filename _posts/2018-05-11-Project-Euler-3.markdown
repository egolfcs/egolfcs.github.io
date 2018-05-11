---
layout: default
title:  Project Euler - Problem 3
date:   2018-05-11 16:00:00 -0400
categories: ProjectEuler
permalink: "/pe/3"
---

# Project Euler Problem 3

>
The prime factors of 13195 are 5, 7, 13 and 29.<br/><br/>
What is the largest prime factor of the number 600851475143 ?

Factoring numbers is one of those problems that gets a lot of attention because of how important it is. I won't pretend to have any new insights into the problem. Plenty of people much more brilliant than myself have developed very impressive approaches to factoring numbers efficiently. With that being said, I will implement a time inefficient approach, a space inefficient approach, and a reasonable approach. This reasonable approach still won't be state of the art.

## Naive Approaches

One approach to factoring n is to check for divisibility by the numbers less than n. Start from 2 and increment by 1. If the current number divides n, it is a factor and it might be a prime factor. If it isn't a prime factor, it will be a multiple of a smaller prime factor. That is, if p is prime and p\*k divides n, p is smaller than p\*k and p must also divide n. Because we're checking in ascending order, p is guaranteed to be checked first. So, each factor can be checked against a running list of prime factors. If none of the numbers in this list divide the factor, it too is a prime and we add it to the list. This approach looks something like this:

{% highlight python %}
def max_prime_factor1(n):
    p_facts = []
    for i in xrange(2,n+1):
        if n % i == 0:
            p_facts += [i]
            for j in p_facts[:-1]:
                if i % j == 0:
                    p_facts = p_facts[:-1]
                    break
    return p_facts[-1]
{% endhighlight %}

This approach worked for the test case of 13195. I attempted to run it on the actual input, and it took about a minute to check the first million numbers. It would take about 60,000 minutes, or about 40 days to find the answer using this method. This isn't exactly plausible.

Perhaps we can use math to make the problem smaller? No number larger than n/2 can divide n except n itself. Perhaps we can instead check the interval [2, n/2]...

{% highlight python %}
def max_prime_factor2(n):
    p_facts = []
    for i in xrange(2,int(n*0.5)+1):
        if n % i == 0:
            p_facts += [i]
            for j in p_facts[:-1]:
                if i % j == 0:
                    p_facts = p_facts[:-1]
                    break
    return p_facts[-1]
{% endhighlight %}

While this is twice as fast, it will still take 20 days to find the answer. Still not plausible, I mean what if the power goes out while our program is running?

There is one more mathematical trick along this line of thought. If p divides n, n/p = q also divides n. That is p\*q = n. Call p and q complementary factors. If q is larger than n\*\*0.5, p must be smaller than n\*\*0.5. With this in mind, we can check the numbers smaller than or equal to sqrt(n) and keep track of the q's. Once we have a list of prime factors smaller than sqrt(n), we can check the list of q's for divisibility by these prime factors and add them to the list if they aren't divisible by any of them. This looks like this:

{% highlight python %}
def max_prime_factor3(n):
    p_facts = []
    candidates = []
    for i in xrange(2,int(n**0.5)+1):
        print i
        if n % i == 0:
            p_facts += [i]
            candidates += [n/i]
            for j in p_facts[:-1]:
                if i % j == 0:
                    p_facts = p_facts[:-1]
                    break

    for i in candidates:
        p_facts += [i]
        for j in p_facts[:-1]:
            if i % j == 0:
                p_facts = p_facts[:-1]
                break

    return p_facts[-1]

{% endhighlight %}

We need only check about 800,000 numbers for this solution, so it spits out the correct answer in less than a minute. While this works in a tractable amount of time, it still isn't a very good solution and doesn't scale well at all.

## Prime Finding

If we had a list of primes less than n, we could check if they divide n. A nice method for generating primes is the Sieve of Eratosthenes. This consists of iterating over a list of numbers and checking if a given number is marked as composite. If it isn't marked as composite, we mark it's multiples as composite and then add it to a list of primes. A generator in python might look like this:

{% highlight python %}
def primes(bound):
    sieve = xrange(2,bound+1)
    marked = (bound+1)*[0]
    for i in sieve:
        if marked[i]:
            continue

        yield i

        for j in xrange(i**2,bound+1,i):
            marked[j] = 1

{% endhighlight %}

Notice the line `for j in xrange(i**2,bound+1,i).` Rather than starting at i and incrementing by i to mark multiples, we can start at i\*\*2 and increment by i. This is because all numbers less than i, call one such number k, are divisible by another prime smaller than i. If k is divisible by a smaller prime, so is k\*i and therefore it has already been marked as composite. This is a somewhat time efficient method for generating primes. Notice though that it requires populating a huge list of numbers. There are ways to use less space, but I won't implement them here.

## A Faster Approach

With our prime generator in hand, we might iterate over the primes less than or equal to n/2 and check for divisibility, returning the largest. If n itself is prime, none of these primes will divide it. In this case we will simply return n. Implementing this will look something like:

{% highlight python %}
def max_prime_factor4(n):
    bound = n/2
    m = 1
    for i in primes(n/2):
        if n % i == 0:
            m = i

    if m == 1:
        return n
    else:
        return m
{% endhighlight %}

Unfortunately, generating a list of 600851475143/2 booleans isn't space friendly and this program fails to run. At this point we could attempt to make a more space efficient version of our prime generator, but I have a better idea.

Again it's sometimes sufficient to check only the numbers less than sqrt(n). Perhaps then we can generate a list of primes less than sqrt(n) and keep track of their complementary factors with respect to n. It turns out keeping track of the complements isn't necessary.

If a prime p divides n and k is the number of times p divides n, one of two things is true:
- p is the largest prime factor of n.
- n/p**k is divisible by a prime larger than p.

In other words, if prime p divides n, we can reduce the problem to factoring n/p**k. Such an algorithm might look like this:

{% highlight python %}
def max_prime_factor5(n):
    for i in primes(int(n**0.5)):
        while n % i == 0:
            n /= i
        if n < i:
            return i
{% endhighlight %}

During each iteration, the current prime factor is being removed from the factorization of n. This does not change the factorization with respect to any other prime. If repeated division by i causes n to be smaller than i, n cannot be divisible by any primes larger than i. Therefore, i is the largest prime which divides n.

This solution is much better and produces a solution in mere milliseconds.
