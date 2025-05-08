---
layout: post
title:  "LLM Scale beyond the limit - Long context length"
date:   2025-05-07 10:39:13 -0400
categories: PIM
tags: PIM
---

{% raw %}
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML"></script>
{% endraw %}

I was asked for scaling up a system very often recently. Scaling up normally means you adds more resource for better result before you reach the limit. We also want to go beyond the limit sometime. I recently needs to understand a legacy system with 100 MB code. I cannot put it to the latest Open AI GPT 4.1 with up to 1,047,576 tokens context window. I tried several ways with GPT 4.1. Each call costs me about 1 dollar with long context length and the output is not good. Why LLM cannot scale up to long context length even I already pay for the expensive call. I will explain it by math, hardware here.


## Mathematical Background: How Attention Works

Attention weights are computed through the softmax function applied to attention scores. Specifically, given:

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}}\right) V$$

* \\(Q, K, V\\) are \\(n \times d\\) matrices.
* \\(n\\) is the context length.
* \\(d\\) is the embedding dimension.

Softmax is applied row-wise on the scaled dot-product matrix:

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{j=1}^n e^{x_j}}$$

Each attention output is thus a **weighted average** of all tokens in the context.


## Mathematical issues for long context length

| Issue                        | Mathematical Explanation                                     | Consequence                                       |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------- |
| Softmax dilution             | $$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{j=1}^n e^{x_j}}$$ | Less focused attention weights                    |
| Noise accumulation           | Signal-to-noise ratio decreases as $$O(\frac{1}{n^2})$$        | Relevant info obscured by irrelevant tokens      |
| Positional embedding degradation | Accuracy $$\sim \frac{1}{n}$$                                 | Positional info becomes less reliable            |
| Numerical instability        | Exponential scaling $$e^{QK^T}$$                             | Computational instability                         |
| Limited parameters           | Capacity per token $$\sim \frac{\text{Params}}{n}$$            | Reduced representation per token                  |

Thus, from the theoretical and mathematical perspective, increasing context length without increasing model complexity or employing specialized architectural modifications (**sparse attention**, **rotary embeddings**, **ALiBi**, **sliding window attention**) inevitably results in degraded performance.

Numerical instability already address hardware limit but the more general one is
# Precision & Numerical Stability
GPUs perform computations typically using lower-precision arithmetic (FP16, INT8) to boost performance:

-   Lower precision (FP16) arithmetic has limited dynamic range and precision.
    
-   For very large context sizes, small errors compound during attention computation and softmax (exponential functions), resulting in numerical instability.
    

**Hardware limitation**:
-   Increased context → Accumulation of small arithmetic errors → Numerical instability (overflow/underflow) → Decreased inference accuracy.

## Go beyond the limit
Google's Gemini 2.5-Pro performance excellent for long context length. One reason in my opinion that is Google's ASICs TPU goes beyond hardware limit to handle numerical operation very well with FP16 than Nvidia GPU.

However, even with Google Gemini 2.5-Pro I still cannot dump 100M code to LLM. My solution go beyond the limit here is

## Go beyond the limit with removing deduplication
My quick solution here is to removing logic duplication code based on my knowledge. Deduplication reduces the code from 100M to 50M even 20M. With prompt engineering, it's quite easy to generate a nice system architecture diagram by LLM today.



[my Resume]: https://bobbercheng.github.io/blog/resume/2024/04/07/Bobber-Resume.html
[my Github]: https://github.com/bobbercheng
[my Linkedin]: https://www.linkedin.com/in/bobbercheng/
[my Kaggle]:   https://www.kaggle.com/bobber
[my Huggingface]: https://huggingface.co/bobber
[My twitter]: https://twitter.com/bobbercheng
