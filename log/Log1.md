# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR
**File:** `log/SESSION_CONTEXT_BAB2.md`  
**Waktu Pembaruan:** 2026-10-21  
**Tujuan Dokumen:** Menjadi sumber konteks lengkap (*context handoff*) untuk sesi percakapan baru agar AI dan pengguna dapat langsung melanjutkan pekerjaan tanpa kehilangan riwayat rancangan, keputusan metodologi, maupun draf teks yang telah disepakati.

---

## 1. IDENTITAS PENELITIAN & PROFIL SISTEM

* **Judul Tugas Akhir:**  
  *"The System Knows Your Team's Performance, But Takes Forever to Tell You": A Comparative Study of Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application*
* **Program Studi & Kampus:** TRPL (Teknologi Rekayasa Perangkat Lunak), Politeknik Negeri Batam.
* **Artefak / Objek Penelitian:**  
  Modul *Decision Support System* (DSS) pada aplikasi **Koladi** (*Collaborative Virtual Office*, SaaS).
* **Tech Stack Sistem:**
  * Framework: Laravel 12.x (PHP 8.2+)
  * Database: PostgreSQL 15 (Relasional)
  * Cache Engine: Redis 7 (via ekstensi `phpredis`)
  * Containerization: Docker Compose (jaringan internal `koladi_net`)
* **Endpoint yang Diteliti:**
  1. `GET /statistik/suggestions` (Rekomendasi manajerial berbasis 5 aturan DSS)
  2. `GET /statistik/modal-data` (Payload lengkap DSS: metrik utama, tugas mendesak, distribusi beban kerja)
* **Permasalahan Teknis Utama (Root Causes):**
  1. *Database-backed cache bottleneck*: Baseline menggunakan tabel `workspace_performance_snapshots` sehingga cache hit masih mengeksekusi kueri SQL relasional (*partial cache hit*).
  2. *Unnecessary Data Retrieval*: Pemanggilan relasi `assignedUsers` tanpa batasan kolom pada `getTasksInPeriod()`.
  3. *N+1 Query Problem*: Relasi `checklists` pada fungsi `getProgressPercentage()` dipanggil tanpa *eager loading*.
  4. *Query-in-loop*: Fungsi `getWorkloadDistribution()` mengeksekusi `Task::whereHas()` di dalam perulangan *members*.
* **Intervensi Teknis Solutif:**
  1. Penerapan **Redis Cache-Aside** dengan invalidasi *event-driven* berbasis Laravel Observer (`TaskObserver` & `TaskAssignmentObserver`).
  2. Optimasi Kueri Eloquent: *Selective eager loading* `with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`, serta refaktorisasi `getWorkloadDistribution()` menjadi *batch retrieval* dan agregasi *in-memory*.

---

## 2. STATUS PROGRES PENULISAN PROPOSAL

### A. BAB I: PENDAHULUAN (Status: SELESAI di `proposal.md`)
Telah mencakup:
* 1.1 Latar Belakang (Fenomena kerja terdistribusi, DSS, metrik kinerja, inefisiensi Koladi, solusi terpadu, research gap)
* 1.2 Rumusan Masalah (Pengaruh penerapan Redis Cache-Aside & optimasi ORM terhadap 4 metrik kinerja)
* 1.3 Tujuan Penelitian (Menerapkan, membandingkan, dan menganalisis secara komparatif)
* 1.4 Batasan Masalah (Fokus modul DSS Koladi, pengujian sekuensial, 240 sampel valid, tanpa load/stress test, tanpa survei pengguna)
* 1.5 Manfaat Penelitian (Manfaat teoretis & manfaat praktis bagi pengembang)

---

### B. BAB II: TINJAUAN PUSTAKA & METODOLOGI (Status: DRAF FINAL TERVALIDASI)

Bab II telah dirumuskan dengan standar akademik kampus dan siap digabungkan:

#### 1. Sub-bab 2.1. Penelitian Terkait (Literature Review)
* **Isi & Karakteristik:** Menganalisis 5 penelitian terkini dengan sintesis dan tabel komparasi:
  1. *Putra et al. (2025)*: Caching Redis pada Laravel (latensi turun hingga 35.1%), namun menggarisbawahi perlunya eksplorasi kombinasi multi-teknik.
  2. *Privalov & Stupina (2024)*: Caching Redis ter-serialisasi (percepatan 10–12x) dengan invalidasi AOP.
  3. *Turcotte et al. (2022)*: Eliminasi kueri N+1 pada ORM via *Reformulator* (kueri konstan 11->2, latensi 38.58x lebih cepat).
  4. *Yusmita et al. (2025)*: Analisis overhead memori ORM (Prisma vs Raw SQL, overhead 2-3x akibat materialisasi objek).
  5. *Kundan & Makwana (2025)*: Pengujian A/B caching vs optimasi kueri pada Laravel e-commerce secara terisolasi.
* **Research Gap Eksplisit:** Belum ada penelitian yang menguji secara komparatif dampak integrasi simultan *Redis Cache-Aside* dan optimasi kueri ORM pada modul DSS aplikasi kerja virtual di bawah variasi beban dataset terkontrol dengan evaluasi metrik terpadu.

#### 2. Sub-bab 2.2. Landasan Teori (Panjang: ±470 kata)
Mencakup 4 pilar teoritis terfokus:
* **2.2.1 Konsep DSS Berbasis Web:** (Cai & Chen, 2021; Koornneef et al., 2020) Definisi DSS semi-terstruktur dan tantangan komputasi agregasi multi-tabel.
* **2.2.2 Arsitektur Caching & Redis:** (Taipalus, 2024; An & Cao, 2022; Kan et al., 2020) Prinsip *Cache-Aside pattern*, latensi memori sub-milidetik, dan *event-driven invalidation*.
* **2.2.3 Teori ORM & Inefisiensi Kueri Eloquent:** (Alshemaimri et al., 2021; Turcotte et al., 2022; Yusmita et al., 2025) Masalah *Unnecessary Data Retrieval*, *N+1 query*, *query-in-loop*, serta solusi *selective eager loading* dan *batch processing*.
* **2.2.4 Profil Objek Penelitian (Aplikasi Koladi):** Arsitektur sistem Koladi, modul DSS lapis 2, kalkulasi 22 metrik performa tim, serta perbandingan konfigurasi *baseline* vs *optimized*.

#### 3. Sub-bab 2.3. Metode Pengembangan Produk (Panjang: ±435 kata, Tanpa Tabel)
* **Kerangka:** *Refactoring Lifecycle* (Mens & Tourwé, 2004, IEEE TSE) diperkuat prinsip *behavior-preserving transformation* oleh Fowler (2018). Dipilih karena pengembangan bersifat *brownfield performance restructuring*.
* **Format:** Mengikuti standar baku kampus (*"Menurut [Penulis, Tahun], tahap ini bertujuan ... Adapun yang dilakukan pada tahap ini adalah ..."*).
* **6 Tahapan:**
  1. *Identify Where to Refactor* (Profiling Laravel Telescope & logging kueri pada 2 endpoint)
  2. *Determine Refactoring Techniques* (Redis Cache-Aside, selective eager loading, batch retrieval)
  3. *Guarantee Behavior Preservation* (Automated regression tests via Pest PHP mengunci kalkulasi 22 metrik)
  4. *Apply the Refactoring* (Implementasi kode di Laravel 12, PostgreSQL 15, Redis 7)
  5. *Assess Quality Effect* (Pengukuran response time, kueri SQL, memori, cache hit ratio)
  6. *Maintain Consistency* (Validasi Docker `koladi_net`, pencegahan side-effects, dokumentasi Git)

#### 4. Sub-bab 2.4. Metode Analisis/Riset yang Digunakan (Panjang: ±480 kata, Tanpa Tabel)
* **Kerangka:** *Experimental Software Engineering* (Wohlin et al., 2000; 2012) dengan rancangan *Controlled Before-After Benchmarking* secara sekuensial terisolasi (Henning & Hasselbring, 2022; Traini et al., 2023).
* **Variabel Bebas:** Konfigurasi (Baseline vs Optimized), Kondisi Cache (Miss vs Hit), Skala Dataset (Small 50, Medium 200, Large 500 tasks), Endpoint (Suggestions vs Modal-data).
* **Variabel Terikat:** Response Time (ms), SQL Query Count (kueri/req), Memory Utilization (MB), Cache Hit Ratio (%).
* **Prosedur & Instrumen:** Middleware kustom Laravel, Runner Node.js, CSV Validator. Eliminasi *cold-start* via 3 *warm-up requests* (dibuang), diikuti 10 repetisi valid.
* **Matriks Data:** $3 \times 2 \times 2 \times 2 \times 10 = \mathbf{240\text{ observasi valid}}$ (120 baseline, 120 optimized).
* **Analisis Data:**
  1. Statistik Deskriptif (Mean, SD, Min, Max).
  2. Formula Persentase Perbaikan ($\text{Improvement } \% = \frac{\text{Baseline} - \text{Optimized}}{\text{Baseline}} \times 100\%$, perubahan <3% dianggap *measurement noise*).
  3. Uji Hipotesis *Paired-Sample t-Test* ($\alpha = 0,05$). Metrik deterministik (kueri SQL) dilaporkan $p < 0,001$ dan difokuskan pada reduksi absolut serta *joint reading* bersama *cache hit ratio*.

---

## 3. NASKAH TEKS LENGKAP HASIL AKHIR DRAF BAB II

Berikut naskah teks lengkap Sub-bab 2.2, 2.3, dan 2.4 yang telah siap dipakai:

```markdown
### 2.2. Landasan Teori

#### 2.2.1. Konsep Decision Support System (DSS) Berbasis Web
Decision Support System (DSS) merupakan sistem informasi berbasis komputer yang dirancang untuk mendukung pengambilan keputusan semi-terstruktur melalui pemrosesan analitis dan manipulasi data interaktif (Cai & Chen, 2021). Pada aplikasi kolaboratif modern, DSS diakses melalui antarmuka web untuk menyajikan ringkasan kinerja tim secara dinamis (Koornneef et al., 2020). Berbeda dari penyajian data operasional biasa, DSS melibatkan pemrosesan komputasi analitis intensif yang menggabungkan banyak entitas relasional guna menghasilkan indikator performa manajerial secara *real-time*.

#### 2.2.2. Arsitektur Caching dan Redis
*Caching* merupakan teknik penyimpanan data sementara pada media akses berkecepatan tinggi guna mengurangi latensi pemrosesan dan beban akses ke basis data utama (Taipalus, 2024). Redis (*Remote Dictionary Server*) adalah *in-memory data structure store* berbasis memori utama (RAM) dengan latensi pembacaan sub-milidetik (An & Cao, 2022). Pola arsitektur yang digunakan adalah *Cache-Aside*, di mana sistem memeriksa ketersediaan data pada Redis terlebih dahulu; jika terjadi *cache miss*, sistem mengambil data dari basis data relasional, mengembalikannya ke pengguna, lalu menyimpannya ke Redis. Untuk menjamin konsistensi data, invalidasi *cache* diterapkan secara otomatis menggunakan mekanisme *event-driven* berbasis Laravel *Observer* saat data tugas atau penugasan mengalami perubahan (Kan et al., 2020).

#### 2.2.3. Teori ORM dan Inefisiensi Kueri Eloquent
*Object-Relational Mapping* (ORM) menjembatani paradigma pemrograman berorientasi objek dengan basis data relasional, namun abstraksinya rentan menyembunyikan biaya eksekusi kueri yang sebenarnya (Alshemaimri et al., 2021). Eloquent ORM pada Laravel secara *default* menerapkan *lazy loading* yang dapat memicu masalah *N+1 query*, yaitu kondisi di mana eksekusi satu kueri induk memicu *N* kueri tambahan untuk setiap baris data relasinya (Turcotte et al., 2022). Selain itu, pemanggilan relasi tanpa pembatasan kolom (*unnecessary data retrieval*) mengakibatkan hidrasi objek memori yang berlebihan pada proses PHP (Yusmita et al., 2025; Kundan & Makwana, 2025). Masalah performa juga terjadi pada pola *query-in-loop*, yaitu eksekusi kueri berulang di dalam iterasi logika aplikasi. Inefisiensi tersebut diselesaikan melalui teknik *selective eager loading* (`with()`), pembatasan atribut kolom, serta penggantian perulangan kueri dengan pengambilan data secara *batch* dan agregasi *in-memory*.

#### 2.2.4. Profil Objek Penelitian: Aplikasi Koladi
Koladi adalah aplikasi *collaborative virtual office* berbasis SaaS yang dikembangkan menggunakan Laravel 12, PostgreSQL 15, dan Docker Compose. Modul DSS Koladi bertugas mengagregasi 22 metrik kinerja tim mentah pada setiap *workspace*, seperti tingkat penyelesaian tugas, progres per anggota, dan status beban kerja. Beban komputasi tertinggi berpusat pada *Layer 2 DSS Modal*, khususnya *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data`. Pada kondisi awal (*baseline*), sistem mengandalkan *caching* berbasis tabel relasional `workspace_performance_snapshots` yang rentan fenomena *partial cache hit* dan dibebani inefisiensi kueri Eloquent. Penelitian ini menguji transformasi arsitektur Koladi ke konfigurasi *optimized* yang menggabungkan Redis *Cache-Aside* dan optimasi kueri ORM.

---

### 2.3. Metode Pengembangan Produk

Pengembangan perangkat lunak pada Tugas Akhir ini tidak membangun sistem dari awal (*greenfield*), melainkan melakukan restrukturisasi internal modul *Decision Support System* (DSS) aplikasi Koladi yang telah berjalan (*brownfield*). Oleh karena itu, metode pengembangan yang digunakan adalah *Refactoring Lifecycle* yang dirumuskan oleh Mens & Tourwé (2004) serta diperkuat prinsip transformasi preservasi perilaku oleh Fowler (2018). Menurut Fowler (2018), *refactoring* merupakan teknik mengubah struktur internal perangkat lunak tanpa mengubah perilaku eksternalnya guna meningkatkan kualitas non-fungsional. Berdasarkan Mens & Tourwé (2004), siklus refaktorisasi terdiri dari enam tahapan terstruktur:

1. **Identifikasi Lokasi Refaktorisasi (*Identify Where to Refactor*)**  
   Menurut Mens & Tourwé (2004), tahap ini bertujuan mendeteksi bagian kode yang bermasalah (*code smells*) atau mengalami degradasi performa. Adapun yang dilakukan pada tahap ini adalah melakukan *profiling* menggunakan Laravel Telescope dan *query logging* pada *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data` untuk mengidentifikasi eksekusi kueri berulang (*N+1 query problem*) serta latensi agregasi data yang tinggi pada modul DSS Koladi.

2. **Penentuan Teknik Refaktorisasi (*Determine Refactoring Techniques*)**  
   Menurut Mens & Tourwé (2004), tahap ini bertujuan memilih pola transformasi kode yang tepat untuk menyelesaikan kelemahan yang telah diidentifikasi. Adapun yang dilakukan pada tahap ini adalah merancang strategi optimasi *backend*, meliputi penentuan teknik *selective eager loading* (`with()`), transformasi perulangan kueri menjadi *batch retrieval* dengan agregasi *in-memory*, serta integrasi *in-memory caching* berbasis Redis menggunakan pola *Cache-Aside* dan Laravel Model *Observer* untuk siklus *cache invalidation*.

3. **Penjaminan Preservasi Perilaku (*Guarantee Behavior Preservation*)**  
   Menurut Fowler (2018) dan Mens & Tourwé (2004), tahap ini bertujuan memastikan bahwa modifikasi struktur kode sama sekali tidak mengubah luaran fungsional sistem. Adapun yang dilakukan pada tahap ini adalah menyusun rangkaian pengujian regresi otomatis (*automated regression tests*) menggunakan Pest PHP guna mengunci integritas kalkulasi 22 metrik performa tim dan 5 komponen rekomendasi manajerial, sehingga respons JSON sebelum dan sesudah refaktorisasi terbukti identik.

4. **Penerapan Refaktorisasi (*Apply the Refactoring*)**  
   Menurut Mens & Tourwé (2004), tahap ini merupakan eksekusi modifikasi kode sumber secara terkontrol pada lingkungan pengembangan. Adapun yang dilakukan pada tahap ini adalah mengimplementasikan transformasi kode pada lapisan servis Laravel 12, yaitu merefaktorisasi metode `getTasksInPeriod()` dan `getWorkloadDistribution()`, mengonfigurasi Redis 7 sebagai *cache store*, serta mengaktifkan mekanisme *eager loading* dan *batch processing* pada repositori data PostgreSQL 15.

5. **Evaluasi Efek Kualitas (*Assess Quality Effect*)**  
   Menurut Mens & Tourwé (2004), tahap ini bertujuan mengukur dampak perbaikan struktur kode terhadap atribut kualitas perangkat lunak. Adapun yang dilakukan pada tahap ini adalah menguji performa teknis sebelum dan sesudah refaktorisasi, mencakup pengukuran waktu respons (*response time*), jumlah kueri SQL, konsumsi memori (*memory usage*), dan rasio keberhasilan *cache* (*cache hit ratio*).

6. **Pemeliharaan Konsistensi Sistem (*Maintain Consistency*)**  
   Menurut Mens & Tourwé (2004), tahap ini memastikan artefak perangkat lunak yang diperbarui tetap konsisten dan terintegrasi baik dengan seluruh lingkungan sistem. Adapun yang dilakukan pada tahap ini adalah memverifikasi kestabilan layanan kontainer Redis pada Docker Compose (`koladi_net`), memastikan ketiadaan efek samping (*side effects*) pada modul Koladi lainnya, serta memperbarui dokumentasi arsitektur kode pada repositori Git.

---

### 2.4. Metode Analisis/Riset yang Digunakan

Metode riset yang digunakan dalam Tugas Akhir ini mengacu pada kerangka kerja ***Experimental Software Engineering*** yang dirumuskan oleh Wohlin et al. (2000; 2012). Metode ini merupakan pendekatan kuantitatif formal untuk menguji hipotesis rekayasa perangkat lunak melalui manipulasi variabel bebas secara terkontrol terhadap artefak sistem dan mengukur dampaknya terhadap variabel terikat secara presisi. Rancangan riset yang diterapkan adalah ***Controlled Before-After Benchmarking*** (Henning & Hasselbring, 2022), yang membandingkan performa modul *Decision Support System* (DSS) aplikasi Koladi sebelum dioptimasi (*baseline*) dan sesudah dioptimasi (*optimized*). Pengujian dilakukan secara sekuensial terisolasi (*isolated sequential execution*) di dalam jaringan kontainer Docker Compose (`koladi_net`) guna meniadakan variabel pengganggu eksternal seperti latensi jaringan, *thread contention*, dan beban *traffic* paralel (Putra et al., 2025; Yusmita et al., 2025).

Tahapan pelaksanaan metode riset eksperimental ini terbagi ke dalam empat fase utama sesuai kaidah Wohlin et al. (2000):

1. **Definisi Ruang Lingkup dan Desain Eksperimen (*Definition & Scoping*)**  
   Menurut Wohlin et al. (2000), tahap ini bertujuan memformalisasi tujuan eksperimen dan menetapkan variabel penelitian. Variabel bebas (*independent variables*) terdiri atas: konfigurasi sistem (*baseline* vs *optimized*), kondisi *cache* (*cache miss* vs *cache hit*), skala volume dataset sintetis bertingkat (*Small*: 50 tugas/5 anggota; *Medium*: 200 tugas/15 anggota; *Large*: 500 tugas/30 anggota), serta jalur *endpoint* DSS (`GET /statistik/suggestions` dan `GET /statistik/modal-data`). Adapun variabel terikat (*dependent variables*) mencakup empat metrik performa teknis: *response time* (ms), *SQL query count* (kueri/request), *memory utilization* (MB), dan *cache hit ratio* (%) (Alshemaimri et al., 2021; Kundan & Makwana, 2025).

2. **Perencanaan Instrumen dan Protokol Pengujian (*Planning & Instrumentation*)**  
   Menurut Wohlin et al. (2000), tahap ini merancang instrumen pengukuran agar data yang diperoleh bersifat objektif dan dapat direplikasi. Pengujian memanfaatkan *middleware* kustom Laravel untuk mencatat eksekusi kueri SQL dan puncak memori internal (`memory_get_peak_usage`), *script runner* berbasis Node.js untuk mengeksekusi HTTP *request* sekuensial dan merekam latensi, serta *CSV validator* untuk verifikasi integritas data. Untuk menghindari bias *cold-start* (seperti *JIT/OPcache warming* dan *database connection pooling*), setiap kombinasi diawali dengan 3 kali *warm-up requests* yang nilainya dibuang dari analisis (Traini et al., 2023; Japke et al., 2025).

3. **Operasionalisasi dan Pengumpulan Data (*Operation & Data Collection*)**  
   Menurut Wohlin et al. (2000), tahap ini merupakan eksekusi eksperimen sesuai protokol faktorial penuh. Setelah fase *warm-up*, sistem merekam 10 repetisi valid untuk setiap kombinasi perlakuan. Struktur faktorial menghasilkan total matriks data valid sebesar:  
   $$\text{Total Data} = 3\text{ skala data} \times 2\text{ endpoint} \times 2\text{ konfigurasi} \times 2\text{ kondisi cache} \times 10\text{ repetisi} = 240\text{ observasi valid}$$  
   (terdiri dari 120 observasi *baseline* dan 120 observasi *optimized*).

4. **Analisis Data dan Uji Inferensial (*Analysis & Interpretation*)**  
   Menurut Wohlin et al. (2000), data mentah diolah untuk memvalidasi hipotesis. Analisis dilakukan melalui tiga instrumen statistik:
   * **Statistik Deskriptif:** Menghitung nilai *Mean*, *Standard Deviation*, *Minimum*, dan *Maximum* untuk mengevaluasi stabilitas pengukuran (Japke et al., 2025).
   * **Persentase Perbaikan (*Practical Significance*):** Menghitung magnitudo perbaikan performa dengan formula:  
     $$\text{Improvement (\%)} = \left(\frac{\text{Mean Baseline} - \text{Mean Optimized}}{\text{Mean Baseline}}\right) \times 100\%$$  
     Perubahan di bawah 3% dikategorikan sebagai *measurement noise* (Japke et al., 2025).
   * **Uji Hipotesis Komparatif:** Melakukan *Paired-Sample t-Test* pada taraf signifikansi $\alpha = 0,05$ untuk membuktikan signifikansi perbedaan performa berpasangan (Uddin & Lu, 2024). Untuk metrik deterministik dengan variansi nol (*SQL query count*), signifikansi dilaporkan secara ilmiah sebagai $p < 0,001$ dengan penekanan analisis pada reduksi absolut kueri dan status *bypass* basis data via *cache hit ratio* (An & Cao, 2022).
```

---

## 4. ATURAN PENULISAN & PREFERENSI PENGGUNA
1. **Panjang Sub-bab:** Maksimal 400–500 kata per sub-bab agar padat dan fokus pada substansi.
2. **Penggunaan Tabel:** Dihindari pada bagian metode pengembangan produk (2.3) dan metodologi riset (2.4), diformat dalam poin bernomor dengan struktur baku.
3. **Pemisahan Peran Sub-bab 2.3 vs 2.4:**
   * **2.3 Metode Pengembangan Produk:** Mengatur siklus modifikasi perangkat lunak (*Refactoring Lifecycle* Mens & Tourwé 2004 / Fowler 2018).
   * **2.4 Metode Analisis/Riset:** Mengatur metodologi eksperimental pembuktian kinerja (*Experimental Software Engineering* Wohlin et al. 2000 / Henning & Hasselbring 2022).
4. **Validitas Metrik:** Murni instrumen komputasional (*response time*, *query count*, *memory*, *cache hit ratio*). Dilarang memasukkan survei kepuasan atau klaim psikologis pengguna.

---

## 5. LANGKAH BERIKUTNYA UNTUK SESI BARU
1. Menyatukan draf Sub-bab 2.1, 2.2, 2.3, dan 2.4 ke dalam file utama `proposal.md` di bawah heading `BAB II TINJAUAN PUSTAKA`.
2. Melanjutkan penyusunan Bab III (Perancangan Sistem & Prosedur Implementasi Pengujian Detail) jika proposal membutuhkan bab metodologi terpisah.
3. Melakukan sinkronisasi daftar pustaka (Daftar Referensi) agar seluruh sitasi (Wohlin, Fowler, Mens & Tourwé, Putra, Turcotte, Yusmita, dll.) terdaftar rapi.
