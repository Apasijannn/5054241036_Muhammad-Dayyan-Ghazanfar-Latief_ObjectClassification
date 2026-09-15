# Hasil Klasifikasi Pakaian - Sesudah Perbaikan (Crop 20%)

## Setup
- Konfigurasi: **Crop 20%**
- Center crop: margin 20% tiap sisi, lalu resize 192x256
- Augmentasi (jika dipakai): kelas ['Kemeja'], flip horizontal, rotasi +/-10 derajat, brightness x0.7-1.3 (45 citra tambahan, hanya dari data training tiap fold)
- PCA (jika dipakai): n_components dipilih grid search dari [0.8, 0.95] (proporsi varians)
- Model: StandardScaler + SVC (one-vs-one, class_weight=balanced); grid kernel ['linear', 'rbf'], C [0.01, 0.1, 1, 10, 100]
- Evaluasi: nested stratified 5-fold CV, diulang 5x

## Ablation Study
| Konfigurasi | Akurasi | Std | Selisih vs baseline | F1 Kemeja |
|---|---|---|---|---|
| Baseline (tanpa crop) | 0.833 | 0.087 | +0.0 poin | 0.554 |
| Crop 10% | 0.851 | 0.097 | +1.8 poin | 0.662 |
| Crop 20% | 0.862 | 0.082 | +2.9 poin | 0.648 |
| Crop 20% + Augmentasi Kemeja | 0.784 | 0.103 | -4.9 poin | 0.577 |
| Crop 20% + PCA | 0.833 | 0.083 | +0.0 poin | 0.588 |
| Crop 20% + Augmentasi Kemeja + PCA | 0.778 | 0.089 | -5.6 poin | 0.541 |

## Akurasi Cross-Validation
**0.862 +/- 0.082** (rata-rata +/- std dari 25 fold uji)

Hyperparameter terpilih di tiap outer fold: `C=0.01, kernel=linear` (22x); `C=1, kernel=rbf` (3x)

## Classification Report (akumulasi 450 prediksi out-of-fold)
```
              precision    recall  f1-score   support

        Kaos      0.889     0.800     0.842       100
        Topi      0.922     0.956     0.939       160
      Kemeja      0.671     0.627     0.648        75
      Celana      0.871     0.939     0.904       115

    accuracy                          0.862       450
   macro avg      0.838     0.831     0.833       450
weighted avg      0.860     0.862     0.860       450
```

## Confusion Matrix
Baris = kelas aktual, kolom = kelas prediksi. Gambar: `confusion_matrix.png`

| Aktual \ Prediksi | Kaos | Topi | Kemeja | Celana |
|---|---|---|---|---|
| Kaos | 80 | 5 | 15 | 0 |
| Topi | 1 | 153 | 3 | 3 |
| Kemeja | 7 | 8 | 47 | 13 |
| Celana | 2 | 0 | 5 | 108 |

## Citra yang Salah Diklasifikasikan
20 dari 90 citra pernah salah (dari 5 kali diprediksi sebagai data uji). Kolom margin = rata-rata margin keyakinan saat salah; batas yakin = 0.44 (median margin prediksi benar).

| File | Kelas asli | Diprediksi sebagai | Frekuensi salah | Margin saat salah | Kategori |
|---|---|---|---|---|---|
| Kaos_6.heic | Kaos | Kemeja (5x) | 5/5 | 0.16 | ragu-ragu |
| Kaos_13.heic | Kaos | Kemeja (5x) | 5/5 | 0.14 | ragu-ragu |
| Kemeja_7.heic | Kemeja | Celana (5x) | 5/5 | 0.10 | ragu-ragu |
| Kemeja_8.heic | Kemeja | Topi (5x) | 5/5 | 0.29 | ragu-ragu |
| Kemeja_10.heic | Kemeja | Kaos (5x) | 5/5 | 0.20 | ragu-ragu |
| Kemeja_12.heic | Kemeja | Celana (5x) | 5/5 | 0.37 | ragu-ragu |
| Kaos_5.heic | Kaos | Topi (4x) | 4/5 | 0.32 | ragu-ragu |
| Celana_17.heic | Celana | Kemeja (3x) | 3/5 | 0.06 | ragu-ragu |
| Kaos_8.heic | Kaos | Kemeja (3x) | 3/5 | 0.13 | ragu-ragu |
| Kemeja_1.heic | Kemeja | Celana (3x) | 3/5 | 0.11 | ragu-ragu |
| Kemeja_3.heic | Kemeja | Topi (3x) | 3/5 | 0.19 | ragu-ragu |
| Topi_2.heic | Topi | Kemeja (2x), Celana (1x) | 3/5 | 0.23 | ragu-ragu |
| Topi_4.heic | Topi | Celana (2x), Kemeja (1x) | 3/5 | 0.21 | ragu-ragu |
| Celana_7.heic | Celana | Kaos (2x) | 2/5 | 0.08 | ragu-ragu |
| Celana_18.heic | Celana | Kemeja (2x) | 2/5 | 0.12 | ragu-ragu |
| Kemeja_11.heic | Kemeja | Kaos (2x) | 2/5 | 0.11 | ragu-ragu |
| Kaos_3.heic | Kaos | Kemeja (1x) | 1/5 | 0.27 | ragu-ragu |
| Kaos_18.heic | Kaos | Kemeja (1x) | 1/5 | 0.01 | ragu-ragu |
| Kaos_20.heic | Kaos | Topi (1x) | 1/5 | 0.05 | ragu-ragu |
| Topi_5.heic | Topi | Kaos (1x) | 1/5 | 0.04 | ragu-ragu |

## Kelas Paling Mudah dan Paling Sulit Dipisahkan
| Kelas | Precision | Recall | F1-score |
|---|---|---|---|
| Kaos | 0.889 | 0.800 | 0.842 |
| Topi | 0.922 | 0.956 | 0.939 |
| Kemeja | 0.671 | 0.627 | 0.648 |
| Celana | 0.871 | 0.939 | 0.904 |

- Kelas paling mudah dikenali (F1 tertinggi): **Topi** (0.939)
- Kelas paling sulit dikenali (F1 terendah): **Kemeja** (0.648)

| Pasangan | Total tertukar | A -> B | B -> A |
|---|---|---|---|
| Kaos <-> Kemeja | 22 | Kaos->Kemeja: 15 | Kemeja->Kaos: 7 |
| Kemeja <-> Celana | 18 | Kemeja->Celana: 13 | Celana->Kemeja: 5 |
| Topi <-> Kemeja | 11 | Topi->Kemeja: 3 | Kemeja->Topi: 8 |
| Kaos <-> Topi | 6 | Kaos->Topi: 5 | Topi->Kaos: 1 |
| Topi <-> Celana | 3 | Topi->Celana: 3 | Celana->Topi: 0 |
| Kaos <-> Celana | 2 | Kaos->Celana: 0 | Celana->Kaos: 2 |

- Paling sulit dipisahkan: **Kaos vs Kemeja** (22 kali tertukar)
- Paling mudah dipisahkan: **Kaos vs Celana** (2 kali tertukar)

## Diagnostik Keyakinan Model
Dari 62 prediksi salah, **4 (6%)** termasuk *yakin tapi salah* (margin >= 0.44), sisanya *ragu-ragu*.

| Arah kesalahan | Jumlah | Median margin | % yakin tapi salah |
|---|---|---|---|
| Kaos -> Kemeja | 15 | 0.14 | 0% |
| Kemeja -> Celana | 13 | 0.12 | 15% |
| Kemeja -> Topi | 8 | 0.23 | 12% |
| Kemeja -> Kaos | 7 | 0.18 | 0% |
| Kaos -> Topi | 5 | 0.20 | 20% |
| Celana -> Kemeja | 5 | 0.07 | 0% |
| Topi -> Kemeja | 3 | 0.25 | 0% |
| Topi -> Celana | 3 | 0.16 | 0% |
| Celana -> Kaos | 2 | 0.08 | 0% |
| Topi -> Kaos | 1 | 0.04 | 0% |

Grafik: `diag_1_error_rate_per_kelas.png`, `diag_2_breakdown_kesalahan.png`, `diag_3_scaling_hog.png`, `diag_4_confidence.png`, `prediction_visualization.png`