---
published: true
title: SoundMap, assistive device for blind and visually impaired
collection: ml
layout: single
author_profile: true
read_time: true
categories: [project]
header :
    teaser : "https://maelfabien.github.io/assets/images/proj_24.png"
comments : true
toc: true
toc_sticky: true
sidebar:
    nav: sidebar-sample
---

My team recently won the [International Create Challenge](https://www.createchallenge.org/) (ICC) in Martigny. We won both the AI 1st prize and the AI healthcare award for a total of 7'000 CHF, with a project called SoundMap. We made a small website that explains the solution: http://soundmap.ml/

# What is SoundMap

SoundMap is a smart wearable belt, equipped with a camera, able to provide real-time information on the surrounding environment of a person through Audio Augmented Reality (Audio AR).

Similarly to the way we easily identify the position of whistling birds, we aim to scan in real-time the surrounding environment and produce directional sounds (through audio AR) in the earphones connected to the device. Blind and visually impaired people are, therefore, able to map and understand their environment.

We aim to improve this prototype over the course of the next months, and we are looking for interested partners (universities, associations, blind or visually impaired).

![](https://maelfabien.github.io/assets/images/soundmap1.jpg)

![](https://maelfabien.github.io/assets/images/soundmap2.jpg)

# How does it work?

1. Object Recognition

Our object recognition technologies relies on deep-learning to identify more than 20 classes of objects like chairs, tables, benches, bikes, cars... We process the images in real-time, up to 25 frames per second, using optimized hardware, a Raspberry Pi, and a simple camera. We estimate distance of objects, and their direction.