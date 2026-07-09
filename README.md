# Predictive Maintenance - Capstone Project
Project ini merupakan sistem Predictive Maintenance yang dikembangkan sebagai proyek akhir (Capstone Project) pada program Asah. Tujuan utama dari proyek ini adalah **memprediksi kemungkinan terjadinya kegagalan (failure) pada mesin** industri serta **mengestimasi sisa waktu operasional sebelum kerusakan terjadi (Remaining Useful Life).**

## 📊 Dataset & Data Enrichment

untuk memprediksi kegagalan mesin menggunakan pendekatan *machine learning*. Dataset awal diperoleh dari Kaggle, kemudian diperkaya dengan fitur tambahan berupa estimasi waktu operasional mesin (*Prediksi_Waktu*) untuk meningkatkan akurasi prediksi.

### 1. Dataset Awal (Kaggle)
Dataset yang digunakan adalah **"Machine Predictive Maintenance Classification"** dari Kaggle dengan file `predictive_maintenance.csv`. Dataset ini memiliki **10.000 sampel** dengan fitur-fitur operasional mesin seperti:

- `Air temperature [K]`
- `Process temperature [K]`
- `Rotational speed [rpm]`
- `Torque [Nm]`
- `Tool wear [min]`

Fitur target:
- `Target` : 0 = Normal, 1 = Gagal
- `Failure Type` : Tipe kegagalan (*Power Failure*, *Overstrain Failure*, dll.)

### 2. Data Enrichment (`data_enrichment.ipynb`)
Sebelum masuk ke tahap pemodelan, dilakukan proses **pengayaan data** dengan menambahkan fitur baru yang sangat krusial yaitu:

#### ➕ `Prediksi_Waktu`
- **Metode**: Menggunakan fungsi logika berbasis kondisi parameter mesin. Jika mesin berada pada kondisi ekstrem (suhu terlalu tinggi, torsi besar, atau *tool wear* hampir habis), maka sisa waktu operasional akan diproyeksikan mengecil.
- **Standarisasi**: Nilai distandarisasi dengan penambahan sedikit efek *noise* alami agar data berdistribusi normal.
- **Hasil**: Dataset baru disimpan sebagai `predictive_maintenance_enriched_dataset.csv` dan siap digunakan untuk pemodelan.

---

## 🤖 Pengembangan Model Machine Learning (`predictive_maintenance_model.ipynb`)

Pada tahap ini, dilakukan langkah-langkah berikut:
- **Exploratory Data Analysis (EDA)**
- **Penanganan Missing Value** (0 data kosong)
- **Pengecekan Duplikat**
- **Pemisahan Data Latih & Uji**

### Model yang Digunakan (2 Algoritma Klasifikasi & 1 Algoritma Regresi):

| No | Model | Deskripsi |
|----|-------|-----------|
| 1  | **XGBoost Classifier** | Model ini berfungsi sebagai gerbang utama untuk mendeteksi apakah suatu mesin berada dalam kondisi normal atau berpotensi mengalami kegagalan fungsi berdasarkan parameter operasional saat itu. |
| 2  | **XGBoost Classifier** | Ketika Model 1 mendeteksi adanya kegagalan (Target = 1), Model 2 ini bertugas mengidentifikasi akar penyebabnya (root cause) secara spesifik dengan mengklasifikasikan tipe kerusakan teknis yang sedang terjadi pada mesin. |
| 3  | **XGBoost Regressor** | Berbeda dengan dua model klasifikasi sebelumnya, model ketiga ini bekerja pada ranah regresi untuk memprediksi nilai kontinu dari fitur hasil data enrichment (Prediksi_Waktu). Model ini memberikan estimasi kuantitatif mengenai berapa hari sisa waktu operasional yang aman bagi mesin (Remaining Useful Life) sebelum kerusakan fatal benar-benar terjadi, berdasarkan tren beban kerja teknisnya. |

> *Catatan: Nama model dapat disesuaikan dengan implementasi akhir di notebook pemodelan.*

### Evaluasi Model
Metrik yang digunakan untuk mengevaluasi performa ketiga model:
- **Akurasi**
- **F1-Score**
- **Precision**
- **Confusion Matrix** 

---

## 📂 Struktur Repositori
```

├── data/
│ ├── predictive_maintenance.csv # Dataset mentah dari Kaggle
│ └── predictive_maintenance_enriched.csv # Dataset hasil pengayaan
├── notebooks/
│ ├── data_enrichment.ipynb # Pembuatan fitur Prediksi_Waktu
│ └── predictive_maintenance_model.ipynb # EDA, pelatihan, & evaluasi model
└── README.md # Dokumentasi proyek (file ini)
```

---

## 🚀 Deployment & API Testing

Model yang telah dilatih juga disiapkan untuk diintegrasikan dengan aplikasi lain melalui **REST API** yang di-*host* di **Hugging Face Spaces**.

### Contoh Pemanggilan API dengan Python:

```python
import requests

api_url = "https://reihann321-predictive-maintenance-api.hf.space/predict"
data = {
    "Type": "M",
    "Air_temperature_K": 298.0,
    "Process_temperature_K": 230,
    "Rotational speed_rpm": 1500,
    "Torque_Nm": 45.0,
    "Tool_wear_min": 180
}

response = requests.post(api_url, json=data)
result = response.json()
print(result)
