---
layout: post
title: Markov Chains and Text Mining 
---

## **Markov Chains applied on Reddit subreddits**

Project: [Markov Chains](https://github.com/alicjawil/Reddit-Markov-Chains)


Abstract

The primary objective of this project was to provide a visual representation of a text data from most popular topis on Reddit.
Most frequent and prominent terms are examined to find possible inherent trends within text data.
[Automated dashboard](https://github.com/alicjawil/Reddit-Markov-Chains/blob/master/webapp.md)
is created to provide easy access to the wordcloud and to the text generator.

Markov Chain usage can be seen in various places, for example Google Page Rank was founded on Markov Chains. General idea of 
Markov Chains is that there are mathematical systems that calculate probabilities of one state hopping to another state.

We follow a HMMs approach. Hidden Markov Model(HMM) is found on joint probability maximization. Its foundation is based on the Bayes 
probability rule. HMM is a generative approach that learns the data by estimating P(x|y) which we use to to estimate P(y|x).
Since it belong to Naive Bayes approaches, its main disadvantage is that it assumes independance of each word from its context.
HMMs assumes that each state s is dependant on previous state s-1, and that each observation word x depends on current state s-1.

Project improvement: explore CRFs(Conditional Random Fields). 



