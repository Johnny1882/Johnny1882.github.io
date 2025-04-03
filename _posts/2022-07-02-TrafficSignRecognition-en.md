---
layout:     post
title:      Traffic Sign Recognition Research Project
subtitle:   2022 Summer Research
date:       2022-07-02
author:     JY
header-img: img/post-bg-github-cup.jpg
catalog: true
lang: en
tags:
    - Computer Vision
    - Machine Learning
    - Project
---

In this research project, Our contribution include:
1. Done extensive research and organize existing works on Traffic Sign Recognication task
2. Demonstrate the feasibility of state-of-the-art Algorithm.
3. Evaluate the effectiveness of PCA dimensionality reduction method in the state-of-the-art Algorithm.

The published paper can be viewed [here](https://www.ewadirect.com/proceedings/ace/article/view/4452)

My contribution mainly include Literature Review, including Detection Algorithms and Classification Algorithms. Here's a concise overview:

Traffic Sign Detection Algorithm include Region of Interest Detection, and Classification.

Detection:

## Detection

### Color-Based Detection
- Color is an easy-to-extract feature but can be influenced by illumination and weather.
- Euclidean distance in RGB space is used to extract colors.
- Other color spaces like HSI, HSV, and YCbCr are employed to mitigate illumination effects.
- White balance is applied to improve illumination conditions.

### Shape-Based Detection
- Shape-based methods are less sensitive to illumination.
- Hough transform detects pixel relationships to identify shapes.
- Radial symmetry transformation is used to detect shapes by analyzing gradients and magnitudes.

### Hybrid-Based Detection
- To reduce inaccuracies caused by similar background objects, both color and shape methods are combined.
- RGB ratios and Douglas-Peucker algorithm are used for segmentation and shape identification.

### Deep Learning Detection
- Deep learning methods offer high accuracy but require significant data and resources.
- Methods like SVM, FCN, and Faster R-CNN have been applied but with trade-offs in speed and accuracy.

## Classification

### Based on Color and Shape
- Traffic sign classification can be done by extracting color and shapes.
- Circularity and Distance to Border (DtB) vectors, classified using SVM, are effective for shape identification.

### Based on Deep Learning
- Deep learning methods like SVM, CNN, and radial-based neural networks are widely used for traffic sign classification.

## State-of-the-Art
- A combination of HOG and color histogram features is extracted.
- PCA reduces dimensionality, speeding up the process.
- SVM classifies traffic signs, balancing accuracy and efficiency without relying on deep learning.
