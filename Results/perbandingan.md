# Perbandingan Sebelum vs Sesudah Perbaikan

- Sebelum: HOG citra utuh -> akurasi **0.833 +/- 0.087**
- Sesudah: Crop 20% -> akurasi **0.862 +/- 0.082** (+2.9 poin)

## Ablation Study
| Konfigurasi | Akurasi | Std | Selisih vs baseline | F1 Kemeja |
|---|---|---|---|---|
| Baseline (tanpa crop) | 0.833 | 0.087 | +0.0 poin | 0.554 |
| Crop 10% | 0.851 | 0.097 | +1.8 poin | 0.662 |
| Crop 20% | 0.862 | 0.082 | +2.9 poin | 0.648 |
| Crop 20% + Augmentasi Kemeja | 0.784 | 0.103 | -4.9 poin | 0.577 |
| Crop 20% + PCA | 0.833 | 0.083 | +0.0 poin | 0.588 |
| Crop 20% + Augmentasi Kemeja + PCA | 0.778 | 0.089 | -5.6 poin | 0.541 |

## Metrik per Kelas (sebelum -> sesudah)
| Kelas | Precision | Recall | F1-score |
|---|---|---|---|
| Kaos | 0.811 -> 0.889 | 0.770 -> 0.800 | 0.790 -> 0.842 |
| Topi | 0.880 -> 0.922 | 0.963 -> 0.956 | 0.919 -> 0.939 |
| Kemeja | 0.655 -> 0.671 | 0.480 -> 0.627 | 0.554 -> 0.648 |
| Celana | 0.864 -> 0.871 | 0.939 -> 0.939 | 0.900 -> 0.904 |

## Perubahan Citra (mayoritas salah = salah > 2 dari 5 ulangan)
- Diperbaiki (salah -> benar), 9 citra: Kaos_3.heic (Kaos), Kaos_4.heic (Kaos), Kaos_16.heic (Kaos), Topi_3.heic (Topi), Kemeja_2.heic (Kemeja), Kemeja_4.heic (Kemeja), Kemeja_11.heic (Kemeja), Kemeja_13.heic (Kemeja), Kemeja_15.heic (Kemeja)
- Jadi salah (benar -> salah), 8 citra: Kaos_6.heic (Kaos, jadi Kemeja), Kaos_8.heic (Kaos, jadi Kemeja), Topi_2.heic (Topi, jadi Kemeja), Topi_4.heic (Topi, jadi Celana), Kemeja_3.heic (Kemeja, jadi Topi), Kemeja_7.heic (Kemeja, jadi Celana), Kemeja_12.heic (Kemeja, jadi Celana), Celana_17.heic (Celana, jadi Kemeja)
- Tetap salah, 5 citra: Kaos_5.heic (Kaos), Kaos_13.heic (Kaos), Kemeja_1.heic (Kemeja), Kemeja_8.heic (Kemeja), Kemeja_10.heic (Kemeja)

Detail: `sebelum/results.md` dan `sesudah/results.md`. Grafik: `perbandingan.png`