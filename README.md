# Cats vs. Dogs Image Classification with a CNN

Project for **ST2AIM — AI and Machine Learning for IT Engineers** (EFREI, ING2).
Building, training and evaluating a convolutional neural network that classifies photographs of
cats and dogs, then improving it with two regularisation strategies.

## Contents

| File | Description |
|---|---|
| `Projet_CNN_Cats_Dogs.ipynb` | Full notebook: code, outputs and answers to every question of the brief |
| `Presentation_CNN_Cats_Dogs.pptx` | Slides for the 10-minute defence (speaker notes included) |
| `Oral_Script.md` | Timed speaking script and prepared answers to likely questions |
| `ST2AIM___Project.pdf` | The assignment brief |

## Data

`cats_and_dogs_filtered` — 2,000 training images and 1,000 validation images, perfectly balanced.
**Not versioned** (too large for GitHub). Download it and place it at the root of the project:

```
Projet/
├── Projet_CNN_Cats_Dogs.ipynb
└── cats_and_dogs_filtered/
    ├── train/       (cats/ + dogs/)
    └── validation/  (cats/ + dogs/)
```

Source: https://storage.googleapis.com/mledu-datasets/cats_and_dogs_filtered.zip

## Method

- Images resized to 150 × 150 × 3, pixels rescaled to [0, 1] inside the model
- 80/20 split of the `train` directory: 1,600 training images, 400 held-out test images
- Architecture: 4 × (`Conv2D` + ReLU + `MaxPooling2D`) with 32 / 64 / 128 / 128 filters,
  then `Flatten` → `Dense(128)` → `Dense(1, sigmoid)`
- **1,043,905 parameters**, computed by hand and verified against `model.summary()`
- Adam (1e-3), `binary_crossentropy`, 15 epochs, batch size 100 (16 batches per epoch)

## Results (400 held-out test images)

| Model | Accuracy | Precision | Recall | F1 | Train/val gap |
|---|---|---|---|---|---|
| A — baseline CNN | 0.728 | 0.839 | 0.619 | 0.712 | 0.179 |
| B — data augmentation | 0.750 | 0.730 | 0.858 | 0.789 | **0.009** |
| C — dropout + batch normalization | **0.780** | 0.766 | 0.858 | **0.810** | 0.085 |

The baseline overfits: its validation loss bottoms out at epoch 13 and rises afterwards while the
training loss keeps falling. Augmentation almost eliminates the generalisation gap; dropout combined
with batch normalization reaches the best final score. A transfer-learning bonus with a frozen
MobileNetV2 backbone is included in the notebook.

## Running the project

```bash
pip install tensorflow scikit-learn matplotlib pandas notebook
jupyter notebook Projet_CNN_Cats_Dogs.ipynb
```

A full re-run takes roughly 20 minutes on CPU.

## References

- ST2AIM lecture notes, chapters 1–5 — A. Tay, F. Chaieb, H. Kchok, A. Gabis, EFREI
- [TensorFlow / Keras — Image classification tutorial](https://www.tensorflow.org/tutorials/images/classification)
- M. Sandler et al., *MobileNetV2: Inverted Residuals and Linear Bottlenecks*, CVPR 2018
