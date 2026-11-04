# MicroLung-Net v2

### An Ultra-Lightweight CNN for Lung Cancer Histopathology Classification

## Overview

MicroLung-Net v2 is a lightweight deep learning research project for classifying lung histopathology images into three categories: lung adenocarcinoma, lung squamous cell carcinoma, and benign lung tissue.

The implementation uses a custom convolutional neural network with depthwise separable convolution blocks. It investigates classification performance, Grad-CAM-based visual explanations, robustness to image perturbations, and CPU inference efficiency.

**Disclaimer:** This is an experimental research project and is not a clinically validated diagnostic system.

## Objectives

- Develop a compact CNN architecture with fewer than one million parameters.
- Classify lung histopathology images into three categories.
- Evaluate classification performance using standard metrics.
- Generate Grad-CAM visualizations to inspect image regions influencing predictions.
- Evaluate performance under selected image perturbations.
- Measure saved model size and CPU inference latency.

## Dataset

The project uses the lung subset of the LC25000 lung and colon histopathological image dataset.

**Dataset:** [Lung and Colon Cancer Histopathological Images — Kaggle](https://www.kaggle.com/datasets/andrewmvd/lung-and-colon-cancer-histopathological-images)

| Property          | Details             |
| ----------------- | ------------------- |
| Total lung images | 15,000              |
| Number of classes | 3                   |
| Images per class  | 5,000               |
| Input image size  | 224 × 224 × 3       |
| Training split    | 70% (10,500 images) |
| Validation split  | 15% (2,250 images)  |
| Test split        | 15% (2,250 images)  |

### Classification categories

- **Adenocarcinoma:** Lung adenocarcinoma histopathology images.
- **Squamous cell carcinoma:** Lung squamous cell carcinoma histopathology images.
- **Benign:** Benign lung tissue images.

## Methodology

### 1. Preprocessing and augmentation

- Decode and resize images to 224 × 224 pixels.
- Normalize pixel values to the range [0, 1].
- Apply stratified training, validation, and test splitting using a fixed random seed.
- Apply training-time augmentation using horizontal and vertical flips, 90-degree rotations, random cropping, brightness and color adjustments, and cutout masking.

Augmentation is applied to the training data, while validation and test images are processed without training-time augmentation.

### 2. Model architecture

The model is implemented in TensorFlow and Keras and uses depthwise separable convolution blocks to reduce computational requirements.

The architecture includes:

- An initial convolutional stem.
- Depthwise and pointwise convolutions.
- Batch normalization and ReLU6 activations.
- Dropout regularization.
- Global average pooling.
- A three-class softmax output layer.

The Keras model is named `MicroLung-Net-v700K`.

### 3. Training

- Optimizer: Adam.
- Initial learning rate: 0.0003.
- Batch size: 32.
- Maximum configured epochs: 50.
- Early stopping with restoration of the best model weights.
- Random seed: 42.

Training stopped early at epoch 15, and the best validation accuracy was recorded at epoch 7.

### 4. Explainable AI

Grad-CAM is used to generate heatmaps and overlays for selected test images. The notebook also includes a confidence-drop experiment that masks highly activated image regions and measures the change in the predicted class confidence.

These visualizations provide exploratory insights into model behavior; they do not establish clinical validity.

### 5. Robustness evaluation

The model is evaluated on a 300-image subset under the following conditions:

- Gaussian noise.
- Gaussian blur.
- Increased image brightness.

Accuracy and weighted F1-score are measured for each condition.

### 6. Efficiency evaluation

The notebook measures model parameter count, serialized Keras model size, and CPU inference latency. Latency is measured after 20 warm-up runs over 200 timed single-image inference runs.

## Results

The following values are reported by the executed notebook.

### Classification performance

| Metric                       | Result |
| ---------------------------- | -----: |
| Test accuracy                | 94.09% |
| Weighted precision           | 0.9417 |
| Weighted recall              | 0.9409 |
| Weighted F1-score            | 0.9409 |
| Weighted one-vs-rest ROC-AUC | 0.9897 |

The held-out test subset contains 2,250 images, with 750 images from each class.

### Model efficiency

| Metric                      |    Result |
| --------------------------- | --------: |
| Total parameters            |   731,843 |
| Saved model size            | 8.517 MiB |
| Mean CPU inference latency  |  31.33 ms |
| 95th-percentile CPU latency |  36.88 ms |

These efficiency results correspond to the environment and measurement procedure used in the notebook. Latency may vary with hardware and software configuration.

### Robustness results

The notebook reports the following results on a 300-image test subset.

| Condition                   | Accuracy | Weighted F1-score |
| --------------------------- | -------: | ----------------: |
| Clean images                |   94.67% |            0.9467 |
| Gaussian noise (σ = 0.06)   |   74.00% |            0.7357 |
| Gaussian blur (σ = 2 px)    |   91.67% |            0.9168 |
| Brightness increased by 35% |   79.00% |            0.7983 |

The results indicate that model performance decreases under some image perturbations, particularly Gaussian noise and increased brightness.

### Explainability experiment

The notebook reports an overall mean confidence drop of 0.1858 when the top 20% of Grad-CAM heatmap regions are masked in the selected samples. This is an exploratory experiment and should not be interpreted as proof that the model relies exclusively on meaningful pathological features.

## Technologies Used

- Python
- TensorFlow and Keras
- NumPy and Pandas
- Scikit-learn
- Matplotlib and Seaborn
- OpenCV

## Repository Contents

- `kaggle_MicroLung.ipynb` — executed notebook containing dataset loading, preprocessing, model training, evaluation, Grad-CAM visualizations, robustness experiments, and efficiency measurements.
- `README.md` — project documentation.

## How to Run

1. Open the notebook in Kaggle.
2. Add the LC25000 dataset linked above to the notebook.
3. Confirm that the three lung class directories are available under `/kaggle/input`.
4. Select a compatible TensorFlow environment and enable GPU acceleration if available.
5. Execute the notebook cells in order.
6. Review the generated training curves, classification report, confusion matrices, Grad-CAM visualizations, robustness results, and efficiency measurements.

The notebook uses Kaggle-specific paths and expects the dataset to be attached to the Kaggle environment. Paths must be adapted when running elsewhere.

## Limitations

- The data split is stratified at the image level. The notebook does not establish that related or augmented images from the same original source are isolated across splits.
- Potential data leakage should be investigated before drawing strong conclusions about generalization.
- Evaluation uses one dataset; external-dataset validation has not been demonstrated in this notebook.
- Robustness testing covers a limited set of synthetic perturbations.
- Grad-CAM and confidence-drop results are exploratory and do not establish clinical reliability.
- The model has not been established as suitable for clinical diagnosis.

## Future Work

- Investigate source-image relationships and potential data leakage.
- Evaluate the model with a split that prevents related source images from crossing partitions.
- Conduct external-dataset validation.
- Expand quantitative evaluation of explainability.
- Assess performance on different hardware and under additional image acquisition conditions.
