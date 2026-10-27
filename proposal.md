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

BAB III METODOLOGI 

3.1. Alat dan Bahan 

Alat dan bahan digunakan untuk mendukung seluruh rangkaian penelitian, mulai dari proses diagnosis awal (profiling), restrukturisasi kode program modul DSS, penyusunan instrumen pengukuran performa, eksekusi pengujian sekuensial terkontrol (controlled benchmarking), hingga verifikasi data dan analisis statistik. 

3.1.1 Alat 

Alat yang digunakan untuk mendukung penelitian ini mencakup perangkat keras, perangkat lunak, serta perkakas instrumentasi pengukuran teknis sebagai berikut: 

Tabel 2. Alat Penelitian 



Spesifikasi perangkat keras dan perangkat lunak yang digunakan dicatat sebagai bagian dari dokumentasi lingkungan pengujian agar kondisi eksperimen dapat dijelaskan dan direplikasi secara konsisten. Pengujian performa dijalankan pada unit komputer dengan prosesor Apple M2 Pro, kapasitas memori utama (RAM) 16 GB, media penyimpanan Solid State Drive (SSD) 1 TB, dan sistem operasi macOS. 

| Alat | Fungsi |
| :--- | :--- |
| **Laptop / Komputer** | Menjalankan lingkungan pengujian terisolasi, mengorkestrasi kontainer Docker, mengeksekusi skrip benchmark sekuensial, serta mengolah data hasil eksperimen. |
| **Visual Studio Code** | Lingkungan pengembangan terintegrasi (IDE) untuk menelusuri kode sumber Koladi, merefaktorisasi modul DSS, dan menyusun skrip pengujian. |
| **Docker dan Docker Compose** | Menyediakan dan mengisolasi lingkungan pengujian multi-kontainer pada jaringan lokal (`koladi_net`), mencakup layanan Nginx, PHP-FPM, PostgreSQL, dan Redis. |
| **Laravel 12.x (PHP ^8.2)** | Kerangka kerja (*framework*) backend aplikasi Koladi yang menjadi subjek refaktorisasi modul DSS dan penyedia runtime eksekusi pengujian. |
| **PostgreSQL 15** | Sistem Manajemen Basis Data Relasional (RDBMS) utama untuk menyimpan entitas data relasional Koladi dan melayani kueri persisten. |
| **Redis 7.x (via phpredis)** | *In-memory key-value data store* yang berfungsi sebagai media penyimpanan *cache* berbasis RAM pada konfigurasi teroptimasi (Redis Cache-Aside). |
| **Node.js Benchmark Runner (`benchmark-dss.js`)** | Perkakas pengujian klien untuk mengorkestrasi eksekusi permintaan HTTP sekuensial terisolasi, mengatur 3 *warm-up requests*, menjalankan 10 repetisi pengujian valid, serta mencatat nilai *response time* menggunakan `process.hrtime()`. |
| **Middleware Instrumentasi Laravel (`BenchmarkInstrumentation.php`)** | Komponen *server-side* untuk mencatat metrik internal performa pada setiap *request* uji, meliputi jumlah kueri SQL (`DB::listen`), alokasi memori puncak (`memory_get_peak_usage(true)`), dan rasio *cache* Redis. |
| **Skrip CSV Validator (`validate-csv.js`)** | Skrip otomatis untuk memverifikasi integritas struktur berkas CSV, memastikan kelengkapan kombinasi perlakuan, dan memvalidasi keutuhan 240 baris data observasi valid sebelum dianalisis. |
| **Laravel Telescope** | Perkakas diagnostik untuk melakukan *profiling* awal guna mendeteksi antipola kueri (*N+1 query*, *query-in-loop*, dan *column over-fetching*) sebelum pengujian komparatif dijalankan. |
| **Git** | Sistem kontrol versi (*version control system*) untuk melacak riwayat perubahan kode sumber serta mengelola cabang (*branch*) *baseline* dan *optimized*. |
| **Web Browser** | Memeriksa antarmuka pengguna Koladi untuk keperluan verifikasi visual papan tugas (Kanban) dan modal statistik kinerja. |

 

3.1.2 Bahan 

Bahan-bahan yang digunakan dalam penelitian ini berupa berkas kode sumber, data sintetis pengujian, skema basis data, rekaman log, serta literatur pendukung sebagai berikut: 

Tabel 3. Bahan Penelitian 

| Bahan | Fungsi |
| :--- | :--- |
| **Kode sumber (*source code*) aplikasi Koladi** | Objek rekayasa refaktorisasi, khususnya berkas logika kalkulasi DSS (`DSSService`, `SnapshotManager`, `MetricsCalculator`, `TrendAnalyzer`, `RecommendationEngine`, serta *controller* terkait). |
| **Rute dan *endpoint* DSS Koladi** | Titik akses jalur eksekusi komputasi analitis yang diuji, mencakup rute `GET /statistik/suggestions` dan `GET /statistik/modal-data`. |
| **Skema basis data PostgreSQL Koladi** | Struktur tabel relasional, atribut kolom, relasi antar-entitas, dan indeks awal pada tabel `tasks`, `checklists`, `users`, `workspace_members`, `workspaces`, serta `workspace_performance_snapshots`. |
| **Dataset sintetis terstruktur (*Seeded Dataset*)** | Data pengujian terkontrol yang dibangkitkan via Laravel Seeder dan Factory (`ResearchDataSeeder.php`) dalam 3 skala volume: Small (50 tugas/5 anggota), Medium (200 tugas/15 anggota), dan Large (500 tugas/30 anggota). |
| **Log diagnostik Laravel Telescope** | Data penelusuran awal yang menjadi bukti empiris keberadaan inefisiensi kueri relasional dan pola *partial cache hit* pada konfigurasi awal (*baseline*). |
| **Data mentah hasil pengukuran (*Dataset CSV*)** | Berkas rekaman 240 observasi valid (120 data *baseline* dan 120 data *optimized*) yang memuat metrik *response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*. |
| **Rancangan kode refaktorisasi optimasi** | Implementasi teknis pola Redis Cache-Aside, *selective eager loading* (`with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`), dan *batch in-memory aggregation*. |
| **Skrip pengujian verifikasi ekuivalensi** | Kumpulan pengujian otomatis (*self-checking tests*) untuk memverifikasi bahwa respons data JSON 22 metrik produktivitas dan 5 aturan rekomendasi tidak mengalami perubahan semantik (*behavior preservation*). |
| **Literatur ilmiah dan dokumentasi teknis** | Landasan teori, panduan metodologi *Experimental Software Engineering*, prinsip siklus refaktorisasi, serta acuan interpretasi hasil evaluasi kinerja perangkat lunak. |

Bahan-bahan di atas dikelola dan diproses secara terintegrasi sepanjang tahapan penelitian guna menjamin validitas internal, akurasi pengukuran, serta keterulangan (reproducibility) eksperimen. 

 

3.2. Jalannya Penelitian 

Jalannya penelitian menggambarkan urutan kegiatan yang dilakukan secara bertahap dan sistematis dalam melaksanakan optimasi kinerja pada modul Decision Support System (DSS) Koladi. Mengingat penelitian ini berfokus pada evolusi perangkat lunak pada sistem yang telah berjalan (brownfield software evolution), jalannya penelitian dirancang dengan mengadopsi enam aktivitas baku dalam siklus hidup refaktorisasi (Refactoring Lifecycle) yang dirumuskan oleh Mens dan Tourwé (2004) serta berlandaskan prinsip-prinsip refaktorisasi oleh Fowler (2018). Tahapan penelitian meliputi: (1) Identifikasi Titik Hambatan, (2) Penentuan Strategi Optimasi, (3) Penjaminan Preservasi Perilaku, (4) Penerapan Refaktorisasi, (5) Penilaian Efek Kualitas, serta (6) Pemeliharaan Konsistensi Sistem. 


| Fase | Aktivitas / Tahapan |
| :---: | :--- |
| **Fase 1** | Identifikasi Titik Hambatan |
| **Fase 2** | Penentuan Strategi Optimasi |
| **Fase 3** | Penjaminan Preservasi Perilaku |
| **Fase 4** | Penerapan Refaktorisasi |
| **Fase 5** | Penilaian Efek Kualitas |
| **Fase 6** | Pemeliharaan Konsistensi Sistem |
```[cite: 1]

Gambar 1. Tahapan Penelitian 


3.2.1. Identifikasi Titik Hambatan  

Tahap Identifikasi Titik Hambatan dilakukan untuk mendeteksi komponen kode yang mengalami penurunan efisiensi kinerja (performance hot spots) pada modul DSS Koladi dalam kondisi awal sebelum optimasi, sejalan dengan prinsip Fowler (2018) untuk mengukur performa secara empiris menggunakan instrumen profiler dan tidak berspekulasi. Penelusuran difokuskan pada dua endpoint target, yaitu GET /statistik/suggestions dan GET /statistik/modal-data, dengan memanfaatkan perkakas diagnostik Laravel Telescope pada lingkungan pengembangan lokal sebagaimana dirumuskan pada Subbab 2.3.1. Penelusuran ini bertujuan memetakan pola pemanggilan basis data pada mekanisme snapshot relasional bawaan (workspace_performance_snapshots), mengidentifikasi gejala partial cache hit pada endpoint modal-data, mendeteksi pengambilan kolom yang berlebihan (unnecessary data retrieval) pada relasi assignedUsers, memeriksa keberadaan kueri berulang N+1 pada relasi checklists, serta mendeteksi eksekusi kueri di dalam perulangan (query-in-loop) pada kalkulasi distribusi beban kerja tim. Bukti temuan log kueri dari hasil profiling awal ini didokumentasikan sebagai bukti observasi awal pada Lampiran A. 

3.2.2. Penentuan Strategi Optimasi  

Tahap Penentuan Strategi Optimasi dilakukan untuk merancang teknik transformasi kode yang tepat guna mengeliminasi inefisiensi yang telah diidentifikasi pada tahap pertama, sebagaimana dirumuskan pada Subbab 2.3.2. Strategi optimasi yang dirancang menggabungkan restrukturisasi tingkat arsitektur penyimpanan sementara (caching) dan optimasi pada lapisan akses data Object-Relational Mapping (ORM). Pada lapisan caching, dirancang penerapan Redis Cache-Aside berbasis Redis 7.x untuk menggantikan penyimpanan snapshot di tabel basis data. Melalui strategi ini, seluruh hasil kalkulasi DSS pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data disimpan langsung ke dalam memori Redis, sehingga sistem dapat langsung mengembalikan data dari memori tanpa perlu melakukan kueri ulang ke basis data PostgreSQL. Pada lapisan ORM, transformasi dirancang melalui penerapan selective eager loading (->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])) guna membatasi atribut kolom yang ditarik sekaligus mengeliminasi kueri N+1 pada checklists, serta merestrukturisasi algoritma kalkulasi getWorkloadDistribution() dari query-in-loop menjadi penarikan data secara batch satu kali yang dilanjutkan dengan agregasi di dalam memori (in-memory aggregation). Rincian rancangan arsitektur dan alur kerja optimasi ini disajikan lebih lanjut pada Subbab 3.4. 

3.2.3. Penjaminan Preservasi Perilaku (Guarantee Behavior Preservation) Tahap Penjaminan Preservasi Perilaku dilakukan untuk memastikan secara mutlak bahwa seluruh restrukturisasi internal kode yang dirancang tidak mengubah perilaku eksternal, fungsionalitas, maupun semantik sistem (behavior-preserving), sejalan dengan prinsip siklus refaktorisasi Mens dan Tourwé (2004) serta konsep Self-Testing Code dari Fowler (2018). Sebelum transformasi kode diterapkan secara luas, disusun rangkaian pengujian regresi otomatis (automated regression/equivalence tests) sebagaimana dijelaskan pada Subbab 2.3.3. Pengujian ini mengevaluasi dan membandingkan luaran payload JSON dari kalkulasi 22 metrik performa tim dan 5 komponen rekomendasi manajerial DSS antara kondisi sebelum dan sesudah refaktorisasi. Luaran sistem dinyatakan lolos tahap ini apabila seluruh nilai numerik dan struktur data respons terbukti menghasilkan nilai yang identik 100%, sehingga membuktikan bahwa optimasi kueri dan penerapan Redis tidak merusak validitas formula kalkulasi DSS. 

3.2.4. Penerapan Refaktorisasi 

Tahap Penerapan Refaktorisasi merupakan eksekusi aktual transformasi kode program pada modul DSS Koladi sesuai dengan rancangan strategi yang telah ditetapkan, sebagaimana dirumuskan pada Subbab 2.3.4. Transformasi kode dieksekusi pada basis kode Laravel 12 dengan memisahkan aktivitas refaktorisasi dari penambahan fitur baru sesuai metafora "The Two Hats" dari Fowler (2018). Penerapan kode mencakup refaktorisasi metode getTasksInPeriod(), getUrgentTasks(), dan getWorkloadDistribution() pada lapisan servis dan controller, pengaktifan driver Redis 7 sebagai cache store utama melalui integrasi servis SnapshotManager, serta restrukturisasi query-in-loop menjadi penarikan data secara batch satu kali dengan agregasi di dalam memori (in-memory aggregation). Seluruh perubahan kode dieksekusi secara bertahap dan terisolasi, serta diverifikasi kembali menggunakan pengujian ekuivalensi fungsional sebelum sistem dialihkan ke lingkungan pengujian performa. 

3.2.5. Penilaian Efek Kualitas  

Tahap Penilaian Efek Kualitas dilakukan untuk mengukur dan menilai dampak empiris dari refaktorisasi terhadap efisiensi kinerja sistem (software performance), mengacu pada tahapan evaluasi kualitas refaktorisasi menurut Mens dan Tourwé (2004) serta protokol Controlled Benchmarking (Henning & Hasselbring, 2022). Pengujian dilakukan secara sekuensial terisolasi di dalam jaringan kontainer Docker Compose (koladi_net) untuk membandingkan kondisi baseline dan optimized di bawah tiga skala dataset sintetis (Small, Medium, Large) yang dibangkitkan melalui seeder data, sebagaimana dirancang pada Subbab 2.4.1 dan 2.4.2. Pengumpulan data dijalankan menggunakan Node.js Benchmark Runner yang mengeksekusi 3 warm-up requests (dibuang) untuk mengeliminasi bias cold-start, dilanjutkan dengan 10 repetisi pengukuran valid pada kondisi steady-state, dengan pencatatan metrik internal dibantu oleh Middleware Instrumentasi Laravel. Hasil pengujian yang memuat 240 baris data observasi valid kemudian dianalisis menggunakan persentase perbaikan (Improvement Percentage) serta uji signifikansi Paired-Sample t-Test pada taraf α = 0.05 disertai evaluasi keterpaduan antara cache hit ratio dan SQL query count untuk membuktikan terjadinya full database bypass pada kondisi teroptimasi sebagaimana dirumuskan pada Subbab 2.4.4. 

3.2.6. Pemeliharaan Konsistensi Sistem (Maintain Consistency)  

Tahap Pemeliharaan Konsistensi Sistem dilakukan untuk memelihara keselarasan antara kode program yang telah direfaktorisasi dengan seluruh artefak rekayasa dan lingkungan eksekusi sistem (change propagation), sebagaimana dirumuskan pada Subbab 2.3.6. Kegiatan pada tahap ini mencakup tiga tindakan terintegrasi: pertama, memverifikasi stabilitas orkestrasi lingkungan kontainer Docker Compose (koladi_net) guna memastikan layanan Nginx, PHP-FPM, PostgreSQL 15, dan Redis 7 beroperasi secara harmonis; kedua, memverifikasi ketiadaan efek samping (side effects) pada modul manajemen tugas (Kanban) melalui pengujian regresi fungsional untuk memastikan bahwa modifikasi relasi model Eloquent dan dependensi servis tidak menimbulkan regresi atau mengganggu integritas operasi pembuatan tugas, pembaruan status, dan penugasan anggota; serta ketiga, memperbarui artefak nonkode berupa berkas konfigurasi deployment (.env.example), pembaruan diagram arsitektur sistem pada Subbab 3.4.1, serta penyelarasan dokumentasi teknis antarmuka API modul DSS Koladi. 

 

 

 

 

3.3. Tahapan Pengembangan Aplikasi 

Sistem Koladi yang digunakan dalam penelitian ini merupakan aplikasi collaborative virtual office berbasis Software as a Service (SaaS) yang telah berjalan dan menjadi objek studi kasus (brownfield software). Penelitian ini berfokus pada aktivitas rekayasa perangkat lunak (software engineering) berupa refaktorisasi basis kode (code refactoring) dan optimasi arsitektur penyimpanan sementara (in-memory caching) pada modul Decision Support System (DSS). Penelitian ini tidak mencakup pembangunan aplikasi dari awal (greenfield), penambahan fitur baru, perubahan alur bisnis, maupun perancangan ulang antarmuka pengguna (UI/UX). Oleh karena itu, metodologi pengembangan perangkat lunak konvensional yang ditujukan untuk pengembangan fitur baru dari awal seperti Waterfall, Agile/Scrum, maupun Rapid Application Development (RAD) tidak diterapkan sebagai metodologi utama penelitian. 

Meskipun tidak menerapkan metodologi SDLC konvensional, penelitian ini tetap merupakan aktivitas rekayasa perangkat lunak yang sistematis dan terukur pada sistem Koladi. Metodologi rekayasa yang diterapkan berlandaskan pada enam tahapan siklus hidup refaktorisasi (Refactoring Lifecycle) menurut Mens dan Tourwé (2004). Melalui metodologi ini, seluruh modifikasi internal pada kode program difokuskan secara murni untuk mengeliminasi inefisiensi kueri dan latensi komputasi, dengan batasan mutlak bahwa perilaku teramati (observable behavior) dan fungsi sistem tetap dipertahankan secara utuh (behavior-preserving) (Fowler, 2018; Mens & Tourwé, 2004). Hal ini dibuktikan melalui kesetaraan hasil kalkulasi 22 metrik performa tim dan rekomendasi manajerial DSS antara sebelum dan sesudah optimasi. 

Aktivitas rekayasa yang dieksekusi mencakup pemahaman arsitektur analitis DSS, profiling inefisiensi kueri dengan Laravel Telescope, perancangan integrasi Redis Cache-Aside, penerapan selective eager loading, serta restrukturisasi query-in-loop menjadi penarikan data secara batch dengan agregasi di dalam memori (in-memory aggregation). Dengan demikian, tahapan penelitian yang telah dirumuskan secara rinci pada Subbab 3.2 berfungsi sebagai metodologi dan alur utama pelaksanaan Tugas Akhir, sedangkan aplikasi Koladi berperan sebagai objek dan lingkungan studi kasus eksperimental. 

3.4. Rencana Perancangan Sistem 

Perancangan pada penelitian ini tidak berfokus pada pembangunan sistem aplikasi baru dari nol, melainkan pada penyiapan arsitektur optimasi caching, restrukturisasi kueri ORM, spesifikasi dataset sintetis, dan lingkungan eksperimen untuk modul Decision Support System (DSS) Koladi. Rancangan sistem meliputi gambaran umum sistem, alur komputasi modul DSS, rancangan dataset pengujian, rancangan transformasi kueri dan skema Redis, serta rancangan lingkungan pengujian. 

3.4.1 Gambaran Umum Sistem 

Koladi merupakan aplikasi perkantoran virtual kolaboratif (collaborative virtual office) berbasis web yang dibangun menggunakan Laravel sebagai framework aplikasi, PostgreSQL sebagai basis data relasional, dan Redis sebagai media in-memory caching. Sistem mengadopsi arsitektur Software as a Service (SaaS) multi-tenant dengan pemisahan data berdasarkan konteks workspace masing-masing. Alur akses data pada modul Decision Support System (DSS) yang menjadi fokus penelitian digambarkan pada Gambar 2. 

| Komponen / Entitas | Deskripsi / Peran | Keterangan Alur |
| :--- | :--- | :--- |
| **User** | Pengguna Sistem[cite: 2] | Mengirimkan permintaan (*request*) ke antarmuka aplikasi[cite: 2] |
| **Aplikasi Koladi** | Laravel 12[cite: 2] | Menerima dan meneruskan *request* ke modul analitis[cite: 2] |
| **Modul DSS** | `DSSService` & `SnapshotManager`[cite: 2] | Mengelola logika kalkulasi keputusan dan alur *caching*[cite: 2] |
| **Redis 7** | *In-Memory Cache* (Cache-Aside)[cite: 2] | Menyimpan dan memeriksa ketersediaan data *cache*[cite: 2] |
| **PostgreSQL 15** | Eloquent ORM (`Tasks`, `Checklists`)[cite: 2] | Melayani persistensi data relasional (*read/write*) dua arah[cite: 2] |

Gambar 2. Gambaran Umum Sistem dan Lingkungan Penelitian 

Pada penelitian ini, fokus optimasi berada pada lapisan arsitektur penyimpanan sementara (caching) dan akses data Object-Relational Mapping (ORM) pada modul DSS. Source code Laravel digunakan sebagai target refaktorisasi kueri Eloquent, Redis digunakan untuk melayani permintaan data analitis secara instan dari memori RAM saat terjadi cache hit, sedangkan basis data PostgreSQL berfungsi sebagai media penyimpanan persisten tempat data relasional ditarik melalui kueri SQL saat terjadi cache miss. Seluruh interaksi komponen tersebut dijalankan di dalam jaringan kontainer Docker Compose (koladi_net) sebagai lingkungan eksperimen terkontrol untuk mengukur efektivitas optimasi terhadap response time, jumlah kueri SQL, alokasi memori, dan cache hit ratio


3.4.2. Rancangan Alur Komputasi Modul DSS dan Cache-Aside  

Modul Decision Support System (DSS) pada aplikasi Koladi bertugas memproses data aktivitas kerja tim yang saling berelasi menjadi indikator kinerja dan rekomendasi manajerial yang dapat ditindaklanjuti secara tepat waktu sebagaimana dikonseptualisasikan pada Subbab 2.2.1. Layanan analitis ini disajikan secara spesifik melalui dua endpoint utama, yaitu GET /statistik/suggestions untuk menyajikan rekomendasi berbasis aturan evaluasi performa kerja tim, serta GET /statistik/modal-data untuk menyajikan payload analitis komprehensif yang memuat ringkasan metrik utama, daftar tugas mendesak, dan sebaran beban kerja anggota tim. Di lapisan backend, alur komputasi analitis menggabungkan dua pilar rekayasa utama: orkestrasi kalkulasi analitis melalui Eloquent ORM pada DSSService, serta kontrol alur pembacaan data (read path) melalui pola Redis Cache-Aside pada SnapshotManager. DSSService mengoordinasikan empat komponen kalkulasi inti: MetricsCalculator (menghitung 22 metrik produktivitas mentah dari data tugas dan relasi checklists), TrendAnalyzer (menganalisis komparasi fluktuasi kinerja antar-periode), RecommendationEngine (mengklasifikasikan status performa dan merumuskan saran tindakan manajerial), serta fungsi pendukung controller getUrgentTasks() dan getWorkloadDistribution(). Berbeda dengan konfigurasi awal (baseline) yang menyimpan metrik ke dalam tabel relasional PostgreSQL (workspace_performance_snapshots) dan mengeksekusi kueri ORM yang memicu N+1 query serta loop kueri per anggota sebagaimana diidentifikasi pada Subbab 3.2.1, rancangan arsitektur teroptimasi mengintegrasikan Redis 7 sebagai cache store dan merefaktorisasi akses data Eloquent ORM menjadi selective eager loading serta penarikan batch satu kali, sejalan dengan strategi yang dirumuskan pada Subbab 2.3.2 dan Subbab 3.2.2. Interaksi sekuensial antar-komponen dalam melayani permintaan data analitis DSS tersebut dimodelkan pada Gambar 3. 

sequenceDiagram
    autonumber
    actor User as Pengguna (Browser)
    participant Ctrl as StatistikController
    participant SM as SnapshotManager
    participant Cache as Redis 7 (In-Memory)
    participant DSS as DSSService & Pipeline
    participant ORM as Eloquent ORM (Task & Checklist)
    participant DB as PostgreSQL 15
    User->>Ctrl: HTTP GET /suggestions atau /modal-data
    Ctrl->>SM: Cek ketersediaan cache analitis workspace
    SM->>Cache: Periksa kunci cache (GET dss:workspace:{id}:{endpoint})
    
    alt Kondisi Cache Hit (Data Tersedia di RAM Redis)
        Cache-->>SM: Kembalikan payload JSON utuh
        SM-->>Ctrl: Payload analitis siap saji
        Ctrl-->>User: HTTP 200 OK (0 Kueri Komputasi DSS)
    else Kondisi Cache Miss (Data Belum Tersedia di Redis)
        Cache-->>SM: Nilai kosong (Null / Cache Miss)
        SM->>DSS: Picu pipeline kalkulasi analitis
        
        Note over DSS,DB: Alur Eksekusi Kueri Eloquent ORM Teroptimasi
        DSS->>ORM: getTasksInPeriod() via Task::with(['assignedUsers:id', 'checklists:id,task_id,is_done'])
        ORM->>DB: Eksekusi SQL Tasks & Checklists (Batch Eager Loading)
        DB-->>ORM: Kembalikan result set data relasional
        ORM-->>DSS: Hydrate ke Collection Model (Atribut Terbatas)
        
        DSS->>DSS: MetricsCalculator hitung 22 metrik produktivitas
        DSS->>DSS: TrendAnalyzer & RecommendationEngine evaluasi aturan
        
        opt Khusus Endpoint /modal-data (Kalkulasi Beban Kerja & Urgent Tasks)
            DSS->>ORM: Penarikan batch data penugasan anggota (1 kali kueri)
            ORM->>DB: Eksekusi SQL Batch Tugas & Anggota
            DB-->>ORM: Kembalikan dataset tugas aktif
            ORM-->>DSS: Collection data penugasan
            DSS->>DSS: Agregasi in-memory distribusi beban kerja & filter urgent tasks
        end
        
        DSS->>SM: Kembalikan payload analitis lengkap
        SM->>Cache: Simpan payload utuh ke Redis (SETEX dss:key)
        SM-->>Ctrl: Diteruskan ke controller
        Ctrl-->>User: HTTP 200 OK (Respons Tersaji Lengkap)
    end

    Gambar 3. Diagram Sekuensial Alur Komputasi Modul DSS, Pola Cache-Aside, dan Eksekusi Eloquent ORM 

Berdasarkan rancangan alur pada Gambar 3, setiap permintaan analitis yang diterima oleh StatistikController terlebih dahulu diarahkan ke SnapshotManager untuk memeriksa ketersediaan data pada lapisan in-memory Redis 7. Pada kondisi cache hit, sistem langsung mengambil payload JSON dari memori Redis dan mengembalikannya ke pengguna tanpa perlu mengeksekusi pipeline komputasi DSS maupun mengakses PostgreSQL 15, sehingga kueri yang dieksekusi untuk komputasi analitis terpangkas menjadi 0 kueri (full database bypass). Pemanfaatan memori RAM yang memiliki latensi jauh lebih rendah dibandingkan pembacaan disk basis data (Taipalus, 2024; Privalov & Stupina, 2024 sebagaimana diulas pada Subbab 2.2.2) memungkinkan response time dipangkas secara drastis serta tetap stabil tanpa terpengaruh oleh besarnya volume data kerja tim. 

Sebaliknya, pada kondisi cache miss, sistem mengeksekusi penarikan data ke PostgreSQL 15 melalui kueri Eloquent yang telah direfaktorisasi menggunakan teknik selective eager loading (->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])). Pembatasan atribut kolom pada assignedUsers menekan pemborosan alokasi memori akibat materialisasi objek model yang berlebihan (Yusmita et al., 2025), sedangkan pemuatan relasi checklists secara eager loading mengeliminasi antipola N+1 query pada pemanggilan getProgressPercentage() di setiap tugas (Turcotte et al., 2022). Setelah objek tugas dimuat, MetricsCalculator menghitung 22 metrik produktivitas mentah yang kemudian dianalisis oleh TrendAnalyzer dan RecommendationEngine untuk merumuskan saran manajerial. Khusus pada endpoint modal-data, fungsi getWorkloadDistribution() yang semula mengeksekusi kueri Task::whereHas() berulang di dalam perulangan anggota ditransformasikan menjadi penarikan data penugasan secara batch satu kali yang dilanjutkan dengan agregasi langsung di dalam memori server (in-memory aggregation). 

Keunggulan utama dari perancangan alur komputasi ini dibandingkan mekanisme baseline adalah penyelesaian masalah partial cache hit pada endpoint GET /statistik/modal-data. Pada kondisi baseline, mekanisme snapshot relasional hanya menampung indikator metrik utama, sementara komputasi tugas mendesak dan distribusi beban kerja tetap dieksekusi ulang ke basis data pada setiap permintaan. Melalui rancangan alur Cache-Aside yang terintegrasi ini, SnapshotManager mengonsolidasikan seluruh struktur data respons analitis, mencakup metrik utama, 22 metrik performa, daftar urgent tasks, serta sebaran beban kerja hasil agregasi in-memory ke dalam satu payload utuh yang disimpan ke Redis menggunakan perintah SETEX. Dengan demikian, permintaan identik berikutnya dijamin terlayani sepenuhnya dari lapisan memori cepat tanpa menyisakan beban kueri komputasi pada basis data PostgreSQL 15. Penjelasan teknis mengenai perbandingan sintaks kueri ORM sebelum dan sesudah refaktorisasi serta struktur skema kunci Redis disajikan lebih lanjut pada Subbab 3.4.4. 

 

3.4.3. Rancangan Dataset Pengujian  

Pengujian efisiensi kinerja pada modul Decision Support System (DSS) Koladi memerlukan perancangan dataset pengujian terkontrol guna mengevaluasi beban pemrosesan analitis pada aplikasi perkantoran virtual kolaboratif (collaborative virtual office). Seluruh dataset yang digunakan dalam penelitian ini merupakan data sintetis terstruktur (seeded dataset) yang dibangkitkan secara otomatis menggunakan skrip seeder dan factory Laravel berbasis pustaka Faker, bukan berasal dari rekaman log produksi riil. Pendekatan data sintetis ini diadopsi mengikuti metodologi eksperimen perangkat lunak terkontrol (controlled software experiment), di mana data produksi riil umumnya memiliki distribusi relasi yang acak, tidak seimbang, dan dinamis sehingga rentan menjadi variabel perancu (confounding variables) yang merusak validitas pengukuran (Yusmita et al., 2025). Penggunaan skrip seeder memberikan kendali penuh terhadap volume entitas, proporsi status tugas, serta kepadatan relasi tugas-anggota secara deterministik. Karakteristik deterministik ini menjamin keterulangan pengujian (bit-for-bit reproducibility) sekaligus memastikan bahwa konfigurasi baseline dan optimized dievaluasi pada kondisi data yang persis sama (Sasmoko et al., 2024; Yusmita et al., 2025). 

Dalam perancangan eksperimen, volume data disusun ke dalam tiga skenario bertingkat guna menganalisis kecenderungan perubahan performa dan stabilitas sistem seiring dengan pertumbuhan volume entitas relasional yang diproses dalam satu siklus request terkontrol (Wohlin et al., 2000; Henning & Hasselbring, 2022). Pengujian pada volume tunggal berisiko menghasilkan kesimpulan semu yang hanya berlaku pada ukuran data tertentu, sedangkan variasi bertingkat memungkinkan pembuktian empiris apakah optimasi yang diterapkan mampu menahan degradasi performa ketika volume data membesar (Henning & Hasselbring, 2022; Yusmita et al., 2025). 

Penetapan skala dataset pada penelitian ini disesuaikan dengan unit analisis modul DSS Koladi yang beroperasi pada lingkup per-workspace, bukan kueri agregat terhadap keseluruhan tabel basis data aplikasi. Tiga skala volume data dirancang untuk menyimulasikan tingkat kepadatan data yang berbeda pada sebuah workspace: 

Skala Small (50 tugas dan 5 anggota): Menyimulasikan tingkat kepadatan data rendah dengan rasio rata-rata 10 tugas per anggota. 

Skala Medium (200 tugas dan 15 anggota): Menyimulasikan tingkat kepadatan data moderat dengan rasio rata-rata 13 hingga 14 tugas per anggota. 

Skala Large (500 tugas dan 30 anggota): Menyimulasikan tingkat kepadatan data tinggi dengan rasio rata-rata 16 hingga 17 tugas per anggota. 

Variasi bertingkat pada jumlah tugas dan anggota tersebut dirancang secara terukur sebagai pemicu beban (workload stressor) untuk mengevaluasi dua titik inefisiensi kueri Eloquent ORM yang telah diidentifikasi pada Subbab 2.2.3 dan Subbab 3.2.1. Peningkatan jumlah tugas (50, 200, dan 500) dirancang untuk memicu eskalasi antipola N+1 query pada relasi checklists saat kalkulasi progres tugas dieksekusi, sejalan dengan prinsip Turcotte et al. (2022) bahwa inefisiensi lazy loading melipatgandakan kueri basis data secara linier terhadap jumlah entitas yang diproses. Skala 500 tugas ditetapkan sebagai batas atas pengujian karena pada konfigurasi baseline telah memicu hingga 696 kueri SQL dalam satu siklus permintaan HTTP (dengan waktu respons mencapai 679 ms), sehingga beban komputasi tersebut telah memadai untuk mengamati batas degradasi performa sebelum melampaui alokasi memori runtime PHP pada lingkungan lokal. Sementara itu, peningkatan jumlah anggota workspace (5, 15, dan 30 anggota) dirancang berdasarkan jumlah iterasi perulangan untuk menguji eliminasi antipola kueri di dalam perulangan (query-in-loop) pada fungsi getWorkloadDistribution(). Sebagaimana disurvei oleh Alshemaimri et al. (2021) mengenai fragmen kode basis data bermasalah (problematic database code fragments), pengeksekusian kueri di dalam iterasi memicu pertumbuhan beban kueri secara linier yang berbanding lurus dengan banyaknya entitas anggota yang diiterasi. Dengan mengevaluasi rentang 5 hingga 30 anggota, penelitian ini membuktikan secara empiris bahwa transformasi algoritma menjadi penarikan batch satu kali dengan agregasi di dalam memori (in-memory aggregation) berhasil memangkas eksekusi kueri menjadi bernilai tetap atau konstan, tanpa lagi dipengaruhi oleh penambahan jumlah anggota tim. Rincian spesifikasi ketiga skenario dataset pengujian disajikan pada Tabel 5. 

Tabel 5. Spesifikasi Rancangan Dataset Pengujian 

| Skenario Dataset | Jumlah Tugas | Jumlah Anggota | Distribusi Status Tugas |
|---|---|---|---|
| Small | 50 tugas | 5 pengguna | 40% Completed (20 tugas)<br>35% In Progress (17 tugas)<br>25% To-Do (13 tugas) |
| Medium | 200 tugas | 15 pengguna | 40% Completed (80 tugas)<br>35% In Progress (70 tugas)<br>25% To-Do (50 tugas) |
| Large | 500 tugas | 30 pengguna | 40% Completed (200 tugas)<br>35% In Progress (175 tugas)<br>25% To-Do (125 tugas) |



Selain volume data, proporsi status tugas di dalam setiap skenario dikunci secara konsisten pada rasio 40% selesai (completed), 35% dalam proses (in progress), dan 25% belum selesai (to-do), serta dilengkapi subset tugas yang melewati batas waktu (overdue). Pengaturan proporsi ini diperlukan secara fungsional agar seluruh formula dari 22 metrik produktivitas (seperti completion rate, backlog ratio, dan delay rate) serta aturan rekomendasi manajerial pada modul DSS dapat tereksekusi secara valid tanpa menghasilkan nilai pembagian nol atau luaran numerik yang trivial. 

Dataset yang digunakan pada pengujian konfigurasi baseline dan optimized dibuat 100% identik pada setiap skenario volume data. Dengan menyeragamkan volume, relasi, dan parameter filter data antara kedua konfigurasi, variasi hasil pengukuran pada empat metrik performa (response time, SQL query count, memory utilization, dan cache hit ratio) dapat diisolasi dan diatribusikan secara sahih sebagai dampak langsung dari intervensi arsitektur Redis Cache-Aside dan optimasi kueri ORM, bukan akibat fluktuasi struktur data (Wohlin et al., 2000; Yusmita et al., 2025), sekaligus memenuhi asumsi dependensi data berpasangan untuk pengujian statistik paired-sample t-test (Uddin & Lu, 2024). 

 

3.4.4. Rancangan Transformasi Kueri ORM dan Skema Cache Redis  

Perancangan optimasi backend pada modul Decision Support System (DSS) Koladi berfokus pada dua pilar rekayasa utama: restrukturisasi kueri pada lapisan Object-Relational Mapping (ORM) Eloquent serta perancangan skema penyimpanan data sementara pada in-memory cache Redis 7. Sebagaimana telah dimodelkan dalam alur komputasi pada Subbab 3.4.2, efisiensi penarikan data ke basis data PostgreSQL 15 menjadi penentu utama latensi sistem ketika terjadi cache miss, sedangkan efektivitas skema penyimpanan di dalam Redis menentukan apakah permintaan berulang dapat dilayani sepenuhnya dari memori cepat (full database bypass) tanpa menyisakan beban kueri relasional. 

3.4.4.1. Rancangan Transformasi Kueri Eloquent ORM 

Transformasi kueri difokuskan pada dua metode komputasi analitis yang sebelumnya memicu inefisiensi kueri basis data, sebagaimana diilustrasikan pada Gambar 4. 

<?php

// ==========================================
// KASUS 1: Refactoring getTasksInPeriod() 
// Lokasi: app/Services/DSS/MetricsCalculator.php
// ==========================================


// Before: getTasksInPeriod()
->with(['assignedUsers'])

// After: getTasksInPeriod()
->with(['assignedUsers:id', 'checklists:id,task_id,is_done'])

// ==========================================
// KASUS 2: Refactoring getWorkloadDistribution()
// Lokasi: app/Services/DSS/StatistikController.php
// ==========================================


// Before: getWorkloadDistribution()
foreach ($members as $member) {
    Task::whereHas('assignedUsers', ...)->get();
}

// After: getWorkloadDistribution()
$tasks = Task::with(['assignedUsers:id'])->where(...)->get();

foreach ($members as $member) {
    $memberTasks = $tasks->filter(...);
}


Pada metode getTasksInPeriod(), konfigurasi baseline mengeksekusi eager loading tanpa pembatasan kolom (->with(['assignedUsers'])) serta mengabaikan relasi checklists. Pola ini memicu antipola Unnecessary Data Retrieval yang memboroskan memori PHP (Alshemaimri et al., 2021; Yusmita et al., 2025), sekaligus menimbulkan masalah N+1 kueri saat kalkulasi progres memanggil relasi checklists secara lazy loading (Turcotte et al., 2022). Melalui refaktorisasi, sintaks ditransformasikan menjadi ->with(['assignedUsers:id', 'checklists:id,task_id,is_done']). Pembatasan kolom menekan beban serialisasi objek pada RAM, sedangkan eager loading selektif mengompresi ratusan kueri terpisah pada relasi checklists menjadi 1 kueri tunggal yang bernilai konstan. 

Sementara itu, inefisiensi pada metode getWorkloadDistribution() terjadi akibat penempatan kueri kondisional Task::whereHas() di dalam blok perulangan anggota workspace (foreach ($members)). Pola query-in-loop ini menyebabkan volume kueri membengkak secara linier seiring penambahan jumlah anggota tim (Alshemaimri et al., 2021). Refaktorisasi dilakukan dengan menarik seluruh data tugas aktif secara batch di luar blok perulangan menggunakan Task::with(['assignedUsers:id'])->where(...)->get(). Selanjutnya, pembagian beban kerja per anggota diproses sepenuhnya di dalam memori PHP menggunakan operasi filter koleksi $tasks->filter(...). Melalui pendekatan in-memory aggregation ini, eksekusi ke basis data PostgreSQL tereduksi menjadi 1 kueri konstan yang sepenuhnya independen dari penambahan jumlah anggota ruang kerja. 

3.4.4.2. Rancangan Skema Cache Redis 

Sebagai pengganti mekanisme snapshot relasional baseline yang masih memicu pembacaan ke PostgreSQL, perancangan lapisan caching memanfaatkan Redis 7.x dengan driver phpredis melalui Laravel Redis facade. Struktur penyimpanan dirancang untuk menjamin isolasi data multi-tenant serta menyelesaikan masalah partial cache hit, sebagaimana disajikan pada Tabel 5. 

Tabel 5. Spesifikasi Skema Kunci dan Struktur Penyimpanan Redis Cache-Aside 





Berdasarkan rancangan pada Tabel 5, penamaan kunci Redis mengadopsi struktur hierarkis berpemisah titik dua (:) dengan menyertakan pengenal unik ruang kerja (workspace_{workspace_id}) serta rentang tanggal evaluasi guna menjamin isolasi data multi-tenant antar-workspace. Setiap respons analitis dibungkus secara utuh ke dalam format teks JSON dan disimpan menggunakan perintah SETEX berdurasi 3.600 detik (1 jam) sebagai batas retensi memori RAM server. 

Pendekatan penyimpanan payload utuh pada kedua endpoint ini memastikan eliminasi kueri komputasi analitis secara penuh saat terjadi cache hit. Khusus pada endpoint /modal-data, penyimpanan seluruh komponen analitis—termasuk daftar tugas mendesak (urgent tasks) dan sebaran beban kerja tim (workload distribution)—ke dalam Redis menuntaskan masalah partial cache hit yang terjadi pada konfigurasi baseline, di mana kedua komponen tersebut sebelumnya selalu memicu komputasi ulang ke PostgreSQL. 

 

 

 

 

 

3.4.5. Rancangan Lingkungan Pengujian dan Instrumentasi  

Pengujian performa pada modul DSS Koladi dilaksanakan pada lingkungan komputasi lokal yang terisolasi dan terkendali guna menjamin validitas internal serta keterulangan (reproducibility) eksperimen. Berbeda dari pengujian basis data konvensional yang mengeksekusi kueri secara langsung pada DBMS, pengujian pada penelitian ini mengevaluasi siklus permintaan-respons HTTP secara menyeluruh (end-to-end) pada dua endpoint analitis. Untuk mendukung tujuan tersebut, rancangan lingkungan pengujian dibangun di atas dua komponen utama: lingkungan terkontainerisasi berbasis Docker dan sistem instrumentasi pengukuran dua arah (dual-layer instrumentation). 

Topologi lingkungan pengujian dan interaksi alur instrumentasi disajikan pada Gambar 5. 

 
| Aspek Teknis | Endpoint Rekomendasi (/suggestions) | Endpoint Modal Analitis (/modal-data) |
|---|---|---|
| Key Pattern | dss:suggestions:workspace_{workspace_id}:start_{start_date}:end_{end_date} | dss:modal-data:workspace_{workspace_id}:start_{start_date}:end_{end_date} |
| Struktur Data Redis | JSON String | JSON String |
| Penyimpanan | SETEX (TTL: 3.600 detik / 1 jam) | SETEX (TTL: 3.600 detik / 1 jam) |
| Data yang Disimpan | Seluruh data kartu rekomendasi (skor performa, metrik utama, daftar saran DSS) | Seluruh data pop-up modal (metrik lengkap, tren, daftar urgent tasks, sebaran workload) |
| Dampak Eliminasi Kueri | 0 kueri komputasi analitis DSS saat cache hit | 0 kueri komputasi analitis DSS saat cache hit (eliminasi partial cache hit) |
 

Gambar 5. Topologi Lingkungan Pengujian Terkontainerisasi dan Alur Instrumentasi Pengukuran 

Berdasarkan topologi pada Gambar 5, lingkungan pengujian diisolasi di dalam jaringan privat Docker (bridge network) bernama koladi_net. Isolasi ini mencakup tiga layanan utama: kontainer web dan aplikasi (koladi_app_local dan koladi_nginx_local) yang menjalankan PHP 8.2 dan Laravel 12, kontainer basis data (koladi_db_local) yang menjalankan PostgreSQL 15 untuk persistensi seeded dataset, serta kontainer cache (koladi_redis_local) yang menjalankan Redis 7.x untuk lapisan penyimpanan in-memory. 

Sistem instrumentasi pengukuran dirancang secara khusus untuk mengekstraksi empat metrik kinerja secara objektif tanpa menimbulkan interferensi yang mendistorsi waktu eksekusi alami sistem: 

1. Instrumentasi Sisi Klien (Client-Side Benchmark Runner): Diimplementasikan melalui skrip Node.js (scripts/benchmark-dss.js) yang bertindak sebagai pemanggil independen dari host OS. Skrip ini mengorkestrasi pengujian secara sekuensial terkontrol dengan menyertakan header X-Benchmark-Mode: 1. Waktu respons (response time) dihitung secara presisi sejak byte pertama permintaan dikirim hingga seluruh payload JSON diterima menggunakan fungsi process.hrtime.bigint() berpresisi nanodetik. Skrip ini juga mengontrol fase 3 kali warm-up (dibuang dari analisis) dan 10 kali repetisi valid, serta melakukan pembersihan kunci Redis per-workspace sebelum repetisi kondisi cache miss dijalankan. 

2. Instrumentasi Sisi Server (Server-Side Middleware): Diimplementasikan melalui middleware Laravel (app/Http/Middleware/BenchmarkInstrumentation.php) yang aktif secara kondisional saat mendeteksi header pengujian. Middleware ini memanfaatkan fasilitas DB::listen untuk mencatat jumlah kueri SQL yang dieksekusi selama pemrosesan, fungsi memory_get_peak_usage(true) untuk merekam penggunaan RAM puncak mesin virtual PHP, serta membaca statistik keyspace hits/misses dari Redis untuk menentukan cache hit ratio. Nilai metrik internal ini kemudian disuntikkan ke dalam header respons HTTP (X-Debug-Query-Count, X-Debug-Memory-MB, X-Debug-Cache-Hit, dan X-Debug-Cache-Hit-Ratio) agar dapat terekam langsung oleh skrip klien ke dalam berkas CSV. 

Agar seluruh rangkaian pengujian bersifat deterministik dan dapat direplikasi, spesifikasi perangkat keras dan perangkat lunak pengujian ditetapkan secara statis sebagaimana dirinci pada Tabel 6. 

 | Aspek Teknis | Endpoint Rekomendasi (/suggestions) | Endpoint Modal Analitis (/modal-data) |
|---|---|---|
| Key Pattern | dss:suggestions:workspace_{workspace_id}:start_{start_date}:end_{end_date} | dss:modal-data:workspace_{workspace_id}:start_{start_date}:end_{end_date} |
| Struktur Data Redis | JSON String | JSON String |
| Penyimpanan | SETEX (TTL: 3.600 detik / 1 jam) | SETEX (TTL: 3.600 detik / 1 jam) |
| Data yang Disimpan | Seluruh data kartu rekomendasi (skor performa, metrik utama, daftar saran DSS) | Seluruh data pop-up modal (metrik lengkap, tren, daftar urgent tasks, sebaran workload) |
| Dampak Eliminasi Kueri | 0 kueri komputasi analitis DSS saat cache hit | 0 kueri komputasi analitis DSS saat cache hit (eliminasi partial cache hit) |

Tabel 6. Spesifikasi Lingkungan Pengujian Perangkat Keras dan Perangkat Lunak 




Selama seluruh rangkaian pengujian baseline dan optimized berlangsung, spesifikasi pada Tabel 6 dijaga konstan tanpa perubahan konfigurasi. Selain itu, pengujian dijalankan dengan jeda waktu istirahat antar-skenario guna memastikan kondisi sistem kembali stabil (idle state). Dengan protokol ini, variasi pada hasil pengukuran dapat dipastikan murni bersumber dari perbedaan perlakuan optimasi modul DSS, bukan akibat fluktuasi lingkungan komputasi. 

3.5. Rencana Analisis  

Rencana analisis ini mengolah hasil pengujian baseline dan optimized yang diperoleh pada tahap Penilaian Efek Kualitas (Subbab 3.2.5), menggunakan variabel penelitian, parameter, instrumen pengukuran, formula, dan kriteria evaluasi yang telah dijelaskan pada Subbab 2.2 dan Subbab 2.4. 

3.5.1. Analisis Baseline 

Dua endpoint DSS (/suggestions dan /modal-data) dijalankan pada tiga skenario volume data sintetis (Small, Medium, Large) sebelum optimasi diterapkan, mencakup kondisi Cache Miss dan Cache Hit. Sebagaimana dirumuskan pada Subbab 2.4.3, setiap kombinasi perlakuan diawali dengan tiga kali warm-up requests (dibuang dari analisis) dan sepuluh kali repetisi pengukuran valid pada kondisi stabil (steady-state), menghasilkan total 120 data observasi baseline. Hasil yang dicatat meliputi nilai Response Time, SQL Query Count, Memory Utilization, dan Cache Hit Ratio yang diolah menggunakan statistik deskriptif (Mean, Standard Deviation, Min, Max) sesuai Subbab 2.4.4, sebagai kondisi awal pembanding terhadap kondisi optimized. 

3.5.2. Analisis Optimized 

Setelah strategi Redis Cache-Aside dan optimasi kueri Eloquent ORM diterapkan pada modul DSS Koladi, kedua endpoint diuji kembali menggunakan dataset sintetis, skenario volume data, kondisi cache, dan parameter request yang sama persis dengan kondisi baseline untuk menjaga keterbandingan hasil. Sebagaimana pada kondisi baseline, setiap query dan endpoint dijalankan dengan tiga kali warm-up dan sepuluh kali pengukuran valid mengikuti Subbab 2.4.3, menghasilkan 120 data observasi optimized. Parameter yang dicatat identik dengan kondisi baseline dan diolah menggunakan statistik deskriptif untuk mengamati stabilitas serta konsistensi performa sistem setelah refaktorisasi diterapkan. 

3.5.3. Analisis Efisiensi Kueri SQL dan Utilisasi Memori 

Efisiensi penarikan data Eloquent ORM dianalisis dengan membandingkan jumlah kueri SQL (SQL Query Count) dan penggunaan memori (Memory Utilization) antara kondisi baseline dan optimized, terutama pada kondisi Cache Miss saat modul DSS memproses data secara penuh. 

Pengurangan jumlah kueri SQL dianalisis untuk melihat keberhasilan eliminasi masalah N+1 pada relasi checklists dan kueri berulang di dalam perulangan anggota tim (Subbab 2.2.3). Sementara itu, penggunaan memori dianalisis menggunakan persentase perbaikan (Improvement %) sesuai Subbab 2.4.4, untuk melihat seberapa besar penghematan RAM server setelah pengambilan kolom dibatasi hanya pada data yang diperlukan. 

3.5.4. Analisis Response Time dan Skalabilitas Kinerja 

Response Time dianalisis dengan membandingkan durasi latensi end-to-end HTTP antara kondisi baseline dan optimized menggunakan formula Improvement (%) sesuai Subbab 2.4.4. Analisis dipisahkan antara kondisi Cache Miss (untuk mengevaluasi efisiensi pipeline komputasi ORM teroptimasi) dan kondisi Cache Hit (untuk memvalidasi percepatan retrieval data dari memori RAM Redis dibandingkan pembacaan diska PostgreSQL sesuai keunggulan latensi memori pada Subbab 2.2.2). Selain itu, tren Response Time dianalisis di bawah pertumbuhan volume dataset dari Small (50 tugas/5 anggota), Medium (200 tugas/15 anggota), hingga Large (500 tugas/30 anggota) untuk menguji skalabilitas sistem dan membuktikan apakah optimasi mampu menjaga responsivitas modul DSS berbasis web tetap stabil dari degradasi latensi saat data bertumbuh (Subbab 2.2.1). 

3.5.5. Analisis Keterpaduan Cache Hit dan Akses Basis Data 

Hasil Cache Hit Ratio dan SQL Query Count dianalisis secara bersamaan menggunakan pendekatan Joint Cache Analysis (Subbab 2.4.4). Pada konfigurasi optimized, Cache Hit Ratio mengukur keberhasilan pengambilan data langsung dari memori Redis. Analisis keterpaduan ini dilakukan untuk memastikan apakah kondisi Cache Hit benar-benar membebaskan basis data dari pemanggilan kueri SQL (mencapai 0 kueri komputasi). Selain itu, analisis ini membuktikan terselesaikannya masalah partial cache hit pada endpoint /modal-data, di mana seluruh data analitis (termasuk tugas mendesak dan sebaran beban kerja) kini tersaji utuh dari memori Redis tanpa menyisakan kueri tambahan ke PostgreSQL (Subbab 2.2.4). 

3.5.6. Interpretasi Hasil dan Uji Hipotesis 

Hasil baseline dan optimized dibandingkan pada setiap skenario pengujian dengan mempertimbangkan Response Time, SQL Query Count, Memory Utilization, dan Cache Hit Ratio untuk menentukan efektivitas optimasi pada modul DSS Koladi. Uji Paired-Sample t-Test pada taraf signifikansi α = 0.05 digunakan untuk memastikan apakah perbedaan performa yang diperoleh terbukti signifikan secara statistik (Subbab 2.4.4). 

Kesimpulan dari penelitian ini dibatasi pada lingkungan pengujian dan jalur eksekusi modul DSS Koladi (endpoint /suggestions dan /modal-data), sehingga tidak digeneralisasikan secara langsung ke seluruh fitur aplikasi Koladi maupun sistem SaaS lainnya (Subbab 2.2.4). 
