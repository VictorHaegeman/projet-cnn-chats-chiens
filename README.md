
Source: https://storage.googleapis.com/mledu-datasets/cats_and_dogs_filtered.zip


- Resized every image to 150 × 150 × 3, pixels rescaled to [0, 1] inside the model
- Split the `train` folder 80/20: 1,600 images to train on, 400 kept for the final test
- Built 4 blocks of `Conv2D` + ReLU + `MaxPooling2D` (32 / 64 / 128 / 128 filters), then
  `Flatten` → `Dense(128)` → `Dense(1, sigmoid)`
- **1,043,905 parameters**, computed by hand and checked against `model.summary()`
- Trained with Adam (1e-3) and `binary_crossentropy`, 15 epochs, batch size 100 (16 batches per epoch)

## Results (on the 400 test images)

| Model | Accuracy | Precision | Recall | F1 | Train/val gap |
|---|---|---|---|---|---|
| A — baseline CNN | 0.728 | 0.839 | 0.619 | 0.712 | 0.179 |
| B — data augmentation | 0.750 | 0.730 | 0.858 | 0.789 | **0.009** |
| C — dropout + batch normalization | **0.780** | 0.766 | 0.858 | **0.810** | 0.085 |

The baseline overfits: the validation loss stops falling at epoch 13 and goes back up while the
training loss keeps going down. Augmentation almost removes the gap between the two curves, and
dropout with batch normalization gives the best final score.

## Running it

```bash
pip install tensorflow scikit-learn matplotlib pandas notebook
jupyter notebook Projet_CNN_Cats_Dogs.ipynb
```

Running everything again takes about 20 minutes on CPU.
