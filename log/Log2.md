# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 2)
**File:** `log/Log2.md`  
**Waktu Pembaruan:** 2026-10-21  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi dokumen *context handoff* lengkap untuk sesi percakapan baru agar AI dan pengguna dapat langsung memahami seluruh status kemajuan, keputusan metodologi, penyelarasan istilah, dan teks proposal yang telah terverifikasi tanpa kehilangan konteks historis.

---

## 1. IDENTITAS PENELITIAN & OPSI JUDUL

### A. Judul Asli Manuskrip Ilmiah (Bahasa Inggris):
> *"The System Knows Your Team's Performance, But Takes Forever to Tell You": A Comparative Study of Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application*

### B. Opsi Judul Formal Bahasa Indonesia untuk Tugas Akhir (Polibatam):
1. **Opsi 1 (Pola Studi Komparasi Standar - Sangat Direkomendasikan):**  
   `STUDI KOMPARASI STRATEGI CACHING DAN OPTIMASI KUERI ORM PADA MODUL DECISION SUPPORT SYSTEM DALAM APLIKASI COLLABORATIVE VIRTUAL OFFICE: STUDI KASUS KOLADI`
2. **Opsi 2 (Gaya Implementasi Rekayasa Spesifik - Senada dengan Judul Dzakwan):**  
   `IMPLEMENTASI REDIS CACHE-ASIDE DAN OPTIMASI KUERI ELOQUENT ORM UNTUK MENINGKATKAN KINERJA MODUL DECISION SUPPORT SYSTEM: STUDI KASUS KOLADI`
3. **Opsi 3 (Ringkas dan Padat Teknis):**  
   `STUDI KOMPARASI KINERJA REDIS CACHE-ASIDE DAN OPTIMASI KUERI ORM PADA MODUL DECISION SUPPORT SYSTEM APLIKASI KOLADI`
4. **Opsi 4 (Mempertahankan Hook Asli):**  
   `"SISTEM MENGETAHUI PERFORMA TIM, TETAPI MEMBUTUHKAN WAKTU LAMA UNTUK MENAMPILKANNYA": STUDI KOMPARASI STRATEGI CACHING DAN OPTIMASI KUERI ORM PADA MODUL DECISION SUPPORT SYSTEM APLIKASI KOLADI`

---

## 2. ARSITEKTUR SISTEM & RUANG LINGKUP PENELITIAN

* **Objek Penelitian:** Modul *Decision Support System* (DSS) pada aplikasi **Koladi** (*Collaborative Virtual Office*, SaaS multi-tenant).
* **Tech Stack:** Laravel 12.x, PHP ^8.2, PostgreSQL 15, Eloquent ORM, Redis 7 (via `phpredis`), Docker Compose (`koladi_net`).
* **Endpoint Uji:**
  1. `GET /statistik/suggestions` (Rekomendasi manajerial berbasis 5 aturan evaluasi).
  2. `GET /statistik/modal-data` (Payload DSS lengkap: ringkasan metrik, tugas mendesak, distribusi beban kerja).
* **Masalah Baseline (Penyebab Inefisiensi):**
  1. *Database-backed cache*: Snapshot disimpan di tabel relasional `workspace_performance_snapshots` sehingga cache hit tetap memicu kueri SQL; endpoint `modal-data` mengalami *partial cache hit*.
  2. *Unnecessary Data Retrieval*: Pemuatan relasi `assignedUsers` tanpa batasan kolom pada `getTasksInPeriod()`.
  3. *N+1 Query Problem*: Relasi `checklists` pada `getProgressPercentage()` dipanggil tanpa eager loading.
  4. *Query-in-loop*: Pemanggilan `Task::whereHas()` di dalam perulangan anggota workspace pada `getWorkloadDistribution()`.
* **Intervensi Solutif (Diterapkan Simultan):**
  1. **Redis Cache-Aside**: Redis dicek terlebih dahulu; kalkulasi ke PostgreSQL hanya terjadi saat *cache miss*; *cache invalidation* berbasis event Laravel Observer (`TaskObserver` dan `TaskAssignmentObserver`).
  2. **Optimasi Kueri Eloquent**: *Selective eager loading* `with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`, serta refaktorisasi `getWorkloadDistribution()` menjadi *batch retrieval* dan agregasi *in-memory*.

---

## 3. STATUS CAPAIAN NASKAH PROPOSAL (`proposal.md`)

File utama [proposal.md](file:///c:/xampp/htdocs/TugasAkhir/proposal.md) saat ini telah selesai dan sinkron hingga **Bab II (total 204 baris)** dengan rincian:

### A. BAB I: PENDAHULUAN
* **1.1. Latar Belakang:** 4 paragraf organis (Tren kerja terdistribusi & peran DSS $\rightarrow$ Profiling Telescope & inefisiensi arsitektural Koladi $\rightarrow$ Metrik diagnostik terpadu $\rightarrow$ Solusi simultan, research gap, dan positioning rekayasa perangkat lunak).
* **1.2. Rumusan Masalah:** Satu pertanyaan komparatif berfokus pada pengaruh penerapan simultan Redis Cache-Aside dan optimasi ORM terhadap 4 metrik teknis.
* **1.3. Tujuan Penelitian:** 3 poin terukur (menerapkan, membandingkan performa teknis secara kuantitatif, menganalisis signifikansi statistik dan magnitude perbaikan).
* **1.4. Batasan Masalah:** 6 batasan teknis ketat (fokus modul DSS Koladi 2 endpoint, tanpa concurrent/load/stress test, dataset sintetis 3 skala, pengujian sekuensial Docker, evaluasi read & write overhead, tanpa pengujian UX/kepuasan pengguna).
* **1.5. Manfaat Penelitian:** 2 paragraf padat (Manfaat Teoretis & Manfaat Praktis).

### B. BAB II: TINJAUAN PUSTAKA
* **2.1. Penelitian Terkait:** Sintesis komprehensif 5 studi terkini (Putra et al., 2025; Privalov & Stupina, 2024; Turcotte et al., 2022; Yusmita et al., 2025; Kundan & Makwana, 2025). Diakhiri oleh narasi *research gap* dan ditutup langsung oleh **Tabel 1 (Perbandingan Penelitian Sebelumnya)** tanpa trailing text.
* **2.2. Landasan Teori:** Terdiri atas 4 sub-subbab terfokus:
  * `2.2.1. Konsep Decision Support System (DSS) Berbasis Web` (Cai & Chen, 2021; Koornneef et al., 2020).
  * `2.2.2. Arsitektur Caching dan Redis` (Taipalus, 2024; An & Cao, 2022).
  * `2.2.3. Teori ORM dan Inefisiensi Kueri Eloquent` (Alshemaimri et al., 2021; Turcotte et al., 2022; Yusmita et al., 2025).
  * `2.2.4. Profil Objek Penelitian: Aplikasi Koladi` (Modul DSS, 22 metrik, kontras baseline vs optimized).
* **2.3. Metode Pengembangan Produk:** Menggunakan kerangka **Refactoring Lifecycle** (Mens & Tourwé, 2004) dan prinsip *behavior preservation* (Fowler, 2018). Penomoran formal `2.3.1` s.d. `2.3.6` dengan formula template kampus (*"Menurut [Penulis, Tahun], tahap ini bertujuan ... Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah ..."*):
  * `2.3.1 Identifikasi Lokasi Refaktorisasi (Identify Where to Refactor)`
  * `2.3.2 Penentuan Teknik Refaktorisasi (Determine Refactoring Techniques)`
  * `2.3.3 Penjaminan Preservasi Perilaku (Guarantee Behavior Preservation)` (Pengujian verifikasi regresi otomatis / equivalence tests untuk memastikan 22 metrik identik).
  * `2.3.4 Penerapan Refaktorisasi (Apply the Refactoring)`
  * `2.3.5 Evaluasi Efek Kualitas (Assess Quality Effect)` (4 metrik teknis).
  * `2.3.6 Pemeliharaan Konsistensi Sistem (Maintain Consistency)` (Validasi Docker `koladi_net` dan pembaruan codebase modul DSS).
* **2.4. Metode Analisis/Riset yang Digunakan:** Menggunakan kerangka **Experimental Software Engineering** (Wohlin et al., 2000; 2012) dan **Controlled Before-After Benchmarking** (Henning & Hasselbring, 2022). Terbagi menjadi sub-subbab formal:
  * `2.4.1. Desain Penelitian dan Isolasi Lingkungan` (Eksekusi sekuensial terisolasi dalam Docker `koladi_net`).
  * `2.4.2. Variabel Penelitian` (Variabel bebas: konfigurasi, kondisi cache, skala data S/M/L, endpoint; Variabel terikat: response time, SQL query count, memory utilization, cache hit ratio).
  * `2.4.3. Instrumen dan Protokol Pengujian` (Middleware kustom Laravel, Node.js runner, CSV validator, Telescope; 3 warm-up requests dibuang, 10 repetisi valid, total matriks $3 \times 2 \times 2 \times 2 \times 10 = \mathbf{240\text{ observasi valid}}$).
  * `2.4.4. Metode Analisis Data dan Uji Hipotesis` (Statistik deskriptif, formula persentase perbaikan dengan batas noise < 3%, Paired-Sample t-Test $\alpha = 0,05$, kaidah khusus data deterministik SQL query $p < 0,001$, serta evaluasi Cache Hit Ratio bersama Query Count).

---

## 4. KEPUTUSAN PENTING & KLARIFIKASI METODOLOGIS SESI INI

1. **Separation of Concerns Subbab 2.3 vs 2.4:**
   * **Subbab 2.3** adalah ranah *Software Engineering* (cara merombak kode, menjaga rumus tidak rusak, dan mengelola codebase di Koladi).
   * **Subbab 2.4** adalah ranah *Sains/Riset Eksperimental* (metode pengumpulan data 240 observasi, perlakuan warm-up, dan uji hipotesis statistik t-test).
   * Diskusi memastikan bahwa modifikasi pada Subbab 2.3 **tidak mengubah** struktur Subbab 2.4 karena fungsinya memang berbeda.
2. **Klarifikasi Pengujian Preservasi Perilaku (Tahap 3):**
   * Manuskrip jurnal ilmiah tidak menyebutkan detail pengujian unit test karena ekuivalensi fungsional dianggap *implied premise*. Namun proposal TA TRPL wajib mencantumkannya.
   * Redaksi dihaluskan menjadi *"pengujian verifikasi regresi otomatis (automated regression/equivalence tests)"* agar mahasiswa tidak terkunci kaku pada Pest PHP, melainkan bebas menggunakan script perbandingan JSON diff / validator / PHPUnit saat pelaksanaan.
3. **Klarifikasi Dokumentasi & Version Control (Tahap 6):**
   * Redaksi dihaluskan dari *"repositori Git"* menjadi *"dokumentasi teknis arsitektur dan pembaruan basis kode (codebase) modul DSS Koladi"* agar tidak memunculkan pertanyaan teknis berlebih dari dosen terkait branching Git.
4. **Klarifikasi Uji Hipotesis t-Test vs Kaidah Deterministik (Subbab 2.4.4 Opsi 2):**
   * Uji Paired-Sample t-Test ($\alpha = 0,05$) diterapkan untuk data stokastik yang memiliki variasi (Response Time dan Memory Utilization).
   * Kaidah khusus metrik deterministik (SQL Query Count) dijelaskan secara terpisah: karena variansi repetisinya nol ($SD = 0$), $p$-value matematis bernilai 0,000000 sehingga dilaporkan secara ilmiah sebagai $p < 0,001$ dan difokuskan pada penurunan absolut kueri.
   * Cache Hit Ratio ditegaskan tidak diuji t-test sendirian, melainkan dianalisis berdampingan (*joint interpretation*) dengan SQL Query Count untuk memverifikasi ada tidaknya bypass basis data yang sesungguhnya.

---

## 5. REKOMENDASI LANGKAH SELANJUTNYA UNTUK SESI BARU

1. **Menetapkan Judul Formal Bahasa Indonesia:** Menempatkan judul pilihan (Opsi 1 atau Opsi 2) di baris paling atas `proposal.md`.
2. **Menyusun Bab III (Metodologi Penelitian / Prosedur Teknis Detail):** Menyusun rancangan arsitektur sebelum-sesudah, alur diagram eksperimen, spesifikasi environment hardware/software, seeder generator data, dan prosedur eksekusi benchmark runner.
3. **Penyusunan Daftar Pustaka (References):** Mengumpulkan seluruh sitasi dari Bab I dan Bab II (Wohlin, Fowler, Mens & Tourwé, Bassil, Al-Saqqa, Putra, Privalov, Turcotte, Yusmita, Kundan & Makwana, Taipalus, An & Cao, dll.) ke dalam format APA 7th style yang rapi.
