# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 4)
**File:** `TugasAkhir/log/Log4.md`  
**Waktu Pembaruan:** 2026-10-25  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan pada Sesi 4 mengenai verifikasi implementasi kode, justifikasi metodologi refaktorisasi (Subbab 2.3.3 & 2.3.6), dekonstruksi instrumen benchmark (Subbab 2.4.3), serta penyelarasan akademik dengan literatur review dan basis kode aktual `koladi-laravel`.

---

## 1. PENJAMINAN PRESERVASI PERILAKU (SUBBAB 2.3.3)

### A. Status Implementasi di Codebase
* **Pertanyaan Awal:** Apakah pengujian verifikasi regresi otomatis (*automated regression/equivalence tests*) untuk 22 metrik performa tim dan 5 rekomendasi manajerial DSS sudah diterapkan di repositori `koladi-laravel`?
* **Hasil Pengecekan Kode:**
  * Di direktori `tests/Feature/` dan `tests/Unit/`, berkas pengujian ekuivalensi otomatis untuk membandingkan luaran JSON 22 metrik dan 5 rekomendasi **memang belum dibuat**.
  * Berkas pengujian yang saat ini ada adalah `tests/e2e/task-dss.spec.js` (Playwright E2E UI visual), `app/Console/Commands/VerifyObserversCommand.php`, dan `scripts/verify_langkah1_task_observer.php`.
  * **Kesimpulan:** Pada tahap proposal, status narasi di Subbab 2.3.3 adalah **komitmen/janji metodologis (*deliverable*)** yang sah secara akademik dan akan diwujudkan kodenya pada saat pelaksanaan penelitian.

### B. Kesesuaian Metodologis & Kelayakan Teknis (Realistis)
* **Kesesuaian:** SANGAT SESUAI (100%). Menjawab syarat mutlak *behavior-preserving* menurut Fowler (2018) dan Mens & Tourwé (2004), serta membuktikan batasan masalah di Bab I bahwa formula metrik dan kontrak API tidak diubah.
* **Kelayakan (Feasibility):** SANGAT REALISTIS dan mudah diimplementasikan di Laravel.
  * Sifat komputasi DSS bersifat deterministik pada dataset seeder terkontrol (`ResearchDataSeeder.php`).
  * Menggunakan pendekatan **Golden Master / Snapshot Testing**: respon JSON baseline disimpan sebagai acuan (`baseline_expected_dss.json`), lalu test case Laravel (`tests/Feature/DSSEquivalenceTest.php`) memanggil endpoint optimized dan mencocokkan nilainya menggunakan `$this->assertEquals()`.

### C. Pemahaman Konseptual Istilah
* **Regresi (*Software Regression*):** Kondisi ketika fungsionalitas yang tadinya berjalan normal tiba-tiba menjadi rusak, salah hitung, atau meleset akibat perubahan struktur internal kode baru.
* **Verifikasi Ekuivalensi (*Equivalence Testing*):** Membuktikan bahwa input data yang sama menghasilkan luaran semantik/data JSON yang 100% identik sebelum (*before*) dan sesudah (*after*) refaktorisasi.
* **Otomatis (*Automated*):** Dijalankan secara terprogram melalui skrip *test runner* (PHPUnit/Pest), bukan pengecekan manual satu per satu di browser.

---

## 2. PEMELIHARAAN KONSISTENSI SISTEM (SUBBAB 2.3.6)

Menurut Mens & Tourwé (2004), refaktorisasi memicu perambatan perubahan (*change propagation*) yang harus dijaga konsistensinya melalui 3 dimensi terpadu:

### A. Dimensi 1: Stabilitas Lingkungan Eksekusi (Infrastruktur)
* Memverifikasi orkestrasi kontainer Docker Compose (`koladi_net`) berjalan stabil, di mana trio layanan: Laravel 12 (`koladi_app_local`), PostgreSQL 15 (`koladi_db_local`), dan Redis 7 (`koladi_redis_local`) berstatus *Up/Healthy*.

### B. Dimensi 2: Ketiadaan Efek Samping pada Modul Kanban (*Side-Effects Verification*)
* **Mengapa HANYA Modul Kanban?**
  * Berdasarkan prinsip *Change Impact Analysis* & *Regression Test Selection* (Mens & Tourwé, 2004; Harrold, 2000), pengujian tidak perlu menguji seluruh 50 fitur Koladi (chat, dokumen, tagihan, dll.) karena kode fitur-fitur tersebut tidak pernah dimodifikasi.
  * Modul Manajemen Tugas (Kanban) adalah satu-satunya titik pertautan langsung (*coupling point*) tempat `TaskObserver` dan `TaskAssignmentObserver` disusupi pada siklus hidup tugas (`Task::observe()` di `AppServiceProvider`).
* **Apa yang Diuji?**
  * Memastikan penambahan observer untuk invalidasi cache Redis di latar belakang tidak menimbulkan *exception* (error 500) yang memacetkan alur kerja penugasan (operasi *Create, Read, Update, Delete, Assign/Unassign* di papan Kanban tetap berjalan normal).
* **Instrumen di Repo:** Telah dibuat dan dapat diverifikasi via `php artisan dss:verify-observers` serta Playwright test `tests/e2e/task-dss.spec.js`.

### C. Dimensi 3: Penyelarasan Artefak Nonkode
* **Bukan beban terpisah di luar skripsi.** Seluruh pembaruan sudah menyatu di naskah proposal/laporan TA:
  1. Pembaruan file konfigurasi: `.env.example` dan `.env.docker.local.example` (penambahan variabel `REDIS_HOST`, `REDIS_PORT`, `CACHE_STORE=redis`).
  2. Pembaruan diagram arsitektur sistem: digambar di Bab III proposal/laporan (menyisipkan Redis Cache-Aside dan Observers).
  3. Dokumentasi teknis API DSS: mencatat kontrak payload 2 endpoint DSS dan aturan TTL/key Redis di Bab III serta berkas `Dokumentasi_Benchmark_DSS.md`.

---

## 3. INSTRUMEN DAN PROTOKOL PENGUJIAN (SUBBAB 2.4.3)

### A. Alur Kerja Sebenarnya & Dekonstruksi Peran 4 Instrumen
* **Jebakan Metodologis yang Diluruskan:**  
  * Laravel Telescope **BUKAN** alat pengukur 4 metrik data baseline! Pengukuran baseline dan optimized **wajib menggunakan instrumen yang identik** agar adil (*fair comparison*).
  * Laravel Telescope hanya digunakan pada **Fase Profiling Awal** (stetoskop diagnosis masalah). Setelah N+1 dan loop query teridentifikasi, Telescope dimatikan agar tidak menimbulkan *probe/observer effect* (overhead pencatatan log) yang merusak akurasi benchmark.
* **Pembagian Tugas Instrumen Benchmarking:**
  1. **Node.js Benchmark Runner (`scripts/benchmark-dss.js`) — Sisi Klien (*Black-Box*):**
     * Mengatur urutan HTTP sekuensial terisolasi.
     * Mengorkestrasi 3 *warm-up requests* (dibuang) dan 10 repetisi valid per sel kombinasi.
     * Mengatur kondisi *cache miss* (flush) dan *cache hit* (pre-warm).
     * Mengukur parameter **`response_time_ms`** secara presisi menggunakan `process.hrtime()`.
  2. **Middleware Instrumentasi Laravel (`BenchmarkInstrumentation.php`) — Sisi Server (*White-Box*):**
     * Mengukur parameter internal PHP/DB yang tidak dapat dilihat dari luar:
       - **`query_count`**: dihitung via listener `DB::listen()`.
       - **`memory_mb`**: dihitung via `memory_get_peak_usage(true)`.
       - **`cache_hit_ratio`**: dihitung via delta Redis `keyspace_hits` & `keyspace_misses`.
     * Mengirimkan ketiga metrik ke header respons HTTP untuk dibaca oleh Node.js Runner.
  3. **CSV Validator (`scripts/validate-csv.js`) — Sisi Integritas Data:**
     * Memverifikasi keutuhan 120 baris per konfigurasi (total 240 observasi valid).
     * Memastikan ke-12 kombinasi unik ($3 \times 2 \times 2$) terisi lengkap tanpa ada data *warm-up* yang bocor.
     * Menjamin terpenuhinya asumsi *balanced design* sebelum data diuji via *Paired-Sample t-Test*.

### B. Kejujuran Akademik terkait Sitasi (Teori vs. Rekayasa)
* **Klarifikasi:** Peneliti terdahulu (Turcotte, Yusmita, Henning, Wohlin, dll.) **tidak membuat Middleware Laravel atau skrip Node.js Koladi**. Keempat instrumen fisik tersebut adalah **karya rekayasa peneliti sendiri (*custom instrumentation*)**.
* **Fungsi Sitasi:** Sitasi digunakan untuk merujuk pada **prinsip pengukuran dan protokol teknisnya**:
  - `memory_get_peak_usage` & internal SQL tracing $\rightarrow$ Turcotte et al. (2022); Yusmita et al. (2025).
  - Eksekusi sekuensial terisolasi, client-side latency, & 3 warm-up dibuang $\rightarrow$ Henning & Hasselbring (2022); Japke et al. (2025).
  - Validitas data berpasangan & integritas sampel $\rightarrow$ Wohlin et al. (2000); Uddin & Lu (2024).
* **Format Penulisan yang Aman:**
  * Kalimat pembuka Subbab 2.4.3 dipayungi oleh sitasi metodologi: *(Wohlin et al., 2000; Henning & Hasselbring, 2022)*.
  * Poin-poin instrumen ditulis secara ringkas dan fungsional (serupa dengan format proposal Dzakwan).
  * Pada poin Middleware, ditegaskan pencatatan metrik ke-4 (*rasio cache Redis*).

### C. Penyederhanaan Bahasa Protokol Cold-Start & Warm-up
* Redaksi yang lebih ringan dan mengalir:
  > *"Guna menjaga keandalan data dari lonjakan beban awal server (*cold-start*), setiap pengujian diawali dengan 3 kali permintaan pemanasan (*warm-up requests*) yang datanya tidak dimasukkan ke dalam analisis (Henning & Hasselbring, 2022; Traini et al., 2023; Japke et al., 2025)."*

---

## 4. STATUS KELENGKAPAN BASIS KODE (`koladi-laravel`)

Cross-check menyeluruh antara naskah proposal dan repositori membuktikan **kesesuaian 100%**:
1. `app/Http/Middleware/BenchmarkInstrumentation.php` (aktif via header `X-Benchmark-Mode: 1`).
2. `scripts/benchmark-dss.js` (runner 3 warm-up, 10 repetisi, 3 volume data).
3. `scripts/validate-csv.js` (validasi 120 baris per berkas).
4. `database/seeders/ResearchDataSeeder.php` (Small 50, Medium 200, Large 500).
5. `storage/app/benchmark/results_baseline_fixed.csv` (120 baris valid baseline).
6. `storage/app/benchmark/results_optimized.csv` (120 baris valid optimized).
7. `scripts/analyze-statistics.js` (skrip uji Paired-Sample t-Test dan statistik deskriptif).
8. `app/Observers/TaskObserver.php` & `TaskAssignmentObserver.php` (invalidasi cache).
9. `app/Services/DSS/DSSService.php` & `MetricsCalculator.php` (Redis Cache-Aside & selective eager loading).

---

## 5. LANGKAH SELANJUTNYA
1. Mengintegrasikan draf final Subbab 2.4.3 ke dalam naskah utama `proposal.md`.
2. Melanjutkan penulisan draf Bab III (Alat dan Bahan, Alur Penelitian, serta Rencana Perancangan Sistem) dengan mengacu pada pemetaan arsitektur Koladi dan format standar kampus.
