# Data UTS Data Wrangling: Harga Pangan Jawa Timur Januari-Desember 2024

Folder ini berisi data **real** dari tiga sumber publik (lihat `SUMBER_DATA.md`). Dosen sudah mengambil datanya dari sumber asli, jadi Anda **tidak perlu melakukan scraping atau mengakses API**. Data diberikan apa adanya, belum dibersihkan: ada nilai kosong, kode khusus, satuan yang berbeda, format angka yang tidak seragam, kode wilayah bertingkat, dan duplikat. Menemukan dan menangani masalah tersebut adalah inti UTS ini.

Semua mahasiswa memakai data yang sama. Yang berbeda antarpaket adalah komoditas fokus dan pertanyaan analisis (lihat naskah soal).

## Isi folder

| Folder | Format | Isi singkat |
|--------|--------|-------------|
| `1_harga_pasar_tradisional/` | Excel, satu sheet per kab/kota | Harga harian pasar tradisional (PIHPS, Bank Indonesia), tabel lebar: satu kolom per tanggal |
| `2_cuaca_stasiun/` | CSV | Cuaca harian 4 stasiun (NOAA GSOD); baca kamus kolomnya di tautan pada `SUMBER_DATA.md` |
| `3_wilayah/` | CSV (3 file) | Data wilayah resmi Kepmendagri untuk Jawa Timur: jumlah penduduk (pria, wanita, total) dan luas per kab/kota, serta daftar kode seluruh kecamatan dan kelurahan/desa |

## Aturan

1. Jangan mengubah file data secara manual (misalnya mengedit di Excel). Semua perbaikan dilakukan dengan kode.
2. Baca data langsung dari folder ini di notebook Anda (jangan disalin ulang dengan tangan).
3. Cantumkan sumber asli data (lihat `SUMBER_DATA.md`) di laporan dan kamus data Anda.
