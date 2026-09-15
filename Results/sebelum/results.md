# Hasil Klasifikasi Pakaian - Sebelum Perbaikan (HOG + SVM)

## Setup
- Dataset: 90 citra, Kaos 20, Topi 32, Kemeja 15, Celana 23
- Preprocessing: RGB, resize 192x256 (aspect ratio 3:4), **tanpa crop**
- HOG: {'orientations': 9, 'pixels_per_cell': (16, 16), 'cells_per_block': (2, 2), 'block_norm': 'L2-Hys'} -> 5940 dimensi
- Model: StandardScaler + SVC (one-vs-one, class_weight=balanced), tanpa augmentasi, tanpa PCA
- Grid search: kernel ['linear', 'rbf'], C [0.01, 0.1, 1, 10, 100] (inner stratified 4-fold)
- Evaluasi: nested stratified 5-fold CV, diulang 5x (25 fold uji)

## Perbandingan Set Fitur
| Set fitur | Dimensi | Akurasi rata-rata | Std | Selisih vs HOG |
|---|---|---|---|---|
| HOG | 5940 | 0.833 | 0.087 | +0.0 poin |
| Color histogram | 128 | 0.464 | 0.077 | -36.9 poin |
| HOG + Color | 6068 | 0.836 | 0.092 | +0.2 poin |

## Grid Search Kernel dan C (fitur HOG, seluruh data, inner 4-fold)
| Kernel | C | Akurasi rata-rata | Std |
|---|---|---|---|
| linear | 0.01 | 0.822 | 0.055 |
| rbf | 0.01 | 0.269 | 0.095 |
| linear | 0.1 | 0.822 | 0.055 |
| rbf | 0.1 | 0.421 | 0.057 |
| linear | 1 | 0.822 | 0.055 |
| rbf | 1 | 0.788 | 0.070 |
| linear | 10 | 0.822 | 0.055 |
| rbf | 10 | 0.766 | 0.052 |
| linear | 100 | 0.822 | 0.055 |
| rbf | 100 | 0.766 | 0.052 |

Parameter terbaik pada seluruh data: `{'svm__C': 0.01, 'svm__kernel': 'linear'}`

## Akurasi Cross-Validation
**0.833 +/- 0.087** (rata-rata +/- std dari 25 fold uji)

Hyperparameter terpilih di tiap outer fold: `C=0.01, kernel=linear` (22x); `C=1, kernel=rbf` (3x)

## Classification Report (akumulasi 450 prediksi out-of-fold)
```
              precision    recall  f1-score   support

        Kaos      0.811     0.770     0.790       100
        Topi      0.880     0.963     0.919       160
      Kemeja      0.655     0.480     0.554        75
      Celana      0.864     0.939     0.900       115

    accuracy                          0.833       450
   macro avg      0.802     0.788     0.791       450
weighted avg      0.823     0.833     0.825       450
```

## Confusion Matrix
Baris = kelas aktual, kolom = kelas prediksi. Gambar: `confusion_matrix.png`

| Aktual \ Prediksi | Kaos | Topi | Kemeja | Celana |
|---|---|---|---|---|
| Kaos | 77 | 6 | 13 | 4 |
| Topi | 2 | 154 | 4 | 0 |
| Kemeja | 14 | 12 | 36 | 13 |
| Celana | 2 | 3 | 2 | 108 |

## Citra yang Salah Diklasifikasikan
24 dari 90 citra pernah salah (dari 5 kali diprediksi sebagai data uji). Kolom margin = rata-rata margin keyakinan saat salah; batas yakin = 0.49 (median margin prediksi benar).

| File | Kelas asli | Diprediksi sebagai | Frekuensi salah | Margin saat salah | Kategori |
|---|---|---|---|---|---|
| Kaos_5.heic | Kaos | Topi (5x) | 5/5 | 0.31 | ragu-ragu |
| Kaos_13.heic | Kaos | Kemeja (4x), Topi (1x) | 5/5 | 0.06 | ragu-ragu |
| Kemeja_1.heic | Kemeja | Kaos (5x) | 5/5 | 0.17 | ragu-ragu |
| Kemeja_2.heic | Kemeja | Celana (5x) | 5/5 | 0.03 | ragu-ragu |
| Kemeja_8.heic | Kemeja | Topi (5x) | 5/5 | 0.14 | ragu-ragu |
| Kemeja_10.heic | Kemeja | Celana (5x) | 5/5 | 0.11 | ragu-ragu |
| Kemeja_11.heic | Kemeja | Kaos (5x) | 5/5 | 0.29 | ragu-ragu |
| Kemeja_15.heic | Kemeja | Topi (5x) | 5/5 | 0.21 | ragu-ragu |
| Kaos_4.heic | Kaos | Celana (4x) | 4/5 | 0.09 | ragu-ragu |
| Kaos_3.heic | Kaos | Kemeja (3x) | 3/5 | 0.14 | ragu-ragu |
| Kaos_16.heic | Kaos | Kemeja (3x) | 3/5 | 0.16 | ragu-ragu |
| Kemeja_4.heic | Kemeja | Topi (2x), Kaos (1x) | 3/5 | 0.04 | ragu-ragu |
| Kemeja_13.heic | Kemeja | Celana (3x) | 3/5 | 0.06 | ragu-ragu |
| Topi_3.heic | Topi | Kemeja (3x) | 3/5 | 0.07 | ragu-ragu |
| Celana_1.heic | Celana | Topi (2x) | 2/5 | 0.02 | ragu-ragu |
| Celana_17.heic | Celana | Kemeja (2x) | 2/5 | 0.04 | ragu-ragu |
| Celana_21.heic | Celana | Kaos (2x) | 2/5 | 0.06 | ragu-ragu |
| Kaos_18.heic | Kaos | Kemeja (2x) | 2/5 | 0.13 | ragu-ragu |
| Kemeja_6.heic | Kemeja | Kaos (2x) | 2/5 | 0.07 | ragu-ragu |
| Topi_8.heic | Topi | Kaos (2x) | 2/5 | 0.02 | ragu-ragu |
| Celana_22.heic | Celana | Topi (1x) | 1/5 | 0.08 | ragu-ragu |
| Kaos_9.heic | Kaos | Kemeja (1x) | 1/5 | 0.02 | ragu-ragu |
| Kemeja_9.heic | Kemeja | Kaos (1x) | 1/5 | 0.12 | ragu-ragu |
| Topi_2.heic | Topi | Kemeja (1x) | 1/5 | 0.04 | ragu-ragu |

## Kelas Paling Mudah dan Paling Sulit Dipisahkan
| Kelas | Precision | Recall | F1-score |
|---|---|---|---|
| Kaos | 0.811 | 0.770 | 0.790 |
| Topi | 0.880 | 0.963 | 0.919 |
| Kemeja | 0.655 | 0.480 | 0.554 |
| Celana | 0.864 | 0.939 | 0.900 |

- Kelas paling mudah dikenali (F1 tertinggi): **Topi** (0.919)
- Kelas paling sulit dikenali (F1 terendah): **Kemeja** (0.554)

| Pasangan | Total tertukar | A -> B | B -> A |
|---|---|---|---|
| Kaos <-> Kemeja | 27 | Kaos->Kemeja: 13 | Kemeja->Kaos: 14 |
| Topi <-> Kemeja | 16 | Topi->Kemeja: 4 | Kemeja->Topi: 12 |
| Kemeja <-> Celana | 15 | Kemeja->Celana: 13 | Celana->Kemeja: 2 |
| Kaos <-> Topi | 8 | Kaos->Topi: 6 | Topi->Kaos: 2 |
| Kaos <-> Celana | 6 | Kaos->Celana: 4 | Celana->Kaos: 2 |
| Topi <-> Celana | 3 | Topi->Celana: 0 | Celana->Topi: 3 |

- Paling sulit dipisahkan: **Kaos vs Kemeja** (27 kali tertukar)
- Paling mudah dipisahkan: **Topi vs Celana** (3 kali tertukar)

## Diagnostik Keyakinan Model
Dari 75 prediksi salah, **0 (0%)** termasuk *yakin tapi salah* (margin >= 0.49), sisanya *ragu-ragu*.

| Arah kesalahan | Jumlah | Median margin | % yakin tapi salah |
|---|---|---|---|
| Kemeja -> Kaos | 14 | 0.17 | 0% |
| Kaos -> Kemeja | 13 | 0.11 | 0% |
| Kemeja -> Celana | 13 | 0.04 | 0% |
| Kemeja -> Topi | 12 | 0.17 | 0% |
| Kaos -> Topi | 6 | 0.27 | 0% |
| Kaos -> Celana | 4 | 0.09 | 0% |
| Topi -> Kemeja | 4 | 0.04 | 0% |
| Celana -> Topi | 3 | 0.03 | 0% |
| Topi -> Kaos | 2 | 0.02 | 0% |
| Celana -> Kaos | 2 | 0.06 | 0% |
| Celana -> Kemeja | 2 | 0.04 | 0% |

Grafik: `diag_1_error_rate_per_kelas.png`, `diag_2_breakdown_kesalahan.png`, `diag_3_scaling_hog.png`, `diag_4_confidence.png`, `prediction_visualization.png`