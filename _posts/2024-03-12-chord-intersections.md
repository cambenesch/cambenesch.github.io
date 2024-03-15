---
layout: post
title: "Counting Chord Intersections"
categories: Computer Science
author: Cam Benesch
meta: "Chicago"
---

# ChordIntersections
algo to count # intersections of n chords in a circle in $\Theta(n \log n)$ time

## How to run compiled code

Just run `intersections.exe` (compiled from `intersections.cpp` using C++23)

Enter user input followed by Return.

e.g. user might enter `[(0.1,0.2,4.4,5.9),("e1","s1","e2","s1")]`

The first measure in the first tuple corresponds to the first label in the second tuple

We assume no two intersection points are identical.

Enter radian measures in increasing order, with radians in $[0,2\pi)$

Each chord should have exactly one starting point and exactly one ending point.

sample input: `[(0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0),("s3","e2","s1","e5","s4","e1","e3","s2","s5","e4")]`
this contains 8 intersections

## Explanation of algorithm

Key observation 1: consider two chords `i` and `j`, whose endpoints (i.e. starting or ending points) we encounter in the order `{i,j,i,j}` or `{i,j,i,j}`. You can convince yourself geometrically that these chords intersect. The other possibilities are `{i,i,j,j}`, `{j,j,i,i}`, `{i,j,j,i}`, `{j,i,i,j}`. These chords do not intersect. 

Then we create an annotated complete binary tree (custom class) with one leaf for each of the $n$ chords. Adding a leaf node takes $\Theta(\log n)$ time. We add $n$ leaf nodes, which takes $\Theta(n \log n)$ time. In this tree, each leaf node has a label (`val`) corresponding to a chord. Non-leaf nodes have `val=-1`. Each leaf node may be activated (`size=1`) or deactivated (`size=0`, the default). Each node also has a `size`, indicating the number of activated nodes in its subtree. Finally, the tree has a `leafs` attribute providing constant-time access to leaf nodes. 

Key observation 2: For leaf node `i` (`val=i`), we can count the number of higher-valued activated nodes in $\Theta(\log n)$ time as follows. If `i` is a right child of its parent, then its parent contains no higher-valued activated nodes. If `i` is a left child of its parent, then every activated node in its right sibling's subtree is greater than `i`. This count is the `parent.size - i.size`. Repeat this computation for each parent of `i`, and sum the counts to get the number of higher-valued activated nodes.

We also maintain a hashmap to indicate the order in which the first endpoint of each chord was encountered. This facilitates relabeling of the nodes. 

Loop through each (radian measure, chord number) pair, of which there are $2n$. 

- If we encounter a new chord, we relabel it `i` (the order in which it was first encountered) and activate the leaf node with `val=i`. Activating a leaf node involves incrementing $\Theta(\log n)$ `size` attributes, one for each of its ancestors. 

- Chord already seen -> get its relabeled `i` from the hashmap, and deactivate leaf node `i`. Deactivation involves decrementing its $\Theta(\log n)$ ancestors' `size` attributes. But before this deactivation, we compute in $\Theta(\log n)$ time how many higher-valued leaf nodes are activated. A higher-valued leaf node `j>i` is activated IFF we encountered `i`'s and `j`'s endpoints in the order `[i,j,i,j]`. Thus, this count gives us the exact number of intersections between `i` and `j`. 

Running this loop takes $\Theta(n \log n)$ time. Summing the higher-valued-activated-leaf-node counts gives the solution. No step takes more than $\Theta(n \log n)$ time, so the algorithm takes $\Theta(n \log n)$ time. 