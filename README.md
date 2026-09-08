# Classification d'images chats / chiens avec un CNN

Projet du cours **ST2AIM — AI and Machine Learning for IT Engineers** (EFREI, ING2).
Construction, entraînement et évaluation d'un réseau de neurones convolutif (CNN) qui
distingue une photo de chat d'une photo de chien.

## Contenu

| Fichier | Description |
|---|---|
| `Projet_CNN_Chats_Chiens.ipynb` | Le notebook complet, avec le code, les sorties et les commentaires |
| `Presentation_CNN_Chats_Chiens.pptx` | Support de la soutenance (10 min) |
| `ST2AIM___Project.pdf` | Le sujet du projet |

## Données

Dataset `cats_and_dogs_filtered` (2000 images d'entraînement, 1000 de validation),
**non versionné** car trop lourd pour GitHub. À télécharger et à placer à la racine du projet :

```
Projet/
├── Projet_CNN_Chats_Chiens.ipynb
└── cats_and_dogs_filtered/
    ├── train/       (cats/ + dogs/)
    └── validation/  (cats/ + dogs/)
```

Source : https://storage.googleapis.com/mledu-datasets/cats_and_dogs_filtered.zip

## Méthode

- Images redimensionnées en 150 × 150 × 3, pixels normalisés dans [0, 1]
- Split 80/20 du dossier `train` : 1600 images d'entraînement, 400 de test
- Architecture : 4 blocs `Conv2D` + `MaxPooling2D` (32 / 64 / 128 / 128 filtres, ReLU),
  puis `Flatten` → `Dense(128)` → `Dense(1, sigmoid)`
- **1 043 905 paramètres**, calculés à la main puis vérifiés avec `model.summary()`
- Optimiseur Adam (1e-3), perte `binary_crossentropy`, 15 epochs, batchs de 100

## Résultats (sur les 400 images de test)

| Modèle | Accuracy | F1-score | Écart train / validation |
|---|---|---|---|
| A — CNN simple | 0,728 | 0,712 | 0,179 (sur-apprentissage net) |
| B — data augmentation | 0,750 | 0,789 | 0,009 |
| C — dropout + batch normalization | **0,780** | **0,810** | 0,085 |

Le modèle de base sur-apprend clairement. La data augmentation supprime presque
totalement l'écart entre les deux courbes ; le duo dropout + batch normalization
donne la meilleure accuracy finale.

Un bonus de *transfer learning* avec MobileNetV2 (backbone gelé) est inclus dans le notebook.

## Lancer le projet

```bash
pip install tensorflow scikit-learn matplotlib pandas notebook
jupyter notebook Projet_CNN_Chats_Chiens.ipynb
```

Comptez une vingtaine de minutes pour réexécuter l'ensemble sur un CPU.

## Sources

- Supports de cours ST2AIM (C1 à C5) — A. Tay, F. Chaieb, H. Kchok, A. Gabis, EFREI
- [Documentation TensorFlow / Keras](https://www.tensorflow.org/tutorials/images/classification)
- Sandler et al., *MobileNetV2: Inverted Residuals and Linear Bottlenecks*, CVPR 2018
