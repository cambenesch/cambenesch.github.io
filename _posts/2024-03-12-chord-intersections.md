---
layout: post
title: "Counting Chord Intersections"
categories: Computer Science
author: Cam Benesch
meta: "Chicago"
header-includes:
  - \usepackage[ruled,vlined,linesnumbered]{algorithm2e}
---

In this post I'll explain a cool way to use annotated trees to speed up an $O(n^2)$ algorithm to run in $O(n\log n)$ time. Should be an easy read if you're familiar with binary trees, hashmaps, and big O notation. 

## The Problem

Let's start with a circle, then draw a few [chords](https://en.wikipedia.org/wiki/Chord_(geometry)) on its circumference. For simplicity, assume no chords have any identical endpoints. Now let's assign a numeric label to each chord. Starting on the right end of the circle and moving counterclockwise, we can begin searching for chord endpoints. Every time we encounter the endpoint of a newly seen chord, we can assign that chord the next label. For instance, the first endpoint we see will be from Chord 0. (If we encounter the other endpoint of an already-labeled chord, that doesn't impact our labeling.) This is illustrated in Figure 1. 

<p align="center" width="100%">
    <img width="100%" src="/assets/images/chord1.png"> <br>
    Figure 1
</p>

In the right side of Fig 1, we can see 4 intersection points: Chords 0 & 1, 0 & 2, 1 & 2, and 2 & 3. Chords 0 & 3, for instance, do not intersect. 

Each endpoint on the circle corresponds to an angle $0\leq \theta < 360$, indicating the counterclockwise degrees from the circle's rightmost point. For instance, the green dashed line shows $\theta=0$, and the bottom-most point on the circle is $\theta=270$. So a tuple of two angles $(s_i, e_i)$ is sufficient to represent a chord on the circle. 

In general, say we have $n$ chords, given as an unordered set $C=\{(s_1,e_1),...,(s_n,e_n)\}$. We'll discuss how to count the number $I$ of distinct pairs of chords which cross paths - that is, the number of intersections - as quickly as possible. 

## Slow algorithm

Clearly, we can have $I=0$, if all chords are mutually parallel. On the other hand, suppose each chord intersects every other chord. Then, at most, we can have ${n\choose 2} = n(n-1)/2 = O(n^2)$ intersections. This reveals a simple solution. Starting with $I=0$, just look at each pair of distinct chords, see if they intersect, and increment $I$ if they do. 

How can we determine whether chords $C_i=(s_i,e_i)$ and $C_j=(s_j,e_j)$ intersect, for $i<j$? 

First off, note that $(s_i,e_i)$ looks exactly the same as $(e_i,s_i)$. We can freely swap the starting and ending angles of a chord. Thus we can safely assume that $s_i<e_i$ - if not, then swap them. 

Next, remember that we labeled the chords by starting at the green dashed line and searching counterclockwise for new endpoints. So if we sort $s_i,e_i,s_j,e_j$ in increasing order, $s_i$ must appear first. From here, you can convince yourself that there are only 3 possibilities for the sorted sequence: $s_i<s_j<e_i<e_j$, $s_i<s_j<e_j<e_i$, $s_i<e_i<s_j,e_j$. And of these 3, as illustrated in Figure 2, only $s_i<s_j<e_i<e_j$ indicates an intersection between chords $i$ and $j$. 

<p align="center" width="100%">
    <img width="100%" src="/assets/images/chord2.png"> <br>
    Figure 2
</p>

Using this observation, we can write an $O(n^2)$ algorithm which just checks each pair of chords for an intersection:


**Input**: $C=\{(s_1,e_1),...,(s_n,e_n)\}$, where $s_k,e_k$ are angular endpoints of chord $i$. \
**Output**: $I$, the number of intersecting pairs of the given chords. 

> Initialize $I=0$\
**for** $k=1,n$:\
&emsp;**if** $s_k>e_k$ **then** swap$(s_k,e_k)$\
sort $C$ by increasing $s_k$\
\
**for** $i=1,n$ **do**\
&emsp;Get chord endpoints $(s_i,e_i)=C[i]$\
&emsp; **for** $j=i+1,n$ **do**\
&emsp;&emsp;Get chord endpoints $(s_j,e_j)=C[j]$\
&emsp;&emsp;**if** $s_i<s_j<e_i<e_j$ **then**\
&emsp;&emsp;&emsp;increment $I$

The sorting step takes $O(n\log n)$ time, and the for loops take $O(n^2)$ time. 

## Fast Algorithm

Let's try to count intersections without explicitly checking every possible pair of chords. In the slow algo, we sorted $C$. Using this sorted $C$, we'll construct a new list $P$ as follows: for every $(s_i,e_i)$ in $C$, add $(s_i,i)$ and $(e_i,i)$ to $P$. Then sort $P$. After this preprocessing, $P$ is a list of angles of increasing magnitude, and each angle has a label indicating which chord it's an endpoint of. For the example below, $P$ would be $\{(40^{\circ},0),(90^{\circ},1),(110^{\circ},2),(150^{\circ},0),(180^{\circ},3),(270^{\circ},2),(320^{\circ},3),(330^{\circ},1)\}$. Then, as a final step, we'll completely remove the first elements, such that $P=[0,1,2,0,3,2,3,1]$. 

<p align="center" width="100%">
    <img width="60%" src="/assets/images/chord3.png"> <br>
    Figure 3
</p>

Now we'll try counting intersections via a single loop through the $2n$ entries of $P$. As we loop through $P$, let's call a chord "open" if we've only encountered exactly one of its endpoints, and "closed" if we've encountered none or both of its endpoints. 

During our loop, first we come across chord 0, then chord 1, then chord 2 - all three of these chords are now open. Then we come across chord 0 again. This means the endpoints of chords 0 and 1 must appear in the sequence $[0,1,0,1]$, as in Figure 2's left diagram. Therefore, chord 0 intersects chord 1. Likewise for chord 2. Every time we close chord $i$, if we know how many *higher-numbered* chords are open, say $o_i$, then there are simply $o_1+\cdots+o_n$ intersections

Our example would look something like this: open 0, open 1, open 2, close 0 (add two intersections since chords 1 and 2 are open), open 3, close 2 (add one intersection since chord 3 is open), close 3, close 1. This gives $I=2+1=3$, which agrees with the diagram. 

When our loop thru $P$ brings us to an endpoint of chord $i$, we can use a hashset `h` to determine whether to open or close `i`. If `i` isn't a key in `h`, we can set `h[i]=open`, otherwise set `h[i]=closed`. In the latter case where we have to close `i`, the hard part is determining how many higher-numbered chords are still open. 

[comment]: <> (left off here)


To solve this, we create an annotated complete binary tree (custom class) with one leaf for each of the $n$ chords. Adding a leaf node takes $\Theta(\log n)$ time. We add $n$ leaf nodes, which takes $\Theta(n \log n)$ time. In this tree, each leaf node has a label (`val`) corresponding to a chord. Non-leaf nodes have `val=-1`. Each leaf node may be activated (`size=1`) or deactivated (`size=0`, the default). Each node also has a `size`, indicating the number of activated nodes in its subtree. Finally, the tree has a `leafs` attribute providing constant-time access to leaf nodes. 

Key observation: For leaf node `i` (`val=i`), we can count the number of higher-valued activated nodes in $\Theta(\log n)$ time as follows. If `i` is a right child of its parent, then its parent contains no higher-valued activated nodes. If `i` is a left child of its parent, then every activated node in its right sibling's subtree is greater than `i`. This count is the `parent.size - i.size`. Repeat this computation for each parent of `i`, and sum the counts to get the number of higher-valued activated nodes.

We also maintain a hashmap to indicate the order in which the first endpoint of each chord was encountered. This facilitates relabeling of the nodes. 

Loop through each (radian measure, chord number) pair, of which there are $2n$. 

- If we encounter a new chord, we relabel it `i` (the order in which it was first encountered) and activate the leaf node with `val=i`. Activating a leaf node involves incrementing $\Theta(\log n)$ `size` attributes, one for each of its ancestors. 

- Chord already seen -> get its relabeled `i` from the hashmap, and deactivate leaf node `i`. Deactivation involves decrementing its $\Theta(\log n)$ ancestors' `size` attributes. But before this deactivation, we compute in $\Theta(\log n)$ time how many higher-valued leaf nodes are activated. A higher-valued leaf node `j>i` is activated IFF we encountered `i`'s and `j`'s endpoints in the order `[i,j,i,j]`. Thus, this count gives us the exact number of intersections between `i` and `j`. 

Running this loop takes $\Theta(n \log n)$ time. Summing the higher-valued-activated-leaf-node counts gives the solution. No step takes more than $\Theta(n \log n)$ time, so the algorithm takes $\Theta(n \log n)$ time. 