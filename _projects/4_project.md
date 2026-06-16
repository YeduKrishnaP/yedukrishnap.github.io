---
layout: page
title: Patching the Past
description: Efficient and Reversible Federated Unlearning via Low-Rank Adaptation
img: assets/img/FedUnlearn/fedunlearn_thumb.png
importance: 1
category: work
giscus_comments: true
---

### Introduction: The "Right to be Forgotten"

In the era of big data, Federated Learning (FL) has emerged as a transformative paradigm for training robust machine learning models while preserving data privacy. By allowing decentralized clients to collaborate and contribute to learning a shared global model without exchanging raw data, FL addresses the problems of data islands. 

However, despite its privacy-preserving nature, FL faces a critical challenge: once a client contributes to the global model, their data is inherently embedded in the model’s weights. Recent legal frameworks, such as the General Data Protection Regulation (GDPR), mandate the **"Right to be Forgotten,"** granting users the authority to request the permanent erasure of their data from any trained system.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/FedUnlearn/Unlearning_Illustration.jpg" title="Machine Unlearning Illustration" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An illustration of the Machine Unlearning problem. When a request to forget specific data (the "forget set") is made, an unlearning algorithm must remove that data's influence from the pre-trained model. The gold standard is a model retrained entirely from scratch without the forget set; approximate unlearning methods are evaluated on how closely they can mimic this gold standard without the prohibitive computational cost.
</div>

In a centralized setting, this is typically achieved by retraining the model from scratch, but doing so is computationally prohibitive and practically unfeasible in a massive, decentralized FL network. This introduces the need for efficient **Federated Unlearning**.

### Abstract

Federated Learning (FL) is a recent paradigm for privacy-preserving decentralized training, but it faces significant challenges complying with the "Right to be Forgotten." Traditional Federated Unlearning methods are computationally prohibitive, often requiring full-model retraining or suffering from catastrophic forgetting and privacy leakage vulnerabilities. Furthermore, transmitting full-model updates during the unlearning process incurs massive communication overhead.

In this paper, we propose a novel and efficient federated unlearning framework leveraging Low-Rank Adaptation (LoRA). Rather than modifying the core global weights, our approach freezes the backbone and learns a modular "negative patch" through adapter layers to selectively neutralize the influence of the data targeted for deletion. We introduce a progressive three-stage unlearning strategy: 
1. Targeted gradient ascent to destroy specific class memories
2. Lightweight performance recovery on unaffected classes 
3. Dual-loss optimization phase to simultaneously preserve retained knowledge and suppress the forgotten data. 

Evaluated on the CIFAR-10 dataset, our method demonstrates rapid and stable unlearning that successfully defends against Membership Inference Attacks (MIA) while maintaining high utility on the remaining data. Furthermore, by restricting updates to low-rank matrices, our framework drastically reduces server-side aggregation costs and uniquely enables reversible unlearning by simply discarding the targeted adapters.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/FedUnlearn/framework.png" title="Conceptual Framework for Federated Machine Unlearning" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: The general conceptual framework for Federated Unlearning, demonstrating the complex interaction between clients, target forget sets, and the central aggregation server.
</div>

### Methodology

Our proposed framework treats unlearning as a parameter-efficient optimization problem where a negative patch is learned to neutralize the influence of specific data without altering the core global weights.

#### Architecture and Initialization
Let the pre-trained global model be \(f(\cdot; \Theta_G)\) with parameters \(\Theta_G\), frozen to preserve the foundational knowledge acquired during federated training. Let \(f_c\) denote the client who needs to forget \(d\%\) of class \(c\)'s data.

To facilitate unlearning, we inject LoRA layers into the weight matrices \(W \in \mathbb{R}^{d \times k}\) corresponding to the later layers of \(f_c\). The trainable parameters are hence restricted to the low-rank matrices \(\phi = \{A, B\}\), where \(A \in \mathbb{R}^{r \times k}\) and \(B \in \mathbb{R}^{d \times r}\), with the rank \(r \ll \min(d, k)\).

The modified forward pass for any adapted layer can be expressed as:
\[ h = Wx + \frac{\alpha}{r}(BA)x \]

where \(\alpha\) is a scaling factor and \(x\) is the input vector. This structure allows the model to learn an additive "interference signal" (\(BA\)) while keeping the original knowledge \(W\) intact.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/FedUnlearn/lora.png" title="Low Rank Adaptation Diagram" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Visual representation of Low-Rank Adaptation (LoRA), detailing how the pre-trained weights remain frozen while trainable rank decomposition matrices A and B are injected to capture the "negative patch" for unlearning.
</div>

#### Staged Unlearning Strategy

**Stage I: Unlearning class c**
To forget the specific information corresponding to class \(c\), we perform Gradient Ascent to destroy the exact memory of samples contained in gradients by moving along the direction that maximizes the loss corresponding to the class \(c\) whose samples are to be forgotten.
\[ \mathcal{L}_{forget-stage} = + \sum_{(x,y) \in D_c} y \log(f(x; \Theta_G, \phi)) \]

**Stage II: Performance Recovery on Other Classes**
To mitigate the catastrophic effects of Stage I on other classes due to sharing of features, we train the adapters with cross-entropy loss to minimize the following loss:
\[ \mathcal{L}_{final} = - \sum_{(x,y) \in D_{rec}} y \log(f(x; \Theta_G, \phi)) \]
where \(D_{rec} = \{x \in D_r : x \notin D_c\}\) is the set of samples from classes other than the forget class \(c\).

**Stage III: Dual loss for forgetting and recovery**
Now we utilize the original \(D_f\) and \(D_r\) forget and retain sets that were given to perform a simultaneous optimization to suppress the performance on samples in \(D_f\) that could potentially spike up.
\[ \mathcal{L}_{recovery-stage} = - \sum_{(x,y) \in D_f} y \log(f(x; \Theta_G, \phi)) + \mu \cdot \sum_{(x,y) \in D_r} y \log(f(x; \Theta_G, \phi)) \]
where \(\mu\) is annealed over time as \(\mu_{epoch+1} = 0.8 * \mu_{epoch}\).

#### Server-Side Aggregation and Integration
Once local optimization is complete, the client transmits only the optimized low-rank matrices \(\phi^* = \{A^*, B^*\}\) to the central server. For a set of \(K\) clients requesting unlearning, the server aggregates these updates:
\[ \Delta W_{avg} = \frac{1}{K} \sum_{i=1}^{K} (B_i^* A_i^*) \]

The server then integrates this aggregate update into the global weights:
\[ W_{unlearned} = W_{global} + \frac{\alpha}{r} \Delta W_{avg} \]

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <!-- PDF Viewer using an iframe -->
        <iframe src="/assets/pdf/FedUnlearn.pdf" width="100%" height="800px" style="border: 1px solid #ddd; border-radius: 8px;">
            This browser does not support PDFs. Please download the PDF to view it: <a href="/assets/pdf/FedUnlearn.pdf">Download PDF</a>.
        </iframe>
    </div>
</div>
<div class="caption">
    The complete research paper: "Patching the Past: Efficient and Reversible Federated Unlearning via Low-Rank Adaptation".
</div>