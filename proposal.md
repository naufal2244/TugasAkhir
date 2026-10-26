BAB I PENDAHULUAN  

1.1. Latar Belakang 

Peralihan global menuju pola kerja terdistribusi telah mendorong organisasi untuk mengadopsi platform kolaborasi guna mendukung koordinasi tim lintas batas geografis (Froese et al., 2025). Di dalam platform tersebut, Decision Support System (DSS) memegang peran analitis yang krusial. Alih-alih sekadar menyajikan data statis, DSS memproses dan mengagregasi data aktivitas tim yang saling berelasi untuk menghasilkan indikator produktivitas manajerial (Cai & Chen, 2021; Koornneef et al., 2020). Paradigma ini diterapkan pada Koladi, sebuah aplikasi collaborative virtual office berbasis SaaS yang dibangun menggunakan Laravel 12.x dan PostgreSQL 15. Modul DSS Koladi bertugas menghitung 22 metrik produktivitas untuk setiap workspace secara dinamis, meliputi rasio penyelesaian tugas, progres anggota, hingga distribusi beban kerja. Namun, tingginya intensitas komputasi analitis ini memicu beban komputasi server yang berat, sehingga penyajian informasi manajerial sering kali terhambat oleh latensi pemrosesan yang lambat. 

Pemeriksaan langsung terhadap kode sumber Koladi yang divalidasi melalui profiling Laravel Telescope mengungkap akar inefisiensi arsitektural pada modul DSS tersebut. Pada konfigurasi awal, SnapshotManager menyimpan metrik ke dalam tabel relasional (workspace_performance_snapshots), sehingga setiap permintaan terus membebani basis data melalui eksekusi kueri SQL relasional. Selain itu, endpoint modal-data mengalami partial cache hit, di mana hanya metrik ringkasan yang tersimpan, sementara kalkulasi tugas mendesak dan distribusi beban kerja tetap dieksekusi ulang dari awal. Di lapisan Object-Relational Mapping (ORM), ditemukan tiga pola inefisiensi: pemanggilan relasi assignedUsers tanpa pembatasan kolom yang memicu Unnecessary Data Retrieval (Alshemaimri et al., 2021), lazy loading pada relasi checklists yang menimbulkan masalah N+1 query pada setiap tugas (Turcotte et al., 2022), serta eksekusi kueri Task::whereHas() di dalam perulangan anggota workspace (query-in-loop).  

Untuk mendiagnosis dampak inefisiensi tersebut dan mengukur efektivitas perbaikan secara objektif, evaluasi performa memerlukan sekumpulan metrik komputasional yang terstandarisasi. Response time menjadi parameter utama untuk menilai kecepatan dan responsivitas aplikasi di bawah kondisi pengujian terkendali (Putra et al., 2025; Privalov & Stupina, 2024). Karena latensi pemrosesan sangat bergantung pada intensitas operasi basis data, SQL query count menjadi indikator diagnostik kunci untuk memverifikasi reduksi beban kueri redundan (Alshemaimri et al., 2021). Sementara itu, pemodelan objek ORM yang tidak efisien dapat diukur melalui memory utilization guna mencatat konsumsi RAM server akibat materialisasi data persisten yang berlebihan (Yusmita et al., 2025; Makwana & Kundan, 2025). Terakhir, cache hit ratio berfungsi mengukur proporsi permintaan yang berhasil dilayani dari memori cepat, yang harus dianalisis secara terpadu bersama jumlah kueri SQL guna membuktikan apakah mekanisme caching benar-benar meniadakan akses ke basis data utama (An & Cao, 2022; Yang et al., 2020). 

Guna mengentaskan permasalahan tersebut, penelitian ini menerapkan dua intervensi terpadu yang saling melengkapi. Mekanisme cache relasional digantikan dengan pola Redis Cache-Aside berbasis in-memory. Secara bersamaan, optimasi ORM diimplementasikan melalui selective eager loading (with()), pembatasan atribut kolom, serta restrukturisasi query-in-loop menjadi pengambilan data secara batch dan agregasi in-memory. Evaluasi simultan ini sangat krusial karena caching dan optimasi kueri memiliki interaksi fungsional yang erat: efisiensi ORM menentukan beban server saat terjadi cache miss, sedangkan efektivitas Redis menentukan seberapa sering lapisan ORM perlu dieksekusi. Sebagian besar penelitian terdahulu hanya menguji caching atau optimasi ORM secara terpisah (Putra et al., 2025; Turcotte et al., 2022), sehingga bukti empiris mengenai dampak penggabungan keduanya dalam lingkungan kerja virtual terdistribusi masih sangat terbatas.  Selain itu, inefisiensi kueri relasional dan pemuatan data memiliki karakteristik yang memburuk secara linier seiring bertambahnya volume data (Turcotte et al., 2022). Oleh karena itu, pengujian performa perlu dievaluasi secara bertingkat di bawah variasi skala volume data yang terkontrol (Henning & Hasselbring, 2022; Yusmita et al., 2025). Dengan demikian, penelitian ini diposisikan sebagai upaya rekayasa perangkat lunak (engineering optimization) yang melakukan restrukturisasi kode langsung terhadap jalur eksekusi modul DSS Koladi, bukan sebatas observasi pasif. Melalui pendekatan eksperimental komparatif, penelitian ini difokuskan untuk menjawab pertanyaan: sejauh mana response time, SQL query count, memory utilization, dan cache hit ratio pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data berubah setelah penerapan Redis Cache-Aside dan optimasi kueri ORM secara bersamaan?  
 

1.2. Rumusan Masalah 

Berdasarkan latar belakang yang telah diuraikan, rumusan masalah dalam penelitian ini dirumuskan sebagai berikut: 

1. Bagaimana merancang dan menerapkan strategi Redis Cache-Aside serta optimasi kueri Eloquent ORM (selective eager loading dan in-memory aggregation) pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data modul DSS Koladi? 

2. Seberapa besar peningkatan kinerja modul DSS Koladi pada kondisi sesudah optimasi (optimized) dibandingkan kondisi sebelum optimasi (baseline), ditinjau dari parameter response time, SQL query count, memory utilization, dan cache hit ratio? 

3. Seberapa besar pengaruh variasi skala volume dataset sintetis (Small, Medium, Large) terhadap efektivitas optimasi tersebut, serta apakah peningkatan performa yang dihasilkan terbukti signifikan secara statistik melalui paired-sample t-test?  

1.3. Tujuan Penelitian 

1. Menerapkan strategi Redis Cache-Aside serta optimasi kueri Eloquent ORM (selective eager loading dan in-memory aggregation) pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data modul DSS Koladi. 

2. Membandingkan kinerja modul DSS sebelum (baseline) dan sesudah optimasi (optimized) berdasarkan parameter response time, SQL query count, memory utilization, dan cache hit ratio. 

3. Menganalisis pengaruh intervensi simultan tersebut di bawah variasi volume dataset sintetis (Small, Medium, Large) serta mengevaluasi signifikansi perbedaannya melalui paired-sample t-test dan persentase perbaikan performa. 

1.4. Batasan Masalah 

Agar penelitian lebih terarah dan sesuai dengan tujuan yang telah ditetapkan, batasan masalah dalam penelitian ini ditetapkan sebagai berikut. 

1. Objek penelitian difokuskan secara eksklusif pada modul Decision Support System (DSS) aplikasi Koladi, spesifik pada dua endpoint: GET /statistik/suggestions dan GET /statistik/modal-data. 

2. Intervensi optimasi dibatasi pada penerapan Redis Cache-Aside di lapisan caching, serta selective eager loading (with()) dan restrukturisasi query-in-loop menjadi batch retrieval di lapisan Eloquent ORM. 

3. Fungsionalitas sistem dipertahankan secara utuh; penelitian tidak mengubah formula kalkulasi 22 metrik produktivitas maupun kontrak respons API publik. 

4. Parameter evaluasi dibatasi pada empat metrik teknis kuantitatif: response time (ms), SQL query count (kueri/request), memory utilization (MB), dan cache hit ratio (%). 

5. Skenario pengujian menggunakan dataset sintetis terstruktur (seeded data) berskala Small (50 tugas/5 anggota), Medium (200 tugas/15 anggota), dan Large (500 tugas/30 anggota), dieksekusi secara sekuensial terisolasi menghasilkan 240 pengukuran valid setelah eliminasi warm-up. 

6. Penelitian ini tidak mengevaluasi uji konkurensi (concurrent user workload), load testing, stress testing, throughput, maupun aspek persepsi manusia (seperti survei kepuasan atau evaluasi User Experience). 

 

1.5. Manfaat Penelitian 

Adapun manfaat yang diharapkan dari penelitian Tugas Akhir ini adalah:  

1. Manfaat Teoretis 

Memberikan bukti empiris mengenai dampak penggabungan strategi Redis Cache-Aside dan optimasi kueri ORM pada pemrosesan analitis DSS berbasis web, sekaligus menyumbangkan wawasan metodologis mengenai pentingnya evaluasi metrik kinerja secara terpadu, khususnya korelasi antara cache hit ratio dan SQL query count untuk memverifikasi reduksi akses basis data yang sesungguhnya. 

2. Manfaat Praktis 

Menjadi panduan teknis bagi pengembang dalam mengidentifikasi dan menanggulangi masalah database-backed caching, Unnecessary Data Retrieval, kueri N+1, dan query-in-loop pada ekosistem Laravel dan PostgreSQL, serta menghasilkan arsitektur modul DSS Koladi yang lebih responsif dan efisien tanpa membebani infrastruktur basis data relasional. 

BAB II TINJAUAN PUSTAKA  

2.1 Penelitian Terkait 

Penelitian mengenai peningkatan performa dan efisiensi sistem perangkat lunak berbasis web umumnya berfokus pada dua area intervensi utama, yaitu optimasi pada lapisan akses data (kueri dan Object-Relational Mapping/ORM) serta penerapan lapisan penyimpanan sementara (caching). Subbab ini menguraikan lima penelitian terdahulu yang relevan guna memetakan perkembangan terkini, mengidentifikasi keterbatasan studi yang ada, serta menegaskan celah penelitian (research gap) dan kebaruan (novelty) Tugas Akhir ini. 

Penelitian optimasi performa framework dievaluasi oleh Putra et al. (2025) pada tiga framework PHP populer (Laravel, CodeIgniter, dan Symfony) menggunakan basis data MySQL 8.0 dan pengujian beban Apache JMeter. Hasil pengujian membuktikan bahwa penerapan teknik optimasi bawaan—seperti caching, indexing, dan eager loading—mampu menurunkan average response time Laravel sebesar 35,1% di bawah beban kerja tinggi (dari 118,34 ms menjadi 88,58 ms). Kendati demikian, evaluasi tersebut terbatas pada operasi CRUD sederhana tanpa mencatat konsumsi memori server maupun kueri SQL basis data. Putra et al. (2025) secara eksplisit menegaskan adanya keterbatasan literatur, di mana studi komparatif yang mengintegrasikan berbagai teknik optimasi secara bersamaan pada framework PHP masih sangat terbatas, serta menyarankan pengujian lanjutan pada basis data relasional lain seperti PostgreSQL. 

Efektivitas in-memory caching Redis dikaji secara empiris oleh Privalov dan Stupina (2024) pada aplikasi web berbasis Spring Boot dan MySQL. Pengalihan jalur pembacaan data (read path) dari basis data disk ke Redis terbukti memangkas waktu eksekusi kueri hingga 10–12 kali lipat (rata-rata 197,5 ms menjadi 41,2 ms), di mana serialisasi data penuh di Redis mengeliminasi kueri RDBMS saat terjadi cache hit. Namun, penelitian ini murni berfokus pada lapisan caching tanpa merefaktorisasi inefisiensi kueri pada lapisan ORM, serta hanya mengukur metrik latensi pada sistem informasi blog sederhana tanpa mengamati konsumsi memori server atau jumlah kueri basis data. 

Pada lapisan ORM, Turcotte et al. (2022) mengembangkan perkakas reformulator untuk mengatasi antipola N+1 query yang dipicu oleh mekanisme lazy loading pada Sequelize ORM. Melalui analisis aliran data statis (static data-flow analysis) pada delapan proyek open-source, Turcotte et al. (2022) membuktikan bahwa merefaktorisasi kueri looping N+1 menjadi kueri konstan (eager loading) memotong kueri SQL dari 11 menjadi 2 kueri serta memberikan peningkatan kecepatan eksekusi (speedup) hingga 38,58 kali lipat pada volume data 1.000 entitas. Walaupun demikian, penelitian ini murni berfokus pada restrukturisasi kueri relasional dan tidak mengukur penggunaan memori server maupun interaksinya dengan lapisan caching. 

Penjelasan mendalam mengenai beban komputasi internal yang ditimbulkan oleh lapisan ORM diuraikan oleh Yusmita et al. (2025), yang membandingkan performa Prisma ORM dan Raw SQL pada PostgreSQL 15 di dalam lingkungan Docker Compose. Hasil pengujian menunjukkan bahwa eksekusi kueri melalui ORM tanpa pembatasan kolom memicu konsumsi RAM 2 hingga 3 kali lebih besar dibandingkan Raw SQL (rata-rata 977 KB vs 357 KB pada kueri berelasi). Yusmita et al. (2025) menjelaskan bahwa pembengkakan memori ini disebabkan oleh biaya materialisasi grafik objek (object graph materialization expense) saat runtime mengonversi baris tabel mentah menjadi instance objek model. Kendati memberikan landasan teoritis yang kuat mengenai pemborosan memori ORM, Yusmita et al. (2025) menyelesaikan masalah dengan beralih ke Raw SQL (meninggalkan ORM), bukan menguji apakah optimasi internal ORM (selective field loading) yang dipadukan dengan caching dapat menekan overhead tersebut. 

Kajian yang meninjau kombinasi optimasi pada ekosistem Laravel dilakukan oleh Kundan dan Makwana (2025) pada aplikasi e-commerce berbasis cloud. Pengujian A/B menunjukkan bahwa Redis caching memangkas waktu respons dari 800 ms menjadi 150 ms (cache hit ratio 95%), eager loading mereduksi kueri dari 15 menjadi 2 kueri (latensi turun dari 1.200 ms ke 300 ms), dan selective field loading menekan penggunaan memori sebesar 35–40%. Namun, keterbatasan utama studi ini adalah teknik-teknik optimasi tersebut dievaluasi secara terisolasi per skenario A/B test pada listing produk generik, bukan dievaluasi sebagai dampak simultan dalam satu alur pemrosesan terintegrasi. 

Sintesis terhadap kelima penelitian terdahulu menunjukkan bahwa masing-masing teknik memiliki efektivitas yang terbukti kuat: caching Redis memotong latensi pembacaan data secara drastis (Privalov & Stupina, 2024; Kundan & Makwana, 2025), eager loading menghentikan pertumbuhan linier kueri N+1 (Turcotte et al., 2022), dan pembatasan kolom menekan biaya materialisasi memori ORM (Yusmita et al., 2025). Namun, studi-studi tersebut umumnya menguji aplikasi CRUD, blog, atau e-commerce biasa secara terpisah. Berbeda dari studi sebelumnya, Tugas Akhir ini meneliti modul Decision Support System (DSS) pada aplikasi collaborative virtual office (Koladi) yang menjalankan kalkulasi agregasi relasional bertahap untuk menghasilkan 22 metrik performa tim di bawah beban dataset bertingkat. 

Berdasarkan telaah komparatif tersebut, celah penelitian (research gap) yang teridentifikasi adalah belum adanya penelitian empiris yang mengevaluasi dampak simultan antara penerapan strategi Redis Cache-Aside dan optimasi kueri Eloquent ORM (selective eager loading serta batch in-memory aggregation) terhadap empat metrik teknis terpadu—yaitu response time, SQL query count, memory utilization, dan cache hit ratio—pada modul Decision Support System (DSS) berbasis web. Tugas Akhir ini mengisi ruang penelitian tersebut melalui perancangan eksperimen komparatif terkontrol before-after pada dua endpoint DSS Koladi (GET /statistik/suggestions dan GET /statistik/modal-data). 

Ringkasan perbandingan antara kelima penelitian terdahulu dengan penelitian Tugas Akhir ini disajikan pada Tabel 1. 

Tabel 1. Perbandingan Penelitian Sebelumnya 

| Peneliti/Tahun | Judul/Topik Penelitian | Metode/Teknologi | Kelebihan | Kekurangan/Celah |
|---|---|---|---|---|
| Putra et al. (2025) | Analysis Comparative of Performance Optimization Techniques for PHP Framework Testing: Laravel, CodeIgniter, Symfony | Load testing Apache JMeter pada PHP 8.1 dan MySQL 8.0 | Membuktikan reduksi latensi Laravel sebesar 35,1% di bawah beban tinggi | Terbatas pada CRUD MySQL; tanpa metrik kueri SQL/memori; belum menguji integrasi teknik secara simultan. |
| Privalov & Stupina (2024) | Improving Web-Oriented Information Systems Efficiency Using Redis Caching Mechanisms | Eksperimen sekuensial pada Spring Boot, MySQL, dan Redis via AOP | Memangkas latensi eksekusi kueri hingga 10–12 kali lipat via serialized caching | Murni fokus pada lapisan caching tanpa optimasi ORM; hanya menguji latensi kueri pada sistem blog sederhana. |
| Turcotte et al. (2022) | reformulator: Automated Refactoring of the N+1 Problem in Database-Backed Applications | Analisis statis aliran data (CodeQL) pada Sequelize ORM | Mengeliminasi N+1 menjadi kueri konstan (11 ke 2 kueri) dengan speedup hingga 38,58x | Tidak mengukur penggunaan memori server maupun interaksi dengan lapisan caching; fokus pada ORM statis. |
| Yusmita et al. (2025) | Optimizing Database Access Strategy: A Performance Analysis Comparison of Raw SQL and Prisma ORM | Controlled benchmark (Docker) pada Node.js dan PostgreSQL 15 | Menguantifikasi overhead memori ORM (2–3x lebih tinggi) akibat object graph materialization | Mengganti ORM menjadi Raw SQL alih-alih mengoptimasi ORM dari dalam; tidak menguji lapisan caching. |
| Kundan & Makwana (2025) | A Comprehensive Study on Performance Optimization Techniques Database Queries and API Response Time in Laravel-Based Web Applications | Uji A/B empiris pada Laravel 10.x, PostgreSQL 14, dan Redis 6.x | Mengukur dampak Redis, eager loading, dan seleksi kolom pada ekosistem Laravel | Setiap teknik diuji secara terpisah per skenario A/B pada katalog e-commerce generik, bukan dampak simultan. |
| Tugas Akhir Ini | Evaluasi simultan Redis Cache-Aside & optimasi ORM | Controlled before-after benchmarking (Docker) pada Laravel 12 & PostgreSQL 15 | Mengevaluasi dampak simultan Redis Cache-Aside dan optimasi ORM pada jalur komputasi DSS dengan 4 metrik terpadu | Pengujian dibatasi pada benchmark sekuensial terisolasi pada dua endpoint DSS tanpa pengujian beban konkuren. |

2.2. Landasan Teori 

Subbab ini memuat konsep-konsep dasar yang mendasari pelaksanaan Tugas Akhir, berfokus secara spesifik pada teori Decision Support System (DSS) berbasis web, arsitektur in-memory caching menggunakan Redis, pola inefisiensi kueri pada Object-Relational Mapping (ORM) Eloquent, serta profil modul DSS aplikasi Koladi sebagai objek penelitian. 

2.2.1. Konsep Decision Support System (DSS) Berbasis Web 

Decision Support System (DSS) didefinisikan sebagai sistem informasi berbasis komputer yang dirancang untuk mendukung pengambilan keputusan semi-terstruktur melalui interaksi manusia-komputer dan pemodelan analitis data (Cai & Chen, 2021). Pada aplikasi kolaboratif modern, DSS diakses melalui peramban web  guna memfasilitasi akses multi-pengguna secara terdistribusi tanpa memerlukan instalasi perangkat lunak di sisi klien (Koornneef et al., 2020). Berbeda dari penyajian data transaksi operasional biasa, modul DSS menjalankan pipeline agregasi bertahap terhadap data aktivitas kerja yang saling berelasi untuk menghasilkan indikator kinerja manajerial yang dapat ditindaklanjuti secara tepat waktu. 

Tingginya intensitas komputasi analitis pada modul DSS menuntut sistem untuk menyajikan hasil kalkulasi secara tepat waktu agar wawasan manajerial tidak terlambat diterima oleh pengambil keputusan (Cai & Chen, 2021). Dalam evaluasi kinerja perangkat lunak berbasis web, response time menjadi metrik teknis utama untuk menilai tingkat responsivitas sistem tersebut (Alnuhait et al., 2024; Putra et al., 2025). Durasi eksekusi permintaan DSS dibentuk secara langsung oleh intensitas dan biaya operasi akses basis data relasional (disk I/O) pada lapisan infrastruktur (Taipalus, 2024). Oleh karena itu, latensi komputasi yang tinggi pada kueri agregasi berisiko menunda penyajian indikator kinerja tim serta menghambat kelancaran koordinasi kerja organisasi (Cai & Chen, 2021; Koornneef et al., 2020). 

2.2.2. Arsitektur Caching dan Redis 

Akses data ke basis data relasional (RDBMS) berbasis disk merupakan sumber utama latensi pada pemrosesan analitis web karena melibatkan operasi I/O, filtering, sorting, dan pemindaian tabel (Privalov & Stupina, 2024).  In-memory caching berfungsi memotong latensi tersebut dengan menyimpan hasil kalkulasi atau data yang sering diakses ke dalam memori utama (RAM), yang memiliki kecepatan akses sekitar empat orde magnitudo lebih cepat dibandingkan media penyimpanan berbasis disk (Taipalus, 2024). Redis (Remote Dictionary Server) merupakan in-memory NoSQL key-value data store yang dirancang untuk mem-bypass beban akses ke basis data relasional secara penuh pada alur pembacaan (read path) (Privalov & Stupina, 2024). 

Pola arsitektur yang diterapkan dalam penelitian ini adalah Cache-Aside (look-aside caching). Melalui pola ini, aplikasi mengontrol alur pembacaan data secara eksplisit: ketika menerima permintaan, sistem terlebih dahulu memeriksa keberadaan data di memori Redis. Jika terjadi cache hit, data langsung dikembalikan dari RAM tanpa membebani basis data; sedangkan jika terjadi cache miss, sistem mengeksekusi pipeline kalkulasi ke basis data relasional, menyimpan hasil agregasi ke dalam Redis untuk melayani permintaan berikutnya, lalu mengembalikannya ke pengguna (An & Cao, 2022). Mekanisme ini menjamin efisiensi sumber daya server karena komputasi analitis yang berat hanya dijalankan ketika data belum tersedia di dalam memori. 

2.2.3. Teori ORM dan Inefisiensi Kueri Eloquent 

Eloquent ORM pada kerangka kerja Laravel mengadopsi pola Active Record, di mana satu kelas model memetakan satu tabel basis data dan instansi objek mewakili baris data. Meskipun meningkatkan produktivitas pengodean, lapisan abstraksi ORM menyembunyikan biaya eksekusi kueri SQL yang sebenarnya di balik layar, sehingga rentan memicu inefisiensi performa yang parah apabila tidak dikonfigurasi secara cermat (Alshemaimri et al., 2021; Turcotte et al., 2022). Terdapat empat pola inefisiensi utama pada lapisan ORM yang relevan dengan penelitian ini: 

Unnecessary Data Retrieval (UD): Kondisi ketika kueri ORM menarik seluruh kolom tabel (SELECT *) padahal logika kalkulasi hanya memerlukan atribut tertentu. Pola ini memicu pemborosan transmisi I/O dan alokasi memori server (Alshemaimri et al., 2021). 

N+1 Query Problem: Terjadi akibat pemuatan relasi lazy loading di dalam iterasi perulangan, di mana sistem mengeksekusi 1 kueri utama untuk entitas induk dan N kueri tambahan terpisah untuk setiap baris data relasinya. Pola ini melipatgandakan round-trips basis data secara linier (O(N)), yang dapat dieliminasi melalui eager loading (with()) menjadi kueri konstan (Turcotte et al., 2022). 

Query-in-Loop: Eksekusi kueri kondisional berulang di dalam logika perulangan aplikasi (seperti pemanggilan Task::whereHas() per anggota tim), yang membuat beban kueri bertambah seiring bertambahnya jumlah anggota. Masalah ini diselesaikan melalui transformasi ke teknik batch retrieval dan agregasi di dalam memori (in-memory aggregation). 

Object Graph Materialization Expense: Beban alokasi RAM yang dibutuhkan runtime PHP untuk mengonversi baris data mentah relasional menjadi grafik objek model. Pengambilan data tanpa pembatasan kolom pada relasi kompleks terbukti meningkatkan konsumsi memori 2 hingga 3 kali lebih besar dibandingkan kueri yang dibatasi kolomnya (selective field loading) (Yusmita et al., 2025; Kundan & Makwana, 2025). 

2.2.4. Profil Objek Penelitian: Aplikasi Koladi 

Objek penelitian ini adalah Koladi, sebuah aplikasi collaborative virtual office berbasis Software as a Service (SaaS) multi-tenant yang dibangun menggunakan Laravel 12.x, PostgreSQL 15, Eloquent ORM, dan Redis 7.x di dalam lingkungan kontainer Docker Compose (jaringan internal koladi_net). Evaluasi difokuskan secara eksklusif pada Lapisan 2 DSS (DSS Modal) pada modul statistik kinerja, yang diakses melalui dua endpoint utama: 

GET /statistik/suggestions: Menyajikan rekomendasi manajerial berbasis 5 aturan evaluasi kinerja tim. 

GET /statistik/modal-data: Menyajikan payload analitis lengkap yang mencakup ringkasan metrik utama, daftar tugas mendesak (urgent tasks), dan distribusi beban kerja anggota (workload distribution). 

Pipeline komputasi DSS Modal Koladi bertugas mengolah 22 metrik produktivitas mentah per workspace melalui perangkaian lima komponen inti: DSSService (orkestrator alur kerja), SnapshotManager (manajemen cache/snapshot), MetricsCalculator (kalkulasi metrik tugas relasional via getTasksInPeriod()), TrendAnalyzer (analisis komparasi tren performa antar-periode), dan RecommendationEngine (klasifikasi saran rekomendasi manajerial), serta didukung fungsi controller getUrgentTasks() dan getWorkloadDistribution(). 

Pada kondisi awal (baseline), sistem mengandalkan mekanisme snapshot berbasis tabel relasional (workspace_performance_snapshots) yang tetap memicu kueri SQL pada setiap permintaan serta mengalami partial cache hit, diperberat oleh inefisiensi ORM berupa column over-fetching, N+1 query checklists, dan query-in-loop beban kerja. Melalui intervensi Tugas Akhir ini, konfigurasi sistem ditransformasikan ke kondisi teroptimasi (optimized) yang menerapkan Redis Cache-Aside untuk mem-bypass akses basis data secara penuh pada alur pembacaan, selective eager loading (with(['assignedUsers:id', 'checklists:id,task_id,is_done'])), serta batch retrieval dengan in-memory aggregation. 
 

2.3. Metode Pengembangan Produk 

Pengembangan perangkat lunak dalam Tugas Akhir ini tidak membangun sistem baru dari awal (greenfield development), melainkan melakukan evolusi dan restrukturisasi kode pada sistem yang telah berjalan (brownfield software evolution). Oleh karena itu, penelitian ini tidak menggunakan metodologi pengembangan konvensional, seperti Waterfall (Bassil, 2012) atau Agile (Al-Saqqa et al., 2020), melainkan mengadopsi metode Refactoring Lifecycle yang dirumuskan oleh Mens dan Tourwé (2004) dengan berlandaskan pada prinsip dasar refaktorisasi oleh Fowler (2018). Menurut Fowler (2018), refaktorisasi adalah proses perubahan yang dilakukan pada struktur internal perangkat lunak guna meningkatkan efisiensi performa tanpa mengubah perilaku eksternal sistem yang teramati (behavior-preserving). Berdasarkan kerangka kerja Mens dan Tourwé (2004), proses refaktorisasi perangkat lunak dilaksanakan melalui enam tahapan terstruktur sebagai berikut: 

 

2.3.1. Identifikasi Lokasi Refaktorisasi  

Menurut Mens & Tourwé (2004) dan Fowler (2018), tahap ini bertujuan mendeteksi bagian kode yang memiliki kelemahan struktural (code smells) atau menjadi titik hambatan performa (performance hot spots). Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah melakukan profiling awal terhadap modul DSS Koladi menggunakan Laravel Telescope dan pencatatan kueri SQL pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data untuk mengidentifikasi keberadaan kueri berulang (N+1 query problem pada relasi checklists), query-in-loop pada kalkulasi beban kerja tim (getWorkloadDistribution()), column over-fetching, serta latensi pembacaan data pada basis data PostgreSQL 15. 

2.3.2. Penentuan Teknik Refaktorisasi (Determine Refactoring Techniques) 

Menurut Mens & Tourwé (2004), tahap ini bertujuan memilih pola transformasi kode yang tepat untuk mengeliminasi kelemahan struktural dan inefisiensi yang telah diidentifikasi. Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah merancang strategi optimasi backend terpadu, meliputi penentuan teknik selective eager loading (with(['assignedUsers:id', 'checklists:id,task_id,is_done'])), restrukturisasi algoritma perulangan menjadi batch retrieval dengan agregasi di dalam memori (in-memory aggregation), serta perancangan integrasi in-memory caching berbasis Redis menggunakan pola Cache-Aside guna mem-bypass akses basis data relasional pada alur pembacaan data (read path) analitis DSS. 

2.3.3. Penjaminan Preservasi Perilaku (Guarantee Behavior Preservation) 

Menurut Mens & Tourwé (2004) dan Fowler (2018), tahap ini bertujuan menjamin secara mutlak bahwa modifikasi struktur internal kode sama sekali tidak mengubah luaran fungsional maupun semantik sistem melalui penyusunan pengujian mandiri (self-checking tests). Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah menyusun pengujian verifikasi regresi otomatis (automated regression/equivalence tests) untuk menguji dan mengunci integritas kalkulasi 22 metrik performa tim dan 5 komponen rekomendasi manajerial DSS, guna memverifikasi bahwa respons data JSON sebelum dan sesudah refaktorisasi terbukti identik. 

2.3.4. Penerapan Refaktorisasi (Apply the Refactoring) 

Menurut Mens & Tourwé (2004), tahap ini merupakan eksekusi aktual transformasi kode sumber pada basis kode perangkat lunak secara terkontrol. Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah mengimplementasikan transformasi kode pada lapisan servis Laravel 12 (khususnya merefaktorisasi metode getTasksInPeriod(), getUrgentTasks(), dan getWorkloadDistribution()), mengonfigurasi client driver Redis 7 sebagai cache store, serta mengaktifkan pembatasan kolom (selective field loading) dan pemrosesan batch pada repositori data PostgreSQL 15. 

2.3.5. Evaluasi Efek Kualitas (Assess Quality Effect) 

Menurut Mens & Tourwé (2004), tahap ini bertujuan menilai dan mengukur dampak kuantitatif dari restrukturisasi kode terhadap atribut kualitas perangkat lunak, khususnya karakteristik efisiensi kinerja (software performance). Adapun yang akan dilakukan pada tahapan ini dalam konteks Tugas Akhir adalah mengevaluasi performa teknis sebelum dan sesudah refaktorisasi melalui pengukuran empat metrik utama: waktu respons (response time), jumlah kueri SQL (query count), konsumsi memori (memory utilization), dan rasio keberhasilan cache (cache hit ratio). 

2.3.6. Pemeliharaan Konsistensi Sistem (Maintain Consistency) 

Menurut Mens dan Tourwé (2004), tahap ini bertujuan memelihara konsistensi antara kode sumber yang telah direfaktorisasi dengan artefak perangkat lunak lainnya serta lingkungan eksekusi sistem (change propagation). Pada konteks Tugas Akhir ini, tahapan tersebut mencakup tiga tindakan terpadu: (1) memverifikasi stabilitas orkestrasi lingkungan eksekusi kontainer Docker Compose (koladi_net) guna memastikan seluruh layanan (Laravel 12, PostgreSQL 15, dan Redis 7) beroperasi secara normal; (2) memverifikasi ketiadaan efek samping (side effects) melalui pengujian regresi fungsional (functional regression testing) pada modul manajemen tugas (Kanban) guna memastikan modifikasi relasi model data dan dependensi servis tidak mengganggu alur kerja operasional penugasan; serta (3) menyelaraskan artefak nonkode berupa pembaruan berkas konfigurasi deployment (.env.example), diagram arsitektur sistem, dan dokumentasi teknis modul DSS Koladi. 

 

2.4. Metode Analisis yang Digunakan 

Pengujian efisiensi kinerja dalam penelitian ini memerlukan metodologi yang mampu membuktikan hubungan sebab-akibat secara valid sekaligus menyediakan protokol pengukuran teknis yang presisi. Oleh karena itu, penelitian ini menerapkan pendekatan kuantitatif eksperimental dengan rancangan komparatif before-after berbasis kerangka kerja Experimental Software Engineering (Wohlin et al., 2000, 2012). Kerangka kerja tersebut digunakan sebagai landasan formal dalam merancang eksperimen berbasis variabel kontrol serta pengujian hipotesis statistik untuk membandingkan performa modul Decision Support System (DSS) Koladi sebelum optimasi (baseline) dan sesudah optimasi (optimized). 

 

2.4.1. Desain Penelitian dan Isolasi Lingkungan 

Pengujian performa dilakukan secara sekuensial terisolasi (isolated sequential execution) di dalam jaringan kontainer Docker Compose (koladi_net), bukan melalui pengujian beban konkuren (concurrent load testing) atau stress testing (Henning & Hasselbring, 2022; Putra et al., 2025). Pemilihan eksekusi sekuensial terisolasi ini bertujuan mengeliminasi variabel pengganggu eksternal seperti fluktuasi latensi jaringan luar, thread scheduling noise, dan perebutan sumber daya komputasi (Yusmita et al., 2025). Dengan lingkungan yang terkontrol ketat, hubungan sebab-akibat  antara intervensi arsitektur (Redis Cache-Aside dan optimasi ORM) terhadap penggunaan sumber daya server dapat diisolasi dan dibuktikan secara sahih (Wohlin et al., 2000). 

2.4.2. Variabel Penelitian 

Berdasarkan kerangka eksperimen formal (Wohlin et al., 2000), variabel penelitian diklasifikasikan ke dalam dua kelompok: 

Variabel Bebas:  

a. Konfigurasi Sistem (2 tingkat): Konfigurasi Baseline (menggunakan database-backed snapshot pada tabel workspace_performance_snapshots dan unrestricted ORM loading) versus Konfigurasi Optimized (menerapkan Redis Cache-Aside dan optimasi kueri Eloquent ORM). 

b. Kondisi Cache (2 tingkat): Kondisi Cache Miss (seluruh entri cache dikosongkan sebelum request untuk mengukur biaya komputasi penuh pipeline DSS) versus Kondisi Cache Hit (entri cache telah diinisialisasi terlebih dahulu untuk mengukur efisiensi retrieval memori). 

c. Skala Volume Dataset (3 tingkat): Dataset sintetis terstruktur yang dibangkitkan melalui Laravel Seeder dan Factory berskala Small (50 tugas/5 anggota), Medium (200 tugas/15 anggota), dan Large (500 tugas/30 anggota) dengan distribusi status tugas terkontrol (40% selesai, 35% dalam proses, 25% belum selesai). 

d. Endpoint DSS (2 jalur): GET /statistik/suggestions (jalur evaluasi aturan rekomendasi) dan GET /statistik/modal-data (jalur payload analitis lengkap yang mencakup metrik utama, tugas mendesak, dan beban kerja tim). 

Variabel Terikat (Dependent Variables):  

a. Response Time (ms): Durasi latensi end-to-end yang dibutuhkan server untuk memproses dan mengembalikan respons HTTP ke client (Taipalus, 2024). 

b. SQL Query Count: Jumlah total kueri SQL yang dieksekusi ke basis data PostgreSQL dalam satu siklus request (Turcotte et al., 2022). 

c. Memory Utilization (MB): Puncak alokasi memori RAM proses PHP per request yang diukur menggunakan fungsi internal memory_get_peak_usage(true) (Yusmita et al., 2025). 

d. Cache Hit Ratio (%): Rasio keberhasilan permintaan yang dilayani langsung dari lapisan cache (dihitung via Redis keyspace stats), yang diinterpretasikan secara bersamaan dengan SQL query count untuk memverifikasi ada tidaknya bypass basis data yang sesungguhnya (An & Cao, 2022). 

2.4.3. Instrumen dan Protokol Pengujian 

Pengumpulan data kinerja dilakukan menggunakan rangkaian instrumen pengukuran terpadu (benchmark instrumentation suite) yang dikembangkan khusus untuk mengadopsi protokol pengujian perangkat lunak terkontrol (Wohlin et al., 2000; Henning & Hasselbring, 2022): 

Laravel Telescope: Digunakan pada tahap profiling awal untuk mengidentifikasi keberadaan antipola kueri N+1 dan titik hambatan performa modul DSS 

Middleware Instrumentasi Laravel: Berfungsi mencatat eksekusi kueri SQL dan alokasi memori internal, serta rasio cache Redis secara otomatis pada setiap request yang membawa header khusus. 

Node.js Benchmark Runner: Bertugas mengeksekusi permintaan HTTP secara sekuensial terkontrol serta merekam durasi response time dengan presisi tinggi. 

CSV Validator: Skrip pemverifikasi otomatis untuk memastikan keutuhan matriks, kelengkapan kombinasi variabel, dan integritas baris data sebelum dianalisis. 

Untuk memastikan data yang dicatat benar-benar stabil, setiap pengujian diawali dengan 3 kali permintaan pemanasan (warm-up requests). Data pemanasan ini dibuang dari analisis karena pada tahap awal server masih melakukan inisialisasi koneksi basis data dan pengisian memori cache internal (Henning & Hasselbring, 2022; Traini et al., 2023; Japke et al., 2025). Setelah fase warm-up, pengujian dieksekusi sebanyak 10 repetisi valid secara berurutan, mengadaptasi skema repeated measurement dari Privalov & Stupina (2024) guna menangkap kestabilan performa (steady-state) serta meredam variasi fluktuasi sistem (Traini et al., 2023). Skema 10 repetisi ini juga menjamin struktur sampel berpasangan yang seimbang (balanced design) sebagai prasyarat uji Paired-Sample t-Test. Rancangan faktorial penuh (full factorial design) ini menghasilkan total matriks data valid sebesar: 

Total Data Valid = 3 Skala Data × 2 Endpoint × 2 Konfigurasi × 2 Kondisi Cache × 10 Repetisi = 240 Observasi Valid 

(Terdiri atas 120 observasi pada kondisi baseline dan 120 observasi pada kondisi optimized). 

2.4.4. Metode Analisis Data dan Uji Hipotesis 

Data hasil pengujian dianalisis secara bertahap mengikuti alur analisis eksperimen rekayasa perangkat lunak (Wohlin et al., 2000), dengan parameter dan kriteria evaluasi teknis yang merujuk pada literatur benchmarking terkait: 

1. Statistik Deskriptif 

Menghitung nilai rata-rata (Mean), deviasi standar (Standard Deviation), serta nilai minimum dan maksimum. Tujuannya adalah untuk melihat gambaran umum performa sistem dan menilai apakah kinerja sistem stabil pada setiap pengulangan (Japke et al., 2025). 

2. Persentase Perbaikan 

Menghitung magnitudo perbaikan kinerja teknis menggunakan formula: 

$$
\text{Improvement (\%)} = \left( \frac{\text{Mean Baseline} - \text{Mean Optimized}}{\text{Mean Baseline}} \right) \times 100
$$
 

 

Perubahan nilai di bawah 3% dikategorikan secara hati-hati sebagai variasi acak atau measurement noise (Japke et al., 2025). 

3. Uji Signifikansi (Paired-Sample t-Test) 

Menguji signifikansi perbedaan antara konfigurasi baseline dan optimized menggunakan Paired-Sample t-Test pada tingkat signifikansi α = 0,05 (tingkat kepercayaan 95%) (Uddin & Lu, 2024). Uji berpasangan ini sahih karena setiap observasi baseline memiliki pasangan observasi langsung pada optimized dengan skala dataset, endpoint, kondisi cache, dan nomor repetisi yang identik. 

4. Analisis Khusus Kueri SQL dan Cache  

SQL Query Count: Karena jumlah kueri bersifat pasti dan selalu sama di setiap repetisi (deterministik), pembuktian berfokus pada penurunan jumlah kueri secara langsung, dengan signifikansi statistik dilaporkan sebagai P < 0.001 (Japke et al., 2025; Uddin & Lu, 2024). 

Cache Hit Ratio: Dianalisis bersamaan dengan jumlah kueri untuk memastikan apakah kondisi Cache Hit benar-benar membebaskan basis data dari pemanggilan kueri secara penuh (full bypass) (An & Cao, 2022). 