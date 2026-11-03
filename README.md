# PulseUP — Sistem Pemantauan Kesehatan Startup

> Perancangan sistem pemantauan kesehatan startup berbasis **Python** dengan pendekatan **Object-Oriented Programming (OOP)**.

Proyek mata kuliah **Analisis dan Desain Pemrograman Berorientasi Objek (Python)**
Program Studi S1 Sistem Informasi, Fakultas Sains dan Teknologi, Universitas Jambi — Tahun 2026.

| | |
|---|---|
| **Status** | Milestone 1 — Analisis dan Desain (belum ada implementasi kode) |
| **Bahasa** | Python 3 |
| **Antarmuka** | Web GUI berbasis Flask (direncanakan pada Milestone 3) |

---

## Deskripsi

Startup berjalan dalam kondisi ketidakpastian tinggi dengan sumber daya terbatas. Tim startup mahasiswa umumnya masih minim pengalaman membaca kondisi usahanya, sehingga pemantauan manual lewat catatan terpisah sering terlambat disadari. Di sisi lain, dosen pendamping membutuhkan cara cepat untuk mengetahui tim mana yang perlu perhatian lebih.

**PulseUP** membantu dengan menilai kesehatan startup dari tiga sisi (keuangan, produk, dan pengguna), lalu menyimpulkannya menjadi status **SEHAT**, **WASPADA**, atau **KRITIS** beserta rekomendasi tindakan.

## Pengguna dan Fitur

Sistem memiliki dua peran:

**Tim Startup (mahasiswa)**
- Memilih tim dengan memasukkan ID tim
- Menginput data metrik bulanan (kas, pengeluaran, milestone, pengguna)
- Validasi input otomatis dengan pesan kesalahan yang jelas
- Melihat status tiap metrik, status keseluruhan, dan rekomendasi (evaluasi mandiri)

**Mentor (dosen)**
- Melihat rekapitulasi seluruh tim beserta skor dan statusnya
- Memfilter tim berstatus **KRITIS** atau **WASPADA**
- Hanya melihat hasil; mentor tidak dapat mengubah data tim

## Metrik Penilaian

| Metrik | Data Masukan | Dasar Penilaian |
|---|---|---|
| **MetrikKeuangan** | Saldo kas, pengeluaran bulanan | *Runway* = saldo kas ÷ pengeluaran bulanan |
| **MetrikProduk** | Target dan capaian milestone | Persentase milestone yang selesai |
| **MetrikPengguna** | Jumlah pengguna, target pengguna | Pertumbuhan pengguna terhadap target |

**Status keseluruhan** ditentukan dari status **terburuk** di antara semua metrik, dengan urutan `SEHAT < WASPADA < KRITIS`.

### Ambang batas (asumsi awal)

| Metrik | SEHAT | WASPADA | KRITIS |
|---|---|---|---|
| Keuangan (runway) | ≥ 6 bulan | 3 s.d. < 6 bulan | < 3 bulan |
| Produk (pencapaian) | ≥ 50% | di bawah 50% | *ditentukan kelompok* |
| Pengguna | *ditentukan kelompok* | *ditentukan kelompok* | *ditentukan kelompok* |

> Ambang batas merupakan asumsi awal yang ditulis eksplisit agar mudah disesuaikan tanpa mengubah struktur class. Nilai yang masih kosong akan ditetapkan pada Milestone 2.

**Contoh:** kas Rp 5.000.000 dengan pengeluaran Rp 2.000.000/bulan menghasilkan runway 2,5 bulan (**KRITIS**). Jika capaian milestone 8 dari 10 (80%, **SEHAT**), status keseluruhan tetap **KRITIS** dengan saran segera menekan pengeluaran operasional.

## Validasi Input

Sistem menolak:
- angka negatif,
- input yang bukan angka,
- persentase di luar rentang 0–100,
- pengeluaran bulanan bernilai nol (runway tidak dapat dihitung).

Validasi dilakukan di dalam class melalui setter (**enkapsulasi**), sehingga sistem tidak berhenti saat input salah, melainkan menampilkan pesan dan menunggu input yang benar.

## Arsitektur OOP

### Daftar class

| Class | Tanggung Jawab |
|---|---|
| `TimStartup` | Menyimpan data tim, daftar metrik, dan hasil evaluasi (`skor_akhir`, `status_akhir`) |
| `Mentor` | Menyediakan rekap seluruh tim dan penyaringan berdasarkan status |
| `PenilaiStartup` | Menghitung skor akhir, menentukan status keseluruhan, dan menyusun rekomendasi |
| `Metrik` *(abstrak)* | Class induk dengan method abstrak `hitung_skor()` dan `hitung_status()` |
| `MetrikKeuangan` | Menghitung runway, skor, dan status keuangan |
| `MetrikProduk` | Menghitung pencapaian milestone, skor, dan status produk |
| `MetrikPengguna` | Menghitung skor dan status pertumbuhan pengguna |

### Relasi antar class

- `Mentor` membimbing satu atau lebih `TimStartup`
- `TimStartup` memiliki nol atau lebih objek `Metrik` (*composition*)
- `MetrikKeuangan`, `MetrikProduk`, dan `MetrikPengguna` mewarisi `Metrik` (*inheritance*)
- `PenilaiStartup` hanya bergantung pada `Metrik`, tidak pada jenis konkretnya

### Prinsip yang diterapkan

- **Inheritance dan Polymorphism** — setiap subclass `Metrik` mengimplementasikan `hitung_skor()` dan `hitung_status()` dengan caranya sendiri.
- **Open/Closed Principle (OCP)** — `PenilaiStartup` memanggil `hitung_status()` secara polimorfik. Metrik baru (misalnya `MetrikKekompakan`) cukup dibuat sebagai subclass `Metrik` dan ditambahkan lewat `tambah_metrik()`, tanpa mengubah penilai atau metrik yang sudah ada, dan tanpa rantai `if-elif`.
- **Single Responsibility Principle (SRP)** — tiap class punya satu alasan untuk berubah. Penampilan data ditangani GUI, bukan class domain.
- **Enkapsulasi** — validitas data dijaga di dalam class.

## Alur Kerja Singkat

```
Pilih peran
 ├─ Tim Startup → masukkan ID tim → isi data metrik → validasi
 │                → PenilaiStartup meminta tiap Metrik menghitung skor & status
 │                → tampil status per metrik + status keseluruhan + rekomendasi
 │                → hasil disimpan di objek TimStartup
 └─ Mentor      → rekapitulasi seluruh tim → (opsional) filter KRITIS / WASPADA
                  (jika belum ada tim dinilai: tampil pesan "data belum tersedia")
```

## Rencana Struktur File

```
pulseup/
├── metrik.py          # Metrik (abstrak) + MetrikKeuangan, MetrikProduk, MetrikPengguna
├── tim_startup.py     # Class TimStartup
├── penilai.py         # Class PenilaiStartup
├── mentor.py          # Class Mentor
├── data_tim.py        # Data tim contoh (disimpan sementara di memori)
├── app.py             # Titik masuk Flask; tanpa logika penilaian
├── uji_class.py       # Pengujian class domain tanpa GUI
├── requirements.txt   # Dependensi (Flask)
├── templates/         # Halaman: pilih peran, form input, hasil evaluasi, rekap mentor, base.html
└── static/
    └── style.css      # Gaya tampilan, termasuk warna penanda status
```

**Aturan desain:** file lapisan domain (`metrik.py`, `tim_startup.py`, `penilai.py`, `mentor.py`) **tidak boleh mengimpor Flask**, sehingga logika penilaian dapat diuji dan dipakai ulang secara mandiri dari tampilan.

## Batasan Sistem

- Hanya dua peran pengguna: tim startup dan mentor
- Hanya tiga metrik: keuangan, produk, dan pengguna
- Hanya data **bulan berjalan** (tanpa riwayat atau perbandingan antarperiode)
- Data tim berupa data contoh; **tanpa** pendaftaran tim baru
- **Tanpa database dan tanpa login**; data hanya disimpan selama program berjalan

## Rencana Milestone

| Tahap | Fokus | Luaran |
|---|---|---|
| **Milestone 1** | Analisis kebutuhan, analisis class (SRP dan OCP), use case diagram, class diagram, flowchart, skenario penggunaan, rencana struktur file | Proposal dan diagram desain |
| **Milestone 2** | Implementasi class domain, data tim contoh, validasi via setter, penetapan rumus dan ambang batas, pengujian tanpa GUI | Source code class domain yang lulus uji |
| **Milestone 3** | Antarmuka GUI (Flask), integrasi dengan class domain, pengujian berdasarkan skenario, laporan, presentasi | Program final, laporan, presentasi |

## Rencana Menjalankan (setelah implementasi)

> Bagian ini adalah rencana dan baru berlaku setelah Milestone 2 dan 3 selesai.

```bash
# 1. Pasang dependensi
pip install -r requirements.txt

# 2. Uji class domain (tanpa GUI)
python uji_class.py

# 3. Jalankan aplikasi web
python app.py
```

## Tim Pengembang (Kelompok 2)

| Nama | NIM | Peran |
|---|---|---|
| Selvy Amanda Saputri | F1E125069 | Ketua; analisis kebutuhan, desain class, prinsip SOLID |
| M. Akmal Faizhin Hafiz | F1E125073 | Class diagram dan struktur file Python |
| Salma Mahira | — | Use case, flowchart, skenario, rencana milestone |
| Rionza | F1E125015 | Dokumentasi, naskah, referensi, penggabungan dokumen |

**Dosen Pengampu:** Edi Saputra, S.T., M.Sc. · Pariyadi, S.Kom., M.Kom. · Hamzah Alghifari, M.Kom. · Muhammad Damas Fatih, S.Kom., M.Kom.

## Referensi

1. E. Ries, *The Lean Startup*. Crown Business, 2011.
2. D. Skok, "SaaS metrics 2.0 – A guide to measuring and improving what matters," *For Entrepreneurs*.
3. K. A. Lambert, *Fundamentals of Python: First Programs*, 2nd ed. Cengage Learning, 2017.
4. R. C. Martin, *Agile Software Development, Principles, Patterns, and Practices*. Prentice Hall, 2002.
5. Python Software Foundation, "abc — Abstract Base Classes," *Python 3 Documentation*.
