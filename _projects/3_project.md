---
layout: page
title: ClassifyVista
description: Automated Detection and Classification of Gastrointestinal Bleeding in WCE Videos
img: assets/img/Health_and_AI/ClassifyVista.png
importance: 4
category: work
---

### Project Description

Gastrointestinal (GI) bleeding is a critical condition that encompasses a wide range of diseases, from acute to chronic conditions, and represents a major cause of morbidity and mortality worldwide. The conventional diagnostic approach using endoscopy, such as Wireless Capsule Endoscopy (WCE), requires gastroenterologists to manually analyze videos captured by the capsule, a process that can be time-consuming and prone to human error. 

To address this, we introduce **ClassifyViStA**, a novel AI framework that combines classification, attention, and segmentation to automate the detection and classification of GI bleeding in WCE images.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Health_and_AI/classifyvista_model.png" title="ClassifyVista Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Architecture of the proposed model. The framework leverages a multibranch architecture with an ensemble-based classification backbone, augmented by implicit attention and segmentation branches.
</div>

### Methodology and Architecture

The framework is designed with two primary goals: 
1. To improve classification performance by leveraging an ensemble of `ResNet18` and `VGG16` models.
2. To provide interpretable predictions using segmentation masks that identify the bleeding regions in the images.

The backbone of ClassifyViStA comprises two classification models working in an ensemble. Both models process the input WCE images independently, and their prediction probabilities are averaged to provide the final classification output. The use of two diverse architectures ensures complementary feature extraction, enhancing the robustness of the classification.

To complement classification and provide interpretability, a segmentation branch utilizing a `U-Net` style decoder generates segmentation masks from encoder outputs. These masks act as explanations for classification decisions, mimicking the diagnostic reasoning of medical experts who focus on specific regions to identify bleeding.

Additionally, we enhance the detection of bleeding regions by applying Soft Non-Maximum Suppression (Soft-NMS) to `YOLOv8`, improving the accuracy of overlapping box handling. This minimizes the risk of discarding partially correct detections, improving precision and recall on the validation set.

### Results and Interpretability

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Health_and_AI/classifyvista_1.png" title="Classification and Detection Test Sets" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Classification and Detection results from Test-Set 1 (left) and Test-Set 2 (right). The predicted bounding boxes with confidence scores are visually depicted, highlighting YOLOv8's ability to localize bleeding regions.
</div>

The classification performance of the ClassifyViStA framework on the validation set demonstrated high accuracy, precision, recall, and F1-score (all hitting 0.9962), underscoring the robustness of the ensemble-based classification approach. The detection performance using YOLOv8 with Soft-NMS achieved a highly competitive Average Precision (0.7715) and Average IoU (0.6405).

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Health_and_AI/classifyvista_2.png" title="Classification, Detection, and Interpretability Plots" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Comprehensive evaluation plots. (a) Ground truth bounding boxes alongside predicted bounding boxes and classification confidence. (b) Interpretability plots demonstrating the predicted segmentation masks used for intrinsic explainability.
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Health_and_AI/classifyvista_3.png" title="Interpretability Plots from Test Sets" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5 and 6: Interpretability Plots from Test-Set 1 and Test-Set 2. Instead of relying on external explainability methods like LIME or SHAP, ClassifyViStA leverages its segmentation branch for intrinsic explainability, providing a visual explanation of the classification outcome.
</div>

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <!-- PDF Viewer using an iframe -->
        <iframe src="/assets/pdf/ClassifyVista.pdf" width="100%" height="600px" style="border: 1px solid #ddd; border-radius: 8px;">
            This browser does not support PDFs. Please download the PDF to view it: <a href="/assets/pdf/ClassifyVista.pdf">Download PDF</a>.
        </iframe>
    </div>
</div>
<div class="caption">
    The complete research paper detailing the development, architecture, and validation of the ClassifyViStA framework.
</div>