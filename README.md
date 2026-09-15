# Computer Vision – Object Classification Challenge

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.9.1-F7931E?logo=scikitlearn&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5.0-5C3EE8?logo=opencv&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

Klasifikasi citra pakaian ke dalam 4 kelas (**Kaos, Topi, Kemeja, Celana**) dengan pendekatan machine learning klasik: ekstraksi fitur **HOG (Histogram of Oriented Gradients)** dan klasifikasi **Support Vector Machine (SVM)**. Model terbaik mencapai akurasi cross-validation **0.862 ± 0.082**.

---

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Dataset](#dataset)
- [Metodologi](#metodologi)
- [Hasil](#hasil)
- [Visualisasi](#visualisasi)
- [Struktur Repository](#struktur-repository)
- [Cara Menjalankan](#cara-menjalankan)
- [Referensi](#referensi)
- [Author](#author)

---

## Latar Belakang

Project ini dibuat untuk tugas mata kuliah **Visi Komputer** (Aktivitas Minggu 2: *Machine Learning in My Daily Life – Object Classification Challenge*). Tugasnya meminta mahasiswa mengidentifikasi masalah klasifikasi objek dari lingkungan sekitar, mengumpulkan dataset sendiri, lalu memilih dan menjustifikasi algoritma machine learning yang sesuai.

Studi kasus yang dipilih adalah **klasifikasi jenis pakaian** untuk kebutuhan katalog toko online atau toko thrift. Tujuannya agar foto produk bisa dikelompokkan otomatis ke kategori yang tepat.

## Dataset

| Kelas | Jumlah Citra |
|---|---|
| Kaos | 20 |
| Topi | 32 |
| Kemeja | 15 |
| Celana | 23 |
| **Total** | **90** |

- **Sumber**: difoto sendiri di toko **H&M** dan **OHSOME**, Galaxy Mall.
- **Format**: `.heic`, resolusi 3000×4000 piksel (rasio 3:4).
- **Bukti pengumpulan data** (screenshot galeri, foto proses pengambilan, metadata foto) ada di `Dataset/Bukti Pengumpulan Data/`.

```
Dataset/
├── Kaos/      Kaos_1.heic ... Kaos_20.heic
├── Topi/      Topi_1.heic ... Topi_32.heic
├── Kemeja/    Kemeja_1.heic ... Kemeja_15.heic
├── Celana/    Celana_1.heic ... Celana_23.heic
└── Bukti Pengumpulan Data/
```

Datasetnya tidak seimbang: kelas terkecil (Kemeja, 15 citra) kurang dari separuh kelas terbesar (Topi, 32 citra).

## Metodologi

Seluruh pipeline ada di [`Training Model/SVM_ObjectClassification.ipynb`](Training%20Model/SVM_ObjectClassification.ipynb).

```
.heic ─► RGB ─► resize 384×512 ─► center crop 20% ─► resize 192×256 ─► HOG (5940 dim)
      ─► StandardScaler ─► SVM (one-vs-one) ─► prediksi
```

### 1. Preprocessing

- Citra `.heic` dibaca dengan `pillow-heif`, orientasi EXIF diterapkan, lalu dikonversi ke RGB.
- Resize ke 384×512 dengan menjaga aspect ratio. Padding (jika diperlukan) memakai replikasi piksel tepi supaya tidak muncul tepi palsu pada HOG.
- **Center crop 20%**: 20% dibuang dari tiap sisi untuk mengurangi latar belakang toko (rak, lantai, gantungan), lalu hasilnya di-resize ke **192×256**.

### 2. Ekstraksi Fitur HOG

| Parameter | Nilai |
|---|---|
| `orientations` | 9 |
| `pixels_per_cell` | (16, 16) |
| `cells_per_block` | (2, 2) |
| `block_norm` | L2-Hys |
| Dimensi fitur | **5940** |

HOG dihitung dari citra grayscale. Sebagai pembanding, notebook juga menguji **color histogram HSV** (8×4×4 bin = 128 dimensi).

### 3. Model

- Pipeline: `StandardScaler` → `SVC(class_weight="balanced")`.
- `SVC` menangani multi-kelas dengan strategi **one-vs-one** (6 classifier biner untuk 4 kelas).
- `class_weight="balanced"` mengompensasi jumlah citra yang tidak seimbang antar kelas.
- Scaler berada di dalam pipeline, jadi hanya di-fit pada data training tiap fold.

### 4. Evaluasi & Hyperparameter Tuning

- **Nested stratified cross-validation**:
  - *Outer loop*: stratified 5-fold untuk evaluasi, **diulang 5×** dengan pengacakan berbeda (25 fold uji).
  - *Inner loop*: grid search dengan stratified 4-fold, hanya pada data training fold tersebut.
- **Grid search**: `kernel` ∈ {linear, rbf} × `C` ∈ {0.01, 0.1, 1, 10, 100}.
- Classification report dan confusion matrix dihitung dari akumulasi **450 prediksi out-of-fold** (90 citra × 5 ulangan).
- **Ablation study** membandingkan center crop (10% dan 20%), augmentasi kelas Kemeja (flip horizontal, rotasi ±10°, brightness ×0.7–1.3), dan PCA (`n_components` ∈ {0.8, 0.95}).

## Hasil

### Sebelum vs Sesudah Perbaikan

| Pipeline | Akurasi CV |
|---|---|
| Sebelum: HOG citra utuh | 0.833 ± 0.087 |
| **Sesudah: Crop 20%** | **0.862 ± 0.082** (+2.9 poin) |

### Ablation Study

| Konfigurasi | Akurasi | Std | Selisih vs baseline | F1 Kemeja |
|---|---|---|---|---|
| Baseline (tanpa crop) | 0.833 | 0.087 | +0.0 poin | 0.554 |
| Crop 10% | 0.851 | 0.097 | +1.8 poin | 0.662 |
| **Crop 20%** | **0.862** | **0.082** | **+2.9 poin** | **0.648** |
| Crop 20% + Augmentasi Kemeja | 0.784 | 0.103 | -4.9 poin | 0.577 |
| Crop 20% + PCA | 0.833 | 0.083 | +0.0 poin | 0.588 |
| Crop 20% + Augmentasi Kemeja + PCA | 0.778 | 0.089 | -5.6 poin | 0.541 |

### Perbandingan Set Fitur (pipeline awal, tanpa crop)

| Set fitur | Dimensi | Akurasi rata-rata | Std | Selisih vs HOG |
|---|---|---|---|---|
| HOG | 5940 | 0.833 | 0.087 | +0.0 poin |
| Color histogram | 128 | 0.464 | 0.077 | -36.9 poin |
| HOG + Color | 6068 | 0.836 | 0.092 | +0.2 poin |

### Hyperparameter

Hyperparameter yang terpilih di tiap outer fold (model sesudah perbaikan): `C=0.01, kernel=linear` (22×) dan `C=1, kernel=rbf` (3×).

<details>
<summary>Hasil grid search kernel dan C (fitur HOG pipeline awal, seluruh data, inner 4-fold)</summary>

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

</details>

### Classification Report (Sesudah Perbaikan: Crop 20%)

Akumulasi 450 prediksi out-of-fold:

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

Metrik per kelas, sebelum → sesudah:

| Kelas | Precision | Recall | F1-score |
|---|---|---|---|
| Kaos | 0.811 → 0.889 | 0.770 → 0.800 | 0.790 → 0.842 |
| Topi | 0.880 → 0.922 | 0.963 → 0.956 | 0.919 → 0.939 |
| Kemeja | 0.655 → 0.671 | 0.480 → 0.627 | 0.554 → 0.648 |
| Celana | 0.864 → 0.871 | 0.939 → 0.939 | 0.900 → 0.904 |

### Confusion Matrix (Sesudah Perbaikan)

Baris = kelas aktual, kolom = kelas prediksi (akumulasi 450 prediksi).

| Aktual \ Prediksi | Kaos | Topi | Kemeja | Celana |
|---|---|---|---|---|
| Kaos | 80 | 5 | 15 | 0 |
| Topi | 1 | 153 | 3 | 3 |
| Kemeja | 7 | 8 | 47 | 13 |
| Celana | 2 | 0 | 5 | 108 |

### Insight Utama

- **Kelas paling mudah dikenali: Topi** (F1 0.939). **Paling sulit: Kemeja** (F1 0.648), tetapi juga kelas yang paling terbantu oleh center crop (F1 0.554 → 0.648).
- **Pasangan kelas yang paling sering tertukar: Kaos ↔ Kemeja**, 22 kali sesudah perbaikan (Kaos→Kemeja: 15, Kemeja→Kaos: 7) dan 27 kali sebelumnya. Berikutnya Kemeja ↔ Celana (18 kali) dan Topi ↔ Kemeja (11 kali).
- **Pasangan paling mudah dipisahkan** sesudah perbaikan: Kaos ↔ Celana (2 kali tertukar).
- **Warna kurang informatif**: color histogram saja hanya mencapai 0.464, dan menambahkannya ke HOG hanya menaikkan +0.2 poin.
- **Augmentasi dan PCA tidak membantu**: augmentasi Kemeja menurunkan akurasi (-4.9 poin), dan PCA tidak memberi perubahan (+0.0 poin).
- **Kesalahan umumnya terjadi saat model ragu-ragu**: dari 62 prediksi salah sesudah perbaikan, hanya 4 (6%) yang tergolong *yakin tapi salah*. Sebelum perbaikan, angkanya 0 dari 75. Margin keyakinan diukur sebagai jarak ke batas keputusan SVM.
- **Citra yang tetap salah** sebelum maupun sesudah perbaikan: `Kaos_5`, `Kaos_13`, `Kemeja_1`, `Kemeja_8`, `Kemeja_10`.
- **Catatan**: kenaikan +2.9 poin masih di bawah standar deviasi antar fold (±0.082), dan konfigurasi terbaik dipilih dari data CV yang sama. Jadi hasil ini lebih tepat dibaca sebagai indikasi positif, belum bukti peningkatan yang pasti.

Detail lengkap, termasuk daftar semua citra yang salah diklasifikasikan beserta margin keyakinannya, ada di:
- [`Training Model/results/sebelum/results.md`](Training%20Model/results/sebelum/results.md)
- [`Training Model/results/sesudah/results.md`](Training%20Model/results/sesudah/results.md)
- [`Training Model/results/perbandingan.md`](Training%20Model/results/perbandingan.md)

## Visualisasi

### Contoh Dataset & Fitur HOG

![Contoh citra per kelas](Training%20Model/results/sample_images.png)

![Visualisasi HOG](Training%20Model/results/hog_visualization.png)

### Center Crop & Contoh Augmentasi

![Area center crop dan contoh augmentasi](Training%20Model/results/crop_augmentation_examples.png)

### Perbandingan Sebelum vs Sesudah

![Ablation study dan F1 per kelas](Training%20Model/results/perbandingan.png)

### Confusion Matrix (Sesudah Perbaikan)

![Confusion matrix sesudah perbaikan](Training%20Model/results/sesudah/confusion_matrix.png)

### Diagnostik (Sesudah Perbaikan)

| Akurasi & error rate per kelas | Breakdown kesalahan per kelas |
|---|---|
| ![Error rate per kelas](Training%20Model/results/sesudah/diag_1_error_rate_per_kelas.png) | ![Breakdown kesalahan](Training%20Model/results/sesudah/diag_2_breakdown_kesalahan.png) |

![Pengecekan feature scaling HOG](Training%20Model/results/sesudah/diag_3_scaling_hog.png)

![Keyakinan model SVM](Training%20Model/results/sesudah/diag_4_confidence.png)

### Hasil Prediksi (Sesudah Perbaikan)

Border **hijau** = selalu benar, **oranye** = kadang salah, **merah** = salah di lebih dari 2 dari 5 ulangan.

![Visualisasi prediksi sesudah perbaikan](Training%20Model/results/sesudah/prediction_visualization.png)

<details>
<summary>Visualisasi sebelum perbaikan</summary>

![Confusion matrix sebelum perbaikan](Training%20Model/results/sebelum/confusion_matrix.png)

![Error rate per kelas sebelum](Training%20Model/results/sebelum/diag_1_error_rate_per_kelas.png)

![Breakdown kesalahan sebelum](Training%20Model/results/sebelum/diag_2_breakdown_kesalahan.png)

![Scaling HOG sebelum](Training%20Model/results/sebelum/diag_3_scaling_hog.png)

![Keyakinan model sebelum](Training%20Model/results/sebelum/diag_4_confidence.png)

![Visualisasi prediksi sebelum perbaikan](Training%20Model/results/sebelum/prediction_visualization.png)

</details>

## Struktur Repository

```
SUBMISSION/
├── README.md
├── Dataset/
│   ├── Kaos/                         # 20 citra .heic
│   ├── Topi/                         # 32 citra .heic
│   ├── Kemeja/                       # 15 citra .heic
│   ├── Celana/                       # 23 citra .heic
│   └── Bukti Pengumpulan Data/       # screenshot galeri, foto pengambilan, metadata
├── Requirement/
│   └── Aktivitas Minggu 2.pdf        # deskripsi tugas
└── Training Model/
    ├── SVM_ObjectClassification.ipynb
    └── results/
        ├── sample_images.png
        ├── hog_visualization.png
        ├── crop_augmentation_examples.png
        ├── perbandingan.md
        ├── perbandingan.png
        ├── sebelum/                  # pipeline awal (tanpa crop)
        │   ├── results.md
        │   ├── confusion_matrix.png
        │   ├── prediction_visualization.png
        │   ├── diag_1_error_rate_per_kelas.png
        │   ├── diag_2_breakdown_kesalahan.png
        │   ├── diag_3_scaling_hog.png
        │   └── diag_4_confidence.png
        └── sesudah/                  # pipeline terbaik (crop 20%), isi file sama
```

## Cara Menjalankan

**1. Clone repository dan buat virtual environment** (Python 3.12)

```bash
git clone <url-repository>
cd SUBMISSION
python -m venv .venv
```

Aktifkan environment:

```bash
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
```

**2. Install dependency** (versi yang dipakai saat eksperimen)

```bash
pip install opencv-python==5.0.0.93 scikit-learn==1.9.1 scikit-image==0.26.0 pillow-heif==1.7.0 numpy==2.5.3 matplotlib==3.11.2 seaborn==0.13.2 ipykernel
python -m ipykernel install --user --name cv-svm --display-name "Python (.venv CV SVM)"
```

**3. Sesuaikan path dataset**

Notebook memakai path absolut. Ubah `BASE_DIR` di cell setup sesuai lokasi repository di komputer Anda:

```python
BASE_DIR = Path(r"C:\Semester 5\CV\SUBMISSION")
```

**4. Jalankan notebook**

Buka `Training Model/SVM_ObjectClassification.ipynb`, pilih kernel **Python (.venv CV SVM)**, lalu jalankan semua cell (*Run All*). Bisa juga lewat terminal:

```bash
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=7200 "Training Model/SVM_ObjectClassification.ipynb"
```

Satu kali run penuh memakan waktu sekitar 12 menit, karena ada 8 kali repeated nested CV untuk perbandingan fitur dan ablation. Semua output akan ditulis ulang ke `Training Model/results/`.

## Referensi

1. Dalal, N., & Triggs, B. (2005). Histograms of oriented gradients for human detection. *2005 IEEE Computer Society Conference on Computer Vision and Pattern Recognition (CVPR'05)*, 1, 886–893.
2. Cortes, C., & Vapnik, V. (1995). Support-vector networks. *Machine Learning*, 20(3), 273–297.
3. Hsu, C.-W., Chang, C.-C., & Lin, C.-J. (2003). *A practical guide to support vector classification*. Department of Computer Science, National Taiwan University.
4. Varma, S., & Simon, R. (2006). Bias in error estimation when using cross-validation for model selection. *BMC Bioinformatics*, 7, 91.
5. Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.

## Author

**&lt;Muhammad Dayyan Ghazanfar Latief&gt;**
NRP: &lt;5054241036&gt;
Program Studi: &lt;Rekayasa Kecerdasan Artifisial;&gt;

Tugas mata kuliah Visi Komputer – Aktivitas Minggu 2
