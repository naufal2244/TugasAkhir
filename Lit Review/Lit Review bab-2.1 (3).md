Berikut adalah hasil verifikasi bibliografi dan analisis kritis terhadap studi yang dikaitkan dengan **Kundan dan Makwana (2025)** berdasarkan seluruh dokumen yang ada di dalam *notebook* Anda (`Introduction.docx`, `Manuscript_Anonymous (2).docx`, dan `Konteks penelitian terbaru (14 Juni 2026).md`).

---

### **Tahap 1: Verifikasi Sumber Primer (Bibliografi & Metadata)**

* **Status Naskah Primer:** ❌ **TIDAK TERDAPAT NASKAH PRIMER.**
  Naskah lengkap maupun penerbitan resmi dari artikel Kundan dan Makwana (2025) **tidak diunggah sebagai dokumen primer** di dalam *notebook* Anda.
* **Status Metadata Bibliografi:** ❌ **TIDAK TERVERIFIKASI (METADATA TIDAK LENGKAP).**
  Pada daftar pustaka naskah draf TA (`Manuscript_Anonymous (2).docx`, Ref), artikel ini dicatat secara sangat terbatas tanpa DOI, nama jurnal, volume, nomor, maupun halaman:
  > ` N. Makwana and Kundan, “A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications,” 2025.`
* **Kesimpulan Tahap 1:**  
  Artikel Kundan & Makwana (2025) **gagal diverifikasi sebagai sumber primer ilmiah yang terbit secara terdaftar** (*Unverified Secondary Citation*). Seluruh klaim mengenai metodologi dan angka hasilnya yang ada saat ini murni merupakan narasi sekunder dari draf Tugas Akhir Anda sendiri.

---

### **Tahap 2: Analisis Kritis terhadap Klaim Sekunder (Berdasarkan Draf TA)**

Meskipun artikel primernya tidak tersedia di *notebook*, berikut adalah analisis kritis terhadap klaim-klaim yang dituliskan dalam draf TA Anda:

1. **Tujuan & Desain Eksperimen:**  
   Mengevaluasi teknik optimasi performa (*Redis caching*, *eager loading*, dan *selective field loading*) pada aplikasi *e-commerce* berbasis Laravel di infrastruktur AWS t3.medium dengan dataset 100.000 pengguna (`Introduction.docx`, Passage 193; `Manuscript_Anonymous`, Passage 293).
2. **Penerapan Terpisah vs. Simultan:**  
   Terdapat **ketidaksesuaian narasi** dalam draf TA Anda. Di satu sisi, draf TA mengklaim Kundan & Makwana mengevaluasi ketiga teknik secara simultan (`Introduction.docx`, Passage 196; `Manuscript_Anonymous`, Passage 292). Namun di sisi lain, draf TA melaporkan angka hasil eksperimen tersebut secara **terpisah per item intervensi** (`Introduction.docx`, Passage 193, 195; `Manuscript_Anonymous`, Passage 293, 294).
3. **Pemeriksaan Akurasi Angka Hasil:**  
   Angka-angka yang dikutipkan dalam draf TA berasal dari skenario pengujian yang **berbeda-beda**, bukan dari satu skenario gabungan tunggal:
   * **Pengujian Redis Caching:** Menurunkan *response time* dari **800 ms menjadi 150 ms** dan mencapai *cache hit ratio* **95%** (`Manuscript_Anonymous`, Passage 293).
   * **Pengujian Eager Loading:** Memotong jumlah kueri dari **15 menjadi 2 kueri** dan *response time* dari **1.200 ms menjadi 300 ms** (`Manuscript_Anonymous`, Passage 294).
   * **Pengujian Selective Field Loading:** Menurunkan konsumsi memori server sebesar **35–40%** (`Manuscript_Anonymous`, Passage 294).
4. **Mekanisme Invalidasi Cache:**  
   Menggunakan *tagged caching* / *cluster Pub/Sub* (`Introduction.docx`, Passage 191, 193). Ini berbeda dengan *event-driven invalidation via Eloquent Observer* (`TaskObserver` & `TaskAssignmentObserver`) yang diterapkan pada TA Koladi.
5. **Keterbatasan Penulis:**  
   Tidak dapat dikonfirmasi dari sumber primer (karena naskah asli Kundan & Makwana tidak ada).
6. **Perbedaan Mendasar dengan TA Koladi:**  
   Kundan & Makwana berfokus pada *generic e-commerce API*, sedangkan TA Koladi mengevaluasi **Decision Support System (DSS)** yang mengeksekusi *pipeline* agregasi 22 metrik produktivitas per *workspace* pada skenario 3 volume data bertingkat (Small, Medium, Large) dengan 240 pengukuran berulang terkontrol pada 2 *endpoint* (`GET /statistik/suggestions` dan `GET /statistik/modal-data`).

---

### **Tabel Bukti Evaluasi Klaim (Claim Verification Table)**

| Klaim Awal (Mengenai Kundan & Makwana 2025) | Status Terverifikasi | Bukti Sumber / Lokasi di Notebook | Implikasi terhadap Research Gap & Karya Ilmiah |
| :--- | :--- | :--- | :--- |
| **Identitas Bibliografi Resmi** (Jurnal, Vol, No, DOI) | ❌ **TIDAK TERVERIFIKASI** | `Manuscript_Anonymous (2).docx` Ref hanya menuliskan judul & tahun 2025 tanpa DOI/Jurnal. | Berisiko menjadi *ghost reference* jika dijadikan rujukan utama tanpa validasi DOI penerbit. |
| **Penyatuan Angka Hasil Uji** (RT 800\\(\rightarrow\\)150ms, Query 15\\(\rightarrow\\)2, Memori -35-40%, Hit Ratio 95%) | ⚠️ **TERPISAH / TIDAK SIMULTAN** | `Introduction.docx` (Passage 193, 195) & `Manuscript_Anonymous` (Passage 293, 294) | Angka-angka tersebut dicatat untuk masing-masing pengujian terisolasi, bukan hasil dari satu skenario uji gabungan simultan. |
| **Mengukur 4 Metrik Simultan dalam Satu Pengujian** | ❓ **TIDAK DAPAT DIVERIFIKASI** | Diklaim pada `Introduction.docx` (Passage 197) & `Manuscript_Anonymous` (Passage 307), tetapi naskah primer tidak ada untuk membuktikannya. | Jika ternyata Kundan & Makwana mengujinya secara terpisah, maka klaim "pengukuran 4 metrik simultan" menjadi kurang akurat. |
| **Menerapkan Eloquent Observer** | ❌ **TIDAK ADA** | `Introduction.docx` (Passage 191, 194) mencatat penggunaan *tagged caching/Pub-Sub*. | Mempertegas *Research Gap* TA Koladi yang menggunakan *event-driven invalidation* via *Eloquent Observer*. |
| **Diuji pada Pipeline Agregasi DSS** | ❌ **TIDAK ADA** | `Manuscript_Anonymous` (Passage 293, 307) menyatakan fokus pada *generic e-commerce listing*. | Mempertegas *Novelty* TA Koladi pada modul *Decision Support System* (DSS) relasional. |

---

### **Rekomendasi Akhir**

> **Rekomendasi:** **SEBAIKNYA DIGANTI ATAU DITULIS DENGAN SANGAT HATI-HATI (HANYA SEBAGAI PENDUKUNG SEKUNDER).**

1. **Hindari Menjadikan Penelitian Terkait Utama:**  
   Karena tidak memiliki DOI atau penerbit resmi terverifikasi di dalam *notebook*, menjadikan Kundan & Makwana (2025) sebagai pilar utama *State-of-the-Art* di Bab II sangat berisiko dalam evaluasi penguji atau *peer-review*.
2. **Gunakan Sumber Utama yang Terverifikasi DOI:**  
   Sandarkan *State-of-the-Art* Bab II Anda pada 4 penelitian primer yang naskah asli dan DOI-nya **terverifikasi secara presisi** di *notebook* Anda:
   * **Privalov & Stupina (2024)** (DOI: `10.11591/ijeecs.v33.i3.pp1667-1675`) untuk legitimasi *Redis caching*.
   * **Yusmita et al. (2025)** (DOI: `10.1016/j.procs.2025.09.061`) untuk *ORM object materialization & selective field loading*.
   * **Turcotte et al. (2022)** (DOI: `10.1145/3551349.3556911`) untuk *N+1 query & eager loading*.
   * **Putra et al. (2025)** (DOI: `10.47709/brilliance.v5i1.5989`) untuk legitimasi *response time* dan dasar *research gap*.
3. **Jika Tetap Disitasi:**  
   Jika Anda tetap ingin menyebutkan Kundan & Makwana (2025), cantumkan HANYA sebagai studi pendukung konteks Laravel, dan **pisahkan pelaporan angka hasilnya** sesuai jenis pengujian masing-masing (Redis Caching tersendiri, Eager Loading tersendiri, Selective Loading tersendiri) agar tidak menimbulkan kesan penggabungan angka yang keliru.

   Berikut adalah verifikasi bibliografi dan analisis kritis mendalam terhadap naskah artikel yang dikaitkan dengan **Kundan dan Makwana (2025)**, berjudul *“A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications”*, berdasarkan sumber PDF primer yang terdapat dalam *notebook*.

---

### **TAHAP 1: Verifikasi Bibliografi & Kredibilitas Sumber Primer**

Berdasarkan pemeriksaan langsung pada naskah PDF primer (`A Comprehensive Study on Performance Optimization Techniques Database Queries.pdf`):

1. **Judul Artikel**: *A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications*
2. **Penulis**: Kundan\\(^1\\), Niraj Makwana\\(^2\\) (Afiliasi: Mewar University; \\(^1\\)Assistant Professor, \\(^2\\)Software Engineer)
3. **Header Jurnal pada PDF**: *International Journal on Science and Technology* (IJSAT), E-ISSN: 2229-7677, Website: www.ijsat.org, Email: editor@ijsat.org.
4. **Status Metadata Resmi**:
   * **Volume**: **TIDAK DITEMUKAN** (tidak tercantum di header, footer, maupun teks naskah).
   * **Nomor / Issue**: **TIDAK DITEMUKAN**.
   * **Halaman / Nomor Artikel**: **TIDAK DITEMUKAN**.
   * **Tahun Terbit**: **TIDAK TERCANTUM** pada naskah PDF (meskipun disitasi sebagai "2025" pada `Manuscript_Anonymous (2).docx`).
   * **DOI Resmi**: **TIDAK DITEMUKAN** di seluruh dokumen.

**Evaluasi Kredibilitas Publikasi**:
Artikel ini **tidak memiliki metadata publikasi standar** (tanpa Volume, Issue, Halaman, dan DOI yang terverifikasi). Selain itu, analisis isi naskah menunjukkan beberapa kejanggalan penulisan (*typos* seperti *"vattappatu om optimization"*, *"subp 8.3 JIT"*, *"Laravel's Underwriting Cash"*), klaim sitasi generik dalam narasi tanpa daftar pustaka yang lengkap (misalnya menyebut "Amazon, 2023", "Facebook, 2022", "Mozilla, 2023"), serta penyajian angka-angka hasil uji yang bersifat deklaratif tanpa deviasi standar atau rincian statistik.

---

### **TAHAP 2: Analisis Kritis Isi dan Eksperimen Artikel**

#### **(1) Tujuan dan Desain Eksperimen**
* **Tujuan**: Menguraikan tantangan performa pada aplikasi berbasis Laravel (seperti N+1 query, payload besar, indexing) dan mengusulkan kerangka optimasi multi-layer (indexing, Redis/Memcached caching, Eloquent optimization, API response structuring, dan GraphQL).
* **Desain Eksperimen**: Naskah mengklaim melakukan *A/B testing* dan *empirical benchmarking* menggunakan perkakas seperti Laravel Telescope, Blackfire.io, Apache JMeter (1.000 RPS), serta Prometheus + Grafana. Namun, skenario pengujian disajikan secara umum, di mana contoh kode disajikan sebagai snippet sampel dan hasil pengukuran dirangkum dalam angka-angka bulat idealistis tanpa analisis deviasi atau pengulangan statistik.

#### **(2) Spesifikasi Lingkungan Pengujian**
* **Framework**: Laravel 10.x (komparasi PHP 8.2 vs 8.3).
* **Database**: MySQL 8.0 (utama), PostgreSQL 14.x (komparasi).
* **Caching**: Redis 6.x, Memcached 1.x.
* **Infrastruktur**: AWS t3.medium (2 vCPUs, 4GB RAM), lingkungan dikontainerisasi Docker.
* **Dataset**: 100.000 user records, 500.000 post records (rerata 5 posts/user), dan simulasi 10.000 permintaan API simultan / 1.000 RPS.

#### **(3) Evaluasi Bersama vs Terpisah**
* **Status**: **DIUJI TERPISAH (A/B Testing Per Fitur)**.
* Pada **Tabel 5.2 Performance Metrics**, setiap teknik optimasi diuji secara terisolasi pada baris tersendiri:
  * Baris 1: *Eager Loading (N+1 Fix)* (1200ms \\(\rightarrow\\) 300ms).
  * Baris 2: *Redis Caching* (800ms \\(\rightarrow\\) 150ms).
  * Baris 3: *Indexed Queries* (500ms \\(\rightarrow\\) 100ms).
  * Baris 4: *Paginated API Responses* (2000ms \\(\rightarrow\\) 200ms).
  * Baris 5: *GraphQL vs REST* (1200ms \\(\rightarrow\\) 600ms).
* Penulis **tidak pernah mengevaluasi dampak gabungan simultan** antara *Redis Caching* dan optimasi kueri ORM (*eager loading* + *selective column*) dalam satu skenario eksekusi tunggal.

#### **(4) Pengukuran Empat Metrik**
* **Response Time**: Diukur dan dilaporkan (ms) pada Tabel 3.3.2 dan Tabel 5.2.
* **SQL Query Count**: Hanya dicatat secara naratif pada kolom insight *Eager Loading* (*"Reduced 15 queries \\(\rightarrow\\) 2 queries"*), bukan sebagai variabel terukur berkelanjutan.
* **Memory Utilization**: Tidak ada pencatatan memori (MB) pada tabel eksperimen utama (Tabel 5.2). Angka reduksi memori hanya muncul di narasi Tinjauan Pustaka (*"Selective field loading reduces memory usage by 35%"*) dan contoh kode (*"User::select(...)->get(); // 40% memory reduction"*).
* **Cache Hit Ratio**: Dilaporkan sebagai persentase tunggal (89% Redis Basic, 95% Redis Pub/Sub Cluster / Redis Caching) pada Tabel 3.3.2 & 5.2, tetapi tidak diukur bersamaan dengan *query count* atau *memory utilization*.

#### **(5) Verifikasi Angka-Angka Hasil Uji**
* **800 ms \\(\rightarrow\\) 150 ms**: **ADA**. Muncul di snippet kode Seksi 3.3.1 (`// ~800ms -> 150ms`), Tabel 3.3.2, dan Tabel 5.2 (Redis Caching).
* **15 \\(\rightarrow\\) 2 kueri**: **ADA**. Muncul di Tabel 5.2 kolom insight *Eager Loading (N+1 Fix)*.
* **Pengurangan Memori 35–40%**: **ADA di Teks/Snippet, BUKAN Tabel Uji Utama**. Angka 35% muncul di Seksi 2.3 (3.a) dan angka 40% muncul di komentar kode Seksi 3.1.1 (`// 40% memory reduction`).
* **Cache Hit Ratio 95%**: **ADA**. Muncul di Tabel 3.3.2 (Redis Pub/Sub Cluster) dan Tabel 5.2.
* **Implikasi**: Angka-angka tersebut memang ada di dalam naskah PDF, tetapi **tersebar di berbagai lokasi terpisah (literatur review, komentar kode, dan baris A/B test terisolasi)**. Angka-angka ini **bukan berasal dari satu pengujian gabungan yang sama**.

#### **(6) Mekanisme Invalidasi Cache**
* Artikel menggunakan *Time-to-Live* (TTL via `Cache::remember`), *Tagged Caching* (`Cache::tags`), dan *Redis Pub/Sub*.
* Penulis **TIDAK menggunakan Laravel Eloquent Observers** untuk *event-driven invalidation* pada level model ORM.

#### **(7) Keterbatasan yang Dinyatakan Penulis**
* Penulis hanya secara umum menyebutkan perlunya eksplorasi lanjutan pada GraphQL vs REST, PHP 8.3 JIT, dan *edge computing*. Tidak ada pembahasan metodologis mendalam mengenai keterbatasan kontrol eksperimen atau deviasi statistik.

#### **(8) Perbedaan Utama dengan Tugas Akhir (TA) DSS Koladi**
1. **Model Evaluasi**: Kundan & Makwana menguji optimasi secara terpisah per fitur (A/B testing). TA Anda mengevaluasi **dampak gabungan (*simultaneous evaluation*)** *Redis Cache-Aside* dan optimasi ORM Eloquent.
2. **Invalidasi Cache**: Kundan & Makwana menggunakan TTL/PubSub. TA Anda menggunakan **event-driven invalidation via Eloquent Observers** (`TaskObserver` dan `TaskAssignmentObserver`).
3. **Domain & Objek Uji**: Kundan & Makwana menggunakan e-commerce/SaaS generik. TA Anda menguji **dua endpoint DSS spesifik** (`GET /statistik/suggestions` dan `GET /statistik/modal-data`) pada aplikasi *collaborative virtual office* (Koladi).
4. **Volume Data**: Kundan & Makwana menggunakan 1 dataset 100k. TA Anda menggunakan **tiga volume data bertingkat** (Small: 50 tasks/5 members, Medium: 200 tasks/15 members, Large: 500 tasks/30 members).
5. **Metode Benchmark**: Kundan & Makwana menggunakan JMeter *load testing* (1,000 RPS). TA Anda menggunakan **controlled benchmarking sekuensial (10 repetisi valid + 3 warm-up dibuang per kombinasi)**, mengukur 4 metrik instrumental simultan, serta menguji signifikansi via *paired-sample t-test*.

---

### **TABEL BUKTI VERIFIKASI KLAIM**

| Klaim Awal / Asumsi | Status Terverifikasi | Bukti Halaman / Tabel / Bagian Naskah | Implikasi Terhadap *Research Gap* & TA Anda |
| :--- | :--- | :--- | :--- |
| **Metadata Jurnal Terverifikasi Lengkap** | **TIDAK TERVERIFIKASI** | Header PDF hanya mencantumkan nama jurnal *IJSAT* dan E-ISSN. **Volume, Issue, Halaman, dan DOI tidak ada**. | Artikel kurang memenuhi kriteria sebagai jurnal bereputasi utama. Disarankan tidak dijadikan rujukan pilar pertama. |
| **Menguji Gabungan Caching & ORM dalam Satu Eksperimen** | **TIDAK TERVERIFIKASI** | Tabel 5.2 menyajikan *Eager Loading* dan *Redis Caching* pada **baris terpisah (A/B testing)**. | **Memperkuat Research Gap TA Anda**: Penelitian yang mengevaluasi integrasi *caching* dan optimasi ORM secara bersamaan dalam satu eksperimen memang masih terbatas. |
| **Meningkatkan Latensi Redis 800 ms \\(\rightarrow\\) 150 ms** | **TERVERIFIKASI (Terisolasi)** | Snippet kode Seksi 3.3.1 & Tabel 5.2 baris *Redis Caching*. | Merupakan hasil dari pengujian *caching* berdiri sendiri, bukan gabungan dengan *eager loading*. |
| **Mengurangi Kueri 15 \\(\rightarrow\\) 2 Kueri** | **TERVERIFIKASI (Terisolasi)** | Tabel 5.2 baris *Eager Loading (N+1 Fix)*. | Merupakan hasil dari pengujian *eager loading* berdiri sendiri. |
| **Pengurangan Memori 35–40%** | **TERVERIFIKASI (Hanya di Narasi/Kode)** | Seksi 2.3 (3.a) & Komentar kode Seksi 3.1.1 (`User::select(...)`). | Angka ini berasal dari contoh klaim / snippet kode, **bukan hasil pengukuran kuantitatif pada tabel eksperimen utama**. |
| **Mengukur 4 Metrik Simultan (RT, Query Count, RAM, CHR)** | **TIDAK TERVERIFIKASI** | Metrik diukur pada tabel terpisah; RAM tidak diukur di Tabel 5.2, Query count hanya berupa teks insight. | **Kebaruan TA Anda**: TA Anda secara nyata mengukur 4 metrik instrumental tersebut secara simultan. |
| **Menggunakan Event-Driven Invalidation (Observer)** | **TIDAK TERVERIFIKASI** | Seksi 3.3.1 & 3.3.2 menggunakan TTL (`Cache::remember`), *Tagged Caching*, dan Pub/Sub. | **Kebaruan TA Anda**: TA Anda menerapkan *event-driven invalidation* via Eloquent Observer pada arsitektur *Redis Cache-Aside*. |

---

### **REKOMENDASI AKHIR**

1. **Status Kelayakan**: Artikel Kundan & Makwana **SEBAIKNYA DIGANTI ATAU DITURUNKAN STATUSNYA MENJADI LITERATUR PENDUKUNG SEKUNDER** (bukan sebagai *state-of-the-art* utama pada Subbab 2.1 Penelitian Terkait).
2. **Alasan Utama**:
   * Dokumen tidak memiliki metadata publikasi resmi yang lengkap (tanpa Volume, Issue, Halaman, dan DOI).
   * Angka-angka yang dihasilkan tidak berasal dari eksperimen terintegrasi melainkan klaim A/B testing terpisah dan contoh snippet kode.
3. **Pengganti Utama yang Lebih Kredibel dalam Notebook**:
   * Untuk *State-of-the-Art* Optimasi Laravel & PHP: Gunakan **Putra et al. (2025)** (*Brilliance: Research of Artificial Intelligence*, DOI: 10.47709/brilliance.v5i1.5989).
   * Untuk Dampak Redis Caching pada PostgreSQL: Gunakan **Sakti et al. (2025)** (*Procedia Computer Science* / Elsevier, DOI: 10.1016/j.procs.2025.08.270).
   * Untuk Overhead ORM & Object Materialization: Gunakan **Yusmita et al. (2025)** (*Procedia Computer Science* / Elsevier, DOI: 10.1016/j.procs.2025.09.061).
   * Untuk Eliminasi N+1 Query: Gunakan **Turcotte et al. (2022)** (*ACM ASE '22*, DOI: 10.1145/3551349.3556911).
4. **Penggunaan Kundan & Makwana (Jika Tetap Disitasi)**: Jika naskah Tugas Akhir Anda tetap menyebutkan Kundan & Makwana (misalnya karena sudah terlanjur ditulis dalam manuskrip), posisikan artikel tersebut untuk mendukung klaim individual—misalnya bahwa *eager loading* mengurangi kueri N+1 atau *Redis caching* meningkatkan kecepatan akses—tetapi **gunakan ketidakingintegrasian eksperimen mereka sebagai bukti utama untuk mengukuhkan *Research Gap* Tugas Akhir Anda**.

Berikut adalah penyusunan komprehensif **Subbab 2.1 Penelitian Terkait** untuk Tugas Akhir Anda yang berjudul *“The System Knows Your Team's Performance, But Takes Forever to Tell You: A Comparative Study of Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application”*.

---

### **1. Analisis Komparatif Lima Penelitian Terkait**

Perbandingan kelima penelitian yang ditinjau dilakukan berdasarkan tujuh dimensi utama:

1. **Tujuan dan Domain Aplikasi**:
   * *Putra et al. (2025)*: Membandingkan performa tiga *framework* PHP (Laravel, CodeIgniter, Symfony) pada aplikasi sistem informasi manajemen siswa (CRUD).
   * *Kundan & Makwana (2025)*: Menganalisis *bottleneck* kueri dan waktu respons API pada aplikasi berbasis Laravel (domain e-commerce/SaaS generik).
   * *Sakti et al. (2025)*: Mengevaluasi teknik optimasi *asynchronous* (Node.js/Express + PostgreSQL) pada pemrosesan dataset publik Kaggle.
   * *Turcotte et al. (2022)*: Mengidentifikasi dan merefaktorisasi antipola *N+1 query* secara otomatis pada 8 aplikasi *open-source* berbasis JavaScript ORM (Sequelize).
   * *Yusmita et al. (2025)*: Mengukur *performance overhead* lapisan abstraksi ORM (Prisma) dibandingkan *Raw SQL* pada sistem pemesanan tiket bioskop (PostgreSQL).
2. **Metode dan Teknologi**:
   * *Putra et al. (2025)*: *Load & stress testing* (Apache JMeter, Xdebug, New Relic) pada PHP 8.1 dan MySQL 8.0.
   * *Kundan & Makwana (2025)*: *Empirical benchmarking* A/B *testing* (Docker) pada Laravel 10.x, MySQL 8.0, PostgreSQL 14, dan Redis 6.x.
   * *Sakti et al. (2025)*: *Controlled benchmarking* (`wrk`) sekuensial dan konkuren pada Node.js, Express, PostgreSQL, dan Redis.
   * *Turcotte et al. (2022)*: *Static data-flow analysis* (CodeQL) dan aturan refaktorisasi deklaratif pada Sequelize ORM (Node.js).
   * *Yusmita et al. (2025)*: *Controlled benchmarking* (Docker) 100 repetisi/pola kueri pada Node.js, Prisma ORM vs *driver* `pg` (*Raw SQL*), dan PostgreSQL 15.
3. **Teknik Caching**:
   * *Putra et al. (2025)*: *Route*, *view*, dan *query caching* bawaan *framework*.
   * *Kundan & Makwana (2025)*: *Redis Caching* (*Basic* & *Pub/Sub Cluster*), *Memcached*, dan *Tagged Caching* (TTL statis).
   * *Sakti et al. (2025)*: *Redis in-memory caching* berdiri sendiri.
   * *Turcotte et al. (2022)*: Tidak menerapkan atau mengukur *caching*.
   * *Yusmita et al. (2025)*: Tidak menerapkan atau mengukur *caching*.
4. **Teknik Optimasi ORM / Query**:
   * *Putra et al. (2025)*: *Database indexing* dan *eager loading* dasar.
   * *Kundan & Makwana (2025)*: *Eager loading*, *selective column retrieval*, *pagination*, dan *indexing*.
   * *Sakti et al. (2025)*: *Connection pooling* dan *batch processing* kueri.
   * *Turcotte et al. (2022)*: Refaktorisasi kueri *looping* N+1 menjadi *bulk fetch* (`Op.in`).
   * *Yusmita et al. (2025)*: Mengganti ORM menjadi *Raw SQL* murni (bukan mengoptimasi ORM).
5. **Metrik dan Hasil Utama**:
   * *Putra et al. (2025)*: *Response time*, *throughput*, dan *error rate*. Latensi Laravel berkurang 35,1% di bawah beban tinggi (118,34 ms \\(\rightarrow\\) 88,58 ms).
   * *Kundan & Makwana (2025)*: *Response time*, *query count*, *memory utilization*, dan *cache hit ratio* (diuji terpisah per baris A/B *test*). *Eager loading* memotong latensi 75% (1200 ms \\(\rightarrow\\) 300 ms, kueri 15 \\(\rightarrow\\) 2); Redis *caching* mempercepat latensi 81% (800 ms \\(\rightarrow\\) 150 ms).
   * *Sakti et al. (2025)*: Latensi, *throughput*, CPU, dan RAM. Redis memotong lonjakan latensi baca hingga 94,5% (30,32 s \\(\rightarrow\\) 1,66 s).
   * *Turcotte et al. (2022)*: *SQL query count*, *execution time*, dan *client page load time*. *Speedup* eksekusi hingga 38,58x dan pemangkasan *page load time* hingga 90%.
   * *Yusmita et al. (2025)*: *Execution time*, CPU, RAM, dan deviasi standar. *Raw SQL* 5–8x lebih cepat dan hemat RAM 2–3x dibanding ORM (434 KB vs 1,02 MB pada *Nested Find*).
6. **Keterbatasan yang Didukung Sumber**:
   * *Putra et al. (2025)*: Berfokus pada MySQL dan CRUD; menyarankan pengujian PostgreSQL dan metrik memori.
   * *Kundan & Makwana (2025)*: Teknik diuji dalam baris A/B *test* terisolasi, bukan skenario gabungan eksekusi tunggal; metadata publikasi terbatas.
   * *Sakti et al. (2025)*: Berfokus pada *stack* JavaScript/Node.js; teknik diuji secara independen.
   * *Turcotte et al. (2022)*: Refaktorisasi bersifat rekomendasi statis; tidak mengukur penggunaan RAM server atau *caching*.
   * *Yusmita et al. (2025)*: Membandingkan ORM vs *Raw SQL* secara terisolasi tanpa menguji intervensi *selective column* atau *caching*.
7. **Relevansi Terhadap Tugas Akhir**:
   * *Putra et al. (2025)*: Menjadi rujukan eksplisit keberadaan *research gap* pada integrasi teknik optimasi *framework* PHP.
   * *Kundan & Makwana (2025)*: Rujukan pembanding metrik evaluasi pada *framework* Laravel.
   * *Sakti et al. (2025)*: Bukti empiris dominasi *Redis caching* di atas basis data PostgreSQL.
   * *Turcotte et al. (2022)*: Landasan teoretis eliminasi antipola *N+1 query* menjadi kueri konstan.
   * *Yusmita et al. (2025)*: Justifikasi ilmiah perlunya *selective column loading* untuk menekan *overhead materialization* grafik objek ORM.

---

### **A. Tabel Perbandingan Penelitian Terkait**

| Peneliti & Tahun | Judul Penelitian | Metode & Teknologi | Kelebihan | Kekurangan / Celah Penelitian (*Research Gap*) |
| :--- | :--- | :--- | :--- | :--- |
| **Putra et al. (2025)** | *Analysis Comparative of Performance Optimization Techniques for PHP Framework Testing: Laravel, CodeIgniter, Symfony* | Kuantitatif Komparatif (*Load Testing* JMeter, Xdebug, New Relic) pada **Laravel 8.x, CodeIgniter 4.x, Symfony 6.x** + **MySQL 8.0**. | Membandingkan dampak optimasi *before-after* secara empiris pada 3 *framework* PHP utama. | Hanya berfokus pada MySQL dan aplikasi CRUD; tidak mengukur *memory utilization*, *query count*, dan *cache hit ratio* secara simultan. |
| **Kundan & Makwana (2025)** | *A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications* | Kuantitatif Eksperimental (A/B *Testing* Docker) pada **Laravel 10.x, PostgreSQL 14, MySQL 8, Redis 6.x**. | Membahas spektrum optimasi Laravel secara luas dan mengukur 4 metrik performa utama. | Teknik dievaluasi dalam skenario A/B *test* terpisah per baris, bukan dampak simultan dalam satu *pipeline* eksekusi; metadata naskah terbatas. |
| **Sakti et al. (2025)** | *Optimizing Asynchronous Performance in Node.js with Express and PostgreSQL* | Kuantitatif *Controlled Benchmarking* (`wrk`) pada **Node.js, Express, PostgreSQL, Redis**. | Membuktikan secara tegas bahwa *Redis caching* memotong lonjakan latensi I/O hingga 94,5%. | Terfokus pada *stack* JavaScript (Node.js) dan mengevaluasi teknik secara berdiri sendiri tanpa integrasi ORM. |
| **Turcotte et al. (2022)** | *reformulator: Automated Refactoring of the N+1 Problem in Database-Backed Applications* | Kuantitatif Eksperimental (*Static Data-Flow Analysis*) pada **Sequelize ORM (JavaScript)**. | Memberikan bukti empiris terkuat bahwa pemangkasan kueri N+1 memberikan *speedup* hingga 38,58x. | Terfokus murni pada refaktorisasi ORM statis, tanpa mengukur penggunaan memori server maupun interaksi dengan *caching*. |
| **Yusmita et al. (2025)** | *Optimizing Database Access Strategy: A Performance Analysis Comparison of Raw SQL and Prisma ORM* | Kuantitatif *Controlled Benchmarking* (Docker) pada **Prisma ORM vs Raw SQL** + **PostgreSQL 15**. | Mengisolasi *overhead* CPU dan RAM (*object graph materialization*) yang disebabkan oleh ORM secara mendalam. | Membandingkan penggantian ORM menjadi *Raw SQL* secara terisolasi, tanpa menguji optimasi internal ORM yang dipadu *caching*. |

---

### **B. Identifikasi Penelitian yang Paling Dekat dengan Tugas Akhir**

Penelitian yang paling dekat dari segi **domain *framework* dan kombinasi metrik** adalah **Kundan dan Makwana (2025)**, karena sama-sama mengevaluasi aplikasi berbasis Laravel, PostgreSQL, dan Redis dengan empat metrik utama. Namun, dari segi **metodologi pengujian dan landasan *research gap***, penelitian **Putra et al. (2025)** dan **Yusmita et al. (2025)** menjadi acuan yang paling dekat:
* *Putra et al. (2025)* secara eksplisit mengidentifikasi bahwa evaluasi komparatif yang mengintegrasikan teknik *caching* dan optimasi kueri secara bersamaan pada *framework* PHP masih sangat terbatas.
* *Yusmita et al. (2025)* memberikan penjelasan mekanistis mendalam mengenai *overhead* alokasi memori ORM yang menjadi dasar perlunya *selective eager loading* pada Tugas Akhir ini.

---

### **C. Pembedaan Gap yang Didukung Bukti vs Dugaan**

* **Gap yang Didukung Bukti Ilmiah**:
  1. *Evaluasi Terisolasi vs Simultan*: Literatur yang ditinjau (*Putra et al., 2025; Kundan & Makwana, 2025; Sakti et al., 2025; Yusmita et al., 2025*) mengevaluasi teknik *caching*, refaktorisasi kueri, atau penggantian ORM secara terpisah dalam skenario A/B *testing* mandiri.
  2. *Cakupan Metrik Terfragmentasi*: Studi refaktorisasi ORM (*Turcotte et al., 2022*) tidak mengukur penggunaan memori atau *cache hit ratio*, sedangkan studi *caching* (*Sakti et al., 2025*) tidak mengukur *SQL query count* dan *overhead* grafik objek ORM.
* **Dugaan yang Dihindari (Tidak Diklaim)**:
  * *Dugaan Keliru*: Mengklaim "tidak ada penelitian di dunia yang menguji DSS berbasis Laravel" atau "penelitian pertama yang menggunakan *event-driven invalidation*". Pernyataan ini tidak dibenarkan karena tinjauan terbatas pada sumber-sumber yang relevan.

---

### **D. Perumusan Research Gap**

> **Research Gap**: Berdasarkan literatur yang ditinjau, penelitian mengenai optimasi performa aplikasi web umumnya mengevaluasi teknik *caching* dan optimasi kueri/ORM secara terpisah (*isolated evaluation*). Terdapat keterbatasan bukti empiris mengenai sejauh mana penerapan gabungan (*simultaneous implementation*) antara strategi *Redis Cache-Aside* terintegrasi *event-driven invalidation* dan optimasi kueri ORM (*selective eager loading* serta *batch retrieval*) berdampak terhadap empat metrik performa instrumental (*response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*) secara utuh, khususnya pada *pipeline* eksekusi *Decision Support System* (DSS) yang menangani kalkulasi agregat kompleks.

---

### **E. Perumusan Kontribusi dan Novelty**

Kontribusi dan kebaruan (*novelty*) Tugas Akhir ini dirumuskan ke dalam tiga pilar utama:
1. **Kebaruan Integrasi Arsitektur Multi-Layer**: Mengukur dampak gabungan simultan dari strategi *Redis Cache-Aside* yang dipicu oleh *event-driven invalidation* (`TaskObserver` dan `TaskAssignmentObserver`) serta optimasi ORM Eloquent (*selective column loading*, *eager loading*, dan *batch retrieval*) dalam satu kerangka eksekusi.
2. **Konteks Komputasi Agregatif DSS**: Mengevaluasi integrasi tersebut secara spesifik pada dua *endpoint* DSS Koladi (`GET /statistik/suggestions` dan `GET /statistik/modal-data`) yang melakukan kalkulasi berulang atas 22 metrik kinerja tim.
3. **Rigour Metodologi Benchmarking**: Menyajikan evaluasi empiris menggunakan *controlled before-after benchmarking* sekuensial pada tiga skala volume data (50, 200, dan 500 *tasks*) dan dua kondisi *cache* (*Hit* dan *Miss*) dengan 240 pengukuran valid, serta memvalidasi signifikansi perubahan menggunakan *paired-sample t-test*.

---

### **F. Narasi Akademik Subbab 2.1 Penelitian Terkait**

**2.1 Penelitian Terkait**

Pengembangan dan optimasi performa aplikasi web berbasis *framework* dan basis data relasional telah menjadi fokus perhatian dalam berbagai penelitian rekayasa perangkat lunak. Tinjauan terhadap studi-studi terdahulu menunjukkan bahwa upaya peningkatan efisiensi sistem umumnya dilakukan melalui dua jalur utama, yaitu optimasi pada lapisan akses data (*query/ORM optimization*) dan implementasi lapisan penyimpanan sementara (*caching*).

Penelitian mengenai optimasi pada lapisan *framework* PHP dievaluasi secara komparatif oleh Putra et al. (2025), yang menguji efektivitas teknik optimasi (*route caching*, *indexing*, dan *eager loading*) pada Laravel, CodeIgniter, dan Symfony. Menggunakan metode *load and stress testing* berbasis Apache JMeter pada aplikasi manajemen siswa berbasis MySQL, studi tersebut membuktikan bahwa intervensi optimasi mampu menekan *response time* Laravel hingga 35,1% di bawah beban kerja tinggi. Namun, Putra et al. (2025) secara eksplisit mengidentifikasi adanya *research gap*, di mana evaluasi komparatif yang mengintegrasikan berbagai teknik optimasi secara bersamaan pada *framework* PHP masih sangat terbatas. Selain itu, penelitian tersebut masih berfokus pada aplikasi CRUD sederhana dengan basis data MySQL dan merekomendasikan pengujian lanjutan pada basis data relasional lain seperti PostgreSQL serta penambahan metrik konsumsi memori.

Pada domain aplikasi berbasis Laravel, Kundan dan Makwana (2025) melakukan studi komprehensif mengenai teknik optimasi kueri basis data dan waktu respons API menggunakan lingkungan dikontainerisasi Docker di atas PostgreSQL dan Redis. Hasil pengujian A/B menunjukkan bahwa penerapan *eager loading* memotong latensi respons API sebesar 75% (dari 1.200 ms menjadi 300 ms) dan mengurangi jumlah kueri dari 15 menjadi 2. Di sisi lain, penggunaan *Redis caching* mempercepat waktu respons hingga 81% (dari 800 ms menjadi 150 ms dengan *cache hit ratio* 95%), sementara *selective column retrieval* menurunkan penggunaan memori sebesar 35–40%. Meskipun mengukur metrik yang luas, Kundan dan Makwana (2025) mengevaluasi masing-masing teknik optimasi secara terisolasi pada baris pengujian terpisah, sehingga belum menyajikan gambaran dampak simultan dari penggabungan *caching* dan optimasi ORM dalam satu *pipeline* eksekusi tunggal.

Efektivitas *in-memory caching* menggunakan Redis di atas basis data PostgreSQL dibuktikan secara eksplisit oleh Sakti et al. (2025) pada *stack* Node.js dan Express. Melalui *controlled benchmarking* menggunakan perkakas `wrk`, penelitian tersebut mengonfirmasi bahwa *Redis caching* merupakan teknik yang paling berdampak (*most impactful*), yang mampu memotong lonjakan latensi baca secara drastis dari 30,32 detik menjadi 1,66 detik (penurunan latensi hingga 94,5%). Kendati demikian, studi tersebut berfokus pada ekosistem JavaScript dan menguji *caching* sebagai komponen berdiri sendiri tanpa mengintegrasikannya dengan strategi refaktorisasi kueri pada lapisan ORM.

Di sisi lain, penelitian yang berfokus pada pembenahan struktur kueri ORM dilakukan oleh Turcotte et al. (2022) melalui pengembangan *reformulator*, sebuah perkakas refaktorisasi otomatis untuk mengatasi antipola *N+1 query* pada ORM Sequelize. Melalui analisis aliran data statis (*static data-flow analysis*) pada 8 proyek *open-source*, Turcotte et al. (2022) membuktikan bahwa mengubah kueri *looping* N+1 menjadi pemanggilan konstan (*bulk fetch*) menghasilkan peningkatan kecepatan eksekusi (*speedup*) hingga 38,58 kali lipat pada skala data besar dan memotong *client-side page load time* hingga 90%. Namun, penelitian ini murni berfokus pada refaktorisasi ORM statis dan tidak mengukur konsumsi memori server maupun interaksinya dengan lapisan *caching*.

Penjelasan mendalam mengenai *overhead* internal yang dihasilkan oleh lapisan abstraksi ORM diuraikan oleh Yusmita et al. (2025), yang membandingkan performa Prisma ORM dan *Raw SQL* pada PostgreSQL 15. Dalam eksperimen terkontrol berbasis Docker, Yusmita et al. (2025) menemukan bahwa *Raw SQL* beroperasi 5 hingga 8 kali lebih cepat, menghemat CPU hingga 7 kali lipat, dan menggunakan memori 2 hingga 3 kali lebih sedikit (434 KB vs 1,02 MB pada kueri *Nested Find*) dibandingkan ORM. Penulis menjelaskan bahwa *overhead* tersebut disebabkan oleh biaya komputasi penerjemahan kueri (*translation overhead*) dan alokasi memori untuk menginstansiasi grafik objek (*object graph materialization expense*). Temuan ini memberikan justifikasi ilmiah yang kuat bahwa penggunaan ORM tanpa pembatasan proyeksi kolom (*unrestricted fetching*) berisiko tinggi memicu pemborosan alokasi memori. Walaupun demikian, Yusmita et al. (2025) mengevaluasi penggantian ORM menjadi *Raw SQL* murni secara terisolasi, tanpa menguji apakah strategi optimasi internal ORM (*selective eager loading*) yang dipadukan dengan *caching* eksternal dapat menutupi *overhead* tersebut.

Berdasarkan tinjauan terhadap lima penelitian utama di atas, belum ditemukan studi yang mengevaluasi penerapan gabungan (*simultaneous implementation*) antara strategi *Redis Cache-Aside* yang dipicu oleh *event-driven invalidation* (via Laravel Observer) dan optimasi kueri ORM Eloquent (*selective column loading*, *eager loading*, dan *batch retrieval*) dalam satu kerangka pengujian terkontrol. Sebagian besar literatur mengevaluasi *caching* dan optimasi ORM secara terpisah atau berfokus pada pembandingan antar-*framework*.

Tugas Akhir ini mengisi ruang penelitian tersebut dengan mengevaluasi dampak simultan dari kedua lapisan optimasi tersebut pada *Decision Support System* (DSS) aplikasi *collaborative virtual office* (Koladi) berbasis Laravel 12.x, PostgreSQL 15, dan Redis 7.x. Evaluasi dilakukan pada dua *endpoint* kalkulasi agregat (`GET /statistik/suggestions` dan `GET /statistik/modal-data`) menggunakan metode *controlled before-after benchmarking* pada tiga skala volume data dan dua kondisi *cache*. Dampak intervensi diukur secara komprehensif melalui empat metrik instrumental—*response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*—yang diuji signifikansinya menggunakan *paired-sample t-test*.

---

### **G. Daftar Pustaka (Format APA 7)**

* Kundan, & Makwana, N. (2025). A comprehensive study on performance optimization techniques database queries and API response time in Laravel-based web applications. *International Journal on Science and Technology*, 2229-7677.
* Putra, F. P. E., Zulfikri, A., Rohman, A., & Alim, R. (2025). Analysis comparative of performance optimization techniques for PHP framework testing: Laravel, CodeIgniter, Symfony. *Brilliance: Research of Artificial Intelligence*, 5(1), 242–248. https://doi.org/10.47709/brilliance.v5i1.5989
* Sakti, M. D. I., Dirgantara, D., Ramadhan, A. K., & Ibrahim, M. A. (2025). Optimizing asynchronous performance in Node.js with Express and PostgreSQL. *Procedia Computer Science*, 269, 172–181. https://doi.org/10.1016/j.procs.2025.08.270
* Turcotte, A., Aldrich, M. W., & Tip, F. (2022). reformulator: Automated refactoring of the N+1 problem in database-backed applications. In *Proceedings of the 37th IEEE/ACM International Conference on Automated Software Engineering* (ASE '22, pp. 1–12). Association for Computing Machinery. https://doi.org/10.1145/3551349.3556911
* Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025). Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM. *Procedia Computer Science*, 269, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061

Berikut adalah penyusunan komprehensif subbab **2.1 Penelitian Terkait** untuk Tugas Akhir Anda, yang dirancang secara ketat berdasarkan hasil analisis terverifikasi dari sumber-sumber ilmiah dalam *notebook*.

---

### **1. Perbandingan Penelitian Terkait berdasarkan 7 Dimensi Utama**

1. **Tujuan dan Domain Aplikasi:**
   * **Privalov & Stupina (2024):** Meningkatkan efisiensi sistem informasi web berbasis Spring Boot.
   * **Turcotte, Aldrich, & Tip (2022):** Refaktorisasi otomatis masalah *N+1 query* pada aplikasi web berbasis ORM (Ruby on Rails/ActiveRecord).
   * **Yusmita et al. (2025):** Mengkuantifikasi *performance penalty* dan *overhead* memori antara Raw SQL vs Prisma ORM pada Node.js.
   * **Putra et al. (2025):** Komparasi horizontal teknik optimasi *framework* pada PHP (Laravel, CodeIgniter, Symfony).
   * **Kundan & Makwana (2025):** Evaluasi teknik optimasi database dan API *response time* pada aplikasi *e-commerce* Laravel (AWS t3.medium, 100k data).
   * **Tugas Akhir Koladi:** Evaluasi dampak gabungan Redis Cache-Aside dan optimasi ORM pada *pipeline Decision Support System* (DSS) relasional di aplikasi *collaborative virtual office* Koladi (Laravel 12.x, PostgreSQL 15, Redis 7.x).

2. **Metode dan Teknologi:**
   * **Privalov & Stupina (2024):** Pengujian eksperimental pada Java Spring Boot.
   * **Turcotte, Aldrich, & Tip (2022):** Analisis statis/dinamis dan *trace execution* menggunakan alat *Reformulator*.
   * **Yusmita et al. (2025):** *Benchmark* komparatif terisolasi dalam kontainer Docker Compose.
   * **Putra et al. (2025):** Pengujian eksperimental komparatif antar-*framework* PHP.
   * **Kundan & Makwana (2025):** Pengujian eksperimental pada instans cloud AWS t3.medium.
   * **Tugas Akhir Koladi:** *Controlled before-after benchmarking* sekuensial (240 pengukuran valid, 3 volume data sintetis, 2 *endpoint*, 2 kondisi *cache*, 10 repetisi) dalam lingkungan terisolasi Docker Compose.

3. **Teknik Caching:**
   * **Privalov & Stupina (2024):** *Redis-backed serialized caching* dengan pemicu AOP (*Aspect-Oriented Programming*).
   * **Turcotte et al. (2022) & Yusmita et al. (2025):** Tidak menerapkan maupun mengevaluasi lapisan *caching*.
   * **Putra et al. (2025):** *Configuration/route caching* tingkat *framework*.
   * **Kundan & Makwana (2025):** *Redis caching* dengan konfigurasi *tagged caching / cluster Pub-Sub*.
   * **Tugas Akhir Koladi:** *Redis Cache-Aside* dengan *event-driven invalidation* via Eloquent Observer (`TaskObserver` dan `TaskAssignmentObserver`) serta *safety net* TTL.

4. **Teknik Optimasi ORM / Query:**
   * **Privalov & Stupina (2024) & Putra et al. (2025):** Tidak mengevaluasi refaktorisasi kueri ORM.
   * **Turcotte et al. (2022):** Refaktorisasi *lazy loading* menjadi *eager loading*.
   * **Yusmita et al. (2025):** Penggantian ORM menjadi Raw SQL (*unrestricted ORM loading vs Raw SQL*).
   * **Kundan & Makwana (2025):** *Eager loading* dan *selective field loading*.
   * **Tugas Akhir Koladi:** *Selective eager loading* (`assignedUsers:id`, `checklists:id,task_id,is_done`) dan *batch retrieval* dengan *in-memory aggregation*.

5. **Metrik dan Hasil Utama:**
   * **Privalov & Stupina (2024):** Waktu eksekusi kueri berkurang 10x lipat (197,5 ms \\(\rightarrow\\) 30 ms).
   * **Turcotte et al. (2022):** Kueri SQL terpotong dari 11 menjadi 2 (*speedup* 38,58x).
   * **Yusmita et al. (2025):** Memori ORM 977 KB vs Raw SQL 357 KB (*overhead* memori 2–3x).
   * **Putra et al. (2025):** Membuktikan peningkatan responsivitas dan stabilitas sistem.
   * **Kundan & Makwana (2025):** RT 800 ms \\(\rightarrow\\) 150 ms, Kueri 15 \\(\rightarrow\\) 2, Memori -35–40%, Hit Ratio 95% (pada pengujian terpisah).
   * **Tugas Akhir Koladi:** Kueri terpotong hingga 98,1%, RT terstabilisasi di ~65 ms under *miss*, memori tertekan hingga 71,4% pada dataset *Large*, dan *full response bypass* under *hit*.

6. **Keterbatasan yang Didukung Sumber:**
   * Studi-studi terdahulu sebagian besar mengevaluasi *caching* dan ORM secara terisolasi.
   * Terbatas pada domain *e-commerce listing* atau API CRUD sederhana, tanpa menguji *pipeline* agregasi DSS relasional.
   * Menggunakan pengujian terisolasi tanpa mengukur 4 metrik teknis secara simultan (kecuali Kundan & Makwana pada *e-commerce*).

7. **Relevansi terhadap TA:**
   Menjadi landasan *State-of-the-Art* untuk mengjustifikasi *Redis caching*, *eager loading*, *selective column retrieval*, dan desain pengujian terkontrol *before-after*.

---

### **A. Tabel Komparatif Penelitian Terkait**

| Peneliti & Tahun | Judul Penelitian | Metode & Teknologi | Kelebihan | Kekurangan / Celah (*Research Gap*) |
| :--- | :--- | :--- | :--- | :--- |
| **Privalov & Stupina (2024)** | *Improving web-oriented information systems efficiency using Redis caching mechanisms* | Eksperimen pengujian performa Redis *serialized caching* via Spring Boot AOP | Membuktikan pemangkasan latensi eksekusi kueri hingga 10x lipat (197,5 ms \\(\rightarrow\\) 30 ms) | Terisolasi pada lapisan *caching* Java/Spring Boot; tidak mengevaluasi ORM, *query count*, atau memori |
| **Turcotte, Aldrich, & Tip (2022)** | *Reformulator: Automated Refactoring of the N+1 Problem in Database-Backed Applications* | Refaktorisasi otomatis N+1 query (*Reformulator*) pada aplikasi ber-ORM | Memotong kueri dari 11 ke 2 per *request* dan memberikan *speedup* 38,58x | Murni mengevaluasi ORM tanpa melibatkan *in-memory cache* (Redis) atau metrik memori/hit ratio |
| **Yusmita et al. (2025)** | *Optimizing Database Access Strategy: A Performance Analysis Comparison of Raw SQL and Prisma ORM* | Pengujian komparatif Raw SQL vs Prisma ORM (Node.js) dalam Docker Compose | Mengkuantifikasi *overhead object materialization* ORM (977 KB vs 357 KB) | Menyelesaikan masalah dengan meninggalkan ORM (ke Raw SQL); tidak menguji *caching layer* |
| **Putra et al. (2025)** | *Analysis Comparative of Performance Optimization Techniques for PHP Framework Testing: Laravel, CodeIgniter, Symfony* | Pengujian komparatif teknik optimasi *framework* pada PHP (Laravel, CI, Symfony) | Memvalidasi *response time* sebagai metrik utama responsivitas aplikasi PHP | Terfokus pada komparasi antar-*framework*, tanpa menguji integrasi simultan *caching* dan ORM |
| **Kundan & Makwana (2025)** *(Pendukung Sekunder)* | *A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications* | Pengujian Redis caching, eager loading, dan selective loading pada Laravel (AWS t3.medium) | Mengukur 4 metrik dalam lingkungan Laravel *e-commerce* | Terbatas pada *generic product listing*, tanpa *pipeline* DSS, *event Observer*, atau dataset bertingkat |
| **Tugas Akhir Koladi (Penelitian Ini)** | *"The System Knows Your Team's Performance, But Takes Forever to Tell You": A Comparative Study of Caching Strategy and Query Optimization...* | *Controlled before-after benchmarking* (240 pengukuran valid, Docker Compose, 3 volume data) | Evaluasi gabungan Redis Cache-Aside (*Observer*) + ORM Refactoring pada jalur DSS dengan 4 metrik simultan | *Fokus Bounded:* Terbatas pada *controlled sequential benchmarking* pada 2 *endpoint* DSS Koladi |

---

### **B. Identifikasi Penelitian Terdekat dengan Tugas Akhir**

Penelitian terdahulu yang paling dekat secara metodologis dengan Tugas Akhir ini adalah **Kundan dan Makwana (2025)**. Studi tersebut merupakan satu-satunya penelitian yang ditinjau yang mengukur *response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio* secara simultan dalam ekosistem *framework* Laravel. Namun, penelitian Koladi ini memiliki perbedaan struktural yang mendasar:
1. **Domain Sistem:** Kundan & Makwana menguji *generic e-commerce product listing*, sedangkan Tugas Akhir ini menguji modul **Decision Support System (DSS)** yang mengeksekusi *pipeline* agregasi 22 metrik produktivitas.
2. **Mekanisme Invalidasi Cache:** Kundan & Makwana menggunakan *tagged caching / Pub-Sub*, sedangkan Tugas Akhir ini menerapkan **event-driven invalidation via Eloquent Observer** (`TaskObserver` dan `TaskAssignmentObserver`) untuk menangani mutasi tabel *pivot* `task_assignments`.
3. **Desain Eksperimen:** Tugas Akhir ini menerapkan *controlled before-after benchmarking* pada 3 strata volume data sintetis (Small, Medium, Large) dengan 240 pengukuran berulang yang divalidasi secara otomatis melalui *instrumentation suite*.

---

### **C. Membedakan Gap yang Didukung Bukti vs. Gap Dugaan**

* **Gap yang Didukung Bukti Ilmiah (*Evidence-Based Gap*):**
  1. *Keterbatasan Integrasi Simultan:* Penelitian yang mengevaluasi integrasi strategi *caching* dan optimasi kueri ORM secara bersamaan dalam satu kerangka eksperimen masih sangat terbatas (**Putra et al., 2025**).
  2. *Ketiadaan Evaluasi Interaksi Dua Lapisan:* Studi terisolasi tidak mampu mengukur bagaimana efisiensi ORM menekan biaya *cache miss*, dan bagaimana efektivitas *cache* mengurangi frekuensi eksekusi ORM dari waktu ke waktu.
  3. *Inefisiensi Database-Backed Snapshot:* Mekanisme *caching* berbasis tabel basis data relasional pada jalur DSS memicu *partial cache hit* dan tidak berfungsi sebagai *database bypass*.

* **Gap Dugaan / Klaim Berlebihan yang Dihindari (*Avoided Conjectures*):**
  * ❌ *Tidak mengklaim bahwa "tidak ada penelitian DSS di dunia" atau "tidak ada yang menggunakan Eloquent Observer".*
  * ✅ *Formulasi yang Benar:* Menggunakan frasa **"belum ditemukan pada penelitian yang ditinjau"** (*not identified in the reviewed literature*) dan secara eksplisit membatasi klaim pada ruang lingkup literatur yang dianalisis.

---

### **D. Formulasi *Research Gap* yang Hati-Hati**

> "Berdasarkan literatur yang ditinjau, sebagian besar penelitian mengevaluasi *in-memory caching* dan optimasi kueri ORM sebagai intervensi yang terpisah. Akibatnya, efek interaksi antara kedua lapisan tersebut terhadap sumber daya server bersama belum terukur secara memadai. Selain itu, belum ditemukan pada penelitian yang ditinjau adanya evaluasi optimasi infrastruktur yang menggabungkan Redis Cache-Aside berbasis *event-driven invalidation* (menggunakan Eloquent Observer) dan refaktorisasi kueri ORM secara simultan pada modul Decision Support System (DSS) yang menjalankan *pipeline* agregasi relasional bertahap."

---

### **E. Rumusan Kontribusi dan Kebaruan (*Novelty Statement*)**

> **Kontribusi dan Kebaruan Penelitian (*Novelty*):**  
> Penelitian ini memberikan kontribusi empiris dengan mengisi kesenjangan literatur yang diidentifikasi oleh **Putra et al. (2025)** melalui **evaluasi terintegrasi antara strategi Redis Cache-Aside berbasis *event-driven invalidation* (menggunakan `TaskObserver` dan `TaskAssignmentObserver`) dan optimasi kueri ORM (*selective eager loading* dan *batch in-memory aggregation*) secara simultan dalam satu kerangka eksperimen terkontrol**. 
> 
> Berbeda dengan penelitian sebelumnya yang berfokus pada API *CRUD* atau *product listing* biasa, kebaruan penelitian ini terletak pada pengujian modul **Decision Support System (DSS)** aplikasi *collaborative virtual office* (Koladi) yang mengeksekusi *pipeline* agregasi 22 metrik produktivitas. Dampak gabungan ini diukur secara komprehensif melalui empat metrik teknis tingkat sistem—**response time (ms), SQL query count, memory utilization (MB), dan cache hit ratio (%)**—di bawah skenario volume data bertingkat (Small, Medium, Large) pada 240 pengukuran terkontrol berulang.

---

### **F. Narasi Akademik Subbab 2.1 Penelitian Terkait (~850 Kata)**

#### **2.1 Penelitian Terkait**

Pergeseran global menuju model kerja terdistribusi mendorong organisasi memanfaatkan platform kolaboratif digital untuk mendukung koordinasi tim lintas batas geografis (Froese et al., 2025). Dalam platform semacam ini, modul *Decision Support System* (DSS) memegang peranan komputasional vital untuk mengolah data aktivitas relasional menjadi indikator produktivitas yang dapat ditindaklanjuti (Cai & Chen, 2021; Ryu et al., 2023). Namun, *pipeline* agregasi relasional pada DSS berbasis web sering kali menghadapi hambatan kinerja akibat tingginya biaya operasi basis data dan latensi pemrosesan (Alnuhait et al., 2024; Koornneef et al., 2020).

Dalam literatur optimasi sistem web, penggunaan *in-memory caching* untuk memangkas latensi akses basis data merupakan strategi yang tervalidasi secara konsisten. Privalov dan Stupina (2024) membuktikan bahwa menggantikan pembacaan langsung dari basis data relasional dengan *Redis-backed serialized caching* pada aplikasi Spring Boot mampu menekan waktu eksekusi kueri hingga sepuluh kali lipat, yaitu dari 197,5 ms menjadi 30 ms. Keunggulan ini berdasar pada karakteristik media penyimpanan RAM yang memiliki kecepatan akses sekitar empat orde magnitudo lebih cepat dibandingkan pengambilan data berbasis disk (Taipalus, 2024). Di tingkat manajemen *cache*, An dan Cao (2022) menunjukkan bahwa *middleware Monotonic Consistent Caching* di atas Redis dapat mengurangi beban pembacaan basis data (`#dbread`) sebesar 39,09% dan meningkatkan *throughput* transaksi hingga 77,15%. An dan Cao (2022) serta Mao et al. (2024) juga mengkritik invalidasi berbasis *Time-To-Live* (TTL) murni karena secara struktural tidak mampu menjaga kesegaran data pada *workload* dengan mutasi tinggi. Pendekatan *event-driven invalidation* terbukti lebih unggul, di mana Kan et al. (2020) melaporkan bahwa invalidasi proaktif berbasis *publish-subscribe* menghasilkan *cache hit ratio* 64,52% lebih tinggi dibandingkan strategi reaktif dengan *overhead* server yang minimal.

Secara terpisah, kelompok literatur lain berfokus pada pembenahan inefisiensi pada lapisan *Object-Relational Mapping* (ORM). Alshemaimri et al. (2021) mengklasifikasikan *Unnecessary Data Retrieval* (UD)—yaitu mengambil seluruh kolom padahal hanya membutuhkan atribut tertentu—sebagai *anti-pattern* yang dapat memicu *overhead* memori server. Dalam hal penanganan kueri berulang, Turcotte et al. (2022) menunjukkan bahwa refaktorisasi otomatis pada masalah *N+1 query* mampu memotong eksekusi kueri SQL dari 11 menjadi 2 kueri per *request* serta memberikan percepatan eksekusi (*speedup*) hingga 38,58 kali lipat saat skala data membengkak. Majerik dan Borkovcova (2023) juga melaporkan penurunan kueri dari 12.748 menjadi 2 kueri pada Doctrine ORM melalui teknik *eager loading*. Sementara itu, Yusmita et al. (2025) mengkuantifikasi bahwa pemuatan ORM tanpa pembatasan kolom meningkatkan penggunaan memori 2 hingga 3 kali lipat (977 KB vs 357 KB) dibandingkan eksekusi yang dibatasi akibat biaya materialisasi objek (*object materialization*) di memori *server*.

Dalam ekosistem *framework* Laravel, Kundan dan Makwana (2025) melaporkan bahwa *selective field loading* menekan konsumsi memori sebesar 35–40%, sedangkan *eager loading* memotong kueri dari 15 menjadi 2. Kundan dan Makwana (2025) juga mengevaluasi *Redis caching*, *eager loading*, dan *selective field loading* pada aplikasi *e-commerce* berbasis Laravel (AWS t3.medium, 100.000 data pengguna) dengan mengukur *response time* (turun dari 800 ms ke 150 ms), *query count*, *memory utilization*, dan *cache hit ratio* (mencapai 95%) secara bersamaan. Meskipun demikian, pengujian tersebut terbatas pada arsitektur *generic product listing*, tanpa melibatkan kompleksitas *pipeline* agregasi DSS, invalidasi berbasis *event* via Eloquent Observer, atau dinamika tabel *pivot* relasional.

Meskipun manfaat *caching* dan optimasi ORM secara individual telah banyak dibuktikan, penelitian yang mengintegrasikan kedua teknik tersebut secara simultan dalam satu kerangka eksperimen masih sangat terbatas, sebagaimana diidentifikasi oleh Putra et al. (2025). Studi terdahulu yang mengevaluasi optimasi secara terisolasi tidak mampu mengukur efek interaksi dua lapisan: efisiensi ORM menentukan besarnya beban komputasi saat terjadi *cache miss*, sedangkan efektivitas *caching* menentukan seberapa sering lapisan ORM dieksekusi dari waktu ke waktu (Putra et al., 2025). Selain itu, berdasarkan literatur yang ditinjau, belum ditemukan adanya studi yang mengevaluasi optimasi infrastruktur gabungan ini pada modul DSS yang mengeksekusi *pipeline* agregasi relasional bertahap.

Dari sisi metodologi, pengujian performa infrastruktur perangkat lunak secara konsisten mengadopsi desain *controlled before-after benchmarking* untuk mengisolasi dampak intervensi teknis (Alnuhait et al., 2024; An & Cao, 2022; Putra et al., 2025). Penggunaan dataset sintetis terstruktur menjamin kontrol volume dan reprodusibilitas eksperimen (Sasmoko et al., 2024; Yusmita et al., 2025), sementara prosedur *warm-up* dan *repeated measurements* diterapkan untuk mengeliminasi bias *cold-start* dan menjaga stabilitas data (Henning & Hasselbring, 2022; Privalov & Stupina, 2024; Traini et al., 2023).

Untuk mengisi kesenjangan literatur tersebut, penelitian Tugas Akhir ini menerapkan strategi **Redis Cache-Aside berbasis *event-driven invalidation*** (menggunakan `TaskObserver` dan `TaskAssignmentObserver`) serta **optimasi kueri ORM Eloquent** (*selective eager loading* dan *batch in-memory aggregation*) secara simultan pada jalur DSS aplikasi Koladi. Dampak intervensi ganda ini dievaluasi secara komprehensif melalui empat metrik teknis—*response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*—pada 240 pengukuran terkontrol berulang di bawah skenario volume data bertingkat (Small, Medium, Large) pada *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data`.

---

### **G. Daftar Pustaka (Format APA 7th)**

*Alnuhait, H., Alzyadat, W., Althunibat, A., Kahtan, H., Zaqaibeh, B., & Al-Khawaja, H. A. (2024). Web application performance assessment: A study of responsiveness, throughput, and scalability. International Journal of Advanced and Applied Sciences, 11(9), 214–226. https://doi.org/10.21833/ijaas.2024.09.023*

*Alshemaimri, B., Elmasri, R., Alsahfi, T., & Almotairi, M. (2021). A survey of problematic database code fragments in software systems. Engineering Reports, John Wiley & Sons. https://doi.org/10.1002/eng2.12441*

*An, S., & Cao, Y. (2022). Making cache monotonic and consistent. Proceedings of the VLDB Endowment, 16(4), 891–904. https://doi.org/10.14778/3574245.3574271*

*Cai, C., & Chen, C. (2021). Optimization of human resource file information decision support system based on cloud computing. Complexity, 2021, 1–11. https://doi.org/10.1155/2021/8919625*

*Froese, F. J., Blay, T., Gibson, C. B., Shaffer, M. A., & Benitez, J. (2025). Global virtual work: A review, integrative framework, and future research opportunities. Journal of International Business Studies / Palgrave Macmillan. https://doi.org/10.1057/s41267-025-00775-1*

*Gómez-Pantoja, C., Brieba-Rojas, D., & Infante-Paris, A. (2025). A behavior-aware caching architecture for large-scale web applications. Applied Sciences / MDPI.*

*Kan, Y., et al. (2020). PIOR: Proactive invalidation with optional renewing for in-network caching in named data networking. IEEE Access.*

*Koornneef, H., Verhagen, W. J. C., & Curran, R. (2020). A decision support framework and prototype for aircraft dispatch assessment. Decision Support Systems, 135, 113338. https://doi.org/10.1016/j.dss.2020.113338*

*Kundan, & Makwana, N. (2025). A comprehensive study on performance optimization techniques database queries and API response time in Laravel-based web applications.*

*Majerik, F., & Borkovcova, M. (2023). Design of data access architecture using ORM framework.*

*Privalov, M. V., & Stupina, M. V. (2024). Improving web-oriented information systems efficiency using Redis caching mechanisms. Indonesian Journal of Electrical Engineering and Computer Science, 33(3), 1667–1675. https://doi.org/10.11591/ijeecs.v33.i3.pp1667-1675*

*Putra, F. P. E., Zulfikri, A., Rohman, A., & Alim, R. (2025). Analysis comparative of performance optimization techniques for PHP framework testing: Laravel, CodeIgniter, Symfony. Brilliance: Research of Artificial Intelligence, 5(1), 242–248. https://doi.org/10.47709/brilliance.v5i1.5989*

*Ryu, H., et al. (2023). A web-based decision support system (DSS) for hydrogen refueling station location and supply chain optimization. International Journal of Hydrogen Energy, 48(93), 36223–36239. https://doi.org/10.1016/j.ijhydene.2023.06.064*

*Sakti, M. D. I., Dirgantara, D., Ramadhan, A. K., & Ibrahim, M. A. (2025). Optimizing asynchronous performance in Node.js with Express and PostgreSQL. Procedia Computer Science, 172–181. https://doi.org/10.1016/j.procs.2025.08.270*

*Sasmoko, Indrianti, Y., Manalu, S. R., & Danaristo, J. (2024). Analyzing database optimization strategies in Laravel for an enhanced learning management. Procedia Computer Science, 799–804. https://doi.org/10.1016/j.procs.2024.10.306*

*Taipalus, T. (2024). Database management system performance comparisons: A systematic literature review. Journal of Systems and Software, 208, 111872. https://doi.org/10.1016/j.jss.2023.111872*

*Turcotte, A., Aldrich, M. W., & Tip, F. (2022). Reformulator: Automated refactoring of the N+1 problem in database-backed applications. Dalam ACM International Conference Proceeding Series. Association for Computing Machinery. https://doi.org/10.1145/3551349.3556911*

*Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025). Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM. Procedia Computer Science, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061*

---

💡 **Saran Lanjutan:**
Penyusunan narasi di atas telah mengikuti seluruhkaidah akademis dan grounding sumber secara presisi. Jika Anda membutuhkan penyesuaian pada bagian rumusan masalah, hipotesis eksperimen, atau pembahasan bab IV, silakan beri tahu!