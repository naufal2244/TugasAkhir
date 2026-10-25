Berikut adalah pemetaan komprehensif (*high-level overview*) dari seluruh dokumen yang tersedia di dalam *notebook* Anda untuk menyusun tiga sub-bab proposal Tugas Akhir.

---

### **1. PEMETAAN UNTUK 2.2. LANDASAN TEORI**

#### **a. Decision Support System (DSS) dalam Aplikasi Web / Kolaborasi Tim**
* **Definisi & Konsep Utama**:
  * DSS didefinisikan sebagai sistem informasi berbasis komputer yang membantu manajer atau pengambil keputusan dalam menyelesaikan masalah semi-terstruktur melalui interaksi manusia-komputer dan pemodelan data **(Cai & Chen, 2021)**.
  * Paradigma *web-based DSS* memungkinkan sistem diakses melalui peramban web (*browser*) untuk mendukung akses multi-pengguna secara terdistribusi dan berskala besar **(Koornneef et al., 2020)**.
* **Peran Kalkulasi & Agregasi Metrik Kinerja Tim**:
  * Pada aplikasi *collaborative virtual office*, DSS tidak sekadar menampilkan data statis, melainkan menjalankan *staged aggregation pipeline* di atas data aktivitas relasional tim untuk menghasilkan indikator produktivitas yang dapat ditindaklanjuti **(Konteks Penelitian; Manuscript Anonymous)**.
  * Proses ini melibatkan penarikan dan pemrosesan data intensif (seperti kalkulasi status tugas, beban kerja, dan pencapaian target) yang memerlukan kecepatan pemrosesan *real-time* agar tidak menghambat efektivitas keputusan **(Koornneef et al., 2020; Mahmoodi et al., 2024)**.
  * Waktu respons (*response time*) diakui sebagai metrik evaluasi utama pada lingkungan *decision support* karena kueri analitis/agregasi berdurasi panjang sangat menentukan kecepatan penyajian informasi kepada pengguna **(Taipalus, 2024; Hussain & Ullah, 2023)**.

#### **b. Arsitektur Caching & Redis**
* **Teori In-Memory Caching**:
  * *Caching* adalah mekanisme menyimpan data atau hasil komputasi yang sering diminta ke dalam media penyimpanan sementara berkecepatan tinggi (RAM) untuk menghindari eksekusi kueri berulang pada basis data relasional (RDBMS) yang berbasis disk I/O **(Privalov & Stupina, 2024; Zhe Yang et al., 2020)**.
  * Redis (*REmote DIctionary Server*) adalah *in-memory NoSQL key-value store* berkinerja tinggi yang terbukti mem-bypass RDBMS (seperti PostgreSQL) secara penuh dan mereduksi lonjakan latensi pembacaan data hingga >90% **(Sakti et al., 2025; Privalov & Stupina, 2024)**.
* **Pola Cache-Aside (Lazy Caching)**:
  * Aplikasi mengontrol alur pembacaan: memeriksa Redis terlebih dahulu saat menerima *request* (*cache check*). Jika terjadi *cache hit*, data langsung dikembalikan dari RAM; jika *cache miss*, sistem menjalankan *pipeline* kalkulasi ke basis data utama, menyimpan hasilnya ke Redis, lalu mengembalikannya ke pengguna **(Mao et al., 2024; Chronopoulos et al., 2022)**.
* **Strategi Invalidasi Data (TTL vs. Event-Driven Invalidation)**:
  * *Time-To-Live (TTL)*: Menentukan masa berlaku statis untuk pengosongan *cache*. Namun, TTL statis berisiko menyajikan data usang (*stale data*) jika data pada basis data utama berubah sebelum durasi TTL berakhir **(Mao et al., 2024; Sultan et al., 2024)**.
  * *Event-Driven Invalidation*: Mekanisme invalidasi dinamis yang dipicu langsung saat terjadi operasi penulisan/perubahan data (*write-triggered event*) pada model basis data utama untuk menjamin konsistensi data (*data freshness*) **(Mao et al., 2024; Kan et al., 2020)**.
  * *Implementasi*: Pada *framework* Spring Boot digunakan AOP `@AfterReturning` **(Privalov & Stupina, 2024)**, sedangkan pada Laravel diimplementasikan melalui *Eloquent Observers* (`TaskObserver` dan `TaskAssignmentObserver`) **(Konteks Penelitian; Manuscript Anonymous)**.

#### **c. Object-Relational Mapping (ORM) & Eloquent**
* **Teori Pola Active Record**:
  * Eloquent Laravel mengadopsi pola *Active Record*, di mana satu kelas model mewakili tabel basis data dan satu instansi objek mewakili baris data **(Alshemaimri et al., 2021; Firmansyah & Sudarmilah, 2025)**.
* **Eager Loading vs. Lazy Loading**:
  * *Lazy Loading*: Memuat data relasi secara bertahap hanya saat properti relasi diakses di kode, yang berisiko memicu eksekusi kueri berulang di dalam *loop* **(Majerik & Borkovcova, 2023)**.
  * *Eager Loading (`with()`)*: Memuat entitas utama beserta relasinya secara bersamaan di awal dalam kueri terpisah yang ter-indeks, sehingga menekan jumlah interaksi dengan basis data **(Kundan & Makwana, 2025; Klochkov & Mulawka, 2021)**.
* **Antipola N+1 Query**:
  * Terjadi ketika kueri awal menghasilkan \\(N\\) baris data, dan aplikasi melakukan iterasi (*looping*) untuk memanggil kueri tambahan pada setiap baris, menghasilkan \\(N+1\\) *database round-trips* yang memperburuk latensi secara linier/eksponensial **(Turcotte et al., 2022; Alshemaimri et al., 2021)**.
* **Unnecessary Data Retrieval (UD) / Column Over-fetching**:
  * *Antipattern* di mana aplikasi menarik seluruh kolom (`SELECT *`) dari tabel relasional padahal hanya membutuhkan kolom tertentu (misalnya hanya kolom `id`), yang memicu pemborosan I/O dan alokasi memori **(Alshemaimri et al., 2021; Kundan & Makwana, 2025)**.
* **Object Graph Materialization Expense**:
  * *Overhead* komputasi dan memori yang dibutuhkan ORM untuk memetakan baris tabel relasional menjadi grafik objek (*object graph*) dalam RAM server. Hal ini menyebabkan penggunaan memori ORM 2 hingga 3 kali lebih besar dibandingkan *Raw SQL* **(Yusmita et al., 2025)**.

#### **d. Profil Objek Penelitian (Koladi)**
* **Arsitektur Sistem**: Platform *Collaborative Virtual Office SaaS* multi-tenant berbasis Laravel 12.x, PostgreSQL 15, Eloquent ORM, Redis 7.x (`redis:7-alpine`, `phpredis`), Nginx + PHP-FPM, dan Docker Compose dalam jaringan terisolasi `koladi_net` **(Konteks Penelitian)**.
* **Modul DSS Modal yang Diteliti**: Lapisan 2 DSS (DSS Modal) pada statistik kinerja yang dipicu saat pengguna menekan tombol *"Lihat Tindak Lanjut"*, diakses melalui dua *endpoint*:
  1. `GET /statistik/modal-data`
  2. `GET /statistik/suggestions`
* **Alur Pipeline Kalkulasi**:
  * `DSSService` \\(\rightarrow\\) `SnapshotManager` (pemeriksaan/penyimpanan *cache*) \\(\rightarrow\\) `MetricsCalculator` (menghitung 22 metrik mentah produktivitas) \\(\rightarrow\\) `TrendAnalyzer` (komparasi tren antar-periode) \\(\rightarrow\\) `RecommendationEngine` (klasifikasi indikator *critical/warning/positive*) \\(\rightarrow\\) `SnapshotManager` (penyimpanan hasil).
  * Ditambah pemanggilan fungsi terpisah di *controller*: `getUrgentTasks()` dan `getWorkloadDistribution()` **(Konteks Penelitian; Manuscript Anonymous)**.

---

### **2. PEMETAAN UNTUK 2.3. METODE PENGEMBANGAN PRODUK**

* **a. Keberadaan Metode SDLC Spesifik**:
  * Di dalam dokumen sumber (`Konteks penelitian terbaru.docx` dan `Manuscript_Anonymous.docx`), **TIDAK ADA penyebutan atau penggunaan metode SDLC rekayasa perangkat lunak tradisional** (seperti Waterfall, Agile/Scrum, Prototyping, atau Spiral) untuk siklus pengembangan aplikasi Koladi secara keseluruhan.
  * Sumber-sumber yang ada secara eksplisit memposisikan penelitian ini sebagai **studi eksperimental komparatif kuantitatif (*Experimental Software Engineering*)**, bukan studi pengembangan produk baru dari nol **(Konteks Penelitian; Manuscript Anonymous)**.
* **b. Referensi Teori**:
  * Kerangka kerja yang digunakan adalah *Experimental Software Engineering* **(Wohlin et al., 2000)**.
* **c. Tahapan Optimasi / Refaktorisasi Sistem Koladi**:
  * Walaupun metode SDLC formal tidak dinamai, tahapan intervensi teknis/refaktorisasi sistem Koladi dicatat secara sistematis dalam empat fase **(Manuscript Anonymous)**:
    1. *Baseline Profiling*: Mendiagnosis *bottleneck* awal menggunakan Laravel Telescope untuk mengidentifikasi kueri berlebih pada *database-backed cache* (`workspace_performance_snapshots`), *partial cache hit*, *column over-fetching* (`assignedUsers`), N+1 query (`checklists`), dan *query-in-loop* (`getWorkloadDistribution`).
    2. *Implementation (Cache Refactoring / Solusi A)*: Mengganti *database-backed cache* menjadi *Redis Cache-Aside* (`phpredis`) dan membangun `TaskObserver` serta `TaskAssignmentObserver` untuk invalidasi *event-driven*.
    3. *Implementation (ORM Refactoring / Solusi B)*: Menerapkan *selective eager loading* `->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])` dan merefaktorisasi `getWorkloadDistribution()` dari *query-in-loop* (`Task::whereHas()`) menjadi *batch retrieval* + *in-memory aggregation*.
    4. *Controlled Benchmarking & Evaluation*: Menjalankan pengujian *before-after* terkontrol untuk mengevaluasi dampak perbaikan.
* **d. Catatan Penting untuk Proposal**:
  * **Bagian metode SDLC rekayasa perangkat lunak secara eksplisit belum ada di dalam sumber**. Jika panduan proposal di institusi Anda mewajibkan sub-bab 2.3 berisi metode SDLC, Anda disarankan untuk menambahkan metode yang sesuai dengan konteks refaktorisasi sistem, seperti **Iterative Enhancement / Refactoring Lifecycle** atau **Agile/Scrum (Sprint Refactoring)**.

---

### **3. PEMETAAN UNTUK 2.4. METODE ANALISIS / RISET YANG DIGUNAKAN**

#### **a. Desain Riset / Eksperimen**
* **Kerangka Kerja**: *Experimental Software Engineering Framework* **(Wohlin et al., 2000)**.
* **Desain Eksperimen**: *Controlled before-and-after comparative benchmarking experiment* (studi eksperimental komparatif sebelum dan sesudah optimasi dalam kondisi terkontrol) **(Putra et al., 2025; Wohlin et al., 2000)**.
* **Sifat Pengujian**: Murni kuantitatif berbasis instrumen sistem terisolasi. Tidak ada data primer dari manusia (tanpa survei, wawancara, kuesioner, atau uji kepuasan UX) **(Konteks Penelitian)**.

#### **b. Variabel Eksperimen**
* **Variabel Bebas (Independen)**:
  1. *Konfigurasi Sistem*: Baseline (database-backed cache, unrestricted eager loading, tanpa eager load checklists) vs. Optimized (Redis Cache-Aside, TaskObserver, TaskAssignmentObserver, selective eager loading, batch retrieval).
  2. *Skala Volume Data (Dataset Size)*: Small (50 tasks, 5 members), Medium (200 tasks, 15 members), Large (500 tasks, 30 members).
  3. *Kondisi Cache*: Cache Miss vs. Cache Hit.
  4. *Endpoint DSS*: `GET /statistik/suggestions` dan `GET /statistik/modal-data`.
* **Variabel Terikat (Dependen)**:
  1. *Response Time (ms)*: Waktu latensi eksekusi permintaan API dari *client* hingga menerima respons **(Sismadi et al., 2022; Taipalus, 2024)**.
  2. *SQL Query Count*: Jumlah kueri SQL yang dieksekusi ke PostgreSQL per permintaan **(Alshemaimri et al., 2021; Turcotte et al., 2022)**.
  3. *Memory Utilization (MB)*: Konsumsi memori puncak PHP/server saat memproses permintaan (`memory_get_peak_usage()`) **(Yusmita et al., 2025; Kundan & Makwana, 2025)**.
  4. *Cache Hit Ratio (%)*: Persentase permintaan yang berhasil dilayani dari *cache* **(Zhe Yang et al., 2020; An & Cao, 2022)**.

#### **c. Prosedur dan Instrumen Pengujian**
* **Instrumen & Tools Ukur**:
  * Middleware kustom Laravel untuk mencatat *SQL query count* dan *memory utilization* per *request*.
  * *Script runner* Node.js sekuensial untuk mengirimkan permintaan HTTP dan mencatat *response time*.
  * *CSV Validator* untuk memastikan kelengkapan dan integritas baris data hasil pengujian.
  * Pemantauan dan verifikasi internal menggunakan Laravel Telescope **(Konteks Penelitian; Manuscript Anonymous)**.
* **Prosedur Pengujian**:
  * Pengujian dieksekusi secara sekuensial (non-concurrent) dalam jaringan terisolasi Docker Compose (`koladi_net`) untuk mengeliminasi variansi jaringan luar **(Yusmita et al., 2025)**.
  * *Warm-up Request*: Setiap kombinasi diawali dengan **3 warm-up requests yang dibuang (*discarded*)** dari analisis untuk mengeliminasi bias *cold-start* dan pengisian memori awal **(Henning & Hasselbring, 2022; Traini et al., 2023; Japke et al., 2025)**.
  * *Repetisi Pengujian*: **10 pengulangan valid** dicatat per kombinasi **(Privalov & Stupina, 2024)**.
  * *Total Data Valid*: 3 volume data \\(\times\\) 2 endpoint \\(\times\\) 2 konfigurasi \\(\times\\) 2 kondisi cache \\(\times\\) 10 repetisi = **240 pengukuran valid** **(Manuscript Anonymous)**.

#### **d. Rencana Analisis Data**
* **Statistik Deskriptif**: Menghitung Rata-rata (*Mean*), Deviasi Standar (*Standard Deviation* / SD), Minimum, dan Maksimum untuk setiap kombinasi **(Kundan & Makwana, 2025; Japke et al., 2025)**.
* **Persentase Peningkatan (*Percentage Improvement*)**:
  \\[\text{Improvement (\%)} = \frac{\text{Baseline} - \text{Optimized}}{\text{Baseline}} \times 100\%\\]
* **Uji Hipotesis Statistik**:
  * *Paired-Sample t-Test* (Uji t berpasangan) pada tingkat signifikansi \\(\alpha = 0,05\\) untuk memvalidasi apakah perbedaan rata-rata performa baseline vs. optimized signifikan secara statistik **(Uddin & Lu, 2024; Confirming the statistically.pdf)**.
* **Interpretasi Khusus**:
  * Untuk data yang bersifat deterministik (seperti *SQL Query Count* yang bernilai konstan per repetisi), nilai \\(p < 0,001\\) dilaporkan dengan penekanan pada **signifikansi praktis (*practical significance*)** dan penurunan absolut jumlah kueri **(Traini et al., 2023; Japke et al., 2025)**.
  * Metrik *Cache Hit Ratio* diinterpretasikan secara bersamaan dengan *SQL Query Count* untuk memverifikasi apakah *cache hit* benar-benar melakukan *bypass* penuh terhadap basis data relasional **(An & Cao, 2022; Zhe Yang et al., 2020)**.

  Berikut adalah pemetaan gambaran besar (*high-level overview*) dari seluruh sumber yang tersedia di dalam *notebook* untuk menyusun tiga sub-bab proposal Tugas Akhir Anda:

---

### **1. PEMETAAN UNTUK 2.2. LANDASAN TEORI**

#### **a. Decision Support System (DSS) dalam Aplikasi Web & Kolaborasi Tim**
* **Definisi & Peran Komputasional:**  
  DSS berbasis web didefinisikan sebagai sistem terkomputerisasi yang membantu manajer dalam pengambilan keputusan semi-terstruktur melalui interaksi manusia-komputer dan pemodelan data. DSS dapat diakses melalui antarmuka *browser* untuk mendukung akses multi-pengguna dan pengelolaan data berskala besar dalam lingkungan kerja terdistribusi. Pada platform kolaboratif, DSS tidak hanya menyajikan data statis, melainkan menjalankan *pipeline* agregasi bertahap (*staged aggregation pipeline*) terhadap data aktivitas relasional tim untuk mengolah data mentah menjadi indikator produktivitas yang dapat ditindaklanjuti.
* **Sitasi & Sumber Rujukan:**
  * **Cai & Chen (2021):** Definisi DSS berbasis web untuk pemodelan data dan keputusan semi-terstruktur *(Introduction.docx; Manuscript_Anonymous)*.
  * **Koornneef et al. (2020):** DSS berbasis *browser* untuk akses multi-pengguna skala besar *(Introduction.docx; Manuscript_Anonymous)*.
  * **Ryu et al. (2023):** Paradigma arsitektur DSS berbasis web modern *(Introduction.docx; Manuscript_Anonymous)*.
  * **Froese et al. (2025):** Konteks pergeseran *global virtual work* dan kebutuhan platform kolaboratif digital *(Introduction.docx; Global virtual work.pdf)*.

#### **b. Arsitektur Caching & Redis**
* **Teori *In-Memory Caching* & Kecepatan RAM:**  
  *In-memory caching* mengalihkan *read path* dari penyimpanan berbasis disk (RDBMS) ke RAM. Pengambilan data dari RAM memiliki latensi akses yang sekitar empat orde magnitudo lebih cepat dibandingkan *database* berbasis disk.
* **Pola *Cache-Aside* (*Look-Aside Caching*):**  
  Aplikasi memeriksa ketersediaan data di memori *cache* terlebih dahulu. Jika terjadi *cache hit*, data dikembalikan langsung ke klien tanpa menyentuh *database*. Jika *cache miss*, sistem mengeksekusi *pipeline* komputasi/database, menyimpan hasilnya ke *cache*, lalu mengembalikannya ke klien.
* **Strategi Invalidasi (*TTL vs. Event-Driven Observer*):**  
  * **Keterbatasan TTL:** Kebijakan berbasis *Time-To-Live* (TTL) secara struktural tidak mampu merespons mutasi data (*data writes*) secara *real-time*, sehingga menimbulkan *stale cache hits* dan *overhead polling* eksponensial ketika interval kesegaran mendekati *real-time*.
  * **Event-Driven Invalidation:** Menggunakan pemicu kejadian (*event triggers* seperti Eloquent Observer pada Laravel atau AOP pada Spring) untuk langsung menghapus/memperbarui *key cache* yang relevan saat terjadi pembuatan, pembaruan, atau penghapusan data (`created`, `updated`, `deleted`, serta penambahan/penghapusan anggota pada tabel *pivot*).
  * **Strategi Hibrida:** Menggabungkan *event-driven invalidation* untuk mutasi data aktual dan TTL sebagai *safety net* khusus untuk metrik berbasis waktu (*urgentTaskRatio*, *overdueRate*) yang berubah secara temporal tanpa adanya operasi *write*.
* **Sitasi & Sumber Rujukan:**
  * **Taipalus (2024):** Teori perbedaan kecepatan akses RAM vs. disk *(Introduction.docx; Manuscript_Anonymous)*.
  * **Privalov & Stupina (2024):** Reduksi latensi kueri 10x lipat (197,5 ms \\(\rightarrow\\) 30 ms) menggunakan *Redis-backed caching* *(Introduction.docx; Manuscript_Anonymous)*.
  * **An & Cao (2022):** Kebijakan *Monotonic Consistent Caching* (MCC), reduksi `#dbread` sebesar 39,09%, dan kritik terhadap ketidakmampuan TTL pada *high-write workload* *(Making Cache Monotonic and Consistent.pdf; Introduction.docx)*.
  * **Mao et al. (2024):** Keterbatasan struktural TTL dan lonjakan *overhead polling* pada kesegaran *real-time* *(Introduction.docx; Manuscript_Anonymous)*.
  * **Kan et al. (2020):** Strategi invalidasi proaktif (*PIOR*) berbasis *publish-subscribe* menghasilkan *cache hit ratio* 64,52% lebih tinggi daripada strategi reaktif *(A Cache Invalidation Strategy Based.pdf; Introduction.docx)*.
  * **Gómez-Pantoja et al. (2025):** Teori *web caching* untuk meredakan beban *backend* dan memperpendek *response time* *(A Behavior-Aware Caching Architecture for Web.pdf; Introduction.docx)*.

#### **c. Object-Relational Mapping (ORM) & Eloquent**
* **Pola *Active Record* & Inefisiensi Kueri:**  
  Eloquent ORM berbasis pola *Active Record*, di mana setiap baris tabel dipetakan menjadi instans objek PHP.
* **Eager Loading vs. Lazy Loading & Antipola N+1 Query:**  
  *Lazy loading* mengeksekusi kueri terpisah secara individu untuk setiap entitas relasional saat diakses di dalam iterasi *loop* (*N+1 query problem*), menyebabkan pertumbuhan jumlah kueri secara linier (\\(O(N)\\)). *Eager loading* menggantikan kueri berulang tersebut menjadi *batch retrieval* tunggal.
* **Unnecessary Data Retrieval (UD) / Column Over-Fetching:**  
  Mengambil seluruh kolom tabel (misalnya `SELECT * FROM users`) padahal aplikasi hanya membutuhkan atribut tertentu (misalnya `id`).
* **Beban Memori ORM (*Object Graph Materialization Expense*):**  
  *Runtime* ORM harus mengalokasikan RAM untuk melakukan instansiasi (*materialization*) setiap atribut dan struktur objek kompleks di memori PHP. Pengambilan kolom tanpa pembatasan membebankan penggunaan memori 2 hingga 3 kali lipat dibandingkan kueri yang dibatasi kolomnya.
* **Sitasi & Sumber Rujukan:**
  * **Turcotte, Aldrich, & Tip (2022):** Analisis masalah *N+1 query* dan pembuktian *eager loading* mereduksi kueri dari 11 ke 2 serta memberikan *speedup* 38,58x *(Introduction.docx; Manuscript_Anonymous)*.
  * **Alshemaimri et al. (2021):** Klasifikasi formal antipola *Unnecessary Data Retrieval* (UD) dan dampak penurunan *response time* hingga 71% *(Introduction.docx; Manuscript_Anonymous)*.
  * **Yusmita et al. (2025):** Kuantifikasi *overhead* memori *object materialization* pada ORM (977 KB vs 357 KB pada Raw SQL) *(Introduction.docx; Manuscript_Anonymous)*.
  * **Majerik & Borkovcova (2023):** Penanganan *lazy loading* ORM dari 12.748 kueri menjadi 2 kueri *(Introduction.docx; Manuscript_Anonymous)*.
  * **Kundan & Makwana (2025):** Penurunan penggunaan memori 35–40% melalui *selective field loading* di Laravel *(Introduction.docx; Manuscript_Anonymous)*.

#### **d. Profil Objek Penelitian (Koladi)**
* **Arsitektur Sistem Koladi:**  
  Aplikasi SaaS *collaborative virtual office* multi-tenant berbasis Laravel 12.x, PostgreSQL 15, Redis 7.x, dan dikontainerisasi dengan Docker Compose (`koladi_net`).
* **Modul & Jalur DSS yang Diteliti:**  
  Jalur DSS pada halaman statistik kinerja yang diuji melalui dua *endpoint*:
  1. `GET /statistik/suggestions` (merepresentasikan jalur rekomendasi DSS via `SnapshotManager`).
  2. `GET /statistik/modal-data` (merepresentasikan respons DSS lengkap: metrik utama, daftar *urgent tasks*, dan *workload distribution*).
* **Komponen & Alur Kalkulasi DSS:**  
  `DSSService` \\(\rightarrow\\) `SnapshotManager` \\(\rightarrow\\) `MetricsCalculator` \\(\rightarrow\\) `TrendAnalyzer` \\(\rightarrow\\) `RecommendationEngine`, serta fungsi spesifik `getUrgentTasks()` dan `getWorkloadDistribution()`. Menghasilkan 22 metrik produktivitas mentah per *workspace* pada setiap *request*.
* **Sumber Informasi:** *(Konteks penelitian terbaru.md; Introduction.docx; Manuscript_Anonymous)*.

---

### **2. PEMETAAN UNTUK 2.3. METODE PENGEMBANGAN PRODUK**

* **Status Penyebutan Metode SDLC Spesifik:**  
  ❌ **Bagian metode SDLC konvensional (seperti Waterfall, Agile/Scrum, atau Prototyping) belum disebutkan secara eksplisit di dalam sumber yang tersedia.**
* **Kerangka Rekayasa Perangkat Lunak yang Tercatat dalam Sumber:**  
  Sumber-sumber yang ada mendokumentasikan proses refaktorisasi dan optimasi sistem Koladi melalui pendekatan **Experimental Software Engineering (Wohlin et al., 2000)** yang disusun ke dalam alur *pipeline* 5 fase:
  1. **Fase 1: *Baseline Profiling*** (Diagnosis awal *bottleneck* menggunakan Laravel Telescope untuk mendeteksi *database-backed snapshot*, *N+1 checklists*, *column over-fetching*, dan *query loop*).
  2. **Fase 2: *Implementation / Refactoring*** (Pengembangan dan penerapan intervensi optimasi pada *codebase* Koladi).
  3. **Fase 3: *Controlled Performance Measurement*** (Pengujian *benchmark* sekuensial terkontrol di Docker Compose).
  4. **Fase 4: *Measurement and Data Collection*** (Pengumpulan data mentah 240 pengukuran via *instrumentation suite* ke CSV).
  5. **Fase 5: *Statistical Evaluation*** (Analisis deskriptif, persentase perubahan, dan *paired-sample t-test*).

* **Tahapan Refaktorisasi / Optimasi Produk Spesifik (Fase Implementation):**
  * **Optimasi Caching:** Mengganti *database-backed snapshot* (`CACHE_STORE=database`) menjadi Redis *Cache-Aside* (`CACHE_STORE=redis`) dan mengimplementasikan `TaskObserver` (`created`, `updated`, `deleted`) serta `TaskAssignmentObserver` (mutasi tabel *pivot* `task_assignments`).
  * **Optimasi ORM Point 1:** Merestrukturisasi `getTasksInPeriod()` dari `->with(['assignedUsers'])` menjadi `->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])` untuk membatasi pemuatan kolom dan menghapus *N+1 query* pada `getProgressPercentage()`.
  * **Optimasi ORM Point 2:** Merestrukturisasi `getWorkloadDistribution()` dari kueri `Task::whereHas()` di dalam *loop* anggota menjadi *batch retrieval* dan *in-memory aggregation*.
* **Sumber Informasi:** *(Konteks penelitian terbaru.md; Introduction.docx; Manuscript_Anonymous)*.

---

### **3. PEMETAAN UNTUK 2.4. METODE ANALISIS / RISET YANG DIGUNAKAN**

#### **a. Desain Riset / Eksperimen**
* **Desain:** *Quantitative experiment* dengan desain *Controlled Before-After Benchmarking* berbasis kerangka *Experimental Software Engineering* (Wohlin et al., 2000).
* **Karakteristik:** *Isolation testing* dan *configuration testing* sekuensial (bukan *concurrent load testing* atau *stress testing*) untuk mengisolasi dampak intervensi teknis pada kondisi data dan parameter *request* yang identik.
* **Sitasi Metodologi:** Wohlin et al. (2000), Alnuhait et al. (2024), Privalov & Stupina (2024), Putra et al. (2025).

#### **b. Variabel Eksperimen**
* **Variabel Bebas (Independen):**
  1. **Konfigurasi Sistem:** *Baseline* (Database-backed snapshot & unrestricted ORM) vs. *Optimized* (Redis Cache-Aside + Event-Driven Observer Invalidation & ORM Query Optimization).
  2. **Kondisi Cache:** *Cache Miss* (diukur setelah pengosongan *cache*) vs. *Cache Hit* (diukur setelah *populasi cache*).
  3. **Volume Data Sintetis:** *Small* (50 task, 5 anggota), *Medium* (200 task, 15 anggota), dan *Large* (500 task, 30 anggota).
* **Variabel Terikat (Dependen) / 4 Metrik Kinerja:**
  1. **Response Time (ms):** Durasi *end-to-end* HTTP request-response.
  2. **SQL Query Count (query/request):** Jumlah kueri SQL yang dikirim ke PostgreSQL per *request*.
  3. **Memory Utilization (MB):** Puncak alokasi RAM proses PHP per *request* via `memory_get_peak_usage(true)`.
  4. **Cache Hit Ratio (%):** Proporsi *request* yang dilayani dari *cache* (dihitung via Redis *keyspace stats* `hits / (hits + misses)` dan dibaca bersama *query count*).

#### **c. Prosedur dan Instrumen Pengujian**
* **Instrumen Pengukuran (*Benchmark Instrumentation Suite*):**
  * **Middleware Laravel Benchmark:** Menangkap metrik internal (`query count` via query listener dan `memory peak usage`) khusus *environment* local/testing dengan header HTTP khusus.
  * **Node.js Benchmark Runner:** Mengeksekusi HTTP *request* sekuensial dan mencatat *end-to-end response time*.
  * **CSV Validator:** Mengutifikasi dan memvalidasi keutuhan data CSV (240 baris valid).
  * **Laravel Telescope:** Digunakan hanya pada tahap *baseline profiling* awal untuk diagnosis diagnosis visual *bottleneck*.
* **Perlakuan *Warm-up* & Repetisi:**
  * **Warm-up:** 3 *request warm-up* di awal setiap kombinasi untuk mengeliminasi bias *cold-start* (dikeluarkan dari CSV analisis).
  * **Repetisi Valid:** 10 repetisi valid per kombinasi.
  * **Total Data Valid:** \\(3\text{ volume data} \times 2\text{ endpoint} \times 2\text{ konfigurasi} \times 2\text{ kondisi cache} \times 10\text{ repetisi} = 240\text{ pengukuran valid}\\) (120 *baseline* + 120 *optimized*).
* **Sitasi Prosedur:** Henning & Hasselbring (2022), Traini et al. (2023), Japke et al. (2025), Yusmita et al. (2025).

#### **d. Rencana Analisis Data**
* **Statistik Deskriptif:** Menghitung *Mean*, *Minimum*, *Maximum*, dan *Standard Deviation* untuk menyajikan kecenderungan umum dan stabilitas sebaran hasil.
* **Persentase Perubahan (*Practical Significance*):** Menghitung persentase penurunan/peningkatan metrik (\\(\frac{\text{Baseline} - \text{Optimized}}{\text{Baseline}} \times 100\%\\)) untuk menilai dampak nyata optimasi.
* **Uji Hipotesis Komparatif (*Paired-Sample t-Test*):**  
  Membandingkan 10 pasang data repetisi *baseline* vs. *optimized* pada tingkat signifikansi \\(\alpha = 0,05\\). Uji berpasangan ini tepat karena setiap observasi *baseline* memiliki pasangan langsung yang identik secara parameter pada *optimized*. Hasil signifikan secara statistik jika \\(p < 0,05\\) (untuk data deterministik seperti *query count* dengan variansi nol, dilaporkan sebagai \\(p < 0,001\\)).
* **Sitasi Rencana Analisis:** Uddin & Lu (2024), Japke et al. (2025), Privalov & Stupina (2024).
* **Sumber Informasi:** *(Konteks penelitian terbaru.md; Introduction.docx; Manuscript_Anonymous)*.

---

💡 **Saran Nudge:**
Pemetaan di atas mencakup seluruh detail teoretis, profil sistem, teknis ORM/Redis, hingga struktur kuantitatif 240 pengukuran. Jika Anda memerlukan bantuan menyusun *drafting* kalimat utuh untuk paragraf pada salah satu dari ketiga sub-bab tersebut, silakan beri tahu!