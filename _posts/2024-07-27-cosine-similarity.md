---
layout: post
title: "Dimension-Weighted Cosine Similarity"
author: Cam Benesch
meta: "Chicago"
---

Cosine similarity is a frequently used scalar metric to evaluate multi-output (i.e. vector) predictions. It's often advised that target dimensions (i.e. vector components) should be centered and scaled before computing cosine similarity, but there isn't much material on how precisely this should be done. Depending on the targets' distributions and the desired metric properties, traditional standardization may not be sufficient. 

Here I'll justify zero-centering, then briefly define our goal in scaling. I'll show how to properly scale in 2 cases: unknown target distributions (approximate scaling), and known normal target distributions (exact scaling). These methods reveal a clean way to compute dimension-weighted cosine similarity: in short, scale each target dimension by its desired weight's square root. Skip to the summary if you just want to see the recommended methods. 

## Topics
1. [Cosine similarity](#s1)
2. [Zero-centering](#s2)
3. [Scaling](#s3)
3. [Normally distributed targets](#s4)
5. [Dimension weighting](#s5)
6. [Summary](#s6)

<a name="s1"></a>

## Cosine similarity

<!-- We'll abbreviate [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) as "cossim". It's a score from -1 to 1; just the cosine of the angle between 2 vectors. Say you have a model which processes some input features, then predicts some $D$-dimensional target vector $t=[t_1,...,t_D]$. Let's call your model's prediction $y=[y_1,...,y_D]$ (others might refer to it as $\hat{t}$). 

Warning: cossim is usually not appropriate for comparing vectors. For example, suppose we have a model which predicts a device's exact movement based on noisy GPS data. There are 2 target dimensions (N/S movement and E/W movement), hence they comprise a vector. But if we're using this model to actually locate the device, then cossim is useless: it treats \[1 inch, 2 inches\] the same as \[10 miles, 20 miles\] (cossim = 1). On top of that, it treats those both very differently (in fact maximally differently) than \[-1 inch, -2 inches\] (cossim = -1). Cossim ignores magnitude, and is very sensitive to small perturbations near the origin. So if you want to model the direction the device has moved, and if you're confident it has moved far enough to withstand some noise, then go ahead and use cossim. Otherwise, leave this page and find another metric. 

If you're still here, you don't care if $y$ doesn't match $t$ exactly - all you want is for them to point in the same direction. Here's the formula for cossim:

$$\\ \cos(\theta) = \frac{t\cdot y}{||t|| ||y||} = \frac{t_1y_1+\cdots +t_Dy_D}{\sqrt{(t_1^2+\cdots +t_D^2)(y_1^2+\cdots + y_D^2)}} \\$$ -->

<a name="s2"></a>

## Zero-centering

<!-- The target vector $t$ is a draw from some $D$-dimensional distribution $f$. Our dataset contains $N$ L2-normalized iid samples $t^{(1)},...,t^{(N)}\sim f$. Now we'll justify zero-centering each of the $D$ target dimensions. 

Sample 2 target vectors $a, b$ from $t^{(1)},...,t^{(N)}$. For the sake of contradiction, suppose $\mathbb{E}\left\lbrack a\cdot b \right\rbrack > 0$. Then we can construct a naive model which at test time discards the input features, samples a random $t'$ from the training dataset's target vectors, and predicts $y=t'$. Upon evaluation, this model produces a positive average cossim $y\cdot t>0$ without even using the input features. (The $\mathbb{E}\left\lbrack a\cdot b \right\rbrack < 0$ case is similar; just set $y=-t'$.) To thwart this naive model, we thus require its cossim's numerator to be zero:

$$\\ \mathbb{E}\left\lbrack a\cdot b \right\rbrack = 0 \\$$

which by linearity of expectation gives

$$\\ \mathbb{E}\left\lbrack a_1b_1 + \cdots + a_Db_D \right\rbrack \\$$
$$\\ = \mathbb{E}\left\lbrack a_1b_1 \right\rbrack + \cdots + \mathbb{E}\left\lbrack a_Db_D \right\rbrack = 0 \\$$

and since $a,b$ are iid,

$$\\ \mathbb{E}\left\lbrack a_1 \right\rbrack \mathbb{E}\left\lbrack b_1 \right\rbrack + \cdots + \mathbb{E}\left\lbrack a_D \right\rbrack \mathbb{E}\left\lbrack b_D \right\rbrack \\$$

$$\\ = \mathbb{E}\left\lbrack a_1 \right\rbrack^2 + \cdots + \mathbb{E}\left\lbrack a_D \right\rbrack^2 = 0 \\$$

$$\\ \implies \mathbb{E}\left\lbrack a_i \right\rbrack = 0 \\$$

for each target dimension $i$. This condition is easily enforced by zero-centering the target dimensions. In particular, compute $\mu = \frac1N (t^{(1)} + \cdots + t^{(N)})$ and center the data by subtracting $\mu$ from each $t^{(k)}$.  -->

<a name="s3"></a>

## Scaling

<!-- In many cases you'll want to scale your target dimensions so that each target dimension has an equal **opportunity** to contribute to the final cossim. Note that equal opportunity doesn't mean equal contribution. If we're very good (or uncannily bad) at predicting $t_i$ but not $t_j$, then ${\mid}y_it_i{\mid} > {\mid}y_jt_j{\mid}$ in expectation. 

There are, however, scenarios where we **should** expect equal contributions. For instance, let's revisit $a, b\sim f$, which were sampled from our dataset's target vectors. Consider any 2 target dimensions $i,j$. Since $a,b$ are independently drawn, $a_i,a_j$ cannot explain any variance in $b_i,b_j$. In this case, the cossim's $i$ and $j$ terms must have the same average magnitudes. In particular, we require

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

$$\\ C_i^2 \mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack = C_j^2 \mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack \\$$

$$\\ C_i^2 \frac1{C_i^2} = C_j^2 \frac1{C_j^2} \\$$

The last step uses our approximation, under which Condition 1 is clearly true/fulfilled.  -->

<a name="s4"></a>

## Normally distributed targets

<!-- Consider the case where the target vector $t$ is a draw from $D$-dimensional multivariate normal distribution $f=\mathcal{N}(0,\Sigma)$. Of course, $f$ is zero-centered. Since $f$ is known, approximation is no longer necessary. Condition 1 is 

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack = \mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack \\$$

Each target dimension follows a univariate normal distribution $a_i\sim \mathcal{N}(0,\Sigma_{ii})$ and $a_j\sim \mathcal{N}(0,\Sigma_{jj})$. Therefore, ${\mid}a_i{\mid}, {\mid}a_j{\mid}$ each follow the [folded normal distribution](https://en.wikipedia.org/wiki/Folded_normal_distribution), whose expectation is known:

$$\\ \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack = \sqrt{\frac{2\Sigma_{ii}^2}{\pi}} \\$$

So in this case, Condition 1 is equivalent to $\Sigma_{ii} = \Sigma_{jj}$. This is easily enforced by standardizing each target dimension, which means scaling by $C = \left\lbrack \frac1{\sqrt{\Sigma_{11}}}, ..., \frac1{\sqrt{\Sigma_{DD}}} \right\rbrack$.  -->

<a name="s5"></a>

## Dimension weighting

<!-- We've been designing the scaling factor $C$ to give each target dimension an **equal** opportunity to contribute to cossim. But what if we care more about some target dimensions than others? Maybe there's an importance vector $P = \left\lbrack P_1, ..., P_D \right\rbrack$, and instead of Condition 1, we want to achieve the following:

$$\\ \frac{\mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack}{\mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack} = \frac{P_i}{P_j} \text{  (Condition 2)} \\$$

Earlier, we computed equal-weighting scaling factors $C_i$ which fulfilled Condition 1. These can be reweighted: $Q_i = C_i\sqrt{P_i}$, and you can scale by $Q$ instead of $C$. Condition 2 is now fulfilled, as shown:

$$\\ \frac{\mathbb{E}\left\lbrack {\mid}(Q_ia_i)(Q_ib_i){\mid} \right\rbrack}{\mathbb{E}\left\lbrack {\mid}(Q_ja_j)(Q_jb_j){\mid} \right\rbrack} \\$$

$$\\ = \frac{Q_i^2 \mathbb{E}\left\lbrack {\mid}a_ib_i{\mid} \right\rbrack}{Q_j^2\mathbb{E}\left\lbrack {\mid}a_jb_j{\mid} \right\rbrack} \\$$

$$\\ = \frac{P_i}{P_j} \frac{C_i^2 \mathbb{E}\left\lbrack {\mid}a_i{\mid} \right\rbrack^2}{C_j^2\mathbb{E}\left\lbrack {\mid}a_j{\mid} \right\rbrack^2} \\$$

$$\\ = \frac{P_i}{P_j} \frac{C_i^2 (1 / C_i^2)}{C_j^2 (1 / C_j^2)} = \frac{P_i}{P_j} \\$$ -->

<a name="s6"></a>

## Summary

<!-- You have a dataset containing input features, and output target vectors $t^{(1)},...,t^{(N)}$. Each sampled target vector $t^{(k)}$ is $D$-dimensional. You've also decided on importance weights $P = \left\lbrack P_1, ..., P_D \right\rbrack$ for the target dimensions.You want to compute a weighted cossim in which on average, each target dimension $i$'s contribution to the final cossim is proportional to $P_i$. 

1. Zero-center your targets by averaging $t^{(1)},...,t^{(N)}$ then subtracting that mean from each $t^{(k)}$. 
2. If you're confident $t^{(1)},...,t^{(N)}$ were drawn from a multivariate normal distribution, then standardize each dimension of each $t^{(k)}$. This means dividing each dimension by its standard deviation. 
3. If you're not confident $t^{(1)},...,t^{(N)}$ were drawn from a multivariate normal, then divide each dimension by its average absolute value $\frac1N \sum_{k=1}^N {\mid}t^{(k)}_i{\mid}$. 
4. If your importance weights $P_i$ are all equal, this step isn't necessary. Otherwise, multiply each target vector's $i$-th dimension by $\sqrt{P_i}$.  -->

<!-- Now that you've preprocessed your dataset's target vectors, you can train a model to take some input features and produce a prediction $y$ of the target vector associated with those inputs. The correct target vector from your dataset is $t$. Finally, you can compute $\cos(\theta) = \frac{t\cdot y}{{\mid}{\mid}t{\mid}{\mid} {\mid}{\mid}y{\mid}{\mid}}$, the cossim between your prediction and the true value.  -->