Berdasarkan dokumen sumber yang tersedia di dalam *notebook*, berikut adalah penjelasan mengenai metode penelitian, rancangan eksperimen, serta alur pengujian yang digunakan untuk mengevaluasi kinerja sistem:

---

### **1. Kerangka Kerja Riset Utama dan Rujukan Ahli**

*   **Nama Kerangka Kerja Utama**:  
    Penelitian ini mengadopsi kerangka kerja **_Experimental Software Engineering_**.
*   **Nama Ahli dan Sitasi Rujukan**:  
    Kerangka kerja ini merujuk langsung pada karya fundamental **Claes Wohlin, Per Runeson, Martin Höst, Magnus C. Ohlsson, Björn Regnell, dan Anders Wesslén (2000)** melalui buku mereka:  
    > *Wohlin, C., Runeson, P., Höst, M., Ohlsson, M. C., Regnell, B., & Wesslén, A. (2000). Experimentation in Software Engineering.*
*   **Paradigma Metodologis**:  
    Penelitian ini dirancang murni sebagai penelitian **kuantitatif eksperimental berbasis pengukuran sistem secara instrumental**. Sumber menegaskan bahwa penelitian ini tidak menggunakan data primer manusia (seperti survei, wawancara, kuesioner, evaluasi UX, atau observasi perilaku pengguna). Seluruh data dikumpulkan langsung melalui instrumen perangkat lunak untuk membuktikan hubungan kausalitas teknis antara intervensi arsitektur dan efisiensi eksekusi.

---

### **2. Rancangan Desain Eksperimen dan Kondisi Lingkungan**

*   **Model Desain Eksperimen**:  
    Menerapkan rancangan **_Controlled Before-and-After Comparative Experiment_** (studi eksperimental komparatif sebelum dan sesudah optimasi dalam kondisi terkontrol).
*   **Cara Pengujian dan Pembanding**:
    1.  **Komparasi Konfigurasi (*Before vs. After*)**:
        *   **_Baseline_ (Sebelum Optimasi)**: Menggunakan *database-backed snapshot cache* (tabel `workspace_performance_snapshots`), *unrestricted eager loading* (tanpa pembatasan kolom), tanpa *eager loading* pada relasi `checklists` (menimbulkan masalah *N+1 query*), dan kueri berulang di dalam *loop* (*query-in-loop*) pada fungsi `getWorkloadDistribution()`.
        *   **_Optimized_ (Sesudah Optimasi)**: Menggunakan *Redis Cache-Aside* (driver `phpredis`), *event-driven invalidation* via *Eloquent Observers* (`TaskObserver` dan `TaskAssignmentObserver`), *selective eager loading* (`assignedUsers:id`, `checklists:id,task_id,is_done`), serta *batch retrieval* dengan agregasi di memori.
    2.  **Kondisi Eksekusi Sekuensial Terisolasi**:  
        Pengujian dijalankan secara **sekuensial (*sequential/non-concurrent requests*)** menggunakan *script runner* berbasis Node.js. Penelitian ini secara tegas **bukan pengujian beban konkuren (*concurrent load testing*), *stress testing*, atau *throughput testing***. Tujuannya adalah mengisolasi eksekusi kueri agar terbebas dari *confounding noise* (seperti *network latency*, persaingan thread, atau *OS scheduling*) sehingga perbedaan performa dapat diatribusikan langsung pada intervensi kode dan arsitektur.
    3.  **Lingkungan Pengujian yang Terstandarisasi**:  
        Sistem dijalankan dalam lingkungan kontainerisasi **Docker Compose** pada jaringan internal terisolasi (`koladi_net`), mencakup Laravel 12.x, PostgreSQL 15, dan Redis 7.x. Standarisasi kontainer ini merujuk pada metodologi *reproducible benchmarking* yang diadvokasi oleh **Henning & Hasselbring (2022)** serta **Yusmita et al. (2025)**.
    4.  **Dataset Sintetis Terkontrol**:  
        Data pengujian dibangkitkan menggunakan fitur Seeder dan Factory Laravel berbasis pustaka Faker. Penggunaan data sintetis bertingkat (*Small*: 50 tugas/5 anggota, *Medium*: 200 tugas/15 anggota, *Large*: 500 tugas/30 anggota) dirancang untuk mengontrol densitas relasi secara identik antara kondisi *baseline* dan *optimized*, sejalan dengan justifikasi eksperimen terkontrol pada **Turcotte et al. (2022)** dan **Yusmita et al. (2025)**.

---

### **3. Alur Hubungan Kerangka Metodologi dengan Prosedur Teknis Pengujian**

Di dalam dokumen draf manuskrip (*Manuscript Anonymous*), seluruh prosedur eksperimen dihubungkan dalam alur sekuensial multi-fase (*Research Process Flow*) agar proses identifikasi masalah, intervensi, dan pengujian dapat dipisahkan secara analitis:

```
[Fase 1: Baseline Profiling]
            │
            ▼
[Fase 2: Implementation / Refactoring]
            │
            ▼
[Fase 3: Controlled Performance Measurement]
            │
            ▼
[Fase 4: Measurement & Data Collection]
            │
            ▼
[Fase 5: Statistical Evaluation]
```

Hubungan rinci antar-tahapannya dijelaskan sebagai berikut:

1.  **Fase 1: _Baseline Profiling_**  
    Pemeriksaan kode sumber dan *profiling* awal menggunakan Laravel Telescope untuk memetakan sumber inefisiensi arsitektural pada modul DSS Koladi (dua *endpoint*: `GET /statistik/suggestions` dan `GET /statistik/modal-data`). Ditemukan bahwa *database-backed caching* masih memicu kueri SQL, relasi `assignedUsers` mengalami *column over-fetching*, relasi `checklists` mengalami antipola *N+1 query*, dan kalkulasi beban kerja anggota tim mengalami inefisiensi *query-in-loop*.
2.  **Fase 2: _Implementation / Refactoring_**  
    Melakukan intervensi teknis pada kode aplikasi Koladi: mengintegrasikan *Redis Cache-Aside* dengan *event-driven invalidation* (Solusi A), serta merefaktorisasi pemanggilan ORM Eloquent melalui *selective eager loading* dan *batch retrieval* (Solusi B).
3.  **Fase 3: _Controlled Performance Measurement_ (Perlakuan Warm-up & Repetisi)**  
    Mengeksekusi *request* secara sekuensial terhadap kedua *endpoint* pada kedua konfigurasi (*Baseline* vs. *Optimized*) dan dua kondisi *cache* (*Miss* vs. *Hit*):
    *   **Perlakuan Warm-up**: Setiap kombinasi diawali dengan **3 *warm-up requests* yang dibuang (*discarded*)** dari analisis guna menghilangkan bias *cold-start*, pengisian memori awal, dan kompilasi JIT/OPcache. Prosedur ini mengacu pada prinsip kestabilan pengukuran dari **Henning & Hasselbring (2022)** serta **Traini et al. (2023)**.
    *   **10 Repetisi Valid**: Setelah fase *warm-up*, pengujian diulang sebanyak **10 kali pengulangan valid** untuk setiap sel kombinasi guna memastikan data berada pada kondisi stabil (*steady-state*), merujuk pada prosedur **Privalov & Stupina (2024)**.
    *   **Matriks Pengukuran**: Rancangan *full factorial* (3 ukuran dataset × 2 *endpoint* × 2 konfigurasi × 2 kondisi *cache* × 10 repetisi) menghasilkan total **240 observasi valid**.
4.  **Fase 4: _Measurement & Data Collection_ (Pencatatan 4 Metrik Dependen)**  
    Instrumen pengukuran (*middleware* kustom Laravel, Node.js runner, dan CSV validator) merekam empat metrik kinerja teknis secara simultan:
    *   *Response Time* (ms): diukur melalui waktu tempuh *request-response* pada *runner*.
    *   *SQL Query Count*: dicatat melalui pendeteksi kueri internal framework.
    *   *Memory Utilization* (MB): dicatat menggunakan fungsi PHP `memory_get_peak_usage()`.
    *   *Cache Hit Ratio* (%): dipantau melalui `Redis INFO STATS` dan log *snapshot*.
5.  **Fase 5: _Statistical Evaluation_ (Analisis Data Komparatif)**  
    Data yang terkumpul dianalisis secara statistik melalui dua pendekatan:
    *   **_Percentage Improvement_**: Menghitung besarnya perbaikan praktis; perubahan di bawah 3% diinterpretasikan secara hati-hati sebagai variasi acak (*measurement noise*) sesuai rujukan **Japke et al. (2025)**.
    *   **_Paired-Sample t-Test_ (\\(\alpha = 0{,}05\\))**: Menguji signifikansi statistik perbedaan rata-rata performa sebelum dan sesudah optimasi, merujuk pada metodologi komparasi berpasangan dari **Uddin & Lu (2024)**. Khusus data deterministik (*SQL Query Count* yang variansinya mendekati nol), hasil dilaporkan sebagai \\(p < 0{,}001\\) dengan penekanan analisis pada signifikansi praktis (penurunan absolut jumlah kueri). Metrik *Cache Hit Ratio* dianalisis secara bersamaan dengan *query count* untuk membuktikan apakah kueri PostgreSQL benar-benar dieliminasi secara penuh.

    Berdasarkan dokumen sumber yang tersedia di dalam *notebook* (khususnya *Manuscript_Anonymous (2).docx*, *Introduction.docx*, dan *Konteks penelitian terbaru (14 Juni 2026).md*), metode penelitian dan metode analisis yang digunakan diuraikan sebagai berikut:

---

### **1. Nama Kerangka Kerja / Metodologi Riset Utama dan Ahli yang Dirujuk**

* **Nama Kerangka Kerja Utama:**  
  Penelitian ini menggunakan kerangka kerja **Experimental Software Engineering**.
* **Nama Ahli / Sitasi Rujukan:**  
  **Wohlin et al. (2000)** melalui buku rujukan standar *“Experimentation in Software Engineering”*.
* **Definisi Operasional:**  
  Dalam kerangka tersebut, eksperimen didefinisikan sebagai *investigasi formal, ketat, dan terkontrol*, di mana hipotesis diuji secara empiris terhadap artefak perangkat lunak (dalam penelitian ini adalah modul *Decision Support System* / DSS aplikasi Koladi) dengan memanipulasi variabel independen secara sengaja dan mengukur variabel dependen secara presisi.

---

### **2. Rancangan atau Desain Eksperimen yang Diterapkan**

Rancangan eksperimen dirancang secara terkontrol untuk menetapkan hubungan kausal langsung antara intervensi optimasi dan perubahan performa:

* **Desain Eksperimen:**  
  Mengadopsi rancangan ***Controlled Before-and-After Comparative Experiment / Benchmarking***.
* **Cara Pengujian (*Isolation / Configuration Testing*):**  
  Pengujian dilakukan melalui pengiriman *request* HTTP sekuensial terkontrol (*non-concurrent*), bukan melalui *load testing*, *stress testing*, *concurrency testing*, atau simulasi *virtual users*. Pendekatan ini bertujuan murni untuk mengisolasi efek perubahan arsitektur internal sistem (*isolation/configuration testing*) terhadap sumber daya komputasi server tanpa dikacaukan oleh latensi jaringan eksternal atau perebutan *thread* paralel.
  * *Sitasi rujukan isolasi & konfigurasi:* **Alnuhait et al. (2024)**, **An & Cao (2022)**, dan **Gómez-Pantoja et al. (2025)**.
* **Objek Pembanding (*Baseline vs. Optimized*):**
  * **Konfigurasi Baseline:** Sistem sebelum optimasi yang menggunakan *database-backed snapshot* (`CACHE_STORE=database`) dan pemuatan kueri ORM tanpa pembatasan (*unrestricted loading* / *N+1 lazy loading*).
  * **Konfigurasi Optimized:** Sistem setelah penerapan Redis *Cache-Aside* (`CACHE_STORE=redis`) berbasis *event-driven invalidation* (via Eloquent Observer) dan optimasi kueri Eloquent ORM (*selective eager loading* dan *batch in-memory aggregation*).
* **Kondisi Lingkungan & Kontrol Eksperimen:**
  * **Lingkungan Kontainer Terisolasi:** Seluruh stack aplikasi (Nginx, PHP-FPM/Laravel 12.x, PostgreSQL 15, dan Redis 7.x) dikontainerisasi dalam Docker Compose pada jaringan internal terisolasi (`koladi_net`) untuk menjamin konsistensi konfigurasi dan reprodusibilitas lingkungan (*Sitasi: Yusmita et al., 2025; Privalov & Stupina, 2024*).
  * **Dataset Sintetis Bertingkat (*Synthetic Seeded Datasets*):** Menggunakan data sintetis terstruktur dari Laravel *Seeder* dan *Factory* untuk menjaga kontrol volume data, rasio status tugas (40% *done*, 35% *in progress*, 25% *todo*), dan kepadatan relasi anggota secara deterministik (*Sitasi: Sasmoko et al., 2024; Yusmita et al., 2025*). Skala data dibagi tiga tingkatan: *Small* (50 tugas, 5 anggota), *Medium* (200 tugas, 15 anggota), dan *Large* (500 tugas, 30 anggota).
  * **Kondisi Cache (2 Variasi):** *Cache Miss* (cache dikosongkan sebelum pengujian untuk mengukur biaya eksekusi penuh *pipeline* DSS) dan *Cache Hit* (cache dipopulasi terlebih dahulu melalui *request* awal untuk mengukur efisiensi *retrieval* memori) (*Sitasi: An & Cao, 2022; Privalov & Stupina, 2024*).
  * **Prosedur *Warm-up* dan Repetisi:** Setiap kombinasi pengujian diawali oleh **3 *warm-up requests*** yang nilainya **dibuang dari analisis** guna mengeliminasi bias *cold-start* (seperti *OPcache filling*, inisialisasi koneksi), kemudian diikuti oleh **10 repetisi pengukuran valid** (*Sitasi: Henning & Hasselbring, 2022; Traini et al., 2023; Privalov & Stupina, 2024; Japke et al., 2025*).
  * **Matriks Pengukuran Faktorial:** Menghasilkan total \\(3\text{ volume} \times 2\text{ endpoint} \times 2\text{ konfigurasi} \times 2\text{ kondisi cache} \times 10\text{ repetisi} = \mathbf{240}\text{ pengukuran valid}\\] (120 *baseline* dan 120 *optimized*).

---

### **3. Alur Hubungan Kerangka Metodologi dengan Prosedur Teknis Pengujian**

Kerangka *Experimental Software Engineering* diterjemahkan ke dalam prosedur teknis terstruktur melalui **alur pipa eksperimen lima fase (*five-phase sequential pipeline*)**:

```
[Fase 1: Baseline Profiling]
            ↓
[Fase 2: Implementation / Refactoring]
            ↓
[Fase 3: Controlled Performance Measurement]
            ↓
[Fase 4: Measurement and Data Collection]
            ↓
[Fase 5: Statistical Evaluation]
```

1. **Fase 1: *Baseline Profiling* (Diagnosis Masalah Awal)**  
   Sebelum intervensi dilakukan, peneliti menggunakan **Laravel Telescope** untuk memeriksa dan mengidentifikasi pola *bottleneck* kueri SQL pada modul DSS, membuktikan adanya kueri redundan ke tabel `workspace_performance_snapshots`, *partial cache hit* pada *modal-data*, konsumsi memori akibat *column over-fetching*, dan *lazy loading N+1* pada relasi `checklists`. Fase ini menjadi dasar empiris perancangan intervensi teknis.
2. **Fase 2: *Implementation* (Penerapan Intervensi Teknis)**  
   Menerapkan perbaikan kode pada *codebase* Koladi: mengganti *database snapshot* dengan pola Redis *Cache-Aside* yang dikawal invalidasi berbasis *event* via `TaskObserver` dan `TaskAssignmentObserver`, serta merestrukturisasi kueri ORM pada `getTasksInPeriod()` (*selective eager loading*) dan `getWorkloadDistribution()` (*batch retrieval* dan *in-memory aggregation*).
3. **Fase 3: *Controlled Performance Measurement* (Eksekusi Pengujian)**  
   Mengeksekusi *request* HTTP terkontrol secara otomatis dan sekuensial terhadap *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data` pada konfigurasi *baseline* dan *optimized* di dalam lingkungan kontainer Docker Compose.
4. **Fase 4: *Measurement and Data Collection* (Perekaman Data Instrumental)**  
   Data dikumpulkan secara otomatis menggunakan *benchmark instrumentation suite* khusus (bukan manual) yang terdiri dari:
   * *Middleware instrumentasi Laravel:* mencatat metrik internal (*SQL query count* via query listener dan puncak alokasi RAM proses PHP via `memory_get_peak_usage(true)`).
   * *Node.js runner script:* mengeksekusi *request* HTTP sekuensial dan mencatat *end-to-end response time* presisi tinggi.
   * *Redis metrics & Snapshot checker:* menghitung *cache hit ratio* melalui delta `keyspace_hits` dan `keyspace_misses` pada Redis.
   * *CSV Validator:* memverifikasi kelengkapan seluruh 240 baris data valid dan pemisahan data *warm-up* ke dalam format CSV.
5. **Fase 5: *Statistical Evaluation* (Analisis Data Statistik)**  
   Mengevaluasi hasil pengukuran melalui tiga pendekatan statistik:
   * **Statistik Deskriptif:** Menghitung *Mean*, *Minimum*, *Maksimum*, dan *Standar Deviasi* untuk menyajikan stabilitas sebaran pengukuran antar-repetisi (*Sitasi: Privalov & Stupina, 2024; Japke et al., 2025*).
   * **Persentase Perubahan (*Practical Improvement*):** Mengukur besarnya dampak optimasi praktis pada *response time*, *query count*, dan *memory utilization*.
   * **Uji Hipotesis Komparatif (*Paired-Sample t-Test*, \\(\alpha = 0{,}05\\)):** Menguji signifikansi perbedaan antara kondisi *baseline* dan *optimized* (*Sitasi: Uddin & Lu, 2024*). Uji-t berpasangan ini tepat secara metodologis karena setiap observasi *baseline* memiliki pasangan observasi langsung (*paired*) pada *optimized* dengan skala dataset, *endpoint*, kondisi *cache*, parameter *request*, dan urutan repetisi yang identik.

---

💡 *Seluruh alur dan rancangan metodologis ini memastikan bahwa data yang dihasilkan bersifat terverifikasi, deterministik, dan dapat dipertanggungjawabkan secara akademik.* Jika Anda ingin mengonversi ringkasan ini menjadi narasi proposal Bab 3, beri tahu saya!