---
layout: post
title: Haskell Meetup
date: '2017-05-01 03:58:08'
tags:
- programming
- haskell
---

### History
Recently I started the [Beaverton Haskell Meetup](https://www.meetup.com/Beaverton-Haskell-Meetup/) because I was interested in finally learning Haskell thoroughly and doing it along with other folks with the same interest.  I was first introduced to Haskell as a configuration language for the superb window manager system [XMonad](http://xmonad.org/).  Haskell seemed obscure at the time because even though it was a functional language the syntax was nothing like LISP which I am far more familiar with.  I had some familiarity with Standard ML from my college days but not much.  At the time Haskell didn't make any sense to me and I eventually decided I would like to learn it a little better.  I've learned many languages over the last 20 or so years but nothing like this one.  So many times I tried and gave up or lost interest.

### Why
This time around I am fully dedicated to becoming a proficient Haskell programmer.  Why?  For a couple of reasons.  I've become convinced that functional programming is the best way forward to design systems that are low maintenance, elegant, and secure.  I can write far fewer lines of Haskell to implement algorithms than would be possible in an imperative language.  The type system is very expressive as well and allows the programmer to very clearly define a series of types for a particular domain of interest.  Because Haskell is a pure functional programming language there are no side effects.  Of course all of the interesting things you want to do with a programming language require side effects, but Haskell allows for things like IO to take place without compromising the purity of its functions.

As an example how about we generate some primes?

```haskell
primes :: Int -> [Int]
primes 0 = []
primes n = take n $ filter isprime [2..]
  where
    isprime :: Int -> Bool
    isprime = \y -> filter (\x -> y `mod` x == 0) [1..y] == [1, y]
```
In this example we are applying two filters to a list from two to infinity (because one is not a prime number).  Infinity, really?  Yes, really.  Because Haskell uses lazy loading it doesn't compute the entire list but only the elements of the list that are necessary to compute.  The key here is the use of `take` which will take only the number of elements off of the list that we want the function to compute.  The filtered list is determined by taking each element and checking it with the `isprime` function.  This function will check the modulus for each value from one to the number we are checking and then compare it against the list of values we would expect to get if this number is a prime.  If the number is a prime then the only numbers that should have a modulus of zero are one and the number itself.  I couldn't imagine having such a clear solution in an imperative language.

### Practical Application
But my real interest in Haskell is for practical applications.  I'm currently working as a DevOps engineer and curious about how Haskell could be used to improve automation and tooling.  The so called "killer app" of Haskell is its ability to define grammars and create DSLs.  That is where I'm most interested in spending my time to start.  How can Haskell be used to define languages that will improve the state of DevOps?  There is already one such project that has gained my curiosity, a Linux distribution called [NixOS](https://nixos.org/) which is a distribution where all system settings are configurable through Haskell code, and another related project called [NixOps](https://nixos.org/nixops/) for defining cloud deployments.

Expect to hear more about Haskell as time goes on!

If you happen to be in the Portland metro area we could always use some more people at the meetup!