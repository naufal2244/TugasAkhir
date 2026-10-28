# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 8)
**File:** `TugasAkhir/log/Log8.md`  
**Waktu Pembaruan:** 2026-10-28  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan pada Sesi 8 mengenai validasi tangkapan layar profiling Laravel Telescope untuk Lampiran A, perumusan caption Gambar 4 (Subbab 3.4.4.1), audit menyeluruh inventaris gambar dan tabel pada naskah `proposal.md`, verifikasi keabsahan hitungan "7 Gambar dan 7 Tabel", serta pemetaan perbaikan teknis penomoran tabel untuk sesi berikutnya.

---

## 1. VALIDASI ARTEFAK PROFILING TELESCOPE (LAMPIRAN A)

### A. Konteks Metodologis (Subbab 3.2.1 Identifikasi Titik Hambatan)
Pada Subbab 3.2.1, profiling awal menggunakan Laravel Telescope difokuskan pada dua endpoint target (`GET /statistik/suggestions` dan `GET /statistik/modal-data`) untuk memperoleh bukti empiris titik hambatan performa (*performance hot spots*) sebelum optimasi diterapkan. Bukti log kueri tersebut dirujuk silang ke **Lampiran A**:
* **Lampiran A.1:** Bukti Kueri Relasional Snapshot Berulang dan Gejala *Partial Cache Hit*.
* **Lampiran A.2:** Bukti Inefisiensi Kueri ORM (*Unnecessary Data Retrieval* dan *N+1 Query*).

### B. Hasil Evaluasi & Validasi Dua Gambar Log Telescope
Dua gambar tangkapan layar Telescope yang diajukan peneliti telah divalidasi dan dinyatakan **100% cocok serta valid** untuk mengisi Lampiran A:

1. **Gambar Pertama $\rightarrow$ Lampiran A.1:**
   * **Fakta Visual:** Menampilkan eksekusi kueri `select * from "workspace_performance_snapshots" where "workspace_id" = ...` sebanyak 2 kali berturut-turut (durasi 0.28ms dan 0.56ms).
   * **Justifikasi Ilmiah:** Membuktikan secara empiris bahwa mekanisme snapshot relasional bawaan Koladi bukanlah *true database bypass*. Permintaan yang sama tetap memicu kueri relasional berulang ke PostgreSQL.
   * **Caption Singkat Disepakati:**  
     > **Gambar A.1. Kueri Berulang pada Mekanisme Snapshot Relasional**

2. **Gambar Kedua $\rightarrow$ Lampiran A.2:**
   * **Fakta Visual:** Header Telescope mencatat secara gamblang: `Queries (41), 26 of which are duplicated. Duration: 18.38ms`, disertai rentetan pemanggilan kueri `select * from "tasks" where ... and exists(...)`.
   * **Justifikasi Ilmiah:** Membuktikan secara mutlak terjadinya inefisiensi ORM berupa:
     1. *Query-in-Loop* & Kueri Duplikat (26 kueri terduplikasi akibat kueri `Task::whereHas()` di dalam perulangan `foreach ($members)` pada metode `getWorkloadDistribution()`).
     2. *Unnecessary Data Retrieval / Column Over-fetching* akibat pemanggilan `SELECT *` yang memboroskan memori PHP.
   * **Caption Singkat Disepakati:**  
     > **Gambar A.2. Inefisiensi Kueri ORM pada Endpoint `/modal-data`**

---

## 2. PERUMUSAN CAPTION GAMBAR 4 (SUBBAB 3.4.4.1)

### A. Konteks Gambar 4
Gambar 4 menampilkan cuplikan kode perbandingan *before-after* refaktorisasi kueri Eloquent ORM pada dua metode kunci:
1. **Kasus 1 (`getTasksInPeriod`):** Transformasi `->with(['assignedUsers'])` menjadi `->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])` (*selective eager loading*).
2. **Kasus 2 (`getWorkloadDistribution`):** Mengeluarkan kueri dari `foreach ($members)` menjadi penarikan batch satu kali `Task::with(['assignedUsers:id'])->where(...)->get()` yang dilanjutkan filter koleksi memori `$tasks->filter(...)` (*batch in-memory aggregation*).

### B. Pilihan Caption yang Dirumuskan
* **Pilihan Lengkap/Akademis (Direkomendasikan):**  
  > **Gambar 4. Perbandingan Transformasi Kueri Eloquent ORM (*Before* dan *After*) pada Metode `getTasksInPeriod()` dan `getWorkloadDistribution()`**
* **Pilihan Singkat:**  
  > **Gambar 4. Optimasi Kueri Eloquent ORM pada Metode `getTasksInPeriod()` dan `getWorkloadDistribution()`**

---

## 3. AUDIT MENYELURUH: VERIFIKASI "7 GAMBAR DAN 7 TABEL"

Peneliti menanyakan validitas hitungan hasil review mandiri, yaitu **7 Gambar dan 7 Tabel**. Dilakukan audit mendalam terhadap seluruh berkas `proposal.md` dan Lampiran:

### A. Audit GAMBAR: **VALID 100% (Tepat 7 Gambar)**
Struktur gambar sudah sangat kokoh, terdiri atas 5 gambar di Bab III dan 2 gambar di Lampiran A:
1. **Gambar 1:** Tahapan Penelitian (Subbab 3.2 - Diagram 6 Fase *Refactoring Lifecycle* Mens & Tourwé).
2. **Gambar 2:** Gambaran Umum Sistem dan Lingkungan Penelitian (Subbab 3.4.1 - Arsitektur DSS, Redis, PostgreSQL).
3. **Gambar 3:** Diagram Sekuensial Alur Komputasi Modul DSS, Pola Cache-Aside, dan Eksekusi Eloquent ORM (Subbab 3.4.2 - *Mermaid Sequence Diagram*).
4. **Gambar 4:** Perbandingan Transformasi Kueri Eloquent ORM pada Metode `getTasksInPeriod()` dan `getWorkloadDistribution()` (Subbab 3.4.4.1).
5. **Gambar 5:** Topologi Lingkungan Pengujian Terkontainerisasi dan Alur Instrumentasi Pengukuran (Subbab 3.4.5).
6. **Gambar A.1:** Kueri Berulang pada Mekanisme Snapshot Relasional (Lampiran A.1).
7. **Gambar A.2:** Inefisiensi Kueri ORM pada Endpoint `/modal-data` (Lampiran A.2).

### B. Audit TABEL: **VALID SECARA STRUKTUR, TETAPI DITEMUKAN DEFECT DI PROPOSAL.MD**
Hitungan 7 Tabel secara substansi **dapat dibenarkan/valid**, namun audit kode markdown pada `proposal.md` menemukan 4 anomali/cacat teknis yang wajib diperbaiki:

#### Temuan Defect di `proposal.md`:
1. **Lompat Nomor:** Di baris 84 ada Tabel 1, baris 261 Tabel 2, baris 288 Tabel 3, namun di baris 457 langsung melompat ke **Tabel 5** (Tabel 4 terlewat).
2. **Nomor Dobel:** Label "Tabel 5" muncul dua kali, yaitu pada baris 457 (*Rancangan Dataset Pengujian*) dan baris 522 (*Skema Kunci dan Struktur Penyimpanan Redis Cache-Aside*).
3. **Tabel Nyasar & Duplikat:** Tabel spesifikasi skema Redis di baris 522 fisiknya kosong, tetapi isi tabelnya malah ter-paste di baris 549 (di atas *Gambar 5*) dan ter-paste duplikat lagi di baris 570 (menimpa isi *Tabel 6*).
4. **Tabel Markdown Tanpa Label Nomor Tabel:** Terdapat tabel daftar fase di baris 311 (dilabeli Gambar 1) dan tabel entitas sistem di baris 370 (dilabeli Gambar 2).

---

## 4. STANDARISASI SUSUNAN 7 TABEL PROPOSAL (SOLUSI PENYELESAIAN)

Untuk menjaga agar total tabel benar-benar **7 Tabel yang berurutan rapi dan berstandar akademik tinggi**, disepakati susunan penomoran baku berikut:

| No | Label Tabel Baku | Lokasi Naskah | Isi & Deskripsi Tabel |
| :---: | :--- | :--- | :--- |
| 1 | **Tabel 1** | Subbab 2.1 | Perbandingan Penelitian Sebelumnya (Sintesis 5 Studi Terdahulu vs TA Ini) |
| 2 | **Tabel 2** | Subbab 2.4.3 | Matriks Desain Eksperimen Faktorial Penuh ($3 \times 2 \times 2 \times 2 \times 10 = 240$ Observasi Valid) |
| 3 | **Tabel 3** | Subbab 3.1.1 | Alat Penelitian (Perangkat Keras, Perangkat Lunak, & Instrumentasi) |
| 4 | **Tabel 4** | Subbab 3.1.2 | Bahan Penelitian (Kode Sumber, Dataset, Skrip Pengujian, Log, dll.) |
| 5 | **Tabel 5** | Subbab 3.4.3 | Spesifikasi Rancangan Dataset Pengujian (Skenario Small, Medium, Large) |
| 6 | **Tabel 6** | Subbab 3.4.4.2 | Spesifikasi Skema Kunci dan Struktur Penyimpanan Redis Cache-Aside |
| 7 | **Tabel 7** | Subbab 3.4.5 | Spesifikasi Lingkungan Pengujian Perangkat Keras dan Perangkat Lunak |

*(Catatan Alternatif: Jika Matriks Desain Eksperimen di Subbab 2.4.3 tetap disajikan dalam bentuk narasi matematika tanpa tabel, maka total tabel inti proposal menjadi **6 Tabel**, yaitu Tabel 1 s.d. Tabel 6).*

---

## 5. STATUS DOKUMEN & RENCANA TINDAK LANJUT (SESI 9)

### A. Status Saat Ini
1. **Bab I, II, dan III:** Teks narasi lengkap telah terpasang di `proposal.md` (mencakup Subbab 3.5 Rencana Analisis hasil Sesi 7).
2. **Inventaris Gambar (7 Gambar):** Telah teridentifikasi dan tervalidasi 100% (5 di Bab III + 2 di Lampiran A).
3. **Inventaris Tabel (7 Tabel):** Telah dipetakan solusinya untuk memperbaiki defect penomoran dan tabel nyasar di `proposal.md`.

### B. Agenda Prioritas Sesi Berikutnya (Sesi 9)
1. **Pembersihan & Refaktorisasi Markdown `proposal.md`:**
   * Memperbaiki penomoran tabel agar berurutan rapi sesuai standar Tabel 1 s.d. Tabel 7.
   * Menghapus duplikasi tabel Redis di atas Gambar 5 dan di atas Tabel Lingkungan Pengujian.
   * Mengembalikan isi fisik tabel spesifikasi lingkungan pengujian perangkat keras dan perangkat lunak ke tempat yang benar.
   * Menyematkan caption resmi Gambar 4 di Subbab 3.4.4.1.
2. **Penyusunan Bagian Lampiran A:**
   * Membuat bagian baru di akhir `proposal.md` berjudul `# LAMPIRAN A. BUKTI OBSERVASI AWAL PROFILING LARAVEL TELESCOPE`.
   * Memasukkan sub-bagian A.1 dan A.2 lengkap dengan caption singkat dan 1–2 kalimat narasi penjelas akademis.
3. **Pemeriksaan Konsistensi Akhir:** Melakukan audit cross-reference (sitasi Gambar dan Tabel di dalam teks paragraf) sebelum proposal final dicetak atau diajukan ke dosen pembimbing/penguji.
