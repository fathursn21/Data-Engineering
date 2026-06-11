# DATA ENGINEERING  
# Proyek : Pengaruh Bahan Material Bangunan Terhadap Tingkat kerusakan Rumah Pasca Gempa Bumi di Indonesia

---

## Kontributor

| Nama Lengkap                       | NIM         | Peran                |
|------------------------------------|-------------|----------------------|
| Fathur Roshin Haddinanta                   | 244311014   | Project Manager        |
| Kevin Rafael Suryatmoko                      | 244311017   | Data Engineer          |
| Muhammad Rifqi Maulana                | 244311018   | Data Analyst        |

---

## Deskripsi Proyek  
Project ini di kembangkan untuk untuk memprediksi tingkat kerusakan infrastruktur bangunan berdasarkan karakteristik struktur fisik bangunan

---

## Manfaat Data / Use Case  
- **Tujuan Proyek:** Menyediakan data terintegrasi yang menggambarkan pola hubungan antara titik, magnitudo, kedalaman dan jarak gempa dari titik pusat kabupaten terhadapa bahan material infrastuktur bangunan sebagai tolak ukur dampak bencana gempa bumi di Indonesia.
- **Manfaat:**  
  - Menyediakan sumber data yang telah melalui proses validasi dan transformasi, sehingga siap digunakan untuk studi lanjutan.  
  - Membuka peluang bagi pengembang teknologi prediktif, seperti model machine learning untuk mitigasi kerusakan infrastruktur bangunan pasca bencana gempa bumi
  - Hasil ETL proyek ini mendukung dashboard visualisasi sebagai upaya mengurangi tingkat kerusakan infrastuktur bangunan pasca gempa bumi di lingkungan yang sering terkena dampak bencana gempa bumi

---

## Serving Analisis  
Data hasil ETL disimpan dalam format .sql yang terstruktur dan dapat diakses untuk eksplorasi lanjutan menggunakan Google Colab atau visualisasi interaktif melalui Looker Studio / Data Studio. Penyimpanan ini memungkinkan analisis korelasi, distribusi wilayah yang sering terkena dampak bencan gempa bumi, dan pembuatan dashboard peta lingkungan.

## Serving Machine Learning  
Dataset bersih digunakan untuk membangun model prediksi tingkat kerusakan menggunakan beberapa algoritma machine learning. proyek ini mengimplementasikan model Machine Learning berbasis Random Forest Regressor untuk memprediksi tingkat kerusakan rumah dan fasilitas umum pasca gempa bumi.

---

# Pipeline
## Extract ( Pengambilan Data ) 
- **Sumber Data:**  
  - Data Kerusakan Infrastuktur – BNPB
    (https://gis.bnpb.go.id/) 
  - Data Gempa – USGS  
    (https://earthquake.usgs.gov/earthquakes/search/) 
  - Data Titik Pusat Kabupaten - Kaggle
    (https://www.kaggle.com/datasets/ayemnadia/latitude-longitude-kabupatenkota-indonesia?resource=download)

- **Metode Pengambilan:**  
**Integrasi Data Spasial & Seismik (File Processing):**  
    - Mengambil data gempa dari USGS dalam format CSV, kemudian dilakukan pemfilteran berdasarkan kriteria geografis (wilayah Indonesia) dan rentang waktu
    - Data kerusakan dari BNPB diekstraksi dari file Excel yang mencakup informasi detail mengenai jumlah rumah rusak dan fasilitas umum terdampak
    - Menggunakan dataset pusat kabupaten dari Kaggle sebagai acuan 

---

## Transform ( Pembersihan & Transformasi )   
- **Pembersihan:**  
  - Menghapus kolom dan baris kosong (`dropna()`)
  - Mengonversi kolom tanggal ke format datetime
  - Memastikan kolom numerik (magnitudo, kedalaman, jumlah rumah/fasum rusak) menggunakan pd.to_numeric()


- **Transformasi:**  
  - : Menggabungkan tiga dataset `Gempa`, `Kerusakan Bencana`, dan `Koordinat Pusat Kabupaten` menjadi satu dataframe menggunakan `pd.merge()` berdasarkan `ID Kabupaten` dan `tanggal`
  - Menghitung `jarak_ke_kabupaten_km` menggunakan rumus Haversine untuk mengukur jarak spasial antara titik episenter gempa dengan titik pusat koordinat kabupaten sebagai prediktor utama tingkat kerusakan
  - Menerapkan teknik Oversampling dengan penambahan distribusi normal untuk memperluas variasi dataset latihan, guna mengatasi keterbatasan data historis yang tersedia dan meningkatkan kemampuan generalisasi model Random Forest

---

## Load ( Pemindahan ke Target ) 
- **Target:**  
  - Sebuah tabel baru di dalam database pada server Aiven. Tabel ini merupakan output utama yang dapat diakses oleh layanan lain untuk melakukan analisis langsung di database.

- **Metode:**  
  - Fungsi to_sql() dari pandas digunakan untuk menulis data dari DataFrame langsung ke tabel di database PostgreSQL
  - konfigurasi fungsi to_sql() diatur dengan parameter-parameter kunci:
    - name diisi dengan yang mendefinisikan nama tabel tujuan
    - con diisi dengan variabel engine, yaitu objek koneksi dari SQLAlchemy
      yang telah dikonfigurasi sebelumnya untuk terhubung ke database Aiven
  - Data diverifikasi dengan membaca 5 baris pertama dari tabel baru tersebut menggunakan pd.read_sql() dan df.head()

---

## Arsitektur / Workflow ETL  
- **Alur Modular:**  
  - Proses ETL diringkas dalam sebuah fungsi transformasi() yang mencakup langkah-langkah membaca, membersihkan, mengisi, menggabungkan, dan mengubah data.
  -  Kode diorganisir secara sekuensial di dalam notebook Google Colab.

- **Tools yang Digunakan:**  
  - Python 3.x
  - Library: `pandas`, `numpy`, `sqlalchemy`, `pymysql`, `folium`, `sklearn`, `scipy`
  - Platform: Google Colab

---

## Kode Program  
- **Struktur Kode:**  
  - Terdapat 2 notebook: satu untuk ETL, satu untuk Machine Learning
  - Nama variabel dan fungsi deskriptif: `pipeline()`, `fetch_real_data()`, `hitung_jarak_spasial()`, k`alkulasi_dampak_realistis()`, dll.
    
- **Machine Learning:**  
  - Model utama: Random Forest Regressor untuk memprediksi jumlah unit rumah rusak
  - Data Augmentation: Teknik Oversampling dengan penambahan distribusi normal
  - Evaluasi: Menggunakan metrik regresi ($R^2$, RMSE, dan MAE)

- **Link Projek:** 
  - ETL Pipeline: [https://colab.research.google.com/drive/1_7lkgpdF1Oql-R47ZcSdVDR4V-Nv1kcV?usp=sharing](https://colab.research.google.com/drive/1_7lkgpdF1Oql-R47ZcSdVDR4V-Nv1kcV?usp=sharing)
  - Machine Learning: [https://colab.research.google.com/drive/1fnqa18_cpDdnY7cOgK7RKvY2bCNlcQ_o?usp=sharing](https://colab.research.google.com/drive/1fnqa18_cpDdnY7cOgK7RKvY2bCNlcQ_o?usp=sharing)
  - Looker : 

---
