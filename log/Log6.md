# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 6)
**File:** `TugasAkhir/log/Log6.md`  
**Waktu Pembaruan:** 2026-10-26  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan pada Sesi 6 mengenai:
1. Finalisasi narasi akademis Subbab 3.4.3 (Dataset Pengujian) dan 3.4.4 (Transformasi ORM & Skema Redis).
2. Penyelesaian perdebatan konsep teknis: peranan TTL di backend vs pengujian Node.js, penyamaan status payload utuh kedua endpoint, dan verifikasi eliminasi partial cache hit.
3. Audit keselarasan 100% antara draf proposal dan implementasi kode di basis data/codebase (`DSSService.php`, `docker-compose.local.yml`, `BenchmarkInstrumentation.php`, `benchmark-dss.js`, `validate-csv.js`).
4. Perancangan Subbab 3.4.5 (Lingkungan Pengujian dan Dual-Layer Instrumentation) serta perbandingan dengan preseden Dzakwan.
5. Panduan teknis Git Commit, Push, dan Pull untuk arsitektur repositori utama ber-submodul.

---

## 1. REKAPITULASI PEMBAHASAN & KEPUTUSAN SUBBAB 3.4

### A. Subbab 3.4.3 (Rancangan Dataset Pengujian)
* **Koreksi Klaim Realistis:** Menghilangkan klaim tidak berdasar mengenai "skala operasional startup nyata/realistis". Digantikan dengan justifikasi metodologis baku: **"simulasi kepadatan beban kerja per-workspace dalam lingkungan SaaS multi-tenant"**.
* **3 Skenario Skala Data (Tabel 4):**
  * *Small*: 50 tugas, 5 anggota workspace (skala pembanding awal).
  * *Medium*: 200 tugas, 15 anggota workspace (skala menengah).
  * *Large*: 500 tugas, 30 anggota workspace (skala beban puncak terkontrol).
* **Alasan Skala 500 Tugas sebagai Batas Atas:**
  Pada baseline, skala 500 tugas telah memicu hingga 696 kueri SQL dan latensi 679 ms, sehingga eskalasi inefisiensi $N+1$ query sudah terbukti sangat jelas sebelum melampaui alokasi memori runtime PHP lingkungan lokal.
* **Justifikasi Data Sintetis Terstruktur (*Seeded Dataset*):**
  Menggunakan `ResearchDataSeeder.php` dengan proporsi status tugas konstan: 40% *completed*, 35% *in-progress*, 25% *to-do/overdue*, serta 2–8 checklist per tugas. Penggunaan data sintetis ini diadopsi berdasarkan prinsip *controlled software experiment* (Wohlin et al., 2000; Henning & Hasselbring, 2022; Yusmita et al., 2025) guna meniadakan variabel perancu (*confounding variables*) akibat skew/distribusi acak data produksi riil.

---

### B. Subbab 3.4.4 (Transformasi Kueri ORM dan Skema Cache Redis)

#### 1. Transformasi Kueri Eloquent ORM (Gambar 4 dari `perubahan.php`)
* **Gambar 4:** Menggunakan cuplikan kode dari [perubahan.php](file:///c:/xampp/htdocs/koladi/perubahan.php) (tidak memerlukan tabel perbandingan dobel agar ringkas dan elegan):
  * **Kasus 1 (`getTasksInPeriod`):** Transformasi `->with(['assignedUsers'])` menjadi `->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`. Menghilangkan antipola *Unnecessary Data Retrieval* (Alshemaimri et al., 2021) dan mengompresi lazy-loading $N+1$ pada relasi checklists menjadi 1 kueri konstan $O(1)$ (Turcotte et al., 2022).
  * **Kasus 2 (`getWorkloadDistribution`):** Mengeluarkan kueri kondisional `Task::whereHas()` dari blok perulangan `foreach ($members)` menjadi penarikan batch satu kali `Task::with(['assignedUsers:id'])->where(...)->get()` yang dilanjutkan dengan filter koleksi memori PHP `$tasks->filter(...)`. Mereduksi kompleksitas kueri dari linier $O(M)$ menjadi konstan $O(1)$.
* **Penyajian Teks:** Ditulis dalam dua paragraf naratif yang mengalir (*cohesive narrative flow*) tanpa pemenggalan sub-heading kaku ("Kasus 1", "Kasus 2").

#### 2. Klarifikasi Konseptual TTL & Payload Utuh (Redis Cache-Aside)
* **Perdebatan TTL (Time-to-Live):**
  * *Perspektif Pengujian:* Pada skrip pengujian Node.js (`benchmark-dss.js`), kondisi *hit/miss* dikendalikan secara manual (skrip melakukan flush cache sebelum *cache miss* dan menembak langsung berurutan untuk *cache hit*).
  * *Perspektif Sistem Backend:* Pada kode aplikasi [DSSService.php](file:///c:/xampp/htdocs/koladi/app/Services/DSS/DSSService.php), perintah `Redis::setex($key, 3600, json_encode($data))` **wajib** menyertakan parameter durasi agar memori RAM server tidak bocor (*memory leak*) oleh data lama.
  * *Solusi di Proposal:* Penjelasan TTL dipangkas dari pembahasan teoretis yang berlebihan menjadi satu kalimat catatan praktis mengenai batas retensi memori RAM server.
* **Penyamaan Konsep Payload Utuh (Tabel 5):**
  * Kedua endpoint (`/suggestions` dan `/modal-data`) sama-sama menyimpan **100% full payload** (`JSON String`) di Redis dan sama-sama mencapai **0 kueri komputasi analitis DSS saat cache hit**.
  * Istilah "terkonsolidasi utuh" ditegaskan khusus untuk endpoint `/modal-data` sebagai solusi penyelesaian cacat desain **_partial cache hit_** pada baseline (di mana baseline lama hanya menyimpan ringkasan metrik di tabel database, sedangkan *urgent tasks* dan *workload* dipaksa dihitung ulang dari basis data PostgreSQL).

#### 3. Hasil Audit Kode Nyata terhadap Draf 3.4.4.2:
* **Facade:** Kode menggunakan `use Illuminate\Support\Facades\Redis;` langsung (bukan *Cache facade*). Di proposal disesuaikan menjadi **Laravel Redis facade**.
* **Key Pattern:** Sesuai fungsi `buildRedisKey()` di `DSSService.php`:
  * Suggestions: `dss:suggestions:workspace_{workspace_id}:start_{start_date}:end_{end_date}`
  * Modal-data: `dss:modal-data:workspace_{workspace_id}:start_{start_date}:end_{end_date}`
* **Dampak Kueri:** Pada data riil benchmark (`results_optimized.csv`), kueri total tercatat 6 kueri (kueri bawaan framework Laravel untuk auth user dan cek workspace), namun kueri komputasi DSS terbukti **0 kueri** (turun drastis dari 32 kueri pada baseline cache hit). Proposal menggunakan istilah presisi: **"0 kueri komputasi analitis DSS saat cache hit"**.

---

### C. Subbab 3.4.5 (Rancangan Lingkungan Pengujian dan Instrumentasi)
* **Perbandingan dengan Proposal Dzakwan:**
  * Dzakwan hanya menguji kueri PostgreSQL langsung via `EXPLAIN ANALYZE` (tanpa request web HTTP, tanpa Redis, tanpa pengukuran memori).
  * Penelitian Naufal mengevaluasi siklus HTTP end-to-end dengan 4 metrik kinerja, sehingga membutuhkan **Sistem Instrumentasi Dua Arah (*Dual-Layer Instrumentation*)**.
* **Verifikasi Komponen Nyata di Basis Kode Koladi:**
  1. *Client-side Runner:* [scripts/benchmark-dss.js](file:///c:/xampp/htdocs/koladi/scripts/benchmark-dss.js) (mengukur *response time* presisi nanodetik via `process.hrtime.bigint()`, 3 warm-ups dibuang, 10 repetisi valid, flush cache miss via Artisan).
  2. *Server-side Middleware:* [app/Http/Middleware/BenchmarkInstrumentation.php](file:///c:/xampp/htdocs/koladi/app/Http/Middleware/BenchmarkInstrumentation.php) (merekam kueri SQL via `DB::listen`, alokasi RAM puncak via `memory_get_peak_usage(true)`, dan keyspace stats Redis, terdaftar aktif di [bootstrap/app.php](file:///c:/xampp/htdocs/koladi/bootstrap/app.php)).
  3. *Verifikator CSV:* [scripts/validate-csv.js](file:///c:/xampp/htdocs/koladi/scripts/validate-csv.js) (memvalidasi tepat 120 baris per konfigurasi = 240 baris total, tanpa data warm-up).
  4. *Lingkungan Kontainer:* [docker-compose.local.yml](file:///c:/xampp/htdocs/koladi/docker-compose.local.yml) (jaringan `koladi_net`, layanan `koladi_app_local`, `koladi_nginx_local`, `koladi_db_local` PostgreSQL 15, `koladi_redis_local` Redis 7).
  5. *Spesifikasi Komputer Pengujian (Tabel 6):* Apple M2 Pro, RAM 16 GB, SSD 1 TB, macOS Sequoia (sinkron dengan Tabel 1 [Manuscript_Anonymous.md](file:///c:/xampp/htdocs/koladi/TugasAkhir/konteks/Manuscript_Anonymous.md)).
* **Format Bersih Tanpa Tabel Dobel:**
  * **Gambar 5:** Satu diagram visual kotak-kotak topologi pengujian (Host OS Node.js $\leftrightarrow$ Docker network `koladi_net` $\leftrightarrow$ 3 kontainer layanan).
  * **Narasi:** Penjelasan dual instrumentasi (klien dan server).
  * **Tabel 6:** Spesifikasi hardware dan software.
  * **Paragraf Kontrol:** Pengendalian *background noise* dan isolasi sekuensial.

---

## 2. PANDUAN TEKNIS GIT: COMMIT, PUSH, DAN PULL (ARTEFAK DENGAN SUBMODUL)

### A. Struktur Relasi Git
1. **Repositori Utama:** `koladi` (remote: `https://github.com/Dzakwan993/koladi.git`, branch: `penelitian-naufal`).
2. **Submodul Mandiri:** `TugasAkhir` (remote: `https://github.com/naufal2244/TugasAkhir`, branch: `main`).

> [!IMPORTANT]
> **Aturan Emas Git Submodule:**
> Submodul `TugasAkhir` adalah repositori git tersendiri di dalam repositori `koladi`. 
> Perubahan di dalam `TugasAkhir` **harus di-commit dan di-push terlebih dahulu ke repositorinya sendiri**, baru kemudian repositori utama `koladi` meng-commit pembaruan referensi commit submodul tersebut.

---

### B. Prosedur Commit & Push (Di Laptop Saat Ini)

Jalankan perintah berikut secara berurutan di terminal:

#### Langkah 1: Commit dan Push Submodul `TugasAkhir`
```bash
# Masuk ke folder submodul TugasAkhir
cd c:\xampp\htdocs\koladi\TugasAkhir

# Pastikan berada di branch main
git checkout main

# Tambahkan seluruh perubahan naskah dan log
git add proposal.md "Lit Review/" log/Log6.md

# Commit perubahan di submodul
git commit -m "Update proposal bab 3 (3.4.2-3.4.5), sinkronisasi kode, dan log sesi 6"

# Push ke repositori TugasAkhir di GitHub
git push origin main
```

#### Langkah 2: Commit dan Push Repositori Utama `koladi`
```bash
# Kembali ke root folder proyek koladi
cd c:\xampp\htdocs\koladi

# Tambahkan perubahan referensi submodule TugasAkhir
git add TugasAkhir

# Commit di repositori utama
git commit -m "chore: update pointer submodule TugasAkhir ke commit sesi 6"

# Push ke repositori koladi di GitHub
git push origin penelitian-naufal
```

---

### C. Prosedur Pull (Di Laptop Rumah / Laptop Lain)

Ketika kamu membuka proyek ini di laptop rumah, gunakan panduan berikut:

#### Kasus 1: Repositori Proyek Sudah Ada di Laptop Rumah (Tinggal Update)
Jika folder `koladi` sudah pernah di-clone sebelumnya di laptop rumah:
```bash
# Masuk ke folder koladi di laptop rumah
cd path/ke/koladi

# Pastikan berada di branch penelitian-naufal
git checkout penelitian-naufal

# Tarik perubahan terbaru di repo utama
git pull origin penelitian-naufal

# Perbarui konten submodul TugasAkhir agar sinkron dengan commit terbaru
git submodule update --init --recursive
```
*(Atau cukup satu perintah singkat:* `git pull --recurse-submodules`*)*.

#### Kasus 2: Baru Pertama Kali Clone di Laptop Rumah
Jika laptop rumah belum ada folder proyeknya:
```bash
# Clone sekaligus mengunduh isi submodul secara otomatis
git clone --recurse-submodules -b penelitian-naufal https://github.com/Dzakwan993/koladi.git
```

---

## 3. STATUS DOKUMEN & ROADMAP SESI BERIKUTNYA (SESI 7)

| Bagian Dokumen | Status Saat Ini | Rencana Aksi Sesi 7 |
| :--- | :--- | :--- |
| **Subbab 3.4.2** | Final (Sequence Diagram Gambar 3 siap) | Disisipkan ke `TugasAkhir/proposal.md` |
| **Subbab 3.4.3** | Final (Tabel 4 Dataset Sintetis siap) | Disisipkan ke `TugasAkhir/proposal.md` |
| **Subbab 3.4.4** | Final (Gambar 4 & Tabel 5 Skema Redis siap) | Disisipkan ke `TugasAkhir/proposal.md` |
| **Subbab 3.4.5** | Final (Gambar 5 Topologi & Tabel 6 Hardware siap) | Disisipkan ke `TugasAkhir/proposal.md` |
| **Subbab 3.5** | Belum Dibuat (Rencana Analisis) | Disusun mengacu formula perbaikan performa, Paired t-Test, dan tabel matriks faktorial |
| **Lampiran A** | Belum Dibuat (Bukti Telescope) | Lampiran A.1 (Kueri snapshot berulang) & A.2 (Inefisiensi ORM) |
