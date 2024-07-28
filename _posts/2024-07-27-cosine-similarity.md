---
layout: post
title: "Feature-Weighted Cosine Similarity"
author: Cam Benesch
meta: "Chicago"
---

Cosine similarity is a frequently used scalar metric to evaluate multi-output (i.e. vector) predictions. It's well-known that targets (i.e. vector components) should be centered and scaled before computing cosine similarity, but there isn't much material on how precisely this should be done. Depending on the targets' distributions and the desired metric properties, traditional standardization may not be sufficient. 

Here I'll briefly define our goal in scaling, then talk about how to achieve that for the case of normally distributed targets. Then I'll show an easy approximate scaling method for arbitrary/unknown target distributions. These methods give a clean way to compute a target-weighted cosine similarity: in short, scale each target by its desired weight's square root.

## Topics
1. [Cosine similarity](#s1)
2. [Normally distributed targets](#s2)
3. [Arbitrarily distributed targets](#s3)
4. [Feature weights](#s4)

<a name="s1"></a>

## Cosine similarity

We'll abbreviate [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) as "cossim". It's a score from -1 to 1; just the cosine of the angle between 2 vectors. Say you have a model which processes some input features, then predicts some $D$-dimensional target vector $t=[t_1,...,t_D]$. Let's call your model's prediction $y=[y_1,...,y_D]$ (others might refer to it as $\hat{t}$). 

I'll start with a warning. Cossim is usually not appropriate for comparing vectors. For instance, say we're trying to predict a device's movement based on GPS data. There are 2 targets (N/S movement and E/W movement), hence they comprise a vector. But if we're using this model to actually locate the device, then cossim is useless: it treats \[1 inch, 2 inches\] the same as \[10 miles, 20 miles\] (cossim = 1). On top of that, it treats those both very differently, in fact maximally differently, than \[-1 inch, -2 inches\] (cossim = -1). Cossim ignores magnitude, and is very sensitive to small perturbations near the origin. So if you want to figure out the direction the device has moved, and if you're confident it has moved far enough to withstand some noise, then go ahead and use cossim. Otherwise, leave this page and find another metric. 

If you're still here, you don't care if $y$ doesn't match $t$ exactly - all you want is for them to point in the same direction. To simplify things, we'll assume you've already L2-normalized your vectors $t,y$. This makes things way easier since the cossim just becomes the dot product:

$$\\
\cos(\theta) = \frac{t\cdot y}{||t|| ||y||} = t\cdot y = t_1y_1+\cdots +t_Dy_D
\\$$

Typically you'll want to scale your targets so that in expectation, each target contributes equally to the final cossim. While arbitrary, it seems obvious enough that "contributes equally" should be defined as (Condition 1)
$$\\
\mathbb{E}\left\lbrack {\mid}t_iy_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}t_jy_j{\mid} \right\rbrack
\\$$
for any 2 targets $i,j$. In words, the cossim's $i$ and $j$ terms should have the same average magnitudes. How can we scale to ensure this? 

<a name="s2"></a>

## Normally distributed targets

We're assuming the target vector $t$ is a draw from $D$-dimensional multivariate normal distribution $\mathcal{N}(\mu,\Sigma)$. 

Given a dataset containing many draws (i.e. many values of $t$), one can roughly reconstruct this distribution. From there, we can imagine a naive baseline model which, regardless of the input features, predicts $y$ as an independent random draw from that same distribution $\mathcal{N}(\mu,\Sigma)$. Under this baseline model, $y$ and $t$ are iid. The baseline model is awful and naive because it just ranodmly chooses something that looks like the observed $t$'s, providing no explanatory or predictive power. So it would make sense to expect an average cossim of (Condition 2)
$$\\
\mathbb{E}\left\lbrack {\mid}t\cdot y{\mid} \right\rbrack = 0
\\$$

Combining Conditions 1 and 2 gives 
$$\\
\mathbb{E}\left\lbrack {\mid}t\cdot y{\mid} \right\rbrack
= \mathbb{E}\left\lbrack {\mid}t_1y_1+\cdots +t_Dy_D{\mid} \right\rbrack
= \mathbb{E}\left\lbrack {\mid}t_1y_1{\mid} \right\rbrack + \cdots + \mathbb{E}\left\lbrack {\mid}t_Dy_D{\mid} \right\rbrack
= \mathbb{E}\left\lbrack {\mid}t_1{\mid} {\mid}y_1{\mid} \right\rbrack + \cdots + \mathbb{E}\left\lbrack {\mid}t_D{\mid} {\mid}y_D{\mid} \right\rbrack
= \mathbb{E}\left\lbrack {\mid}t_1{\mid} \right\rbrack \mathbb{E}\left\lbrack {\mid}y_1{\mid} \right\rbrack + \cdots + \mathbb{E}\left\lbrack {\mid}t_D{\mid} \right\rbrack \mathbb{E}\left\lbrack {\mid}y_D{\mid} \right\rbrack
= \mathbb{E}\left\lbrack {\mid}t_1{\mid} \right\rbrack^2 + \cdots + \mathbb{E}\left\lbrack {\mid}t_D{\mid} \right\rbrack^2
= D\cdot \mathbb{E}\left\lbrack {\mid}t_1{\mid} \right\rbrack^2 = 0
\implies \mathbb{E}\left\lbrack {\mid}t_i{\mid} \right\rbrack = 0
\\$$


$$\\
\mathbb{E}\left\lbrack {\mid}t_iy_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}t_jy_j{\mid} \right\rbrack
\mathbb{E}\left\lbrack {\mid}t_i{\mid} {\mid}y_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}t_j{\mid} {\mid}y_j{\mid} \right\rbrack
\\$$

<a name="s3"></a>

## Arbitrarily distributed targets


 
<a name="s4"></a>

## Feature weights

