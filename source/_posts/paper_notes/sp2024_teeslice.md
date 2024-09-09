---
title: Reading Notes of TEESlice (S&P 2024)
date: 2024-09-09 22:14:55
categories: [论文笔记]
tags: [神经网络, 隐私计算, 可信执行环境]
---

This paper, *No Privacy Left Outside: On the (In-)Security of TEE-Shielded DNN Partition for On-Device ML*, is from [S&P 2024](https://ieeexplore.ieee.org/document/10646815) and focuses on the problem that how to deploy your private model on a untrusted user’s device.


<!--more-->


### Overview

- **Scenario**: **Scenario**: The model provider (e.g., OpenAI) wants its client to use the model (e.g., for flower classification) but doesn't want to make the model public. Thus, although the model is deployed on the client's device, the client shouldn't be able to obtain any information about the model parameters.
- **Basic idea**: Consider an image classification model like ResNet50, which we've fine-tuned with our private data. To protect this model's proprietary nature, we can split it into two parts: one part is placed on the client's device in plaintext, while the other is stored in the device's Trusted Execution Environment (TEE). The TEE ensures that the data inside remains confidential.
- **Existing works' limitation**: Current approaches also consider this basic idea, typically following a uniform workflow of partitioning the model after fine-tuning. However, the paper highlights that some information related to the fine-tuning process leaks when the “offloaded” part is published. Consequently, the partition must occur before training.
- **What it does**: The paper proposes partitioning the model before training. To achieve this, it introduces algorithms that partition the model with minimal “slices” (private parts), thereby reducing the computation required in the TEE.

### Some observations

- The paper employs an effective writing structure, first analyzing the limitations of existing works before presenting its solution through well-defined research questions and corresponding evaluations. This approach enhances the paper's persuasiveness and serves as a valuable model for academic writing.
- Although this is a TEE scheme to solve the privacy-preserving ML problem, we can find similarities to HE/MPC hybrid schemes. The communication between TEE and GPU requires one-time pad to mask the intermediate features. If we consider the online and offline stages as a whole, a full secret forward-pass is needed for every inference. This implies that we cannot really reduce the secret computation needed for an input by protecting fewer parameters. The root cause is that these “model slicing” schemes need to handle front layers, which causes intermediate features to contain secret information.
- The approach doesn't truly “slice” the original model. Rather, it adds new modules (i.e., slices) parallel to the original backbone and combines their outputs at each intermediate layer.
- I don’t really understand why the Freivalds’ algorithm is used. It appears that the offloaded computation could be tampered with by malicious actors. Consequently, it becomes the model provider's responsibility to safeguard the entire computation process against potential attacks.
