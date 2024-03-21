---
layout: post
title: "Counting Chord Intersections"
categories: Computer Science
author: Cam Benesch
meta: "Chicago"
---

This post is about a cool way to speed up an $O(n^2)$ algorithm to $O(n\log n)$ runtime using annotated trees. Easy read if you're familiar with binary trees, arrays, and big O notation. 

## Problem Formulation

Let's draw a few [chords](https://en.wikipedia.org/wiki/Chord_(geometry)) on a circle. A point on the circle's circumference is identified by its [polar angle](https://en.wikipedia.org/wiki/Polar_coordinate_system) $0^{\circ}\leq \theta < 360^{\circ}$, indicating the point's counterclockwise angle from the green dashed line in Figure 1. A chord has 2 endpoints, each a point the circle's circumference. Thus, we will identify a chord using a tuple of its endpoint angles: $C_i = (s_i, e_i)$. 

As input we are given $n$ chords, $C=[(s_1,e_1),...,(s_n,e_n)]$. For simplicity, assume no endpoints are reused (see Figure 1). We'll discuss how to efficiently count the number $I$ of distinct pairs of intersecting chords. 

<p align="center" width="100%">
    <img width="100%" src="/assets/images/chord1.png"> <br>
    Figure 1 - No shared endpoints
</p>

## Chord labeling procedure

Note that a chord doesn't change if the starting and ending angles are swapped. $(s_i,e_i)$ is the same chord as $(e_i,s_i)$. Thus we can safely assume that $s_i<e_i$ (if not, then just go ahead and swap them). This is a preprocessing of $C$, and let's call the preprocessed list $C'$. 

As another preprocessing step, sort $C'$ in increasing order of starting angle $s_i$. We'll call the sorted list $C''$. After this sorting, each chord's number is its position in $C''$. Visually, starting at the green dashed line and proceeding counterclockwise, the chords are numerically labeled in order of first appearance of an endpoint. See the example below for clarity. 

## Example

<p align="center" width="100%">
    <img width="57%" src="/assets/images/chord4.gif"> 
    <img width="42%" src="/assets/images/chord3.png"> 
    <br>
    Figure 2 - Labeling the chords
</p>

Fig 2 shows $n=4$ chords numbered 0 thru 3. There are ${4\choose 2} = 6$ pairs of distinct chords. Four of these pairs intersect: 0 & 1, 0 & 2, 1 & 2, and 2 & 3. Chords 0 & 3 and 1 & 3 do not intersect.

Unprocessed input: $C = [(110^{\circ},270^{\circ}),(320^{\circ},180^{\circ}),(90^{\circ},330^{\circ}),(150^{\circ},40^{\circ})]$

After sorting each tuple: $C' = [(110^{\circ},270^{\circ}),(180^{\circ},320^{\circ}),(90^{\circ},330^{\circ}),(40^{\circ},150^{\circ})]$

After sorting list: $C'' = [(40^{\circ},150^{\circ}),(90^{\circ},330^{\circ}),(110^{\circ},270^{\circ}),(180^{\circ},320^{\circ})]$. \
The order of $C''$ gives the numeric chord labeling. 

## Number of possible intersections

Suppose each chord intersects each other chord. Then, as an upper bound, we have ${n\choose 2} = n(n-1)/2 = O(n^2)$ intersections. This suggests a simple $O(n^2)$ solution. Starting with $I=0$, check each pair of chords, and increment $I$ if they intersect. 

How can we determine whether chords $C_i=(s_i,e_i)$ and $C_j=(s_j,e_j)$ intersect, for $i<j$? 

## Checking whether 2 chords intersect

Remember that we labeled the chords by starting at the green dashed line and searching counterclockwise for new endpoints. So if $i<j$ and we sort $s_i,e_i,s_j,e_j$ in increasing order, $s_i$ must appear first. From here, convince yourself that there are only 3 possibilities for the sorted sequence: $s_i<s_j<e_i<e_j$, or $s_i<s_j<e_j<e_i$, or $s_i<e_i<s_j<e_j$. Of these possibilities, as illustrated in Figure 3, only $s_i<s_j<e_i<e_j$ indicates an intersection between chords $i$ and $j$. 

<p align="center" width="100%">
    <img width="100%" src="/assets/images/chord2.png"> <br>
    Figure 3 - Checking whether 2 chords intersect
</p>

### Slow $O(n^2)$ Algorithm
Using this observation, we can write an $O(n^2)$ algorithm which just checks each pair of chords for an intersection:

<p align="center" width="100%">
    Algorithm 1 - slow intersection counting
</p>
**Input**: $C=[(s_1,e_1),...,(s_n,e_n)]$, where $s_k,e_k$ are endpoint angles of chord $i$. \
**Output**: $I$, the number of intersecting pairs of the given chords. 

> Initialize $I=0$\
**for** $k=0,n-1$:\
&emsp;**if** $s_k>e_k$ **then** swap$(s_k,e_k)$\
sort $C$ by increasing $s_k$\
\
**for** $i=0,n-1$ **do**\
&emsp;Get chord endpoints $(s_i,e_i)=C[i]$\
&emsp; **for** $j=i+1,n-1$ **do**\
&emsp;&emsp;Get chord endpoints $(s_j,e_j)=C[j]$\
&emsp;&emsp;**if** $s_i<s_j<e_i<e_j$ **then**\
&emsp;&emsp;&emsp;increment $I$

The sorting step takes $O(n\log n)$ time, and the nested for loops take $O(n^2)$ time. Next we'll describe a faster algorithm.

## Sorted list of endpoint labels
Starting with $C''$, construct a new list $P$ as follows: for every $(s_i,e_i)$ in $C$, add $(s_i,i)$ and $(e_i,i)$ to $P$. Each entry in $P$ contains the angle of an endpoint, and the numeric label of that endpoint. 

Now sort $P$ by increasing angle. For Figure 2's example, $P$ is now $[(40^{\circ},0),(90^{\circ},1),(110^{\circ},2),(150^{\circ},0),(180^{\circ},3),(270^{\circ},2),(320^{\circ},3),(330^{\circ},1)]$. As a final step, completely remove the angles, such that $P$ is just a sorted list of $2n$ endpoint labels, e.g. $P=[0,1,2,0,3,2,3,1]$. 

## Counting higher-numbered "open" chords
We'll try counting intersections via a single loop thru $P$. As we loop through $P$, let's call a chord **"open"** if we've encountered exactly one of its endpoints, and **"closed"** if we've encountered neither or both of its endpoints. 

Refer to Fig 2. During our loop, first we come across chord 0, then chord 1, then chord 2 - all three of these chords are now open. Then we come across chord 0 again. This means the endpoints of chords 0 and 1 occur in the sequence $[0,1,0,1]$, as in Fig 3's left diagram. Therefore, chord 0 intersects chord 1. Likewise for chords 0 and 2. Every time we close chord $i$, if we know how many **higher-numbered** chords are open, say $o_i$, then there are simply $I=o_1+\cdots+o_n$ total intersections.

<video src="https://github.com/cambenesch/cambenesch.github.io/assets/33947384/35971a3d-981c-465e-98a5-8c0d78c9e321" controls="controls" style="max-width: 730px;">
</video>
<p align="center" width="100%">
    Figure 4 - Slow algorithm illustration
</p>

In this example, we would open 0, open 1, open 2, close 0 (add two intersections since chords 1 and 2 are still open), open 3, close 2 (add one intersection since chord 3 is open), close 3, close 1. This gives $I=2+1=3$, as in Figre 4 above. (The "higher-numbered" condition is crucial. Without it, we would count a false intersection between Chords 1 & 2, since 1 is still open when we close 2.)

This method still has one missing detail: When we close a chord, how do we know how many **higher-numbered** chords are currently open? 

Checking each element of `h` takes $O(n)$ time. Clearly, doing so each time we close a chord just produces another $O(n^2)$ algorithm. Can we do better?

## Indexed complete binary tree

It turns out we can't do better. Just kidding, we can. We can count higher-numbered elements in $O(\log n)$ time by extending a [complete binary tree](https://www.geeksforgeeks.org/complete-binary-tree/). As a quick summary, each leaf will correspond to a chord, each node will store the number of open leafs in its subtree, and via a bubble-up procedure, we can count the number of open leafs to the right of any specified leaf. 

Our tree `T` will have one **leaf** node for each of the $n$ chords. All leaf nodes reside at the same depth level: $d=\lceil \log n \rceil$, where the root has depth 0. (Logs in this post use base 2.)

The leafs have the same left-to-right order as the chord numberings. For instance, Chord 0 corresponds to `T`'s leftmost leaf, and Chord $n-1$ corresponds to the rightmost leaf. Since we know $n$'s value, we can construct `T` by adding leafs one at a time. Each leaf addition takes $d=O(\log n)$ time, so constructing `T` with $n$ leafs takes $O(n\log n)$ time. 

For quick constant-time access to leafs (indexing), we can use an array `leaf` with `leaf[i]` pointing to Chord $i$'s leaf node in `T`. This will make life easier. And for quick two-way traversal of `T`, each node has 3 pointers: left child, right child, and parent. 

## Annotating the tree

Each tree node, leaf or not, is annotated with a "size". Leaf $i$'s size is 1 if Chord $i$ is open, 0 otherwise. A non-leaf's size is the number of open leafs in its subtree. For instance, the root's size is equal to the total number of open leafs. Before looping thru $P$, all nodes have initial value 0. 

What happens when we open chord $i$? Clearly, we should set `leaf[i].size=1`. Each of $i$'s ancestors' sizes should also be incremented, since one leaf in their subtree was newly opened. This takes $O(\log n)$ time. Likewise, closing chord $i$ requires setting `leaf[i].size=0`, and decrementing each of $i$'s ancestors' sizes. 

Now, back to the reason we created `t`: When we close a chord, we need to quickly count how many higher-numbered chords are still open. 
- Suppose node $R$ is a **right** child of its immediate parent $A$. Consider the highest-numbered leaf $B$ in the subtree rooted at $A$. Since the leafs are in increasing order from left to right, $B$ must also be in the subtree rooted at $R$. Thus, $A$ contains no higher-numbered leafs than $R$. 
- Suppose node $L$ is a **left** child of $A$, and $A$ has right child $R$. Then every single open leaf in the subtree rooted at $R$ is higher-numbered than any leaf in the subtree rooted at $L$. Again, this follows from the left-to-right ordering of the leafs in the tree. 

This gives a recursive $O(\log n)$ procedure for counting how many higher-numbered chords are open when we close Chord $i$. See Fig 5 for an illustration. 

<p align="center" width="100%">
    Algorithm 2 - recursively count higher-numbered leafs
</p>
**Input**: Complete binary annotated tree `T`, Chord number $i$ to close \
**Output**: Number $G$ of currently open chords with numeric label $>i$

> Initialize $G=0$, `cur = T[i]`\
**while** `cur.parent` isn't null:\
&emsp;set A to `cur.parent`\
&emsp;**if** `cur` is left child of A **then** \
&emsp;&emsp;add size of A's right child to $G$\
&emsp;set `cur` to A


## Final $O(n\log n)$ algo

<video src="https://github.com/cambenesch/cambenesch.github.io/assets/33947384/fce9c4b2-93bc-4cee-b883-4ccb7cc2e22b" controls="controls" style="max-width: 730px;">
</video>
<p align="center" width="100%">
    Figure 5 - Fast algorithm illustration
</p>

Now that we can count a single chord's higher-numbered intersections in $O(\log n)$ time, we can do this for each chord to get an $O(n\log n)$ algorithm. Illustration in Fig 5, pseudocode below. 

<p align="center" width="100%">
    Algorithm 3 - fast intersection counting
</p>
**Input**: $C=[(s_1,e_1),...,(s_n,e_n)]$, where $s_k,e_k$ are endpoint angles of chord $i$. \
**Output**: $I$, the number of intersecting pairs of the given chords. 

> Initialize $I=0$\
Initialize $n=$length$(C)$\
Initialize depth $d=lceil \log n \rceil$\
Initialize complete binary tree `T`$=$ root node\
&emsp;Any node added to `T` starts with size = 0
Initialize array `leafs` of length $n$\
Initialize empty endpoints array $P$\
\
**for** $k=0,n-1$:\
&emsp;**if** $s_k>e_k$ **then** swap$(s_k,e_k)$\
&emsp;Add a leaf to `T` at depth $d$\
&emsp;Leaf should be as far left as possible\
&emsp;Set `leafs[k]` to point to leaf node\
Sort $C$ by increasing $s_k$\
**for** $k=0,n-1$:\
&emsp;add tuple $(s_k, k)$ to $P$\
&emsp;add tuple $(e_k, k)$ to $P$\
$P$ is now an array of (angle, chord label)\
Sort $P$ by increasing angle\
Prop angles from $P$\
$P$ is now an array of $2n$ chord labels\
\
**for** $i=0,2n-1$ **do**\
&emsp;Get next chord label $m=P[i]$\
&emsp;**if** `leafs[m]` is 0 **then**\
&emsp;&emsp;Increment the leaf's and its ancestors' sizes\
&emsp;**if** `leafs[m]` is 1 **then**\
&emsp;&emsp;Set $G=$ higher-numbered leaf count (Algorithm 3) \
&emsp;&emsp;Update total count: $I=I+G$ \
&emsp;&emsp;Decrement the leaf's and its ancestors' sizes\
