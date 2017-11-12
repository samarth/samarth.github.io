---
layout: post
title:  "Why haskell blew my mind"
date:   2017-11-11 16:48:25
categories: Tech
tags: functional programming, haskell, recursion
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---

I was recently introduced to haskell by a coworker. He told me that this will change the way how you think about programs and needless to say I didn't understand why he said that. I have been programming for over 4 years now and I have sort of accepted the norm that programs are supposed to be dumb instructions that are eventually dumbed down to machine level instructions and that gives you, the programmer a high.

So on a cold dark afternoon in Stockholm, I decided to see what the fuss was about, what functional programming is supposed to be and why haskell is so great. And yes, I have to agree with the rest of the world. It is BRILLIANT!!!


## Patterns
This concept blew my mind. Maybe people have been using this in other languages as well, but I never came across them before. I have seen examples of this in Scala but I just switched from node.js which didn't have them.

Instead of writing dumb if else blocks, you simply punch in patterns on how your input variables are supposed to look and haskell understands you.

For example, lets say I wanted to implement a python like list slicer which says a[:3] (give me the first 3 elements of the list a).

The function definition would be something like

```
slicer' :: (Num i, Ord i) => i -> [a] -> [a]
```

Don't worry about Num and Ord (these are used supposed to be type classes of i, that its a number and can be compared). What the above declration says that, it takes a number i (number of elements to be sliced) and a list [a] and returns a list [a] (which will have i or less elements)

Now a program written lets say in python would look like,

```
   sliced = []
   for e in xrange(i):
       sliced.append[a[e]]
```

But haskell doesn't have for loops. Because of that, you are forced to think of the function definition from *what to do* to *how to do it*
So let's break it down to steps

   - if the number of elements to take are <= 0, return an empty list (you decided to take nothing from the list)
   - if there are no elements in the list, return an empty list (you can't take anything from an empty list)
   - take the first element from the list, then recurse on the remaining list.

and the supporting code for this in haskell would be

```
take' i _
   | i <=0 = []
take' _ [] = []
take' i (x:xs) = x:take' (i-1) xs
```

In the first case, it doesn't care what the list is (*_* suggests that). If the number of elements to be taken out are less than equal to 0, it returns an empty list.

In the second case it doesn't care what the number is, if the input is an empty list, it'll give back an empty list.

And the final step, if the list has one or more elements, it takes the first element x (haskell sugar for taking out one element from list is x:xs where xs would be the remaing list.) and takes i - 1 elements from the remaining list.

The reason that made me feel like writing the blog post was the implementation of quick sort in haskell. This has haunted me since the days of college days where one could cry looking at the ugly imperative code. If you think about what quicksort essentially boils down to, the how of quicksort, its just these simple words:

"All elements less than the head of the list are supposed to be kept on the left of the list and all elements greater than the head of the list are supposed to be kept on the right side of the list. The head lies in between these elements".

And how does haskell solve that problem for me?

```
quicksort :: (Ord a) => [a] -> [a]
quicksort [] = []
quicksort (x:xs) =
  -- filter all elements that are less than the head of the list, keep them in smaller list, sort the smaller list
  let smallerSorted = quicksort [a | a <- xs, a <= x]
  -- filter all elements that are bigger than the head of the list, keep them in bigger list, sort the bigger list
      biggerSorted  = quicksort [a | a <- xs, a > x]
  -- place the head in the middle of these two lists, return
  in  smallerSorted ++ [x] ++ biggerSorted

```

Neat eh? I am so far not been disappointed by haskell. Let's see what more this language has to bring.

PS: I am referring to this [excellent book](http://learnyouahaskell.com/) to learn haskell. I definitely recommend it.