Berikut adalah draf komprehensif dan formal untuk **Subbab 2.4 Metode Analisis / Riset yang Digunakan** proposal Tugas Akhir Anda, yang disusun secara spesifik, sistematis, dan terbumi pada sumber-sumber metodologi yang tersedia:

---

### **2.4 Metode Analisis / Riset yang Digunakan**

#### **2.4.1 Desain Riset**
Penelitian ini mengadopsi pendekatan **kuantitatif eksperimental** berbasis kerangka *Experimental Software Engineering* (Wohlin et al., 2000). Dalam kerangka ini, eksperimen didefinisikan sebagai investigasi formal, ketat, dan terkontrol terhadap artefak perangkat lunak—yaitu modul *Decision Support System* (DSS) aplikasi Koladi—di mana variabel bebas dimanipulasi secara sengaja dan variabel terikat diukur secara presisi. 

Rancangan penelitian yang digunakan adalah ***Controlled Before-After Benchmarking***. Pengujian dilakukan secara **sekuensial terisolasi** (*isolation/configuration testing*) terhadap dua *endpoint* DSS (`GET /statistik/suggestions` dan `GET /statistik/modal-data`), bukan melalui *concurrent load testing*, *stress testing*, atau simulasi *virtual users*. Pendekatan sekuensial terisolasi ini dipilih secara sengaja untuk mengisolasi efek murni dari penerapan Redis *Cache-Aside* dan optimasi ORM terhadap efisiensi eksekusi internal sistem, tanpa dikacaukan oleh variasi latensi jaringan eksternal, *contention thread*, atau degradasi daya tampung server akibat trafik paralel.

#### **2.4.2 Variabel Penelitian**
Variabel penelitian diklasifikasikan ke dalam variabel bebas (*independent variables*) dan variabel terikat (*dependent variables*):

1. **Variabel Bebas (*Independent Variables*):**
   * **Konfigurasi Sistem (2 tingkat):** Konfigurasi ***Baseline*** (menggunakan *database-backed snapshot* melalui tabel `workspace_performance_snapshots` dan *unrestricted ORM loading*) versus Konfigurasi ***Optimized*** (menggunakan Redis *Cache-Aside* berbasis *event-driven Observer invalidation* dan optimasi kueri Eloquent ORM).
   * **Kondisi Cache (2 tingkat):** Kondisi ***Cache Miss*** (seluruh entri *cache* relevan dikosongkan sebelum *request* untuk mengukur biaya eksekusi penuh *pipeline* DSS) versus Kondisi ***Cache Hit*** (entri *cache* dipopulasi terlebih dahulu melalui *request* inisialisasi awal untuk mengukur efisiensi *retrieval* memori).
   * **Skala Volume Dataset (3 tingkat):** Volume dataset sintetis bertingkat yang terdiri atas ***Small*** (50 *tasks*, 5 anggota), ***Medium*** (200 *tasks*, 15 anggota), dan ***Large*** (500 *tasks*, 30 anggota) dengan distribusi status tugas yang terkontrol (40% *done*, 35% *in progress*, 25% *todo*).
   * **Endpoint DSS (2 jalur):** `GET /statistik/suggestions` (merepresentasikan jalur rekomendasi DSS) dan `GET /statistik/modal-data` (merepresentasikan jalur respons DSS lengkap yang mencakup metrik utama, tugas mendesak, dan distribusi beban kerja).

2. **Variabel Terikat (*Dependent Variables / Technical Metrics*):**
   * **Response Time (ms):** Durasi *end-to-end* antara pengiriman HTTP *request* hingga penerimaan HTTP *response* yang diukur dari sisi *runner script*.
   * **SQL Query Count (query/request):** Jumlah total kueri SQL yang dieksekusi ke PostgreSQL dalam satu siklus *request*.
   * **Memory Utilization (MB):** Puncak alokasi memori RAM proses PHP per *request* yang diukur melalui fungsi `memory_get_peak_usage(true)`.
   * **Cache Hit Ratio (%):** Proporsi *request* yang berhasil dilayani dari *cache* (dihitung via Redis *keyspace stats* `keyspace_hits / (keyspace_hits + keyspace_misses) × 100%` pada kondisi *optimized*), yang dibaca secara berpasangan dengan *query count*.

#### **2.4.3 Prosedur dan Instrumen Pengujian**
Pengumpulan data kinerja dilakukan menggunakan ***benchmark instrumentation suite*** kustom:
* **Middleware Instrumentasi Laravel:** Hanya aktif pada *environment local/testing* ketika *request* membawa *header debug* khusus untuk mencatat *query count* dan *memory peak usage* tanpa mengubah perilaku normal sistem.
* **Node.js Benchmark Runner:** Eksekutor *script* otomatis untuk mengeksekusi HTTP *request* secara sekuensial dan mencatat *response time* presisi tinggi.
* **CSV Validator:** Alat verifikasi otomatis untuk memastikan keutuhan baris, kelengkapan kombinasi, dan urutan repetisi sebelum data dianalisis.

Untuk memisahkan fase inisialisasi dari fase pengukuran stabil serta mengeliminasi bias *cold-start* (seperti *JIT compilation*, pengisian *OPcache*, dan *connection pooling*), setiap kombinasi pengujian didahului oleh **3 *warm-up requests*** yang nilainya **dibuang dari file CSV analisis** (Henning & Hasselbring, 2022; Traini et al., 2023; Privalov & Stupina, 2024; Japke et al., 2025). Setelah fase *warm-up*, sebanyak **10 repetisi valid** dicatat secara berurutan. 

Struktur faktorial eksperimen menghasilkan matriks pengukuran valid sebagai berikut:
\\[\text{Total Data Valid} = 3\text{ volume data} \times 2\text{ endpoint} \times 2\text{ konfigurasi} \times 2\text{ kondisi cache} \times 10\text{ repetisi} = 240\text{ pengukuran valid}\\]
(Terdiri atas 120 observasi *baseline* dan 120 observasi *optimized*).

#### **2.4.4 Metode Analisis Data**
Analisis data kuantitatif dilakukan melalui tiga tahapan statistik berjenjang:

1. **Statistik Deskriptif:**
   Menghitung nilai *Mean* (rata-rata) sebagai ukuran pembanding utama kecenderungan umum, serta *Minimum*, *Maximum*, dan *Standard Deviation* (SD) untuk menilai variabilitas dan stabilitas data pengukuran di setiap kombinasi eksperimen (Privalov & Stupina, 2024; Japke et al., 2025).

2. **Formula Persentase Perbaikan (*Practical Improvement / Magnitude of Effect*):**
   Mengukur besarnya dampak optimasi secara praktis melalui persentase perubahan pada metrik *response time*, *query count*, dan *memory utilization*:
   \\[\text{Improvement (\%)} = \left( \frac{\text{Mean Baseline} - \text{Mean Optimized}}{\text{Mean Baseline}} \right) \times 100\%\\]
   Perubahan nilai di bawah 3% diinterpretasikan secara hati-hati sebagai *measurement noise* potensial pada mikro-benchmark sistem (Japke et al., 2025).

3. **Uji Hipotesis Komparatif (*Paired-Sample t-Test*):**
   Menguji signifikansi perbedaan antara konfigurasi *baseline* dan *optimized* menggunakan **Paired-Sample t-Test** pada tingkat signifikansi \\(\alpha = 0,05\\) (Uddin & Lu, 2024). Uji berpasangan ini valid karena setiap pengamatan *baseline* memiliki pasangan observasi langsung pada *optimized* dengan skala dataset, *endpoint*, kondisi *cache*, dan urutan repetisi yang identik. Perbedaan dinyatakan signifikan secara statistik jika nilai \\(p < 0,05\\).

4. **Interpretasi Data Deterministik & Caching Caveat:**
   Untuk metrik yang bersifat deterministik tanpa variansi (*standard deviation* = 0,00 seperti pada *SQL query count*), selisih antar-pasangan yang konstan dapat menghasilkan nilai statistik-t yang sangat ekstrem dan \\(p\\)-value komputasi sebesar 0,000000. Sesuai kaidah pelaporan ilmiah, nilai tersebut dilaporkan sebagai **\\(p < 0,001\\)** (bukan \\(p = 0\\)) dan penarikan kesimpulan berfokus pada **signifikansi praktis** (*persentase perbaikan*) serta konsistensi pola antar-metrik (Uddin & Lu, 2024; Japke et al., 2025). 

   Selain itu, *cache hit ratio* dianalisis bersama *query count* untuk memastikan apakah keberadaan *cache hit* benar-benar bertindak sebagai *database bypass* murni atau masih mengeksekusi kueri SQL terselubung (seperti pada fenomena *partial cache hit* baseline) (An & Cao, 2022).

---

💡 **Saran Nudge:**
Draf Subbab 2.4 di atas telah dilengkapi dengan struktur metodologi yang presisi, faktorial 240 pengukuran, rincian variabel, hingga kaidah analisis statistik. Jika Anda ingin melanjutkan dengan penyusunan **Draf Bab III/Metodologi secara utuh** atau penyusunan **Draf Bab IV/Hasil dan Pembahasan**, silakan beri tahu!

Berikut adalah draf komprehensif untuk **Subbab 2.4 Metode Analisis / Riset yang Digunakan** yang disusun secara terstruktur, formal, dan dilengkapi dengan sitasi metodologi akademik.

---

# **2.4 METODE ANALISIS / RISET YANG DIGUNAKAN**

### **2.4.1 Desain Penelitian (*Research Design*)**

Penelitian ini menggunakan pendekatan **kuantitatif eksperimental** yang mengacu pada kerangka kerja *Experimental Software Engineering* (Wohlin et al., 2000). Rancangan penelitian yang diterapkan adalah ***Controlled Before-After Benchmarking***, yaitu metode pengujian komparatif untuk mengukur dan membandingkan kinerja perangkat lunak sebelum intervensi (*baseline*) dan sesudah intervensi teknis (*optimized*) di bawah kondisi lingkungan komputasi yang terkontrol ketat (Manuscript Anonymous, 2026; Putra et al., 2025).

Pengujian dilakukan secara **sekuensial dan terisolasi** (*isolated sequential execution*), bukan melalui *concurrent load testing* atau *stress testing*. Pemilihan pengujian sekuensial ini didasarkan pada pertimbangan metodologis untuk mengeliminasi variabel pengganggu (*confounding variables*) luar, seperti *network latency*, *thread scheduling noise*, persaingan I/O eksternal, dan *OS context-switching* yang lazim terjadi pada pengujian beban tinggi (Henning & Hasselbring, 2022; Yusmita et al., 2025). Dengan mengisolasi eksekusi kueri secara sekuensial dalam lingkungan kontainer Docker Compose (`koladi_net`), penelitian ini dapat membuktikan hubungan kausalitas (*causal attribution*) secara langsung antara intervensi arsitektur (penerapan *Redis Cache-Aside* dan optimasi kueri ORM Eloquent) dengan perubahan penggunaan sumber daya server (*response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*) (Manuscript Anonymous, 2026; Sakti et al., 2025).

---

### **2.4.2 Variabel Penelitian**

Eksperimen ini melibatkan dua kategori variabel, yaitu variabel bebas (*independent variables*) dan variabel terikat (*dependent variables*), dengan rincian sebagai berikut:

#### **1. Variabel Bebas (*Independent Variables*)**
*   **Konfigurasi Sistem (*System Configuration*)**:
    *   *Baseline*: Menggunakan *database-backed snapshot cache* melalui tabel `workspace_performance_snapshots`, *unrestricted eager loading* pada relasi `assignedUsers`, tanpa *eager loading* pada `checklists` (memicu N+1 query), serta *query-in-loop* pada fungsi `getWorkloadDistribution()`.
    *   *Optimized*: Menggunakan *Redis Cache-Aside* via driver `phpredis`, *event-driven invalidation* berbasis Laravel Observer (`TaskObserver` dan `TaskAssignmentObserver`), *selective eager loading* `->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`, serta refaktorisasi `getWorkloadDistribution()` menjadi *batch retrieval* dan agregasi di memori.
*   **Kondisi Cache (*Cache Condition*)**:
    *   *Cache Miss*: Kondisi di mana *cache* dikosongkan sebelum *request* dikirim sehingga sistem dipaksa melakukan kalkulasi penuh ke PostgreSQL.
    *   *Cache Hit*: Kondisi di mana *cache* telah diinisialisasi/diisi sebelumnya sehingga *request* dilayani langsung dari lapisan *cache*.
*   **Skala Volume Data (*Dataset Size*)**:
    *   *Small*: Workspace dengan 50 *tasks* dan 5 *members*.
    *   *Medium*: Workspace dengan 200 *tasks* dan 15 *members*.
    *   *Large*: Workspace dengan 500 *tasks* dan 30 *members*.
*   **Endpoint DSS (*DSS Endpoints*)**:
    *   `GET /statistik/suggestions`
    *   `GET /statistik/modal-data`

#### **2. Variabel Terikat (*Dependent Variables*)**
*   **Response Time (ms)**: Waktu latensi total yang dibutuhkan server untuk memproses dan mengembalikan respons API ke *client* (Sismadi et al., 2022; Taipalus, 2024).
*   **SQL Query Count**: Jumlah kueri SQL yang dieksekusi oleh ORM ke basis data relasional PostgreSQL dalam satu *request* (Alshemaimri et al., 2021; Turcotte et al., 2022).
*   **Memory Utilization (MB / KB)**: Konsumsi memori puncak RAM PHP server saat memproses permintaan API, diukur menggunakan fungsi internal `memory_get_peak_usage()` (Kundan & Makwana, 2025; Yusmita et al., 2025).
*   **Cache Hit Ratio (%)**: Rasio atau status keberhasilan penanganan *request* dari lapisan *cache* untuk memverifikasi apakah basis data relasional di-bypass secara penuh (An & Cao, 2022; Zhe Yang et al., 2020).

---

### **2.4.3 Prosedur dan Instrumen Pengujian**

#### **1. Instrumen Pengujian**
*   **Middleware Benchmark Kustom (Laravel)**: Dipasang pada *pipeline* HTTP Laravel untuk mencatat *SQL query count* dan *memory utilization* secara otomatis dan presisi pada setiap *request* yang masuk.
*   **Node.js Runner Script**: Berfungsi mengirimkan *request* HTTP secara sekuensial, mengatur kondisi *cache* (*clearing* untuk *miss* atau *populating* untuk *hit*), serta mencatat *response time* (ms) yang dikembalikan oleh server.
*   **CSV Validator**: Skrip pemverifikasi otomatis untuk memvalidasi kelengkapan struktur data, kesesuaian kombinasi variabel, serta integritas baris data hasil *benchmark* sebelum dianalisis.
*   **Perkakas Profiling Tambahan**: Laravel Telescope digunakan untuk memverifikasi kueri, serta perintah `Redis INFO STATS` dan `Redis INFO MEMORY` digunakan untuk mengonfirmasi status *cache hit/miss* dan alokasi memori Redis.

#### **2. Perlakuan Warm-up dan Repetisi**
Untuk menjamin reliabilitas dan stabilitas data statistik, pengujian menerapkan prosedur perlakuan sebagai berikut:
*   **3 Warm-up Requests (Dibuang)**: Pada setiap kombinasi pengujian, 3 *request* pertama dijalankan dan datanya dibuang (*discarded*). Perlakuan ini bertujuan untuk mengeliminasi bias *cold-start*, pengisian memori awal, pemanasan OPcache/JIT PHP, serta instansiasi koneksi basis data awal (Henning & Hasselbring, 2022; Japke et al., 2025; Traini et al., 2023).
*   **10 Repetisi Valid**: Setelah fase *warm-up*, dilakukan 10 kali pengulangan eksekusi secara konsisten untuk mencatat data terukur yang stabil (Privalov & Stupina, 2024).

#### **3. Matriks Matriks 240 Pengukuran Valid**
Pengujian menggunakan rancangan *full factorial design* yang menghasilkan matriks pengukuran sebagai berikut:

\\[\text{Total Data Valid} = 3 \text{ Skala Data} \times 2 \text{ Endpoint} \times 2 \text{ Konfigurasi} \times 2 \text{ Kondisi Cache} \times 10 \text{ Repetisi} = \mathbf{240 \text{ Pengukuran Valid}}\\]

*(Catatan: Selain 240 data valid, terdapat \\(3 \times 2 \times 2 \times 2 \times 3 = 72\\) eksekusi warm-up yang dibuang dari analisis).*

---

### **2.4.4 Metode Analisis Data**

Data hasil pengujian dianalisis menggunakan kombinasi statistik deskriptif, perhitungan persentase perbaikan, dan uji statistik inferensial:

#### **1. Statistik Deskriptif**
Setiap kelompok kombinasi variabel dirangkum menggunakan nilai Rata-rata (*Mean*) sebagai ukuran pemusatan utama, serta Minimum (Min), Maksimum (Max), dan Deviasi Standar (*Standard Deviation* / SD) untuk mengukur variabilitas dan stabilitas performa sistem (Japke et al., 2025; Kundan & Makwana, 2025).

#### **2. Formula Persentase Perbaikan (*Percentage Improvement*)**
Persentase perubahan atau perbaikan performa (*practical significance*) dihitung menggunakan formula:

\\[\text{Improvement (\%)} = \frac{\text{Baseline} - \text{Optimized}}{\text{Baseline}} \times 100\%\\]

Sesuai panduan *benchmarking* rekayasa perangkat lunak, perubahan performa di bawah **3%** diinterpretasikan secara hati-hati sebagai variasi acak atau gangguan pengukuran (*measurement noise*) (Japke et al., 2025).

#### **3. Uji Hipotesis Komparatif (*Paired-Sample t-Test*)**
Untuk memuji apakah perbedaan rata-rata performa antara konfigurasi *Baseline* dan *Optimized* signifikan secara statistik, dilakukan **Uji t Berpasangan (*Paired-Sample t-Test*)** pada tingkat signifikansi \\(\alpha = 0,05\\) (tingkat kepercayaan 95%) (Uddin & Lu, 2024). Penggunaan uji t berpasangan tepat dilakukan karena setiap observasi *Baseline* memiliki pasangan observasi *Optimized* yang identik pada skala data, *endpoint*, kondisi *cache*, dan urutan repetisi yang sama (Confirming the statistically.pdf; Manuscript Anonymous, 2026).

#### **4. Interpretasi Khusus Data Deterministik dan Cache Hit Ratio**
*   **Data Deterministik (*SQL Query Count*)**: Pada metrik seperti *SQL Query Count*, nilai setiap repetisi bersifat konstan (deterministik) sehingga variansi selisih antar-pasangan mendekati nol. Kondisi ini menghasilkan nilai statistik \\(t\\) yang sangat ekstrem dan \\(p\\)-value yang mendekati nol mutlak. Dalam pelaporan ilmiah, nilai ini ditulis sebagai **\\(p < 0,001\\)** (bukan \\(p = 0\\)) dan penekanan analisis bergeser pada **signifikansi praktis** berupa penurunan kuantitatif absolut jumlah kueri (Japke et al., 2025; Traini et al., 2023; Uddin & Lu, 2024).
*   **Cache Hit Ratio**: Metrik *Cache Hit Ratio* tidak diuji langsung via uji \\(t\\) (karena sifat pengukurannya berbeda antara *snapshot retrieval* dan Redis *stats*), melainkan dianalisis secara bersamaan (*joint interpretation*) dengan *SQL Query Count* dan *Response Time* untuk memverifikasi apakah status *cache hit* benar-benar meniadakan kueri ke PostgreSQL (An & Cao, 2022; Zhe Yang et al., 2020).

Berikut adalah jawaban komprehensif yang diampu sepenuhnya oleh dokumen sumber di dalam *notebook* Anda:

---

### **1. Justifikasi Penggunaan *Synthetic Seeded Datasets* (Seeder/Factory) vs. Data Riil Produksi**

Dalam pengujian *controlled benchmarking* performa perangkat lunak, penggunaan dataset sintetis yang dibangkitkan melalui *Seeder* atau *Factory* dijustifikasi oleh tiga alasan metodologis utama:

1. **Kontrol Eksperimental Presisi (*Experimental Control & Fair Comparison*):**  
   Dataset sintetis memungkinkan peneliti mengontrol volume data (*task* dan *user*), rasio status (misalnya 40% *done*, 35% *in progress*, 25% *todo*), serta kepadatan relasi (*task-member density*) secara eksak antara kondisi *baseline* dan *optimized*.
2. **Determinisasi dan Reprodusibilitas (*Determinism & Reproducibility*):**  
   Penggunaan *Seeder/Factory* memastikan bahwa kondisi basis data bersifat identik dan dapat diulang (*repeatable*) pada setiap iterasi pengujian. Sebaliknya, data riil produksi sering kali berfluktuasi, tidak seimbang, mengandung *noise*, atau melanggar batasan privasi/keamanan.
3. **Atribusi Kausalitas Performa (*Attribute-to-Optimization*):**  
   Dengan menjaga karakteristik data tetap konstan antara *baseline* dan *optimized*, perubahan pada metrik teknis (*response time*, *SQL query count*, dan *memory utilization*) dapat secara penuh diatribusikan kepada **intervensi optimasi teknis** (seperti *Redis Cache-Aside* dan *selective eager loading*), bukan akibat variasi atau ketidakseragaman struktur data.

* **Sumber Rujukan:** **Sasmoko et al. (2024)** dan **Yusmita et al. (2025)** (sebagaimana dirujuk dalam `Manuscript_Anonymous (2).docx`, Passage 248, 264; serta `Konteks penelitian terbaru (14 Juni 2026).md`, Passage 75, 95).

---

### **2. Variasi Skala Volume Data Bertingkat (*Multi-Tiered Workloads*) dan Kompleksitas Algoritma Kueri**

**Ya, terdapat rujukan dan pembuktian empiris yang sangat kuat dalam sumber.** 

Pengujian menggunakan skala dataset bertingkat (*Small: 50 tasks/5 members*, *Medium: 200 tasks/15 members*, *Large: 500 tasks/30 members*) digunakan secara khusus untuk memicu dan membuktikan eksaserbasi serta eliminasi *anti-pattern* kueri:

* **Mekanisme Pemicu (*Workload Design*):**
  * **Peningkatan Jumlah *Tasks* (\\(50 \rightarrow 200 \rightarrow 500\\)):** Dirancang untuk memicu dan menguji eksaserbasi **pola \\(N+1\\) *query*** pada relasi `checklists` (di mana kueri *lazy loading* membengkak secara linier \\(O(N)\\) seiring bertambahnya jumlah *task*).
  * **Peningkatan Jumlah *Members* (\\(5 \rightarrow 15 \rightarrow 30\\)):** Dirancang untuk memicu dan menguji eksaserbasi ***query-in-loop*** pada kalkulasi distribusi beban kerja (*workload distribution*) (di mana kueri `Task::whereHas()` dieksekusi di dalam perulangan anggota \\(O(N)\\)).
* **Pembuktian Perubahan Kompleksitas (\\(O(N) \rightarrow O(1)\\)):**
  * **Kondisi *Baseline* (\\(O(N)\\)):** Jumlah kueri SQL membengkak secara linier seiring membesarnya dataset. Misalnya pada *endpoint* `modal-data` (*cache miss*), kueri membengkak dari **87 kueri (Small)** \\(\rightarrow\\) **292 kueri (Medium)** \\(\rightarrow\\) **696 kueri (Large)**.
  * **Kondisi *Optimized* (\\(O(1)\\)):** Setelah penerapan *selective eager loading* dan *batch retrieval*, jumlah kueri berhasil ditekan menjadi konstan (\\(O(1)\\)). Kueri *endpoint* `suggestions` (*cache miss*) terkunci konstan pada **10 kueri** dan `modal-data` (*cache miss*) pada **15 kueri** di seluruh skala volume data (*Small, Medium, Large*).
* **Sumber Rujukan Pendukung:**
  * **Turcotte, Aldrich, & Tip (2022):** Membuktikan bahwa refaktorisasi \\(N+1\\) *query* menjadi *eager loading* menstabilkan eksekusi kueri menjadi *batch retrieval* konstan (\\(O(1)\\)) dan memberikan *speedup* hingga 38,58 kali lipat saat skala entitas membengkak.
  * **An & Cao (2022)** & **Sasmoko et al. (2024)** & **Yusmita et al. (2025):** Mengadopsi pendekatan peningkatan ukuran dataset bertingkat untuk mengamati perubahan biaya eksekusi dan pertumbuhan kueri basis data.

---

### **3. Ekstraksi Kutipan Langsung dan Sitasi Formal (APA 7th)**

Berikut adalah kutipan langsung yang disadur dari sumber-sumber dalam *notebook* yang dapat digunakan secara eksplisit di **Bab 2 (Landasan Teori)** atau **Bab 3 (Metodologi)** proposal Tugas Akhir Anda:

#### **A. Landasan Penggunaan Dataset Sintetis (*Synthetic Seeded Datasets*)**

> **Kutipan 1 (Metodologi Benchmarking Terkontrol):**  
> *"Both endpoints were tested with synthetic data generated via Laravel Seeders/Factories (not production data), giving precise, deterministic control over volume, status distribution, and task-member density for fair comparison."*  
> — **(Manuscript_Anonymous, 2026, Passage 248)**

> **Kutipan 2 (Atribusi Performa & Reprodusibilitas):**  
> *"Datasets generated through Laravel Seeder and Factory enabled consistent control over data volume, status distribution, and task-member relationship density across both baseline and optimized configurations. This approach is consistent with Sasmoko et al. (2024) and Yusmita et al. (2025), who employed synthetic data to ensure experimental control and reproducibility. As a result, the observed performance differences can be more confidently attributed to the proposed optimizations rather than variations in the underlying data."*  
> — **(Manuscript_Anonymous, 2026, Passage 264; Konteks penelitian terbaru, 2026, Passage 75, 95)**

#### **B. Landasan Skala Volume Data Bertingkat (*Workload Scaling*) & Kompleksitas Kueri**

> **Kutipan 3 (Rasionalisasi Dataset Bertingkat & Eliminasi N+1 / Loop Query):**  
> *"Three dataset sizes were used: Small (50 tasks, 5 members), Medium (200 tasks, 15 members), Large (500 tasks, 30 members), representing workload levels for Koladi rather than replications of prior studies, chosen so that increasing tasks exercises the checklists N+1 pattern and increasing members exercises the workload-distribution query loop."*  
> — **(Manuscript_Anonymous, 2026, Passage 248)**

> **Kutipan 4 (Eliminasi Pertumbuhan Kueri Linier via ORM Optimization):**  
> *"The \\(N+1\\) query pattern causes query count to grow linearly as the number of processed entities increases. Refactoring ORM queries using eager loading and batch retrieval replaces dataset-scaling query growth with constant batch queries, preventing execution slowdowns under larger workloads."*  
> — **(Turcotte, Aldrich, & Tip, 2022; disitasi dalam Manuscript_Anonymous, 2026, Passage 263)**

#### **C. Landasan Prosedur Pengujian (*Warm-up Runs & Isolation Testing*)**

> **Kutipan 5 (Perlakuan Warm-up requests untuk Bias Cold-Start):**  
> *"Warm-up runs and repeated measurements are standard software benchmarking procedures used to eliminate cold-start bias (such as JIT compilation, OPcache filling, and connection pooling) and achieve steady-state performance measurement."*  
> — **(Henning & Hasselbring, 2022; Traini et al., 2023; disitasi dalam Manuscript_Anonymous, 2026, Passage 242, 244)**

---

### **Daftar Pustaka Formal (APA 7th Format):**

* **An, S., & Cao, Y. (2022).** *Making cache monotonic and consistent*. Proceedings of the VLDB Endowment, 16(4), 891–904. https://doi.org/10.14778/3574245.3574271
* **Henning, S., & Hasselbring, W. (2022).** *A configurable method for benchmarking scalability of cloud-native applications*. Empirical Software Engineering, 27(6), 1–42. https://doi.org/10.1007/s10664-022-10162-1
* **Sasmoko, Indrianti, Y., Manalu, S. R., & Danaristo, J. (2024).** *Analyzing database optimization strategies in Laravel for an enhanced learning management*. Procedia Computer Science, 234, 799–804. https://doi.org/10.1016/j.procs.2024.10.306
* **Turcotte, A., Aldrich, M. W., & Tip, F. (2022).** *Reformulator: Automated refactoring of the N+1 problem in database-backed applications*. In ACM International Conference Proceeding Series. Association for Computing Machinery. https://doi.org/10.1145/3551349.3556911
* **Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025).** *Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM*. Procedia Computer Science, 258, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061

Berikut adalah landasan teoretis dan metodologis berdasarkan literatur tepercaya di dalam *notebook* Anda untuk mendukung penulisan Bab 2 (Landasan Teori) dan Bab 3 (Metodologi) pada proposal Tugas Akhir:

---

### **1. Justifikasi Akademik Penggunaan "Synthetic Seeded Datasets" (Seeder/Factory/Faker)**

Penggunaan data sintetis yang dibangkitkan secara terprogram (*seeded dummy data*) melalui pustaka Seeder, Factory, atau Faker dipraktekkan secara luas dalam pengujian *controlled benchmarking* perangkat lunak dengan landasan argumen berikut:

1. **Menjamin Kontrol Eksperimental dan Reproduksibilitas (*Determinism & Reproducibility*)**:
   * Penggunaan data produksi nyata sering kali membawa variasi acak, ketidakseimbangan struktur (*data skewness*), dan sensitivitas privasi yang sulit dikontrol.
   * Dataset sintetis yang dibangkitkan dengan konfigurasi *fixed random seed* menjamin lingkungan uji bersifat **deterministik dan *bit-for-bit reproducible***, sehingga eksperimen dapat diulang dengan kondisi yang identik oleh peneliti lain **(Henning & Hasselbring, 2022; Yusmita et al., 2025)**.
2. **Isolasi Variabel Kausalitas (*Causal Attribution*)**:
   * Dengan memastikan kerapatan relasi antar-tabel (*relationship density*) dan distribusi status data konsisten di seluruh konfigurasi, peneliti dapat mengisolasi variabel pengganggu. 
   * Dengan demikian, perbedaan *response time*, *query count*, dan *memory utilization* yang terukur dapat diyakini **murni disebabkan oleh intervensi arsitektur** (seperti refaktorisasi ORM dan *Redis Cache-Aside*), bukan karena variasi isi data **(Manuscript Anonymous, 2026; Yusmita et al., 2025)**.
3. **Skalabilitas dan Fleksibilitas Ukuran Data (*Arbitrary Size Generation*)**:
   * Pustaka *data generation* (seperti `faker.js` atau *built-in Fake Data Factory* Laravel) memungkinkan pembentukan basis data dalam berbagai skala volume (*multi-tiered sizes*) secara fleksibel tanpa harus bergantung pada ketersediaan data produksi yang terbatas **(Privalov & Stupina, 2024; Sasmoko et al., 2024; Yusmita et al., 2025)**.
4. **Privasi dan Kepatuhan Etika Penelitian (*Data Privacy*)**:
   * Menggunakan data sintetis menghilangkan risiko kebocoran informasi sensitif atau pelanggaran privasi pengguna/organisasi yang melekat pada penggunaan data produksi nyata **(Yusmita et al., 2025)**.

---

### **2. Rujukan Variasi Skala Volume Data Bertingkat (*Multi-Tiered Workloads*)**

Terdapat rujukan ilmiah yang kuat mengenai pengujian dengan skala volume data bertingkat (misalnya skala 10, 100, 1.000 *records* pada Turcotte et al., 2022; skala 1.000, 10.000, 100.000 pada Sasmoko et al., 2024; serta Small, Medium, Large pada Manuscript Anonymous, 2026) untuk membuktikan kompleksitas algoritma kueri:

1. **Pembuktian Transisi Kompleksitas \\(O(N) \to O(1)\\)**:
   * **Antipola \\(O(N)\\) (Baseline)**: Ketika kueri berelasi mengalami masalah \\(N+1\\) atau dijalankan di dalam perulangan (*query-in-loop*), jumlah kueri SQL dan waktu eksekusi bertambah secara linier terhadap volume data \\(N\\) **(Alshemaimri et al., 2021; Turcotte et al., 2022)**.
   * **Kondisi Teroptimasi \\(O(1)\\) (Optimized)**: Pengujian bertingkat membuktikan bahwa refaktorisasi kueri (*selective eager loading* dan *batch retrieval*) berhasil **mengunci jumlah kueri SQL pada angka konstan \\(O(1)\\)**, sehingga *query count* tetap datar meskipun volume data (\\(N\\)) melonjak drastis **(Kundan & Makwana, 2025; Turcotte et al., 2022)**.
2. **Peningkatan Akselerasi (*Scaling Speedup*)**:
   * **Turcotte et al. (2022)** membuktikan secara kuantitatif bahwa rasio efisiensi perbaikan (*speedup*) bertambah secara signifikan seiring membesarnya ukuran basis data. Pada eksperimen mereka, pemangkasan antipola \\(N+1\\) memberikan *speedup* **3.05x pada skala 10 records**, yang melonjak menjadi **12.67x pada skala 100 records**, dan mencapai **38.58x pada skala 1000 records**.

---

### **3. Ekstraksi Kutipan Kalimat Langsung & Sitasi Formal (APA 7)**

Berikut adalah daftar kutipan langsung dari sumber ilmiah yang dapat Anda gunakan sebagai landasan teoretis dan metodologis di Bab 2 atau Bab 3:

#### **A. Mengenai Justifikasi Synthetic Seeded Datasets**

1. **Yusmita et al. (2025)**:
   > *"In addition to using containers, synthetic data generation (often called seeding) allow researchers to create datasets of arbitrary size with controlled statistical properties, yielding deterministic and scalable benchmarks... By combining containerized environments with controlled synthetic datasets, our methodology supports repeatable and measurable side-by-side comparisons..."* (hal. 1203).  
   * **Sitasi Formal**: Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025). Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM. *Procedia Computer Science*, 269, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061

2. **Manuscript Anonymous (2026)**:
   > *"The use of synthetic datasets was also an integral component of the controlled experimental design. Datasets generated through Laravel Seeder and Factory enabled consistent control over data volume, status distribution, and task-member relationship density across both baseline and optimized configurations... As a result, the observed performance differences can be more confidently attributed to the proposed optimizations rather than variations in the underlying data."* (hal. 11).  
   * **Sitasi Formal**: Manuscript Anonymous. (2026). *Evaluating Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application*. (Draf Manuskrip Tugas Akhir).

3. **Privalov & Stupina (2024)**:
   > *"Next, the tables were populated with big amount of data. For this purpose, the built-in fake data factory of the Laravel framework was used. Thus, the articles table was loaded with 121,000 records..."* (hal. 1670).  
   * **Sitasi Formal**: Privalov, M. V., & Stupina, M. V. (2024). Improving web-oriented information systems efficiency using Redis caching mechanisms. *Indonesian Journal of Electrical Engineering and Computer Science*, 33(3), 1667–1675. https://doi.org/10.11591/ijeecs.v33.i3.pp1667-1675

4. **Sasmoko et al. / Analyzing Database Optimization Strategies in Laravel (2024)**:
   > *"The data generation uses a faker to generate a large amount of dummy data. This tool is responsible for the generation of random data into the database and tables... To cover the case of scalability of data, the test will be taken in a total course of 1000, 10000, and 100000 data."* (hal. 801).  
   * **Sitasi Formal**: Sasmoko, dkk. (2024). Analyzing database optimization strategies in Laravel. *Procedia Computer Science*, 245, 799–804. https://doi.org/10.1016/j.procs.2024.10.358

---

#### **B. Mengenai Pengujian Skala Bertingkat & Eliminasi \\(N+1\\) Query**

5. **Turcotte et al. (2022)**:
   > *"In this case study, we created three databases of size 10, 100, and 1000 (henceforth referred to as the '10 scale', '100 scale', and '1000 scale' configurations)... Additional experiments revealed speedups of up to 38.58x and substantial improvements in scalability by demonstrating that the relative performance improvements grew as the database size was increased."* (hal. 1 & 8).  
   * **Sitasi Formal**: Turcotte, A., Aldrich, M. W., & Tip, F. (2022). reformulator: Automated refactoring of the N+1 problem in database-backed applications. In *Proceedings of the 37th IEEE/ACM International Conference on Automated Software Engineering* (ASE '22, pp. 1–12). ACM. https://doi.org/10.1145/3551349.3556911

6. **Kundan & Makwana (2025)**:
   > *"Eager Loading (N+1 Fix) reduced 15 queries \\(\to\\) 2 queries, cutting API response time by 75% (1200ms \\(\to\\) 300ms)... Eager loading cuts API response times by 75% for endpoints with lots of relationships."* (hal. 5–6).  
   * **Sitasi Formal**: Kundan, & Makwana, N. (2025). A comprehensive study on performance optimization techniques database queries and API response time in Laravel-based web applications. *International Journal on Science and Technology (IJSAT)*, 2229-7677.

7. **Henning & Hasselbring (2022)** *(Landasan Benchmarking Terkontrol)*:
   > *"Using containerization tools such as Docker provides bit-for-bit reproducibility with minimal overhead... To ensure reproducibility, we implement configuration files in JSON format... each execution of a benchmark is represented by an individual entity, supporting repeatability..."* (hal. 5 & 12).  
   * **Sitasi Formal**: Henning, S., & Hasselbring, W. (2022). A configurable method for benchmarking scalability of cloud-native applications. *Empirical Software Engineering*, 27(6), Article 143. https://doi.org/10.1007/s10664-022-10162-1

---

### **Saran Penerapan pada Paragraf Bab 3 (Metodologi Penelitian)**

> *"Penelitian ini mengadopsi pendekatan data sintetis yang dibangkitkan melalui fitur Seeder dan Factory berbasis pustaka Faker. Langkah ini sejalan dengan metodologi **Henning dan Hasselbring (2022)** serta **Yusmita et al. (2025)**, yang menyatakan bahwa pembuatan dataset terprogram dengan sifat statistik yang terkontrol memungkinkan terciptanya benchmark yang deterministik, presisi, dan teruji secara bit-for-bit reproducible. Skalasi data dibagi ke dalam tiga tingkat populasi—Small (50 tasks, 5 members), Medium (200 tasks, 15 members), dan Large (500 tasks, 30 members)—untuk menguji sensitivitas kompleksitas kueri ORM. Seperti yang dibuktikan oleh **Turcotte et al. (2022)**, pengujian bertingkat ini berfungsi memverifikasi pergeseran sifat kueri dari pola linier \\(O(N)\\) akibat antipola N+1 query menjadi angka konstan \\(O(1)\\) melalui teknik batch retrieval dan selective eager loading, sekaligus membuktikan bahwa rasio perbaikan performa akan semakin signifikan seiring bertambahnya volume data."*