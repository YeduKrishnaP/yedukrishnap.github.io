---
layout: page
title: Masked Face In-Painting
description: Predict the face under the mask using a lightweight Residual Attention Conditioned UNET
img: assets/img/RACUNET/RACUNET_Thumbnail.png
importance: 3
category: work
related_publications: true
---

As face masks emerged as a quintessential accessory during the outbreak of the COVID-19 pandemic, a significant challenge arose in recreating occluded faces. Our established security systems rely heavily on facial recognition, making this an urgent problem to solve. While extensive research has been conducted in the realm of facial in-painting and reconstruction using Deep Neural Networks, the specific task of unmasking faces while preserving subtle features—like occluded emotions—remains largely under-attempted. 

During the final semester of my B.Sc Mathematics program, I tackled this challenge by developing a custom deep learning model. This work was later presented at the `NCVPRIPG 2023` conference held at `IIT Jodhpur` as part of the Student Research Symposium.

We introduced the **RAC-UNET (Residual Attention Conditional UNET)**. Unlike previous models that attempt to "blindly" in-paint missing data, our model uses explicit emotion labels as conditioning factors for generating the unmasked facial images. To achieve this, the network architecture integrates custom Residual blocks alongside Attention Gating mechanisms.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/RACUNET/RACUNET_architecture.jpg" title="RAC-UNET Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The complete architecture of the RAC-UNET. The emotion label is embedded and concatenated with the input tensor, guiding the network to reproduce accurate facial expressions.
</div>

To ensure the model did not suffer from vanishing gradients and could capture fine details, I designed custom residual blocks for both the encoding and decoding phases. Rather than relying on simple MaxPool or bilinear up-sampling, these blocks use 2D and Transposed convolutions to retain as much spatial data as possible.

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/RACUNET/UpscaleBlock.png" title="Upscale Block" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/RACUNET/DownscaleBlock.png" title="Downscale Block" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The custom Upscale (left) and Downscale (right) Residual Blocks. These blocks increase the depth of the network, allowing it to learn highly complex facial features without losing critical data during the downsampling process.
</div>

The model was initially trained from scratch on the RAFDB (Real World Affective Faces Database) with digitally imposed face masks. Training was conducted on Kaggle's P100 GPUs using the MS-SSIM loss function. The results demonstrated that the RAC-UNET could successfully predict unmasked faces while accurately interpreting the conditioning emotion.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/RACUNET/preds1.png" title="RAC-UNET Output 1" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/RACUNET/preds2.png" title="RAC-UNET Output 2" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/RACUNET/preds3.png" title="RAC-UNET Output 3" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Early outputs of the model trained exclusively on the RAFDB dataset. While successful, the limited dataset size occasionally resulted in blurry patches.
</div>

To push the model's capabilities further and prevent overfitting, the dataset was augmented with samples from the AFFECTNET database, creating a balanced dataset of 35,000 images. After training for 200 epochs, the network produced incredibly sharp images that stayed true to the original emotional intent, avoiding the uncanny valley effect.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/RACUNET/outputs_on_rafdb_affectnet.png" title="Final Outputs on RAFDB and AFFECTNET" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/RACUNET/model_comparison.png" title="Model Comparison Table" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Final predictions on the augmented dataset (Input -> Ground Truth -> Prediction). Right: The RAC-UNET significantly outperformed similar unconditional models across metrics like MS-SSIM, PSNR, VGG-Perceptual Loss, and Brisque score.
</div>

To objectively verify the model's ability to recreate emotions accurately, the outputs were evaluated using a RESNET-18 based Facial Emotion Recognition classifier. With a rapid prediction time of just 0.015 seconds per image, RAC-UNET stands as a highly viable candidate for real-time video unmasking applications in the future.

<div class="row mt-4">
    <div class="col-sm mt-3 mt-md-0 text-center">
        <!-- PDF Viewer using an iframe -->
        <iframe src="/assets/pdf/RACUNET_presentation.pdf" width="100%" height="600px" style="border: 1px solid #ddd; border-radius: 8px;">
            This browser does not support PDFs. Please download the PDF to view it: <a href="/assets/pdf/RACUNET_presentation.pdf">Download PDF</a>.
        </iframe>
    </div>
</div>
<div class="caption">
    The complete presentation from NCVPRIPG 2023: "Masked Face Completion using Residual Attention Conditional UNET".
</div>