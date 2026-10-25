Berikut adalah draf materi mendalam dan formal untuk **Subbab 2.2 Landasan Teori** proposal Tugas Akhir Anda, yang disusun secara spesifik, padat, dan terstruktur sesuai ketentuan:

---

### **2.2 Landasan Teori**

#### **2.2.1 Konsep Decision Support System (DSS) Berbasis Web**
Pergeseran arsitektur kerja menuju model terdistribusi (*global virtual work*) menuntut integrasi platform digital yang mampu mengordinasikan tim secara terstruktur. Dalam ekosistem ini, *Decision Support System* (DSS) memiliki peran komputasional yang berbeda dari sistem informasi biasa: DSS tidak sekadar menyajikan data statis, melainkan mengeksekusi *pipeline* agregasi bertahap (*staged aggregation pipeline*) terhadap data aktivitas tim relasional untuk menghasilkan indikator produktivitas yang dapat ditindaklanjuti. DSS berbasis web didefinisikan sebagai sistem terkomputerisasi yang membantu manajer mengambil keputusan semi-terstruktur melalui interaksi manusia-komputer dan pemodelan data (Cai & Chen, 2021). Format berbasis *browser* ini memungkinkan sistem diakses oleh multi-pengguna secara simultan dalam lingkungan terdistribusi berskala besar (Koornneef et al., 2020). 

Karena modul DSS harus mengeksekusi komputasi agregasi relasional pada setiap *request*, *response time* menjadi metrik teknis utama untuk menilai responsivitas sistem (Putra et al., 2025; Alnuhait et al., 2024). Durasi eksekusi *request* DSS dibentuk secara langsung oleh jumlah dan biaya operasi pemrosesan basis data (*database reads*) pada tingkatan infrastruktur. Kecepatan *retrieval* data sangat menentukan apakah indikator kinerja tim dapat disajikan secara *real-time* atau justru mengalami kelambatan akibat tingginya biaya transaksi basis data relasional (Taipalus, 2024; Alnuhait et al., 2024).

#### **2.2.2 Teori Caching & Arsitektur Redis**
*In-memory caching* merupakan teknik optimasi yang mengalihkan jalur pembacaan data (*read path*) dari media penyimpanan berbasis disk (RDBMS) ke memori utama (RAM). Secara teoretis, pengambilan data dari RAM memiliki latensi akses yang sekitar empat orde magnitudo lebih cepat dibandingkan *database retrieval* berbasis disk (Taipalus, 2024). Dalam aplikasi web, *Cache-Aside* (*look-aside caching*) merupakan pola interaksi di mana server aplikasi memeriksa ketersediaan data di dalam *cache store* terlebih dahulu. Jika terjadi *cache hit*, data dikembalikan langsung ke klien tanpa menyentuh basis data; sedangkan jika terjadi *cache miss*, aplikasi akan mengeksekusi kueri ke basis data, menyimpan hasilnya ke dalam *cache*, lalu mengembalikannya kepada pengguna (An & Cao, 2022).

Terkait kesegaran data (*data freshness*), kebijakan pembersihan *cache* berbasis *Time-To-Live* (TTL) murni memiliki keterbatasan struktural karena tidak mampu merespons penulisan/pembaruan data (*data writes*) secara *real-time*, sehingga berisiko menyajikan data usang (*stale data*) atau memicu *overhead polling* eksponensial (Mao et al., 2024; An & Cao, 2022). Untuk mengatasi keterbatasan TTL pada *workload* berintensitas mutasi tinggi, diterapkan pendekatan *Event-Driven Invalidation* melalui pemantau kejadian (*Eloquent Observers* pada Laravel atau *AOP/pub-sub triggers*) yang secara proaktif menginvalidasi entri *cache* saat terjadi pembuatan, pembaruan, atau penghapusan data (Kan et al., 2020; Privalov & Stupina, 2024). Strategi invalidasi berbasis *event* ini terbukti menghasilkan *cache hit ratio* yang jauh lebih tinggi dan menjaga konsistensi kuat (*strong consistency*) dengan beban server yang minimal dibanding strategi berbasis waktu murni (Kan et al., 2020).

#### **2.2.3 Teori ORM & Inefisiensi Kueri Eloquent**
Eloquent ORM mengadopsi pola *Active Record*, di mana setiap tabel basis data dipetakan menjadi objek kelas PHP. Meskipun meningkatkan produktivitas pengembang, abstraksi ORM rentan memicu inefisiensi kueri apabila tidak dikonfigurasi dengan tepat. Salah satu *anti-pattern* utama adalah *Unnecessary Data Retrieval* (UD) atau *column over-fetching*, yaitu kondisi ketika aplikasi mengambil seluruh kolom tabel (seperti `SELECT *`) padahal hanya membutuhkan atribut/kolom tertentu (Alshemaimri et al., 2021).

Selain itu, penggunaan *lazy loading* pada kueri relasional memicu antipola *N+1 query problem*, di mana ORM mengeksekusi 1 kueri utama untuk entitas induk dan \\(N\\) kueri tambahan terpisah untuk setiap \\(N\\) entitas relasi di dalam iterasi *loop*. Pola ini menyebabkan pertumbuhan jumlah kueri SQL membengkak secara linier (\\(O(N)\\)) seiring bertambahnya ukuran data, yang dapat dieliminasi melalui petunjuk *eager loading* untuk mereduksi eksekusi kueri berulang menjadi *batch query* tunggal (Turcotte et al., 2022). Di sisi penggunaan memori, ORM menanggung biaya instansiasi (*Object Graph Materialization Expense*), di mana *runtime* PHP harus mengalokasikan RAM untuk memetakan setiap kolom dan baris menjadi struktur objek kompleks. Pengambilan data ORM tanpa pembatasan kolom terbukti membebankan penggunaan memori 2 hingga 3 kali lipat (misalnya 977 KB vs 357 KB) dibandingkan kueri yang dibatasi kolomnya (*selective field loading*) akibat biaya materialisasi objek di memori server (Yusmita et al., 2025; Kundan & Makwana, 2025).

#### **2.2.4 Profil Objek Penelitian (Koladi)**
Koladi merupakan aplikasi SaaS *collaborative virtual office* multi-tenant yang dibangun menggunakan *framework* Laravel 12.x, PostgreSQL 15, Redis 7.x, Eloquent ORM, dan dikontainerisasi dalam jaringan terisolasi Docker Compose (`koladi_net`). Jalur DSS Koladi bertugas menghitung metrik produktivitas *workspace* yang diakses melalui dua *endpoint* pada halaman statistik kinerja:
1. `GET /statistik/suggestions`: merepresentasikan jalur pengambilan rekomendasi DSS via `SnapshotManager`.
2. `GET /statistik/modal-data`: merepresentasikan respons DSS lengkap yang mencakup metrik utama, daftar tugas mendesak (*urgent tasks*), dan distribusi beban kerja (*workload distribution*).

Alur kalkulasi DSS Koladi mengolah 22 metrik produktivitas mentah per *workspace* melalui perangkaian 5 komponen utama:
- `DSSService`: mengoordinasikan seluruh eksekusi jalur DSS.
- `SnapshotManager`: mengelola pemeriksaan, penciptaan, dan invalidasi *cache/snapshot* DSS.
- `MetricsCalculator`: mengeksekusi kalkulasi dasar metrik tugas relasional (`getTasksInPeriod()`), termasuk persentase penyelesaian dan rasio tugas *overdue*.
- `TrendAnalyzer`: menganalisis perbandingan tren produktivitas antar-periode.
- `RecommendationEngine`: menghasilkan saran tindak lanjut berbasis aturan (*rule-based suggestions*) bagi pengelola *workspace*.

---

💡 **Saran Nudge:**
Draf Landasan Teori di atas telah disesuaikan secara presisi dengan kebutuhan proposal Tugas Akhir Anda. Jika Anda ingin melanjutkannya dengan penyusunan draf kalimat akademik untuk **Subbab 2.3 (Metode Pengembangan Produk)** atau **Subbab 2.4 (Metode Analisis/Riset)**, silakan beri tahu!

Berikut adalah draf materi mendalam untuk **Subbab 2.2 Landasan Teori** proposal Tugas Akhir Anda, yang disusun secara terstruktur, formal, dan berfokus pada konsep teknis optimasi modul Decision Support System (DSS) Koladi:

---

# **2.2 LANDASAN TEORI**

### **2.2.1 Konsep Decision Support System (DSS) Berbasis Web**

*Decision Support System* (DSS) didefinisikan secara komputasional sebagai sistem informasi berbasis komputer yang dirancang untuk membantu manajer atau pengambil keputusan dalam menyelesaikan masalah semi-terstruktur melalui interaksi manusia-komputer dan pemodelan data. Pada paradigma aplikasi web modern, *web-based DSS* mengintegrasikan sumber data heterogen dan terdistribusi untuk diakses secara independen melalui peramban web (*browser*) tanpa memerlukan instalasi perangkat lunak tambahan di sisi klien. Pendekatan ini memungkinkan pemrosesan data berskala masif secara terpusat untuk melayani banyak pengguna secara bersamaan.

Dalam konteks platform kolaborasi tim, modul DSS menjalankan *staged aggregation pipeline* yang mengekstrak data transaksi aktivitas kerja relasional (seperti riwayat tugas, alokasi anggota, dan pencapaian target) menjadi indikator kuantitatif kinerja tim. Proses kalkulasi agregasi ini bersifat intensif karena mengolah puluhan metrik produktivitas secara berulang pada setiap permintaan pengguna. Mengingat pemrosesan kueri analitis pada lingkungan *decision support* (OLAP) membutuhkan waktu komputasi yang relatif panjang, **waktu respons (*response time*)** menjadi metrik evaluasi teknis utama untuk menilai efisiensi sistem. Kualitas data dan kecepatan respons (*response speed*) dari *resource pool* DSS sangat menentukan efektivitas pengambilan keputusan; latensi yang tinggi berisiko memperlambat penyajian wawasan berbasis data (*data-supported insights*) dan mengganggu koordinasi operasional tim.

---

### **2.2.2 Arsitektur Caching dan Redis**

Akses data ke basis data relasional (*Relational Database Management System* / RDBMS) berbasis disk merupakan sumber utama latensi pada aplikasi web karena melibatkan operasi I/O, penyaringan (*filtering*), pengurutan (*sorting*), dan pemrosesan kueri agregasi yang kompleks. Mekanisme *in-memory caching* berfungsi memotong latensi tersebut dengan menyimpan hasil kalkulasi atau data yang sering diakses ke dalam memori kerja berkecepatan tinggi (RAM), sehingga mengalihkan beban kerja I/O dari penyimpanan disk. **Redis (*REmote DIctionary Server*)** adalah *in-memory NoSQL key-value data store* yang menyediakan platform *caching* berkinerja tinggi guna mem-bypass RDBMS secara penuh dan menekan waktu eksekusi kueri.

Secara arsitektural, implementasi *caching* umumnya menerapkan pola **Cache-Aside (*lazy caching*)**. Pada pola ini, aplikasi mengontrol alur pembacaan data secara aktif: ketika menerima permintaan, sistem terlebih dahulu memeriksa keberadaan data di memori Redis (*cache check*). Jika terjadi *cache hit*, data langsung dikembalikan dari RAM; namun jika terjadi *cache miss*, sistem mengeksekusi kueri ke basis data relasional, menyimpan hasil kalkulasinya ke Redis, lalu mengembalikannya ke pengguna. 

Untuk menjaga kesegaran data (*data freshness*), mekanisme *Time-To-Live* (TTL) berbasis durasi waktu sering digunakan sebagai pembatas umur data di *cache*. Namun, kebijakan TTL statis memiliki keterbatasan dalam menangani mutasi data yang dinamis, karena data *cache* berisiko menjadi usang (*stale data*) sebelum durasi TTL berakhir. Oleh karena itu, pendekatan **Event-Driven Invalidation (*write-triggered invalidation*)** diterapkan dengan cara mengirim sinyal invalidasi atau pembersihan entri *cache* secara proaktif segera setelah terjadi operasi penulisan (*insert/update/delete*) pada basis data utama. Pada *framework* Laravel, konsep invalidasi dinamis ini diimplementasikan melalui *Eloquent Observers* (`TaskObserver` dan `TaskAssignmentObserver`) yang memantau peristiwa perubahan data tugas untuk menjamin konsistensi yang kuat (*strong consistency*) pada metrik DSS.

---

### **2.2.3 Teori ORM dan Inefisiensi Kueri Eloquent**

*Object-Relational Mapping* (ORM) seperti Eloquent pada Laravel mengadopsi pola *Active Record*, di mana satu kelas model mewakili tabel basis data dan satu instansi objek mewakili baris data. Walaupun ORM meningkatkan produktivitas pengembangan melalui abstraksi berorientasi objek, lapisan abstraksi ini sering menyembunyikan biaya eksekusi kueri SQL yang sebenarnya. Terdapat tiga antipola (*antipattern*) utama pada lapisan ORM yang menjadi sumber inefisiensi performa:

1. **Unnecessary Data Retrieval (UD) / Column Over-fetching**: Terjadi ketika ORM melakukan eksekusi `SELECT *` untuk mengambil seluruh kolom dari tabel basis data, padahal logika aplikasi hanya membutuhkan sebagian kecil atribut (misalnya hanya kolom `id`). Penarikan data berlebih ini memicu pemborosan I/O basis data dan alokasi memori server.
2. **N+1 Query Problem (*Row-by-Row Processing*)**: Terjadi ketika kueri awal menghasilkan \\(N\\) baris data utama, dan aplikasi melakukan iterasi (*looping*) untuk memanggil kueri tambahan pada setiap baris data tersebut guna memuat relasinya. Antipola ini mengakibatkan akumulasi interaksi basis data (*database round-trips*) sebanyak \\(N+1\\) kali, yang menyebabkan peningkatan latensi eksekusi secara linier atau eksponensial seiring bertambahnya volume data. Mengubah pola kueri N+1 menjadi kueri konstan (*bulk fetch* / *eager loading*) terbukti dapat meningkatkan performa eksekusi hingga puluhan kali lipat.
3. **Object Graph Materialization Expense**: Merupakan *overhead* komputasi dan memori yang dibutuhkan ORM untuk memetakan hasil kueri relasional menjadi grafik objek (*object graph*) di dalam memori PHP. Pengambilan data tanpa pembatasan proyeksi kolom (*unrestricted fetching*) pada kueri berelasi kompleks memaksa ORM mengalokasikan memori RAM 2 hingga 3 kali lebih banyak dibandingkan kueri SQL murni (*Raw SQL*).

---

### **2.2.4 Profil Objek Penelitian: Aplikasi Koladi**

Objek penelitian ini berfokus pada **Koladi**, sebuah platform *Collaborative Virtual Office SaaS* berbasis *multi-tenant*. Sistem ini dibangun di atas *stack* teknologi yang terdiri dari **Laravel 12.x**, **PostgreSQL 15** sebagai basis data relasional utama, **Eloquent ORM**, **Redis 7.x** (`redis:7-alpine` via ekstensi `phpredis`) sebagai *in-memory cache*, serta web server **Nginx** dan **PHP-FPM** yang dijalankan di dalam lingkungan dikontainerisasi **Docker Compose** pada jaringan terisolasi `koladi_net`.

Ruang lingkup evaluasi dibatasi secara eksklusif pada **Lapisan 2 DSS (DSS Modal)** pada modul statistik kinerja, yang dipicu saat pengguna menekan tombol *"Lihat Tindak Lanjut"*. Modul ini diakses melalui dua *endpoint* API utama:
* `GET /statistik/modal-data`
* `GET /statistik/suggestions`

Secara teknis, pemrosesan data pada DSS Modal mengeksekusi **alur kerja 5 komponen kalkulasi utama** untuk menghasilkan 22 metrik produktivitas tim:
1. **`DSSService`**: Berfungsi sebagai pengorkestra utama yang mengoordinasikan pemanggilan modul dan aliran data.
2. **`SnapshotManager`**: Mengelola pencarian, validasi, dan penyimpanan *snapshot* hasil kalkulasi metrik.
3. **`MetricsCalculator`**: Mengeksekusi kalkulasi komputasional atas 22 metrik mentah produktivitas (seperti *task completion rate*, *workload distribution*, dan *progress percentage*) dari data aktivitas kerja.
4. **`TrendAnalyzer`**: Mengkalkulasi perbandingan tren performa kinerja tim antar-periode.
5. **`RecommendationEngine`**: Memproses hasil analisis metrik menggunakan logika aturan bisnis untuk mengklasifikasikan indikator ke dalam kategori rekomendasi *critical*, *warning*, atau *positive*.

Selain kelima komponen utama tersebut, pemanggilan *endpoint* `modal-data` juga mengeksekusi dua fungsi kalkulasi tambahan di tingkat *controller*, yaitu `getUrgentTasks()` dan `getWorkloadDistribution()`, yang berturut-turut menghitung daftar tugas mendesak dan distribusi beban kerja anggota tim. Integrasi *Redis Cache-Aside* dan optimasi ORM Eloquent difokuskan untuk mengeliminasi inefisiensi pada seluruh jalur eksekusi kalkulasi agregasi ini.