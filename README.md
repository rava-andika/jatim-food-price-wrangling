# Data Wrangling: Harga Pangan Jawa Timur 2024 (Paket D)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-2.0%2B-150458.svg?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![Status](https://img.shields.io/badge/Status-Completed-success.svg)]()
[![Institution](https://img.shields.io/badge/UNESA-S1%20Sains%20Data-blueviolet.svg)](https://datascience.fmipa.unesa.ac.id)

Repositori ini memuat implementasi *end-to-end* **Data Wrangling** terhadap data harga pangan di 9 kabupaten/kota Jawa Timur sepanjang tahun 2024. Data harga pangan diintegrasikan dengan data cuaca harian dan data karakteristik wilayah administrasi/demografi untuk mendukung analisis pemantauan harga pangan mingguan bagi tim analitik dinas ketahanan pangan.

Proyek ini merupakan pemenuhan **Ujian Tengah Semester (UTS) Gasal 2026/2027** mata kuliah **Data Wrangling** pada program studi S1 Sains Data, Universitas Negeri Surabaya.

---

## 1. Fokus Analisis Paket D

Analisis utama berfokus pada dua kelompok komoditas strategis:
1. **Beras Antarkualitas**: Beras Kualitas Bawah I, Beras Kualitas Medium I, dan Beras Kualitas Super I.
2. **Minyak Goreng**: Minyak Goreng Curah dan Minyak Goreng Kemasan Bermerk 1.

### Pertanyaan Analisis:
> *"Seberapa besar selisih harga antarkualitas beras (bawah, medium, super) dan antara minyak goreng curah dan kemasan di tiap kabupaten/kota, dan apakah selisih tersebut stabil atau berubah selama Januari–Desember 2024?"*

---

## 2. Sumber Data

Seluruh data mentah diambil dari sumber publik resmi tanpa rekayasa data awal:

| No | Sumber Data | Berkas Mentah | Format & Karakteristik |
|:--:|:---|:---|:---|
| 1 | **PIHPS Nasional** (Bank Indonesia) | `pihps_pasar_tradisional_jatim_2024.xlsx` | Excel 9 sheet (satu sheet per kab/kota), matriks lebar 262 kolom tanggal harian. |
| 2 | **NOAA GSOD** (NCEI, AS) | `noaa_gsod_jatim_2024.csv` | CSV ringkasan cuaca 4 stasiun (Surabaya Perak, Surabaya Juanda, Sumenep, Banyuwangi), satuan °F dan inci, kode sentinel `999.9` / `99.99`. |
| 3 | **Kepmendagri No 300.2.2-2138 Tahun 2025** | `kode_wilayah_jatim.csv`, `penduduk_jatim.csv`, `luas_jatim.csv` | CSV hierarki 9.199 unit wilayah (provinsi s.d. desa) serta data kependudukan dan luas daratan per kabupaten/kota. |

---

## 3. Struktur Repositori

```text
jatim-food-price-wrangling/
│
├── UTS_DataWrangling_Paket_D.ipynb   # Notebook utama: proses wrangling & analisis lengkap
│
├── UTS_DW_data/                      # Folder data mentah asli
│   ├── 1_harga_pasar_tradisional/   # Data mentah PIHPS (Excel)
│   ├── 2_cuaca_stasiun/             # Data mentah NOAA GSOD (CSV)
│   ├── 3_wilayah/                   # Data mentah Kepmendagri (3 file CSV)
│   ├── PETUNJUK.md                  # Petunjuk teknis pengerjaan
│   └── SUMBER_DATA.md               # Dokumentasi asal data resmi
│
├── data_processed/                   # Dataset analitik akhir siap pakai
│   ├── harga_pangan_jatim_2024.csv           # Dataset terintegrasi tingkat harian (46.112 baris)
│   └── harga_pangan_jatim_mingguan_2024.csv  # Dataset terintegrasi tingkat mingguan (9.328 baris)
│
├── kamus_data.md                     # Kamus data lengkap seluruh variabel
├── laporan.pdf                       # Dokumen laporan resmi UTS Data Wrangling
├── .gitignore                        # Konfigurasi file yang diabaikan git
└── README.md                         # Dokumentasi proyek
```

---

## 4. Alur Metodologi Data Wrangling

Alur kerja mengikuti standar 6 fase data wrangling:

1. **Discovery (Penemuan)**: Pemeriksaan metadata, dimensi tabel, tipe data awal, nilai sentinel (`999.9`, `99.99`, `-`), dan inkonsistensi nama entitas wilayah.
2. **Structuring (Penstrukturan)**:
   - *Unpivoting* (melt) 262 kolom tanggal PIHPS menjadi format panjang (*tidy data*).
   - Pemisahan baris agregat kategori (`level == 1`) dan komoditas spesifik (`level == 2`).
   - Ekstraksi hierarki kode wilayah Kepmendagri via regex untuk menghitung jumlah kecamatan (`35.xx.xx`), kelurahan (`35.xx.xx.1xxx`), dan desa (`35.xx.xx.2xxx`).
3. **Cleaning (Pembersihan)**:
   - Pembersihan teks angka berkoma (`"12,900"`) menjadi numerik `float64`.
   - Konversi satuan imperial cuaca ke metrik: Suhu ke Celcius ($^\circ\text{C} = (\text{F}-32) \times 5/9$) dan curah hujan ke milimeter ($\text{mm} = \text{inci} \times 25.4$).
   - Netralisasi kode sentinel kosong `999.9` dan `99.99` menjadi `NaN`.
   - Imputasi 101 missing data harga harian intermiten menggunakan **interpolasi linear dalam grup `(wilayah, komoditas)`** berdasarkan urutan waktu.
4. **Enriching (Pengayaan & Integrasi)**:
   - Kalkulasi kepadatan penduduk ($\text{jiwa/km}^2 = \text{total penduduk} / \text{luas}$).
   - Agregasi rata-rata harian 2 stasiun cuaca Surabaya (Perak & Juanda) untuk eliminasi duplikasi baris.
   - Penyelarasan waktu ke kalender standar ISO (`tahun_iso`, `minggu_ke`).
   - Penggabungan data melalui **`LEFT JOIN`** berbasis kode wilayah resmi `35.xx` (Kepmendagri) sebagai *single source of truth*.
5. **Validating (Validasi)**:
   - Verifikasi integritas jumlah baris (0 baris harga pangan yang hilang pasca-join).
   - Deteksi outlier ganda menggunakan metode **IQR** dan **Z-Score berbasis MAD** (outlier dipertahankan dengan *flagging* karena mencerminkan dinamika riil HBKN Ramadan dan masa paceklik).
   - Evaluasi 4 dimensi kualitas data: *Completeness*, *Validity*, *Consistency*, dan *Uniqueness*.
6. **Publishing (Penyajian)**:
   - Ekspor dataset bersih ke format CSV di `data_processed/`.
   - Penyusunan dokumentasi teknis `kamus_data.md` dan visualisasi analitik tren bulanan.

---

## 5. Ringkasan Temuan Utama Paket D

### A. Selisih Harga Antarkualitas Beras
* Struktur harga beras selalu berjenjang positif ($Bawah < Medium < Super$) di seluruh 9 wilayah.
* **Margin Super I vs Medium I**: Tertinggi berada di **Kota Madiun (Rp 2.789,89/kg)** dan **Kab. Banyuwangi (Rp 2.185,02/kg)**. Selisih paling tipis di **Kota Probolinggo (Rp 469,47/kg)** dan **Kota Malang (Rp 608,59/kg)**.
* **Margin Medium I vs Bawah I** *(pada 6 wilayah yang mencatat)*: Tertinggi di **Kab. Jember (Rp 1.637,50/kg)** dan **Kab. Sumenep (Rp 1.486,45/kg)**. Terendah di **Kota Probolinggo (Rp 383,97/kg)** dan **Kota Surabaya (Rp 477,77/kg)**.
* *Catatan Keterbatasan:* Beras Kualitas Bawah I tidak pernah dicatat oleh PIHPS di Kab. Banyuwangi, Kota Kediri, dan Kota Madiun.

### B. Selisih Harga Minyak Goreng (Kemasan Bermerk 1 vs Curah)
* Minyak Kemasan Bermerk 1 konsisten lebih mahal dibanding Minyak Curah di semua daerah.
* Margin kemasan tertinggi berada di wilayah kabupaten: **Kab. Banyuwangi (Rp 2.964,22/liter)** dan **Kab. Jember (Rp 2.520,33/liter)**.
* Margin tersempit berada di sentra logistik perkotaan: **Kota Madiun (Rp 736,07/liter)** dan **Kota Kediri (Rp 895,42/liter)**.

### C. Stabilitas Selisih Harga Sepanjang 2024
* **Beras**: Berfluktuasi secara musiman di daerah sentra agraris (Kab. Sumenep std Rp 752,43; Kota Madiun std Rp 714,16), menyempit saat harga beras medium melonjak pada musim paceklik sebelum panen raya (Februari–Maret 2024), lalu melebar kembali pasca panen. Sebaliknya di Kota Surabaya dan Kota Malang, harga ketiga mutu beras sangat stabil terkontrol ($CV < 2,2\%$, std selisih Rp 169,62).
* **Minyak Goreng**: Terjadi **penyempitan margin secara tajam dan sistematis dari Januari ke Desember 2024** di hampir semua wilayah (contoh di Kota Surabaya: selisih turun dari Rp 4.210 di awal tahun menjadi -Rp 490 di akhir tahun). Hal ini dipicu oleh tren kenaikan harga Minyak Curah yang merangkak naik secara berkelanjutan sepanjang 2024 ($CV = 6,13\% - 8,28\%$), sementara harga Kemasan Bermerk 1 tertahan kaku di sekitar batas HET pemerintah ($CV = 0,63\% - 4,43\%$).

---

## 6. Teknologi dan Dependensi

Proyek ini dibangun menggunakan Python 3.10+ dengan pustaka-pustaka berikut:
- **`pandas`**: Manipulasi tabular, reshape (*melt/pivot*), merge/join, dan transformasi groupby.
- **`numpy`**: Operasi komputasi numerik, penanganan NaN, dan transformasi logaritmik.
- **`matplotlib`** & **`seaborn`**: Visualisasi tren deret waktu, sebaran distribusi, dan komparasi selisih.
- **`openpyxl`**: Parser berkas multi-sheet Microsoft Excel.
- **`pathlib`**: Manajemen jalur berkas lintas sistem operasi (*cross-platform*).

### Instalasi Lingkungan:
```bash
pip install pandas numpy matplotlib seaborn openpyxl
```

---

## 7. Cara Menjalankan Proyek

1. **Clone repositori ini**:
   ```bash
   git clone https://github.com/rava-andika/jatim-food-price-wrangling.git
   cd jatim-food-price-wrangling
   ```
2. **Jalankan Jupyter Notebook**:
   ```bash
   jupyter notebook UTS_DataWrangling_Paket_D.ipynb
   ```
3. **Eksekusi Notebook**:
   Pilih menu `Kernel` $\rightarrow$ `Restart & Run All`. Seluruh proses dari Tahap 3 (pembacaan) hingga Tahap 8 (penyimpanan dataset akhir) akan dieksekusi secara otomatis.
4. **Periksa Hasil**:
   File dataset akhir akan tersimpan di dalam folder `data_processed/`.

---

## 8. Lisensi & Hak Penggunaan

Data mentah bersumber dari domain publik:
- **PIHPS Nasional (Bank Indonesia)**: Publikasi laporan harian pasar tradisional.
- **NOAA NCEI (Global Summary of the Day)**: Data domain publik pemerintah AS.
- **Kepmendagri No 300.2.2-2138 Tahun 2025**: Dokumen hukum pemerintahan Republik Indonesia.

Kode dan dokumentasi ini dikembangkan untuk tujuan akademik dan portofolio data science oleh **Rava Dwi Andika** (NIM: 25031554039), S1 Sains Data, Fakultas Matematika dan Ilmu Pengetahuan Alam, Universitas Negeri Surabaya.
