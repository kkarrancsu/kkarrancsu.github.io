---
title: "Hybrid Copula Bayesian Networks"
excerpt: " "
header:
  teaser: assets/images/copula/hcbn-th.png
---

## Research Overview

Bayesian networks are a graphical representation of joint probability distribution functions.  By utilizing conditional independence relationships in the data, they can efficiently factor large dimensional joint probability distributions into products of smaller dimensional probability distributions, making them easier to estimate and perform inference over.  However, in practice, modeling the joint probability distribution of discrete and continuous random variables simultaneously is difficult.  One approach is to factorize the joint distribution into a conditional distribution of one set of outcomes and a marginal distribution of the other set.  While this is a valid approach, it suffers from the problem of identifying probability distributions for each discrete outcome.  When multiple discrete random variables are combined, the number of conditional distributions to identify explodes combinatorially.

Previous attempts at modeling them jointly include the [Conditional Linear Gaussian model](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003676), and the [Mixture of Truncated Exponential](https://link.springer.com/chapter/10.1007/3-540-44652-4_15) approaches.  In this research paper, we investigate leveraging the [Copula Bayesian Network](https://papers.nips.cc/paper/3956-copula-bayesian-networks.pdf), pioneered by Gal Elidan, and extend his original framework to allow for nodes in the Bayesian network to be either continuous or discrete random variables.

The full research paper can be found [here](http://proceedings.mlr.press/v52/karra16.pdf).
