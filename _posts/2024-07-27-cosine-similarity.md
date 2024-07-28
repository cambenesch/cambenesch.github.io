---
layout: post
title: "Feature-Weighted Cosine Similarity"
author: Cam Benesch
meta: "Chicago"
---

Cosine similarity is a frequently used scalar metric to evaluate multi-output (i.e. vector) predictions. It's often advised that targets (i.e. vector components) should be centered and scaled before computing cosine similarity, but there isn't much material on how precisely this should be done. Depending on the targets' distributions and the desired metric properties, traditional standardization may not be sufficient. 

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

<a name="s2"></a>

## Zero-centering

The target vector $t$ is a draw from some $D$-dimensional distribution $f$. We've already L2-normalized each target vector in our dataset (let's say there are $N$ of them). Now we'll justify the process of zero-centering each of the $D$ targets. 

Imagine another i.i.d draw $t'\sim f$. For the sake of contradiction, suppose $\mathbb{E}\left\lbrack t\cdot t' \right\rbrack > 0$. Then we can construct a naive model which just draws a random $t'$ from the dataset and predicts $y=t'$. This model produces a positive average cossim $y\cdot t>0$ without even using any input features. (For the $\mathbb{E}\left\lbrack t\cdot t' \right\rbrack < 0$ case, just set $y=-t'$.) To thwart this naive model, we thus require 

$$\\ \mathbb{E}\left\lbrack t\cdot t' \right\rbrack = 0 \\$$

which by linearity of expectation gives

$$\\ \mathbb{E}\left\lbrack t_1t'_1 + \cdots + t_Dt_D' \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack t_1t'_1 \right\rbrack + \cdots + \mathbb{E}\left\lbrack t_Dt'_D \right\rbrack = 0 \\$$

and since $t,t'$ are iid,

$$\\ \mathbb{E}\left\lbrack t_1 \right\rbrack \mathbb{E}\left\lbrack t'_1 \right\rbrack + \cdots + \mathbb{E}\left\lbrack t_D \right\rbrack \mathbb{E}\left\lbrack t'_D \right\rbrack \\$$

$$\\ = \mathbb{E}\left\lbrack t_1 \right\rbrack^2 + \cdots + \mathbb{E}\left\lbrack t_D \right\rbrack^2 = 0 \\$$

$$\\ \implies \mathbb{E}\left\lbrack t_i \right\rbrack = 0 \\$$

for each target $i$. We can easily enforce this condition by zero-centering the targets. In particular, our dataset contains $N$ samples $t^{(1)},...,t^{(N)}$. So wea compute $\mu = (t^{(1)} + \cdots + t^{(N)})/N$ and center the data by subtracting $\mu$ from each $t^{(k)}$. The preprocessing now involves per-sample L2-normalization followed by per-dimension zero-centering.

<a name="s3"></a>

## Scaling

Typically you'll want to scale your targets so that each target has an equal **opportunity** to contribute to the final dot product (the cossim). Note that equal opportunity doesn't mean equal contributions. For instance, if we're very good (or uncannily bad) at predicting $t_i$ but not $t_j$, then ${\mid}y_it_i{\mid} > {\mid}y_jt_j{\mid}$ in expectation. 

There are, however, scenarios where we **should** expect equal contributions. For instance, let's revisit the naive model where $y,t\sim f$ are iid. Consider any 2 targets $i,j$. Under the naive random-draw model, $y,t$ are independently drawn, so we're unable to explain any variance in $t_i$ or $t_j$. In this case, it's clearly fair to expect the cossim's $i$ and $j$ terms to have the same average magnitudes. In particular, we'll require

$$\\ \mathbb{E}\left\lbrack {\mid}t_iy_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}t_jy_j{\mid} \right\rbrack \text{  (Condition 1)} \\$$

Since $y,t$ are iid,

$$\\ \mathbb{E}\left\lbrack {\mid}t_iy_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}t_i{\mid} \cdot {\mid}y_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}t_i{\mid} \right\rbrack \mathbb{E}\left\lbrack {\mid}y_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}t_i{\mid} \right\rbrack^2 \\$$

and likewise for $j$. Thus Condition 1 becomes 

$$\\ \mathbb{E}\left\lbrack {\mid}t_i{\mid} \right\rbrack^2 = \mathbb{E}\left\lbrack {\mid}t_j{\mid} \right\rbrack^2 \\$$
$$\\implies \mathbb{E}\left\lbrack {\mid}t_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}t_j{\mid} \right\rbrack \\$$

We can 

<a name="s4"></a>

## Normally distributed targets

Remember the target vector $t$ is a draw from $D$-dimensional multivariate normal distribution $f=\mathcal{N}(0,\Sigma)$. Of course, $f$ is zero-centered. 

 
<a name="s5"></a>

## Feature weights

