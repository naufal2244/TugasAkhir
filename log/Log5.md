# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 5)
**File:** `TugasAkhir/log/Log5.md`  
**Waktu Pembaruan:** 2026-10-26  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan pada Sesi 5 mengenai eliminasi konsep *Observer/Invalidasi* dari lingkup proposal, penyelarasan metodologi Bab III (Subbab 3.2, 3.3, 3.4), audit berkas krusial Git (.gitignore), dan panduan migrasi lingkungan kerja ke laptop kedua.

---

## 1. KEPUTUSAN ARSITEKTURAL KRITIS: ELIMINASI OBSERVER & INVALIDASI EVENT

### A. Latar Belakang & Akar Masalah
* Sebelumnya, draf proposal (Subbab 2.3.2, 2.3.6, 3.2.2, 3.2.4, dan 3.2.6) sempat memuat istilah `TaskObserver`, `TaskAssignmentObserver`, dan "invalidasi cache otomatis berbasis event".
* Di basis kode lokal memang sempat dibuat observer untuk invalidasi cache, namun di dalam dokumen rujukan utama ([Konteks-TA.md](file:///Users/pinkman/Documents/project/koladi-laravel/TugasAkhir/konteks/Konteks-TA.md)) dan naskah jurnal internasional ([Manuscript_Anonymous.md](file:///Users/pinkman/Documents/project/koladi-laravel/TugasAkhir/konteks/Manuscript_Anonymous.md)), **mekanisme invalidasi ini sama sekali tidak pernah diukur before-after-nya**.
* Skrip *benchmark runner* (Node.js) murni menguji alur pembacaan data analitis (*read path*) pada dua kondisi:
  1. **Cache Miss**: Entri cache dikosongkan sebelum request untuk mengukur beban penuh eksekusi kalkulasi ORM.
  2. **Cache Hit**: Entri cache telah diisi sebelumnya untuk mengukur retrieval instan dari memori RAM Redis.
* **Risiko Fatal jika Dibiarkan:** Menyebut "invalidasi berbasis event" akan memicu pertanyaan jebakan penguji saat sidang: *"Mana data eksperimen latensi proses invalidasi ketika kartu Kanban dipindahkan?"* Padahal penelitian ini fokus pada optimasi pemrosesan analitis DSS dengan 4 metrik kuantitatif (*response time*, *SQL query count*, *memory utilization*, *cache hit ratio*).

### B. Tindakan Perbaikan yang Telah Dilakukan
* Seluruh kata kunci `observer`, `invalida*`, `sinkronisasi`, dan `event` telah dibersihkan 100% dari naskah proposal ([TugasAkhir/proposal.md](file:///Users/pinkman/Documents/project/koladi-laravel/TugasAkhir/proposal.md) baris 148 dan 164).
* **Tahap 6 Mens & Tourwé (Maintain Consistency) Tetap Terpenuhi Sah:**
  Konsistensi sistem dijaga melalui 3 pilar nyata:
  1. Memverifikasi stabilitas orkestrasi kontainer Docker Compose (`koladi_net`).
  2. Memverifikasi ketiadaan efek samping (*side effects*) pada modul Kanban melalui pengujian regresi fungsional untuk memastikan modifikasi relasi model `Task` dan dependensi servis tidak mengganggu operasional penugasan.
  3. Menyelaraskan artefak nonkode (`.env.docker.local.example`, diagram arsitektur sistem di 3.4.1, dan dokumentasi API DSS).

---

## 2. PENYELARASAN METODOLOGI BAB III

### A. Subbab 3.2 (Jalannya Penelitian)
* Mengadopsi **6 Tahapan Siklus Hidup Refaktorisasi menurut Mens & Tourwé (2004)** secara seragam dengan Bab II (2.3), ditulis murni dalam paragraf naratif yang padat tanpa *bullet points* bertingkat:
  * `3.2.1. Identifikasi Titik Hambatan (Identify Where to Refactor)`: Profiling awal via Laravel Telescope pada 2 endpoint target (terdokumentasi di Lampiran A).
  * `3.2.2. Penentuan Strategi Optimasi (Determine Refactoring Techniques)`: Perancangan Redis Cache-Aside dan selective eager loading (`with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`) serta in-memory aggregation.
  * `3.2.3. Penjaminan Preservasi Perilaku (Guarantee Behavior Preservation)`: Pengujian regresi otomatis untuk menjamin 22 metrik performa tim dan 5 rekomendasi menghasilkan nilai 100% identik. Nama "Opdyke (1992)" dihilangkan agar konsisten dengan Subbab 2.3.3.
  * `3.2.4. Penerapan Refaktorisasi (Apply the Refactoring)`: Transformasi aktual pada `DSSService`, `SnapshotManager`, dan controller. Istilah "The Two Hats" dan "perangkuman kueri" disederhanakan menjadi istilah baku "penarikan data secara batch dengan agregasi di dalam memori (*in-memory aggregation*)".
  * `3.2.5. Penilaian Efek Kualitas (Assess Quality Effect)`: Controlled benchmarking sekuensial terisolasi dalam Docker `koladi_net`, 3 skala dataset sintetis, 3 warm-up requests, 10 repetisi valid (240 total observasi valid), dianalisis via Paired t-Test ($\alpha = 0{,}05$). Istilah janggal "prinsip Seksi 3.3" diganti menjadi istilah akademis baku.
  * `3.2.6. Pemeliharaan Konsistensi Sistem (Maintain Consistency)`: Verifikasi kontainer Docker, uji regresi relasi model pada Kanban, dan pembaruan artefak nonkode.

### B. Subbab 3.3 (Tahapan Pengembangan Aplikasi)
* **Klarifikasi Penolakan SDLC Konvensional:** Mengikuti preseden sukses pada proposal Dzakwan (`proposalDzakwan.md`), metodologi seperti Waterfall, Agile/Scrum, atau RAD **secara tegas dinyatakan tidak relevan dan tidak diterapkan** karena penelitian ini bukan pembangunan perangkat lunak baru (*greenfield*), melainkan rekayasa optimasi kode pada sistem yang sudah berjalan (*brownfield software engineering*).
* **Metodologi Pengganti yang Sah:** Dinyatakan secara resmi mengadopsi Siklus Hidup Refaktorisasi Mens & Tourwé (2004) dengan batasan preservasi perilaku (*behavior-preserving*) (Fowler, 2018), dengan alur utama merujuk ke Subbab 3.2.

### C. Subbab 3.4 (Rencana Perancangan Sistem)
* **Adaptasi Template Kampus:** Menghindari pemaksaan Use Case, Wireframe, atau ERD generik yang tidak relevan untuk optimasi backend.
* **3.4.1. Gambaran Umum Sistem:**
  * Diagram 5 Kotak Horizontal (Gambar 2): `User` $\rightarrow$ `Aplikasi Koladi (Laravel 12)` $\rightarrow$ `Modul DSS (DSSService)` $\rightarrow$ `Redis 7 (In-Memory Cache-Aside)` $\leftrightarrow$ `PostgreSQL 15 (Basis Data Relasional)`.
  * Menegaskan bahwa diagram ini adalah **arsitektur target solusi yang dirancang** setelah optimasi.
  * PostgreSQL berfungsi sebagai penyimpan data persisten relasional yang melayani penarikan kueri SQL saat *cache miss*, sedangkan lingkungan eksperimen terisolasi adalah jaringan Docker Compose (`koladi_net`).
* **3.4.2. Rancangan Alur Komputasi Modul DSS dan Cache-Aside:**
  * Menguraikan alur 2 endpoint DSS (`/suggestions` dan `/modal-data`) serta pipeline kalkulasi (`DSSService`, `SnapshotManager`, `MetricsCalculator`, `TrendAnalyzer`, `RecommendationEngine`).
  * Menyajikan **Sequence Diagram UML (Gambar 3)** yang membedakan alur *Cache Hit* (langsung dari RAM, 0 kueri SQL) dan *Cache Miss* (eksekusi kueri teroptimasi + full payload caching untuk meniadakan masalah *partial cache hit* baseline).

---

## 3. AUDIT GIT, SUBMODUL, & PERSIAPAN CLONE LAPTOP KEDUA

### A. Berkas Krusial yang Diselamatkan dari `.gitignore`
* **Folder `storage/app/benchmark/`**:
  * `results_baseline_fixed.csv` (data 240 observasi baseline)
  * `results_optimized.csv` (data 240 observasi optimized)
  * `auth.json` (token pengujian lokal)
* Sebelumnya terabaikan (*ignored*) oleh aturan `*` bawaan Laravel di `storage/app/.gitignore`.
* **Solusi**: Aturan di `storage/app/.gitignore` telah diubah dengan menambahkan `!benchmark/` dan `!benchmark/**`, sehingga berkas-berkas hasil eksperimen ini sekarang ikut ter-track dan ter-push.

### B. Struktur Repositori & Submodul
1. **Repo Utama `koladi-laravel`**:
   * Remote: `origin https://github.com/Dzakwan993/koladi.git`
   * Branch aktif: `penelitian-naufal`
   * Berkas konfigurasi: `.gitmodules` (mendaftarkan `TugasAkhir` sebagai submodul).
2. **Submodul `TugasAkhir/`**:
   * Remote: `origin https://github.com/naufal2244/TugasAkhir`
   * Branch aktif: `main`
   * Berisi: `proposal.md`, `proposalDzakwan.md`, `konteks/`, `Lit Review/`, `log/` (`Log1.md` s.d. `Log5.md`), dan `tips/`.

### C. Prosedur Push (Di Laptop Saat Ini)
```bash
# 1. Push Submodul TugasAkhir terlebih dahulu
cd TugasAkhir
git add .
git commit -m "Update proposal bab 2 & 3, log4, log5, dan pembersihan observer"
git push origin main
cd ..

# 2. Push Repositori Utama koladi
git add .gitmodules TugasAkhir storage/app/.gitignore storage/app/benchmark/
git commit -m "Track data hasil benchmark dan update submodule TugasAkhir"
git push origin penelitian-naufal
```

### D. Prosedur Clone di Laptop Kedua (Perintah 1 Baris)
```bash
git clone --recurse-submodules -b penelitian-naufal https://github.com/Dzakwan993/koladi.git
```

---

## 4. KEBUTUHAN INSTALASI DI LAPTOP KEDUA

* **Apakah perlu install modul / software di laptop kedua?**
  * **TIDAK PERLU SAMA SEKALI** jika tujuannya adalah membuka codebase dan melanjutkan penulisan proposal bersama AI Antigravity.
  * Antigravity hanya memerlukan file kode dan markdown yang ada di folder workspace untuk membaca konteks, memahami relasi model Eloquent, skrip benchmark, serta draf proposal.
  * Anda **tidak perlu** menjalankan `composer install`, `npm install`, ataupun `docker compose up` di laptop kedua kecuali jika nanti Anda ingin menjalankan server aplikasi dan mengeksekusi ulang pengujian benchmark secara lokal.
