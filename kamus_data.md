# Kamus Data
## Proyek UTS Data Wrangling Paket D
**Judul:** Dari Data Mentah ke Dataset Siap Analisis: Harga Pangan Jawa Timur 2024

Dokumen ini menjelaskan variabel yang digunakan dalam proses integrasi dan analisis data harga pangan Jawa Timur tahun 2024. Nama kolom mengikuti nama variabel pada hasil pengolahan notebook.

## 1. Identitas Waktu

| Nama variabel | Tipe data | Deskripsi |
|---|---|---|
| `tanggal` | datetime | Tanggal pengamatan harga harian. |
| `tanggal_mulai_minggu` | datetime | Tanggal awal periode minggu untuk agregasi mingguan. |
| `tahun_iso` | integer | Tahun berdasarkan kalender minggu ISO. |
| `minggu_ke` | integer | Nomor minggu berdasarkan kalender minggu ISO. |
| `bulan` | integer | Nomor bulan pengamatan (1–12). |

**Catatan:** Dataset mingguan memiliki 53 tanggal awal minggu. Sebanyak 52 periode termasuk tahun ISO 2024, sedangkan periode yang dimulai pada 30 Desember 2024 termasuk minggu ISO ke-1 tahun 2025.

## 2. Identitas Wilayah

| Nama variabel | Tipe data | Deskripsi |
|---|---|---|
| `kode` | string | Kode wilayah untuk pencocokan data kabupaten/kota. |
| `wilayah` | string | Nama kabupaten/kota di Jawa Timur. |

## 3. Demografi dan Geografi

| Nama variabel | Tipe data | Deskripsi dan satuan |
|---|---|---|
| `jumlah_penduduk` | numerik | Jumlah penduduk di wilayah (jiwa). |
| `luas` | numerik | Luas wilayah (km²). |
| `kepadatan_penduduk` | numerik | Kepadatan penduduk (jiwa/km²). |

**Catatan:** Data demografi dan geografi bersumber dari tahun 2025 dan digunakan sebagai informasi kontekstual wilayah, bukan sebagai pengukuran yang berubah sepanjang 2024.

## 4. Administrasi Wilayah

| Nama variabel | Tipe data | Deskripsi |
|---|---|---|
| `jumlah_kecamatan` | integer | Jumlah kecamatan pada kabupaten/kota. |
| `jumlah_kelurahan` | integer | Jumlah kelurahan pada kabupaten/kota. |
| `jumlah_desa` | integer | Jumlah desa pada kabupaten/kota. |

## 5. Komoditas dan Harga Pangan

| Nama variabel | Tipe data | Deskripsi dan satuan |
|---|---|---|
| `kategori` | string | Kategori atau kelompok komoditas pangan. |
| `komoditas` | string | Nama komoditas pangan yang diamati. |
| `harga` | numerik (`float64`) | Harga komoditas pada pengamatan harian, dalam Rupiah sesuai satuan pada sumber data. |
| `harga_mingguan` | numerik | Harga hasil agregasi mingguan, dalam Rupiah sesuai satuan pada sumber data. |
| `jumlah_observasi_harga` | integer | Jumlah pengamatan harga yang tersedia dalam satu kelompok agregasi mingguan. |

Komoditas Paket D:
- Beras Kualitas Bawah I
- Beras Kualitas Medium I
- Beras Kualitas Super I
- Minyak Goreng Curah
- Minyak Goreng Kemasan Bermerk 1

## 6. Indikator Cuaca

| Nama variabel | Tipe data | Deskripsi dan satuan |
|---|---|---|
| `TEMP_C` | numerik | Suhu rata-rata (°C). |
| `MAX_C` | numerik | Suhu maksimum (°C). |
| `MIN_C` | numerik | Suhu minimum (°C). |
| `PRCP_MM` | numerik | Curah hujan (mm). |

**Catatan:** Data cuaca berasal dari NOAA. Kode khusus seperti `999.9` dan `99.99` diperlakukan sebagai *missing value*, bukan hasil pengukuran. Data cuaca hanya berhasil dicocokkan dengan Banyuwangi, Sumenep, dan Surabaya.

## 7. Variabel Turunan untuk Analisis

| Nama variabel | Tipe data | Deskripsi dan satuan |
|---|---|---|
| `selisih_medium_bawah` | numerik | Selisih harga Beras Kualitas Medium I dengan Beras Kualitas Bawah I (Rp). |
| `selisih_super_medium` | numerik | Selisih harga Beras Kualitas Super I dengan Beras Kualitas Medium I (Rp). |
| `selisih_kemasan_curah` | numerik | Selisih harga Minyak Goreng Kemasan Bermerk 1 dengan Minyak Goreng Curah (Rp). |
| `harga_normalisasi` | numerik | Harga hasil normalisasi Min-Max per komoditas, dengan rentang 0–1. |

## 8. Ringkasan Dataset Hasil Pengolahan

| Dataset | Jumlah baris | Jumlah kolom | Keterangan |
|---|---:|---:|---|
| Dataset harian | 46.112 | 16 | Data terintegrasi pada tingkat pengamatan harian. |
| Dataset mingguan | 9.328 | 19 | Data harga yang diagregasi berdasarkan wilayah, komoditas, dan periode minggu. |

Unit observasi:
- **Dataset harian:** kombinasi wilayah, komoditas, dan tanggal pengamatan.
- **Dataset mingguan:** kombinasi wilayah, komoditas, dan tanggal awal minggu.

Dataset mingguan memiliki 16 nilai kosong pada `harga_mingguan` karena terdapat kelompok wilayah-komoditas-minggu tanpa observasi harga.

## 9. Keterangan Pengolahan Data

1. Sebanyak 101 nilai harga kosong pada lima komoditas Paket D diisi menggunakan interpolasi waktu per wilayah dan komoditas, khusus untuk celah yang memiliki observasi sebelum dan sesudahnya.
2. Kombinasi Beras Kualitas Bawah I di Banyuwangi, Kediri, dan Madiun tidak memiliki observasi awal sehingga tidak dapat diinterpolasi.
3. Outlier dideteksi menggunakan metode IQR dan Z-score. Nilai yang terdeteksi tetap dipertahankan karena belum dapat dipastikan sebagai kesalahan pencatatan.
4. Normalisasi Min-Max dilakukan per komoditas. Kolom harga asli tetap dipertahankan untuk analisis dalam satuan Rupiah.
5. Data demografi/geografi tahun 2025 digunakan sebagai atribut kontekstual untuk data harga tahun 2024.

---
**Catatan penggunaan:** Tipe data pada tabel merupakan tipe konseptual berdasarkan fungsi variabel. Tipe aktual dapat berbeda setelah proses pembacaan atau transformasi di notebook.
