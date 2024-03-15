---
layout: post
title: "Counting Chord Intersections"
categories: Computer Science
author: Cam Benesch
meta: "Chicago"
---

In this post I'll explain a cool way to use annotated trees to solve a naively $\Theta(n^2)$ problem in $\Theta(n\log n)$ time. Should be an easy read if you know a bit about binary trees, hashmaps, and big O notation. 

## The Problem

Let's start with a circle, then draw a few [chords](https://en.wikipedia.org/wiki/Chord_(geometry)) on its circumference. For simplicity, assume no chords have any identical endpoints, as shown below. 

<p align="center" width="100%">
    <img width="85%" src="/assets/images/chord1.png"> <br>
    Figure 1 [5]
</p>

## Slow algorithm

Compare every pair of chords. 

## Fast Algorithm

Key observation 1: consider two chords `i` and `j`, whose endpoints (i.e. starting or ending points) we encounter in the order `{i,j,i,j}` or `{i,j,i,j}`. You can convince yourself geometrically that these chords intersect. The other possibilities are `{i,i,j,j}`, `{j,j,i,i}`, `{i,j,j,i}`, `{j,i,i,j}`. These chords do not intersect. 

Then we create an annotated complete binary tree (custom class) with one leaf for each of the $n$ chords. Adding a leaf node takes $\Theta(\log n)$ time. We add $n$ leaf nodes, which takes $\Theta(n \log n)$ time. In this tree, each leaf node has a label (`val`) corresponding to a chord. Non-leaf nodes have `val=-1`. Each leaf node may be activated (`size=1`) or deactivated (`size=0`, the default). Each node also has a `size`, indicating the number of activated nodes in its subtree. Finally, the tree has a `leafs` attribute providing constant-time access to leaf nodes. 

Key observation 2: For leaf node `i` (`val=i`), we can count the number of higher-valued activated nodes in $\Theta(\log n)$ time as follows. If `i` is a right child of its parent, then its parent contains no higher-valued activated nodes. If `i` is a left child of its parent, then every activated node in its right sibling's subtree is greater than `i`. This count is the `parent.size - i.size`. Repeat this computation for each parent of `i`, and sum the counts to get the number of higher-valued activated nodes.

We also maintain a hashmap to indicate the order in which the first endpoint of each chord was encountered. This facilitates relabeling of the nodes. 

Loop through each (radian measure, chord number) pair, of which there are $2n$. 

- If we encounter a new chord, we relabel it `i` (the order in which it was first encountered) and activate the leaf node with `val=i`. Activating a leaf node involves incrementing $\Theta(\log n)$ `size` attributes, one for each of its ancestors. 

- Chord already seen -> get its relabeled `i` from the hashmap, and deactivate leaf node `i`. Deactivation involves decrementing its $\Theta(\log n)$ ancestors' `size` attributes. But before this deactivation, we compute in $\Theta(\log n)$ time how many higher-valued leaf nodes are activated. A higher-valued leaf node `j>i` is activated IFF we encountered `i`'s and `j`'s endpoints in the order `[i,j,i,j]`. Thus, this count gives us the exact number of intersections between `i` and `j`. 

Running this loop takes $\Theta(n \log n)$ time. Summing the higher-valued-activated-leaf-node counts gives the solution. No step takes more than $\Theta(n \log n)$ time, so the algorithm takes $\Theta(n \log n)$ time. 