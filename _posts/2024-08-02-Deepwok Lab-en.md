---
layout:     post
title:      Machine Learning Internship
subtitle:   UROP at Deepwok Lab Imperial College
date:       2023-07-01
author:     JY
header-img: img/post-bg-desk.jpg
catalog: true
lang: en
tags:
    - Pytorch
    - Machine Learning
    - Internship
---


### Project Overview
Deep Neural Networks (DNNs) achieve high performance and energy efficiency when deployed on customized hardware accelerators. Dataflow architectures are particularly effective due to their layer-pipelined structure and scalability.

Current methods to exploit weight and activation sparsity often overlook dataflow architectures, missing opportunities for optimization. We propose Hardware-Aware Sparsity Search (HASS), a novel approach for exploiting sparsity in dataflow accelerators through software and hardware co-optimization. HASS systematically finds efficient sparsity solutions, achieving significant improvements over existing designs.

### Contribution

Most of my work is software development on [MASE](https://github.com/jianyicheng/mase-tools), or more specific, on Machop, MASE's software stack.

**[Mase (Machine-Learning Accelerator System Exploration Tools)](https://github.com/jianyicheng/mase-tools)** , provides an efficient and scalable approach for exploring accelerator systems to compute large ML models by directly mapping onto an efficient streaming accelerator system.

1. Mase provide Command Line Interface to train a model from scratch using specified dataset. To estimate efficiency of new dataflow accelerator, I integrated RepVGG models support into Mase, organized and documented detailed procedures for future developers.

2.Exploration of the DNN hardware accelerator required hardware-software co-simulation.  I helped with scripts writing that ship metadata to the hardware team, which would helps verify the integrity of the hardware implementation and could help identify discrepancies. Scripts include: 

-  **Transform Script:** Run a transformation pass on the specified model, e.g. quantization or pruning. 
-  **Search Script:** Apply a given search strategy over a search space to optimize a specified set of hardware/software metrics.

### Paper Published

The research paper is published on FPT. You can view the paper [here](https://arxiv.org/abs/2406.03088).