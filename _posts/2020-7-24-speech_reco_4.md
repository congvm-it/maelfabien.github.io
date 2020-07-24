---
published: true
title: Speaker Adaptation
collection: ml
layout: single
author_profile: true
read_time: true
categories: [machinelearning]
excerpt : "Speech Processing"
header :
    overlay_image: "https://maelfabien.github.io/assets/images/lgen_head.png"
    teaser : "https://maelfabien.github.io/assets/images/wolf.jpg"
comments : true
toc: true
toc_sticky: true
sidebar:
    nav: sidebar-sample
---

<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

When training an ASR system, mismatch can occur between training and test data. In 

# Sources of variation

## Acoustic model

Lots of speaker-specific variations exist in the **acoustic model**:
- speaking styles
- accents
- speech poduction anatomy

There are also variations not related to the speaker itself, but to the channel (telephone, distance to microphone, reverberation...).

## Pronunciation model

Each speaker has a specific way to pronounce some words.

## Language model

Test data might be related to a specific topic, or to user-specific documents. The language model should then be adapted.

To handle all of these potential variations, we perform **speaker adaptation**. The aim is to reduce the mismatch between test data and trained models.

ASR Systems can be said to be:
- Speaker Independent (SI), which does not require too much data for each speaker, but has high WER
- Speaker Dependent (SD), which requires a lot of data per speaker
- Speaker Adaptative, that builds on top of an SI system and requires only a fraction of speaker-specific training data from an SD system

# Speaker Adaptation (SA)

Adaptation can be:
- supervised, if word level transcriptions of adaptation data is known
- unsupervised, if no transcripts are available

It can also be:
- static, if we adapt on adaptation data in a block
- dynamic, if we adapt incrementally

Finally, we want SA to be:
- compact, few speaker-dependent parameters
- efficient, requires low computational resources
- flexible, applicable to different models

## Model-based adaptation

We can adapt the parameters of acoustic models to better match the observed data, using:
- Maximum A Posteriori (MAP) adaptation of HMM/GMM
- Maximum Likelihood Linear Regression (MLLR) of GMMs
- Learnung Hidden Unit Contributions (LHUC) for NNs

### **MAP training of GMMs**

MAP training will balance the estimated parameters of the Speaker Independent data and the new adaptation data.

Using Maximum Likelihood Estimate (MLE) os Speaker Independent model, the mth Gaussian in the jth state has the following mean:

$$ \mu_{mj} = \frac{\sum_n \gamma_{jm}(n) x_n}{\sum_n \gamma_{jm}(n)} $$

Where $$ \gamma_{jm} $$ is the component occupation probability. The aim of a MLE is to maximize: $$ P(X \mid \lambda) $$ where $$ \lambda $$ are the model parameters.

The MAP estimate of the adapted model uses the Speaker Independent models as a prior probability distribution over model parameters to estimate speaker-specific data. MAP training maximizes: $$ P(\lambda \mid X) ∝ P(X \mid \lambda) P_0(\lambda) $$

It gives a new mean:

$$ \hat{\mu} = \frac{ \tau \mu_0 + \sum_n \gamma(n)x_n} {\tau + \sum_n \gamma(n)} $$

Where:
- $$ \tau $$ is a weight factor for the weight of the SI estimate (between 0 and 20 typically)
- $$ x_n $$ is the adaptation vector at time $$ n $$
- $$ \gamma(n) $$ is the probability of this Gaussian at this time
- $$ \mu_0 $$ is the prior mean

As the amount of training data increases, the MAP estimate converges to the ML one.

### **Maximum Likelihood Linear Regression (MLLR)**

One of the issues with MAP is that with few adaptation data, most Gaussians will not be adapted. If instead we share the adaptation across the Gaussians, each adaptation data can affect many of, or all, the Gaussians.

As its name suggests, MLLR will apply a linear transformation on the Gaussian parameters estimated by MLE:

$$ \hat{\mu} = A \mu + b $$

Hence, if the observations have $$ d $$ -dimensions, then A is $$d \times d $$ and b has $$ d $$ dimensions.


![image](https://maelfabien.github.io/assets/images/asr_32.png)

# Conclusion

If you want to improve this article or have a question, feel free to leave a comment below :)

References:
- [ASR 09, University of Edimburgh](http://www.inf.ed.ac.uk/teaching/courses/asr/2019-20/asr09-lvcsr.pdf)
- [Steve Renals course on Speaker Adaptation](https://www.inf.ed.ac.uk/teaching/courses/asr/2008-9/asr-adapt-1x2.pdf)
- [Weighted Finite State Transducers in Automatic Speech Recognition - ZRE lecture](https://www.cs.brandeis.edu/~cs136a/CS136a_Slides/zre_lecture_asr_wfst.pdf)