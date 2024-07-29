---
layout: post
title: "Dimension-Weighted Cosine Similarity"
author: Cam Benesch
meta: "Chicago"
---

Cosine similarity is a frequently used scalar metric to evaluate multi-output (i.e. vector) predictions. It's often advised that targets be scaled before computing cosine similarity, but there isn't much material on how precisely this should be done. Depending on the targets' distributions and the desired metric properties, traditional standardization may or may not be sufficient. 

Here I'll briefly define our goal in scaling, then show how to properly scale in 2 cases: unknown target distributions (approximate L1-normalization), and known normal target distributions (exact standardization). These methods reveal a clean way to compute dimension-weighted cosine similarity: scale each target dimension by its desired weight's square root. I'll end with a quick discussion of baseline cossim, along with an open question. Skip to the summary if you just want to see the suggested transformations. 

## Topics
1. [Cosine similarity](#s1)
2. [Scaling](#s2)
3. [Normally distributed targets](#s3)
4. [Dimension weighting](#s4)
5. [Summary](#s5)
6. [Baseline cossim](#s6)

<a name="s1"></a>

## Cosine similarity

We'll abbreviate [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) as "cossim". It's a score from -1 to 1; just the cosine of the angle between 2 vectors. Say you have a model which takes in some input features, then predicts a $D$-dimensional target vector $t=[t_1,...,t_D]$. Let's call your model's prediction $y=[y_1,...,y_D]$ (others might refer to it as $\hat{t}$). 

Warning: cossim is usually not appropriate for comparing vectors. For example, suppose we have a model which predicts a device's exact movement based on noisy GPS data. There are 2 target dimensions (N/S movement and E/W movement), hence they comprise a vector. But if we're using this model to actually locate the device, then cossim is useless: it treats \[1 inch, 2 inches\] the same as \[10 miles, 20 miles\] (cossim = 1). On top of that, it treats those both very differently (in fact maximally differently) than \[-1 inch, -2 inches\] (cossim = -1). Cossim ignores magnitude, and is very sensitive to small perturbations near the origin. So if you want to model the direction the device has moved, and if you're confident it has moved far enough to withstand some noise, then go ahead and use cossim. Otherwise, leave this page and find another metric. 

If you're still here, you don't care whether $y$ matches $t$ exactly - all you want is for them to point in the same direction. Here's the formula for cossim:

$$\\ \cos(\theta) = \frac{t\cdot y}{||t|| ||y||} = \frac{t_1y_1+\cdots +t_Dy_D}{\sqrt{(t_1^2+\cdots +t_D^2)(y_1^2+\cdots + y_D^2)}} \\$$

We also have a dataset containing $N$ target vectors $t^{(1)},...,t^{(N)}\sim f$, each an iid sample from some $D$-dimensional target distribution $f$. 

<a name="s2"></a>

## Scaling

In many cases you'll want to scale your target dimensions so that each target dimension has an equal **opportunity** to contribute to the final cossim. Note that equal opportunity doesn't mean equal contribution. If we're very good (or uncannily bad) at predicting $t_i$ but not $t_j$, then ${\mid}y_it_i{\mid} > {\mid}y_jt_j{\mid}$ in expectation. 

There are, however, scenarios where we **should** expect equal contributions. For instance, let's sample $a, b\sim f$ iid from our dataset's target vectors $t^{(1)},...,t^{(N)}$. Consider any 2 target dimensions $i,j$. Since $a,b$ are independently drawn, $a_i,a_j$ cannot explain any variance in $b_i,b_j$. In this case, the cossim's $i$ and $j$ terms must have the same average magnitudes. In particular, we require

$$\\ \mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack \text{  (Condition 1)} \\$$

Since $a,b$ are iid, so too are ${\mid}a{\mid}, {\mid}b{\mid}$, and

$$\\ \mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}a_i{\mid} \cdot {\mid}b_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack \mathbb{E}\left\lbrack {\mid}b_i{\mid} \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack^2 \\$$

Likewise for $j$. Thus Condition 1 becomes 

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack^2 = \mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack^2 \\$$
$$\\ \implies \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack \\$$

This is easily enforced. For each target dimension $i$, the dataset allows us to estimate a scaling factor $C_i$:

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack \approx \frac1N \sum_{k=1}^N {\mid}t^{(k)}_i{\mid} = \frac1{C_i} \\$$

, which yields $C= \left\lbrack C_1,...,C_D \right\rbrack$. Now we can scale each target vector $t^{(k)}$ to be the component-wise product $C \odot t^{(k)}$. After this scaling, Condition 1 becomes:

$$\\ \mathbb{E}\left\lbrack {\mid}(C_ia_i)(C_ib_i){\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}(C_ja_j)(C_jb_j){\mid} \right\rbrack \\$$

$$\\ C_i^2 \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack^2 = C_j^2 \mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack^2 \\$$

$$\\ C_i^2 \frac1{C_i^2} = C_j^2 \frac1{C_j^2} \\$$

The last step uses our approximation, under which Condition 1 is clearly true/fulfilled. 

<a name="s3"></a>

## Normally distributed targets

Consider the case where the target vector $t$ is a draw from $D$-dimensional multivariate normal distribution $f=\mathcal{N}(\mu,\Sigma)$. Since $f$ is known, approximation is no longer necessary. As shown above, Condition 1 is 

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack \\$$

Each target dimension follows a univariate normal distribution $a_i\sim \mathcal{N}(\mu,\Sigma_{ii})$ and $a_j\sim \mathcal{N}(\mu,\Sigma_{jj})$. Therefore, ${\mid}a_i{\mid}, {\mid}a_j{\mid}$ each follow the [folded normal distribution](https://en.wikipedia.org/wiki/Folded_normal_distribution), whose expectation is known to be the following ugly formula:

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack = \sqrt{\frac{2\Sigma_{ii}^2}{\pi}} \left\lbrack \exp -\frac{\mu^2}{2\sigma^2} \right\rbrack + \mu - 2\mu \Phi(-\mu/\sigma) \\$$

This is fine - it's evident that we can easily fulfill Condition 1 by ensuring $\mu_i = \mu_j$ and $\Sigma_{ii} = \Sigma_{jj}$. So you can choose any desired $u\in \mathbb{R}, v\in \mathbb{R}^+$, shift the target vectors such that $\mu = \left\lbrack u,...,u \right\rbrack$, then scale the target vectors such that $\text{diag}(\Sigma) = \left\lbrack v,...,v \right\rbrack$. 

If it makes sense with your dataset, though, choose $u=0$, $v=1$ to make life easier. Enforcing Condition 1 then just means standardizing each target dimension. First zero-center by subtracting $\mu$ from each $t^{(k)}$, then scale by $C = \left\lbrack \frac1{\sqrt{\Sigma_{11}}}, ..., \frac1{\sqrt{\Sigma_{DD}}} \right\rbrack$. And for what it's worth, $\mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack$ can now be expressed succinctly as $\sqrt{\frac{2}{\pi}}$. 

<a name="s4"></a>

## Dimension weighting

We've been designing the scaling factor $C$ to give each target dimension an **equal** opportunity to contribute to cossim. But what if we care more about some target dimensions than others? Maybe there's an importance vector $P = \left\lbrack P_1, ..., P_D \right\rbrack$, and instead of Condition 1, we want to achieve the following:

$$\\ \frac{\mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack}{\mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack} = \frac{P_i}{P_j} \text{  (Condition 2)} \\$$

Earlier, we computed equal-weighting scaling factors $C_i$ which fulfilled Condition 1. These can be reweighted: $Q_i = C_i\sqrt{P_i}$, and you can scale by $Q$ instead of $C$ to fulfill Condition 2 instead:

$$\\ \frac{\mathbb{E}\left\lbrack {\mid}(Q_ia_i)(Q_ib_i){\mid} \right\rbrack}{\mathbb{E}\left\lbrack {\mid}(Q_ja_j)(Q_jb_j){\mid} \right\rbrack} \\$$

$$\\ = \frac{Q_i^2 \mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack}{Q_j^2\mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack} \\$$

$$\\ = \frac{P_i}{P_j} \frac{C_i^2 \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack^2}{C_j^2\mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack^2} \\$$

$$\\ = \frac{P_i}{P_j} \frac{C_i^2 (1 / C_i^2)}{C_j^2 (1 / C_j^2)} = \frac{P_i}{P_j} \\$$

<a name="s5"></a>

## Summary

You have a dataset containing input features, and output target vectors $t^{(1)},...,t^{(N)}$. Each sampled target vector $t^{(k)}$ is $D$-dimensional. You've also decided on importance weights $P = \left\lbrack P_1, ..., P_D \right\rbrack$ for the target dimensions.You want to compute a weighted cossim in which on average, each target dimension $i$'s contribution to the final cossim is proportional to $P_i$. 

1. If you're confident $t^{(1)},...,t^{(N)}$ were drawn from a multivariate normal distribution, then standardize each dimension of each $t^{(k)}$. Step 1: (please consider implications first) Zero-center your targets by averaging $t^{(1)},...,t^{(N)}$ then subtracting that mean from each $t^{(k)}$. Step 2: divide each dimension by its standard deviation. 
2. If you're not confident $t^{(1)},...,t^{(N)}$ were drawn from a multivariate normal, then divide each dimension by its average absolute value $\frac1N \sum_{k=1}^N {\mid}t^{(k)}_i {\mid} $. 
3. If your importance weights $P_i$ are all equal, you're all set. If they're not, multiply each target vector's $i$-th dimension by $\sqrt{P_i}$.

Now that you've preprocessed your dataset's target vectors, you can train a model to take some input features and produce a prediction $y$ of the target vector associated with those inputs. The correct target vector from your dataset is $t$. Finally, you can compute the weighted cossim $\cos(\theta) = \frac{t\cdot y}{ \mid \mid t\mid \mid \mid \mid y\mid \mid }$ between your prediction and the true value.

As explained below, I also suggest computing the average pairwise cossim of target vectors within your datasets as a baseline metric. Your model should not underperform this baseline. 

<a name="s6"></a>

## Open question: Recentering

I cautiously recommended zero-centering normally distributed targets. Why not zero-center in general? A simple answer is that angles change, sometimes drastically, when you start shifting data. For some applications, including the GPS example above, shifting is just incorrect and entirely changes the data's meaning. In the rest of this page, assume that the application allows for shifting. 

Choose 2 target vectors $a, b$ from $t^{(1)},...,t^{(N)}$. For the sake of contradiction, suppose $\mathbb{E}\left\lbrack \text{cossim}(a,b) \right\rbrack > 0$. Then we can construct a naive model which at test time discards the input features, samples a random $t'$ from the training dataset's target vectors, and predicts $y=t'$. Upon evaluation, this model produces a positive average cossim without even using the input features. (The $\mathbb{E}\left\lbrack \text{cossim}(a,b) \right\rbrack < 0$ case is similar; just set $y=-t'$.) To thwart this naive model, we would like cossim to have zero expectation:

$$\\ \mathbb{E}\left\lbrack \frac{a\cdot b}{ \mid \mid a\mid \mid \mid \mid b\mid \mid } \right\rbrack = 0 \\$$

Since $a,b$ are iid and expectation is linear, this is equivalent to 

$$\\ \mathbb{E}\left\lbrack \frac{a_i}{ \mid \mid a\mid \mid } \right\rbrack = 0 \\$$

If $D=1$, we end up with $\mathbb{E}\left\lbrack \text{sign}(a) \right\rbrack = 0$, which can be achieved by subtracting $t$'s median from each $t^{(k)}$. If $D>1$, this is extremely hard to enforce, [even for the normal special case](https://stats.stackexchange.com/a/265902). So the open question is how to thwart the naive model for arbitrary $f$ and $D$. 

Until that's solved, I'd recommend just computing cossim for each pair of distinct target vectors in the dataset. Average those, then use that as a baseline when you're evaluating models. That takes $O(N^2D)$ time. With a bit of optimization, we can reduce it to $O(ND)$:

$$\\ \mathbb{E}\left\lbrack \frac{a\cdot b}{ \mid \mid a\mid \mid \mid \mid b\mid \mid } \right\rbrack \\$$

$$\\ = \sum_{i=1}^D \mathbb{E}\left\lbrack \frac{a_ib_i}{ \mid \mid a\mid \mid \mid \mid b\mid \mid } \right\rbrack \\$$

$$\\ = \sum_{i=1}^D \mathbb{E}\left\lbrack \frac{a_i}{ \mid \mid a\mid \mid } \right\rbrack \mathbb{E}\left\lbrack \frac{b_i}{ \mid \mid b\mid \mid } \right\rbrack \\$$

$$\\ = \sum_{i=1}^D \mathbb{E}\left\lbrack \frac{a_i}{ \mid \mid a\mid \mid } \right\rbrack^2 \\$$

Precompute $\mid \mid t^{(k)}\mid \mid$ for each data point, which takes $O(ND)$ time total. Compute $\frac{t^{(k)}_i}{ \mid \mid t^{(k)}\mid \mid }$ for each dimension of each data point, which again takes $O(ND)$ time total. Finally, average those across the data points to get all your $\mathbb{E}\left\lbrack \frac{t_i}{ \mid \mid t\mid \mid } \right\rbrack$. Square and sum the expectations to get your baseline cossim. 