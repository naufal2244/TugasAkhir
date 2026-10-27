IMPLEMENTASI WORKLOAD-BASED INDEX OPTIMIZATION UNTUK MENINGKATKAN PERFORMA QUERY PADA SISTEM SAAS MULTI-TENANT: STUDI KASUS KOLADI

KATA PENGANTAR 

Alhamdulillahirabbil'alamin, segala puji dan syukur penulis ucapkan kepada Allah SWT. atas segala rahmat dan karunia-Nya, sehingga penyusunan proposal Tugas Akhir yang berjudul “Implementasi Workload-Based Index Optimization untuk Meningkatkan Performa Query pada Sistem SaaS MultiTenant: Studi Kasus Koladi” ini dapat diselesaikan dengan baik. Proposal ini disusun sebagai salah satu syarat untuk melanjutkan penelitian menjadi Tugas Akhir pada Program Studi Teknologi Rekayasa Perangkat Lunak, Jurusan Teknik Informatika, Politeknik Negeri Batam. Dalam proses penyusunannya, penulis menyadari bahwa proposal ini dapat terwujud berkat dukungan, bimbingan, dan arahan dari berbagai pihak. Oleh karena itu, pada kesempatan ini penulis ingin menyampaikan ucapan terima kasih kepada: 1. Bapak/Ibu Ketua Program Studi Teknologi Rekayasa Perangkat Lunak Politeknik Negeri Batam. 2. Bapak/Ibu Dosen Pembimbing yang telah meluangkan waktu, tenaga, dan pikiran untuk memberikan bimbingan, arahan, serta masukan selama penyusunan proposal ini. 3. Orang tua dan keluarga tercinta yang senantiasa memberikan doa, dukungan, dan motivasi selama masa perkuliahan hingga penyusunan proposal ini. 4. Rekan-rekan tim Koladi dan teman-teman yang telah memberikan dukungan dan bertukar pikiran selama proses penyusunan proposal ini. 5. Seluruh pihak yang telah memberikan bantuan dan dukungan, baik secara langsung maupun tidak langsung. Penulis menyadari bahwa penyusunan proposal Tugas Akhir ini masih memiliki kekurangan karena keterbatasan ilmu dan pengetahuan yang dimiliki. Oleh karena itu, penulis mengharapkan kritik dan saran yang membangun demi perbaikan dan penyempurnaan penelitian ini. Semoga proposal Tugas Akhir ini dapat memberikan manfaat bagi penulis maupun pihak yang berkepentingan. 

ABSTRAK 

Sistem SaaS multi-tenant menghadapi tantangan penurunan kinerja basis data seiring dengan peningkatan volume data dan aktivitas pengguna yang berbagi infrastruktur. Pada Koladi, sebuah aplikasi kantor digital berbasis Laravel dan PostgreSQL, terdapat fitur kolaborasi yang memicu tingginya operasi pembacaan data. Beberapa query dominan di sistem ini menyaring data menggunakan atribut tenant, workspace, visibilitas, dan soft-delete yang belum sepenuhnya didukung oleh index bawaan sistem, sehingga berpotensi menggunakan Sequential Scan yang kurang efisien. Permasalahan ini tergolong complex engineering problem karena penyelesaiannya menuntut intervensi langsung terhadap arsitektur fisik basis data, bukan sekadar observasi atau analisis log secara pasif. Tugas Akhir ini mengusulkan penyelesaian masalah tersebut melalui implementasi optimasi index berbasis beban kerja (workload-based index optimization) yang secara aktif memodifikasi struktur penyimpanan data melalui perancangan dan penerapan index B-Tree komposit dan parsial. Pendekatan yang akan dilakukan meliputi penelusuran query Laravel Eloquent, konversi ke SQL mentah, analisis rencana eksekusi menggunakan EXPLAIN ANALYZE, serta perancangan index B-Tree komposit dan parsial berdasarkan pola filter. Evaluasi akan dilakukan pada tiga skala data dengan rasio kenaikan seragam: Small (1.000 baris), Medium (10.000 baris), dan Large (100.000 baris). Hasil akhir yang diharapkan dari penelitian ini adalah diperolehnya hasil evaluasi perubahan performa query berdasarkan waktu eksekusi dan cost, serta dampak penerapan index terhadap operasi penulisan (INSERT, UPDATE, DELETE), sehingga dapat dirumuskan rekomendasi index yang sesuai dengan workload dan lingkungan pengujian pada arsitektur SaaS multitenant. Kata Kunci: Workload-based index optimization, SaaS multi-tenant, PostgreSQL, query performance, composite index, partial index, write overhead. 

1.1. Latar Belakang Perkembangan teknologi komputasi awan (cloud computing) telah mempercepat adopsi sistem perangkat lunak dengan arsitektur Software as a Service (SaaS) multi-tenant. Pada arsitektur ini, berbagai perusahaan (klien atau tenant) saling berbagi aplikasi, peladen (server), dan instansi basis data yang sama guna meningkatkan efisiensi biaya, kemudahan pemeliharaan, serta skalabilitas operasional (Pippal et al., 2024). Namun demikian, berbaginya infrastruktur ini menghadirkan tantangan teknis tersendiri. Seiring bertambahnya jumlah tenant dan volume data yang tersimpan, kinerja pemrosesan basis data (query performance) rentan mengalami degradasi (Shah & Bhatt, 2024; Thakur, 2025). Sistem Koladi merupakan aplikasi SaaS kantor digital multi-tenant yang dirancang untuk memfasilitasi pelacakan tugas (task tracking), kolaborasi dokumen, dan pertukaran pesan secara waktu nyata. Fitur-fitur kolaboratif ini secara konstan menghasilkan permintaan operasi baca (read operations) terhadap basis data (Kavuluri et al., 2025). Aplikasi Koladi dibangun menggunakan kerangka kerja Laravel dan beroperasi di atas sistem manajemen basis data PostgreSQL. Meskipun sistem basis data Koladi saat ini telah menerapkan beberapa index dasar pada tingkat skema, penelusuran arsitektur menunjukkan adanya ketidaksesuaian antara index bawaan tersebut dengan pola akses data pada workload aplikasi Koladi. Beberapa query pada Koladi melakukan filtering berdasarkan parameter multi-tenant, seperti kolom workspace_id dan company_id, parameter status pengguna, atribut visibilitas, serta pemeriksaan data yang dihapus lunak (deleted_at IS NULL). Ketidaksesuaian antara pola filtering tersebut dengan struktur index yang tersedia dapat menyebabkan PostgreSQL menggunakan access path yang kurang efisien, termasuk Sequential Scan, terutama ketika volume data meningkat (Taipalus, 2024). Kondisi tersebut berpotensi meningkatkan waktu eksekusi query sehingga perlu dianalisis melalui workload dan execution plan untuk menentukan kebutuhan index tambahan. Optimasi kinerja basis data tingkat lanjut mengamanatkan bahwa struktur index harus dibangun berdasarkan karakteristik beban kerja nyata (workloadbased), bukan sebatas mengandalkan konfigurasi generik (Milicevic & Babovic, 2024). Namun, penerapan index yang berlebihan juga dapat menambah beban pemeliharaan. Ketika data pada tabel mengalami INSERT, UPDATE, atau DELETE, struktur index yang terkait juga perlu diperbarui, sehingga dapat menimbulkan write overhead (Abbasi et al., 2024b). Berdasarkan kompleksitas permasalahan tersebut, penelitian ini akan mengimplementasikan serta mengevaluasi strategi optimasi index berbasis beban kerja (workload-based index optimization) pada PostgreSQL untuk sistem Koladi (Sirigiri, 2025). Penelitian ini bertujuan merancang index komposit dan parsial yang spesifik terhadap query dominan yang lambat, sekaligus memastikan bahwa lonjakan kecepatan operasi baca (read) tidak menurunkan efisiensi waktu pada operasi tulis (write). Dengan demikian, penelitian ini tidak diposisikan sebagai kajian analitis atau observasional terhadap pola query semata, melainkan sebagai upaya rekayasa (engineering optimization) yang melakukan intervensi langsung terhadap arsitektur fisik basis data sistem Koladi yang sudah berjalan. Intervensi tersebut diwujudkan melalui perancangan dan penerapan struktur index B-Tree komposit dan parsial pada tingkat skema, sehingga solusi yang dihasilkan bersifat struktural dan dapat diverifikasi secara eksperimental, bukan sebatas temuan atau rekomendasi berdasarkan pengamatan log secara pasif.  

1.2. Rumusan Masalah Berdasarkan latar belakang yang telah dipaparkan, rumusan masalah dalam penelitian ini adalah sebagai berikut: 1. Bagaimana pola query yang paling dominan pada sistem Koladi? 2. Bagaimana perancangan dan penerapan Workload-Based Index Optimization berdasarkan pola query dominan tersebut? 3. Seberapa besar peningkatan performa query setelah penerapan strategi indexing berbasis workload pada operasi baca? 4. Bagaimana dampak penerapan index terhadap performa operasi tulis seperti INSERT, UPDATE, dan DELETE? 

1.3. Tujuan Penelitian Tujuan yang ingin dicapai melalui pelaksanaan Tugas Akhir ini adalah: 1. Mengidentifikasi pola query dominan pada sistem Koladi melalui pendekatan Application-Level Query Tracing. 2. Merancang dan menerapkan strategi indexing berbasis workload menggunakan composite index dan partial index pada database PostgreSQL sistem Koladi. 3. Mengevaluasi peningkatan performa query sebelum dan sesudah penerapan optimasi index menggunakan execution time dan query cost sebagai parameter utama. 4. Menganalisis dampak strategi indexing terhadap performa operasi tulis (INSERT, UPDATE, dan DELETE) serta menginterpretasikan trade-off antara peningkatan performa baca dan write overhead berdasarkan hasil pengujian. 

1.4. Batasan Masalah Untuk menjaga fokus agar penelitian tetap terarah, batasan masalah ditetapkan sebagai berikut: 1. Optimasi difokuskan secara eksklusif pada tingkat basis data (PostgreSQL), spesifik pada strategi indexing (index komposit dan parsial berbasis BTree). 2. Optimasi pada tingkat aplikasi (seperti implementasi caching internal) dan tingkat infrastruktur (seperti load balancing atau modifikasi perangkat keras) berada di luar ruang lingkup. 3. Query yang dianalisis berasal dari penelusuran tingkat aplikasi (applicationlevel query tracing) pada source code sistem Koladi. 4. Evaluasi kinerja pembacaan data diukur menggunakan parameter Execution Time dan Query Cost memanfaatkan fitur diagnostik EXPLAIN ANALYZE. 5. Skenario pengujian menggunakan data turunan terstruktur (seeded data) berskala Small (1.000 baris), Medium (10.000 baris), dan Large (100.000 baris), bukan menggunakan production query logs. 6. Performa operasi tulis dibatasi pada pengukuran pengorbanan waktu eksekusi (execution time overhead) untuk operasi INSERT, UPDATE, dan DELETE. 7. Penelitian ini tidak mengevaluasi interaksi beban pengguna secara bersamaan (concurrent user workload) dan ukuran penyimpanan index (index storage size). 

1.5. Manfaat Penelitian Adapun manfaat yang diharapkan dari penelitian Tugas Akhir ini adalah: 1. Manfaat Teoritis: Menjadi studi kasus empiris terkait implementasi workload-based index optimization pada arsitektur perangkat lunak SaaS multi-tenant dengan pendekatan model basis data terpusat (shared-database dan shared-schema). 2. Manfaat Praktis: Menghasilkan rekomendasi struktur index yang diharapkan dapat diverifikasi secara eksperimental untuk mengurangi potensi inefisiensi pembacaan data pada sistem Koladi, serta memberikan gambaran mengenai dampak penambahan index terhadap operasi penulisan sistem. 

BAB II TINJAUAN PUSTAKA 

2.1. Penelitian Terkait 
Penelitian ini merujuk pada beberapa studi terdahulu yang secara spesifik membahas mengenai optimasi kinerja basis data, arsitektur multi-tenant, serta pendekatan workload-aware indexing. Penelitian pertama dilakukan oleh Taipalus (2024) dengan judul "Database Management System Performance Comparisons: A Systematic Literature Review". Melalui metode Systematic Literature Review (SLR) terhadap 117 studi perbandingan performa DBMS, penelitian ini menemukan bahwa lebih dari 80% studi tidak melaporkan detail pengujian yang cukup untuk direplikasi, dan hasilnya sering saling bertentangan antar studi. Kelebihan studi ini adalah memberikan landasan konseptual yang kuat bahwa performa basis data tidak dapat digeneralisasi di luar konteks pengujiannya. Namun, keterbatasannya adalah murni bersifat tinjauan teoretis tanpa implementasi solusi teknis. Hal ini memvalidasi urgensi Tugas Akhir ini, yang menggunakan metode eksperimental terukur melalui EXPLAIN ANALYZE dengan skenario workload spesifik dari sistem Koladi. Penelitian kedua oleh Sirigiri (2025) berjudul "Enhancing SQL Query Performance: A Case Study on Optimizing Enterprise Data Processing" menerapkan teknik workload-aware indexing dan execution plan analysis menggunakan EXPLAIN yang dikombinasikan dengan simulasi model machine learning (ML). Hasilnya menunjukkan penurunan latensi query hingga 60% dan peningkatan throughput sebesar 35%. Kelebihan penelitian ini adalah bukti empiris efektivitas analisis execution plan pada skala enterprise, namun keterbatasannya adalah model ML yang digunakan masih bersifat simulasi (pre-trained) dan kurang feasible untuk SaaS skala menengah. Tugas Akhir ini menerapkan workload-based indexing murni yang lebih praktis untuk diimplementasikan langsung pada database Koladi tanpa bergantung pada infrastruktur ML. Penelitian ketiga oleh Pippal et al. (2024) berjudul "Optimizing MultiTenant Database Architecture for Efficient Software as a Service Delivery" mengusulkan arsitektur multi-tenant berbasis XML/JSON untuk menampung atribut spesifik tenant. Pengujian pada lingkungan cloud menunjukkan operasi INSERT dan DELETE yang jauh lebih cepat. Kelebihannya adalah efisiensi pada tingkat arsitektur storage, namun kelemahan utamanya adalah format XML/JSON menyulitkan penerapan index SQL standar. Tugas Akhir ini menutupi kelemahan tersebut dengan mempertahankan skema relasional terstruktur pada Koladi dan melakukan optimasi melalui index B-Tree pada kolom filter kunci (seperti company_id dan workspace_id) tanpa mengubah fundamental arsitektur data. Penelitian keempat oleh Abbasi et al. (2024b) dengan judul "Optimizing Database Performance in Complex Event Processing through Indexing Strategies" mengusulkan Hierarchical Temporal Indexing (HTI) untuk sistem Complex Event Processing (CEP). Eksperimen pada dataset MTA New York City menunjukkan peningkatan performa hingga 64,5%. Kelebihan studi ini adalah pembuktian empiris bahwa penyesuaian index dengan karakteristik workload aktual secara drastis meningkatkan efisiensi, namun keterbatasannya adalah fokus penelitian ini dikhususkan pada pemrosesan event spatio-temporal (single-tenant), bukan arsitektur transaksional SaaS multi-tenant. Tugas Akhir ini mengadopsi prinsip adaptasi workload tersebut, namun dikontekstualisasikan untuk filter visibilitas dan otorisasi multi-tenant. Penelitian kelima, juga oleh Abbasi et al. (2024c), berjudul "Revisiting Database Indexing for Parallel and Accelerated Computing: A Comprehensive Study and Novel Approaches", menguji strategi index yang sadar-perangkat keras (hardware-conscious) seperti cache-conscious B-Tree dan SIMD-optimized hash pada PostgreSQL. Hasilnya menunjukkan peningkatan performa query antara 32,4% hingga 48,6%. Keunggulan penelitian ini adalah integrasi optimasi perangkat keras tingkat rendah, namun keterbatasannya terletak pada kompleksitas implementasi yang tinggi. Tugas Akhir ini membedakan diri dengan berfokus pada optimasi tingkat logis (index komposit dan parsial) yang dapat diimplementasikan pada database Koladi tanpa memerlukan perubahan fundamental pada arsitektur aplikasi. Di luar kelima studi utama tersebut, sejumlah penelitian lain turut memperkuat posisi Tugas Akhir ini. Pada aspek arsitektur multi-tenant, Thakur (2025) dan Jin et al. (2024) menunjukkan bahwa tenant-aware indexing dapat menekan efek noisy-neighbor, sementara El Danaoui et al. (2024) menerapkan prediksi workload berbasis SARIMA untuk memperbaiki pemilihan execution plan pada DBMS paralel multi-tenant, sehingga menegaskan bahwa karakteristik multitenant perlu menjadi pertimbangan eksplisit dalam desain index, sejalan dengan fokus penelitian ini pada filter tenant-isolation. Pada aspek otomatisasi, Brucato et al. (2024) dan Wang et al. (2024) mempercepat proses index tuning melalui reduksi workload dan realokasi anggaran what-if, sedangkan Koukaras dan Tjortjis (2025) serta Liu et al. (2025) menekankan pentingnya menyelaraskan struktur index dengan karakteristik workload aktual. Pendekatan yang lebih kompleks berbasis reinforcement learning dan graph-based optimization (Tong et al., 2025; Sassi & Jaziri, 2025; Abbasi et al., 2024a) maupun tinjauan otomatisasi berbasis AI (Qiao et al., 2025; Boddu, 2025) menjanjikan otomatisasi index tuning, tetapi masih menghadapi kendala kompleksitas implementasi untuk sistem berskala kecilmenengah sehingga memperkuat pilihan penelitian ini pada pendekatan workloadbased indexing yang lebih sederhana dan langsung dapat diterapkan. Berdasarkan tinjauan terhadap penelitian-penelitian di atas, terdapat ruang penelitian pada penerapan workload-based index optimization pada sistem SaaS multi-tenant dengan arsitektur shared-schema di PostgreSQL. Penelitian sebelumnya menunjukkan berbagai pendekatan, antara lain optimasi berbasis perangkat keras, machine learning, perubahan arsitektur penyimpanan, serta indexing yang disesuaikan dengan workload. Penelitian ini memfokuskan penerapan index B-Tree komposit dan parsial pada sistem yang telah berjalan, dengan evaluasi terhadap operasi baca dan tulis. Koladi digunakan sebagai studi kasus untuk menguji pendekatan tersebut secara eksperimental menggunakan EXPLAIN ANALYZE. Perbandingan antara penelitian sebelumnya dengan penelitian ini disajikan pada Tabel 1. 

Tabel 1. Perbandingan Penelitian Sebelumnya

| Peneliti/Tahun | Judul/Topik Penelitian | Metode/Teknologi | Kelebihan | Kekurangan/Celah |
|---|---|---|---|---|
| Taipalus (2024) | Database Management System Performance Comparisons: A Systematic Literature Review | Murni Systematic Literature Review (SLR) | Memberikan fondasi pentingnya metodologi terukur berbasis workload pengujian, bukan sekadar query ad hoc. | Berupa tinjauan literatur teoretis tanpa usulan implementasi optimasi teknis spesifik. |
| Sirigiri (2025) | Enhancing SQL Query Performance: A Case Study on Optimizing Enterprise Data Processing | EXPLAIN, workload-aware indexing, Machine Learning | Menunjukkan peningkatan performa query melalui diagnosis berbasis execution plan. | Bergantung pada model Machine Learning yang kompleks dan kurang feasible untuk diterapkan pada SaaS skala menengah. |
| Pippal et al. (2024) | Optimizing Multi-Tenant Database Architecture for Efficient Software as a Service Delivery | Arsitektur berbasis XML/JSON | Meningkatkan efisiensi storage dan kecepatan modifikasi data pada skema multi-tenant. | Penggunaan tipe data XML/JSON menyulitkan penerapan teknik indexing SQL berorientasi baris secara standar. |
| Abbasi et al. (2024) | Optimizing Database Performance in Complex Event Processing through Indexing Strategies | PostgreSQL, Hierarchical Temporal Indexing (HTI) | Meningkatkan performa eksekusi query hingga 64,5% melalui keselarasan index dengan karakteristik data. | Berfokus pada aliran data spatiotemporal (single-tenant), bukan basis data transaksional SaaS multi-tenant. |
| Abbasi et al. (2024) | Revisiting Database Indexing for Parallel and Accelerated Computing | PostgreSQL, Cache-conscious B-Tree, SIMD-optimized hash | Meningkatkan performa query hingga 48,6% dengan memanfaatkan karakteristik arsitektur perangkat keras. | Memiliki kompleksitas implementasi tingkat tinggi (hardware-level) sehingga sulit diadopsi langsung oleh pengembang aplikasi reguler. |
| Penelitian Ini | Implementasi Optimasi Index Berbasis Workload untuk Meningkatkan Kinerja Query pada Sistem SaaS Multi-Tenant | PostgreSQL, B-Tree (Composite & Partial), Query Tracing | Menggabungkan analisis pola workload dengan optimasi index untuk menangani latensi query multi-tenant, sekaligus mengevaluasi trade-off antara performa baca [...] | [...] |

2.2. Landasan Teori beban operasi tulis. Landasan teori penelitian ini membahas konsep yang berkaitan langsung dengan analisis dan optimasi kinerja query pada database PostgreSQL dalam sistem SaaS multi-tenant. 2.2.1. Database dan PostgreSQL Database merupakan sistem yang digunakan untuk menyimpan dan mengelola data secara terstruktur, sehingga data dapat disimpan, diakses, dan dimanipulasi sesuai kebutuhan aplikasi. Dalam penelitian ini, database menjadi komponen utama yang dianalisis karena sebagian besar aktivitas aplikasi Koladi melibatkan operasi pembacaan maupun perubahan data, khususnya pada aspek yang berkaitan dengan struktur tabel dan index dalam memengaruhi proses pengambilan data.    PostgreSQL merupakan sistem manajemen basis data relasional yang digunakan sebagai database pada Koladi, dan menyediakan berbagai mekanisme untuk mengoptimalkan pemrosesan query, termasuk penggunaan index dan pemilihan execution plan oleh query planner. Dalam penelitian ini, PostgreSQL digunakan sebagai lingkungan eksperimen untuk menganalisis execution plan sebelum dan sesudah implementasi index yang dirancang berdasarkan workload aplikasi. 2.2.2. Sistem SaaS Multi-Tenant Software as a Service (SaaS) merupakan model penyediaan perangkat lunak yang memungkinkan aplikasi digunakan melalui jaringan tanpa pengguna harus mengelola infrastruktur aplikasi secara langsung. Pada sistem multi-tenant, satu aplikasi dan infrastruktur dapat melayani beberapa tenant dengan data yang tetap dipisahkan berdasarkan konteks tenant masing-masing.   Konsep ini relevan dengan penelitian karena query pada Koladi banyak menggunakan atribut seperti workspace_id untuk membatasi data berdasarkan workspace. Kondisi tersebut menjadi pertimbangan dalam menentukan kolom yang digunakan pada index.  2.2.3. Database Workload dan Application-Level Query Tracing Database workload merupakan kumpulan aktivitas query yang dijalankan aplikasi terhadap database, yang menggambarkan pola akses data selama aplikasi digunakan sehingga dapat menjadi dasar penentuan bagian database yang perlu dianalisis dan dioptimalkan. Pada penelitian ini, workload diperoleh melalui application-level query tracing, yaitu proses menelusuri interaksi antara kode aplikasi dan database melalui alur endpoint, controller, model, hingga operasi database yang digunakan (Brucato et al., 2024). Query dengan pola SQL yang sama dikonsolidasikan sebelum dilakukan pemilihan query untuk analisis lebih lanjut.    2.2.4. Database Index: B-Tree, Composite Index, dan Partial Index Database index merupakan struktur data tambahan yang membantu database menemukan baris data yang memenuhi kondisi tertentu tanpa harus memeriksa seluruh isi tabel, sehingga mengurangi jumlah data yang perlu diproses. Namun, index juga membutuhkan ruang penyimpanan dan harus diperbarui setiap kali data pada tabel berubah, sehingga penelitian ini turut mempertimbangkan dampaknya terhadap operasi tulis.   B-Tree merupakan salah satu struktur index PostgreSQL yang sesuai untuk berbagai operasi pencarian berdasarkan nilai kolom, termasuk kondisi perbandingan dan pengurutan (Abbasi et al., 2024b; Koukaras & Tjortjis, 2025). Dalam penelitian ini, B-Tree digunakan melalui dua pendekatan: composite index, yang dibentuk dari lebih dari satu kolom untuk query yang menggunakan beberapa kolom sekaligus dalam filter atau sorting (misalnya workspace_id dengan uploaded_at, atau conversation_id dengan created_at); dan partial index, yang hanya mencakup baris yang memenuhi kondisi tertentu, seperti data yang belum dibaca atau data yang tidak bersifat private. Susunan kolom pada kedua jenis index mempertimbangkan selektivitas kolom untuk memaksimalkan efisiensi. 2.2.5. Query Execution Plan dan EXPLAIN ANALYZE Query execution plan merupakan rencana yang digunakan database untuk menjalankan sebuah query, yang menentukan metode akses data dan operasi yang digunakan selama eksekusi. Dalam penelitian ini, execution plan digunakan untuk melihat apakah query memanfaatkan index atau melakukan pemindaian tabel, dengan metode yang menjadi perhatian meliputi Sequential Scan, Index Scan, Index Only Scan, dan Bitmap Heap Scan.    EXPLAIN ANALYZE merupakan fasilitas PostgreSQL yang menjalankan query sekaligus menampilkan informasi execution plan dan hasil eksekusi aktual (Kavuluri et al., 2025). Penelitian ini menggunakan EXPLAIN ANALYZE untuk membandingkan kondisi sebelum dan sesudah implementasi index, dengan informasi yang diamati meliputi scan method, query cost, execution time, planning time, dan jumlah baris yang tersaring selama eksekusi. 2.2.6. Parameter Evaluasi Query: Cost, Execution Time, Planning Time, dan Rows Removed by Filter Query cost merupakan estimasi biaya eksekusi yang dihitung PostgreSQL query planner berdasarkan execution plan yang dipilih, dan digunakan untuk membandingkan alternatif execution plan. Execution time merupakan waktu aktual yang dibutuhkan PostgreSQL untuk menjalankan query, sedangkan planning time merupakan waktu yang digunakan untuk menentukan execution plan. Rows Removed by Filter menunjukkan jumlah baris yang diperiksa tetapi tidak memenuhi kondisi filter pada suatu bagian execution plan; nilai yang tinggi menunjukkan bahwa database harus memeriksa banyak baris sebelum memperoleh data yang sesuai. Dalam penelitian ini, execution time digunakan sebagai metrik utama karena mencerminkan waktu aktual eksekusi query, sedangkan query cost, planning time, dan Rows Removed by Filter digunakan sebagai metrik pendukung untuk melihat perubahan estimasi biaya, overhead perencanaan, dan efisiensi proses filtering setelah index diterapkan. Query cost tidak diperlakukan sebagai pengganti execution time karena cost merupakan estimasi planner, sedangkan execution time merupakan waktu aktual eksekusi (Taipalus, 2024).    2.2.7. Read dan Write Performance Read performance berkaitan dengan efisiensi operasi pengambilan data, sedangkan write performance berkaitan dengan efisiensi operasi perubahan data seperti INSERT, UPDATE, dan DELETE. Penelitian ini mengevaluasi keduanya karena penambahan index dapat memberikan manfaat pada operasi baca, tetapi pada saat yang sama menambah pekerjaan pemeliharaan index ketika data diubah. Oleh karena itu, evaluasi tidak hanya melihat perubahan pada query baca, tetapi juga mengamati perubahan waktu operasi tulis. 2.2.8. Workload-Based Index Optimization Workload-Based Index Optimization merupakan pendekatan optimasi index yang menggunakan pola akses database dari workload aplikasi sebagai dasar penentuan index yang perlu dirancang (Abbasi et al., 2024b; Kavuluri et al., 2025; Sirigiri, 2025). Pendekatan ini diterapkan melalui lima fase, yaitu Workload Analysis, Execution Plan Analysis, Index Design, Index Implementation, dan Performance Evaluation, sehingga penambahan index diarahkan pada pola query yang menunjukkan kebutuhan optimasi berdasarkan hasil analisis workload dan execution plan. Pendekatan berbasis machine learning atau reinforcement learning untuk index tuning juga berkembang dalam penelitian sebelumnya, sebagaimana dibahas pada Subbab 2.1. Penelitian ini memilih pendekatan workload-based indexing manual karena sesuai dengan kebutuhan optimasi pada database Koladi dan dapat diterapkan tanpa menambahkan infrastruktur machine learning.    2.2.9. Koladi sebagai Objek Penelitian Koladi merupakan aplikasi Digital Office berbasis web yang menjadi objek studi kasus penelitian ini, menggunakan arsitektur SaaS multi-tenant dan PostgreSQL sebagai database. Penelitian difokuskan pada sisi database Koladi, khususnya query yang dihasilkan oleh fitur aplikasi dan pola akses data pada tabel yang berkaitan dengan workload penelitian. Penelitian tidak mencakup pengembangan fitur baru, tetapi berfokus pada analisis dan optimasi struktur index berdasarkan workload aplikasi. Dengan demikian, Koladi digunakan sebagai lingkungan studi kasus untuk menguji bagaimana pendekatan workload-based index optimization dapat memengaruhi execution plan, waktu eksekusi, estimasi query cost, serta efisiensi filtering pada database. 2.3. Metode Pengembangan Produk Penelitian ini menggunakan pendekatan Workload-Based Index Optimization, yaitu strategi optimasi index yang menjadikan pola akses data pada workload aplikasi sebagai dasar perancangan index, bukan konfigurasi index generik (Abbasi et al., 2024b; Brucato et al., 2024; Kavuluri et al., 2025; Sirigiri, 2025). Pendekatan ini dipilih karena penelitian sebelumnya menunjukkan bahwa perancangan index berdasarkan karakteristik workload dapat digunakan untuk menyesuaikan konfigurasi index dengan pola akses query yang diuji (lihat Subbab 2.1), sekaligus lebih praktis diimplementasikan pada sistem operasional seperti Koladi dibandingkan pendekatan berbasis machine learning yang kompleks. Pendekatan ini terdiri atas lima fase berurutan, yaitu Workload Analysis, Execution Plan Analysis, Index Design, Index Implementation, dan Performance Evaluation, dengan setiap fase menghasilkan artefak yang menjadi input bagi fase berikutnya. Uraian konsep tiap fase dijelaskan di bawah ini, sedangkan langkah pelaksanaan spesifik pada sistem Koladi dijelaskan pada Subbab 3.2. 2.3.1. Workload Analysis Workload Analysis merupakan tahap mengidentifikasi query yang menjadi beban kerja database, karena struktur index perlu disesuaikan dengan pola akses query aktual, bukan asumsi umum (Brucato et al., 2024). Tahap ini menghasilkan kumpulan query yang menjadi input bagi Execution Plan Analysis. 2.3.2. Execution Plan Analysis Execution Plan Analysis merupakan tahap menganalisis cara PostgreSQL menjalankan query dan access path yang digunakan, guna menentukan teknik optimasi yang sesuai (Kavuluri et al., 2025; Sirigiri, 2025). Query yang pada hasil EXPLAIN ANALYZE menunjukkan indikasi inefisiensi, seperti penggunaan Sequential Scan pada tabel dengan volume data besar, jumlah Rows Removed by Filter yang tinggi, atau penggunaan existing index yang belum mendukung pola filtering dan sorting secara optimal, dipertimbangkan sebagai kandidat optimasi index. 2.3.3. Index Design Index Design merupakan tahap merancang struktur index berdasarkan karakteristik workload dan hasil execution plan analysis, guna mendukung pola akses data tertentu dan meningkatkan efisiensi pemrosesan query (Abbasi et al., 2024b; Sirigiri, 2025). Index dirancang menggunakan struktur B-Tree dengan pendekatan composite index atau partial index, disesuaikan dengan karakteristik query yang menjadi target optimasi. 2.3.4. Index Implementation Index Implementation merupakan tahap menerapkan rancangan index pada database PostgreSQL Koladi tanpa mengubah fungsi utama aplikasi, sehingga kondisi optimized dapat dibandingkan langsung dengan kondisi baseline menggunakan query dan skenario data yang sama. Validitas penerapan index dipastikan melalui prosedur verifikasi correctness, yang memastikan index hanya mengubah access path yang digunakan PostgreSQL, bukan mengubah semantik maupun hasil query (lihat Subbab 3.2.4). 2.3.5. Performance Evaluation Performance Evaluation merupakan tahap mengukur perubahan performa query setelah index diterapkan dengan membandingkan kondisi baseline dan optimized. Evaluasi dilakukan menggunakan scan method, query cost, execution time, planning time, dan Rows Removed by Filter sebagai metrik pengujian (Abbasi et al., 2024b; Kavuluri et al., 2025). Selain performa baca, tahap ini juga mengevaluasi execution time overhead pada operasi tulis (INSERT, UPDATE, DELETE) untuk melihat trade-off antara peningkatan performa baca dan write overhead. 

2.4. Metode Analisis yang Digunakan Penelitian ini menggunakan metode eksperimen kuantitatif dengan pendekatan komparatif before-after, yang digunakan untuk mengukur perubahan kinerja berdasarkan perbandingan kondisi sebelum dan sesudah suatu perlakuan diterapkan (Wohlin et al., 2024). Dalam penelitian ini, perlakuan berupa penerapan index berbasis workload, sedangkan perubahan kinerja diamati melalui performa query baca dan operasi tulis pada database PostgreSQL. 2.4.1. Desain Eksperimen Eksperimen dilakukan dengan membandingkan dua kondisi, yaitu baseline (konfigurasi index sebelum penerapan index berbasis workload) dan optimized (kondisi setelah index diterapkan). Query, struktur data, parameter query, dan skenario volume data dibuat sama pada kedua kondisi. Perbedaan antara kondisi baseline dan optimized dibatasi pada konfigurasi index. Pengujian dilakukan pada tiga skenario volume data, yaitu Small (1.000 baris), Medium (10.000 baris), dan Large (100.000 baris), guna melihat perubahan performa ketika volume data meningkat serta menjaga kondisi pengujian tetap terdefinisi dan dapat direplikasi (Taipalus, 2024). Pada setiap skenario, setiap query diukur melalui beberapa kali pengulangan eksekusi, bukan melalui eksekusi tunggal, untuk mengurangi pengaruh variasi performa sistem terhadap hasil pengukuran. Rancangan pengulangan dan metode agregasi dijelaskan pada Subbab 2.4.2. 2.4.2. Parameter dan Teknik Pengukuran Evaluasi performa dilakukan terhadap dua kelompok operasi. Read performance diukur menggunakan Execution Time dan Query Cost, dengan perubahan access path diamati melalui Scan Method, menggunakan EXPLAIN ANALYZE. Write performance diukur berdasarkan execution time overhead pada operasi INSERT, UPDATE, dan DELETE, karena penambahan index dapat memberi manfaat pada operasi baca namun menambah beban pemeliharaan index saat data berubah. Planning Time dan Rows Removed by Filter dicatat sebagai informasi pendukung. Untuk menghindari bias pengukuran akibat fluktuasi performa sistem, termasuk efek buffer cache pada PostgreSQL yang menyebabkan eksekusi pertama suatu query cenderung lebih lambat dibandingkan eksekusi berikutnya pada data yang sama, setiap query pada setiap skenario tidak diukur dengan eksekusi tunggal. Setiap query dijalankan sebanyak satu kali eksekusi pemanasan (warm-up) yang hasilnya tidak dicatat, dilanjutkan dengan 10 kali eksekusi EXPLAIN ANALYZE yang hasilnya dicatat sebagai data pengukuran, pada kondisi baseline maupun optimized dengan jumlah pengulangan dan parameter query yang identik. Untuk pengukuran operasi tulis, kondisi dataset dikembalikan ke keadaan awal sebelum setiap pengulangan pengukuran agar setiap percobaan menggunakan kondisi data yang setara. Dengan demikian, operasi INSERT, UPDATE, dan DELETE pada setiap pengulangan tidak dipengaruhi oleh perubahan data yang terjadi pada pengulangan sebelumnya. Dataset dan parameter operasi dibuat identik pada kondisi baseline dan optimized. Dari 10 nilai hasil pengulangan tersebut, nilai median digunakan sebagai nilai utama untuk Execution Time, Planning Time, dan Query Cost, karena median lebih tahan (robust) terhadap outlier yang dapat muncul akibat gangguan sesaat pada sistem (seperti aktivitas latar belakang sistem operasi atau I/O disk), dibandingkan nilai rata-rata (mean) yang sensitif terhadap nilai ekstrem. Nilai mean dan standar deviasi dari 10 pengulangan tetap dihitung dan dilaporkan sebagai informasi pembanding untuk menunjukkan sebaran hasil pengukuran, secara umum dinyatakan sebagai: Median Execution Time = median(T1, T2, ..., T10) dengan T1 hingga T10 merupakan hasil Execution Time dari 10 kali pengulangan eksekusi pada satu kondisi (baseline atau optimized) pada satu skenario volume data. 2.4.3. Analisis Peningkatan Performa Query Perubahan performa query dianalisis dengan membandingkan Execution Time antara kondisi baseline dan optimized, menggunakan persamaan: Improvement (%) = ((T_before − T_after) / T_before) × 100% (1) dengan T_before sebagai nilai median Execution Time dari 10 kali pengulangan pada kondisi baseline, dan T_after sebagai nilai median Execution Time dari 10 kali pengulangan pada kondisi optimized, sebagaimana didefinisikan pada Subbab 2.4.2. Nilai positif menunjukkan waktu eksekusi menurun setelah optimasi, sedangkan nilai negatif menunjukkan peningkatan waktu eksekusi pada skenario tersebut. Selain Execution Time, efisiensi execution plan dianalisis menggunakan Cost Reduction Ratio: Cost Reduction Ratio = Cost_before / Cost_after (2) Nilai lebih besar dari 1 menunjukkan bahwa estimasi cost pada kondisi optimized lebih rendah dibandingkan baseline, dengan nilai rasio menunjukkan perbandingan cost baseline terhadap optimized. Cost Reduction Ratio digunakan sebagai metrik pendukung karena Query Cost merupakan estimasi planner, bukan waktu aktual eksekusi. 2.4.4. Analisis Write Performance Write performance dianalisis dengan membandingkan waktu eksekusi operasi INSERT, UPDATE, dan DELETE sebelum dan sesudah penerapan index, dinyatakan sebagai write execution time overhead: Write Overhead (%) = ((T_after − T_before) / T_before) × 100% (3) Dengan T_before dan T_after merujuk pada nilai median waktu eksekusi operasi tulis dari 10 kali pengulangan, mengikuti teknik pengukuran pada Subbab 2.4.2. Nilai positif menunjukkan tambahan waktu eksekusi setelah index diterapkan, sedangkan nilai negatif menunjukkan waktu eksekusi operasi tulis yang lebih rendah pada kondisi optimized. Analisis ini digunakan untuk mengetahui apakah peningkatan performa baca diikuti tambahan beban pada operasi tulis. 2.4.5. Kriteria Evaluasi Trade-off Strategi indexing dievaluasi melalui pendekatan interpretasi trade-off yang bersifat kontekstual terhadap workload Koladi. Evaluasi mempertimbangkan keseimbangan antara peningkatan performa query baca dan biaya tambahan yang ditimbulkan oleh penerapan index pada operasi tulis. Penelitian mengenai index tuning menunjukkan bahwa pemilihan konfigurasi index dilakukan berdasarkan karakteristik workload dan dampaknya terhadap kinerja query, sehingga efektivitas index perlu dinilai berdasarkan workload yang diuji (Brucato et al., 2024; Wang et al., 2024). Trade-off antara read performance dan write overhead dievaluasi dengan mempertimbangkan beberapa aspek, yaitu: (1) perubahan Execution Time pada query target, (2) konsistensi peningkatan performa pada setiap skenario volume data, (3) perubahan Scan Method pada execution plan, (4) perubahan Query Cost dan Rows Removed by Filter sebagai indikator pendukung, serta (5) besaran write overhead yang ditimbulkan setelah penerapan index. Pendekatan ini mempertimbangkan bahwa hasil evaluasi strategi indexing dapat dipengaruhi oleh volume data, karakteristik query, execution plan, dan workload yang digunakan dalam pengujian (Abbasi et al., 2024b).  Evaluasi trade-off tidak menggunakan satu nilai gabungan atau batas persentase tertentu sebagai dasar untuk menentukan bahwa suatu index diterima atau ditolak. Interpretasi hasil dilakukan dengan mempertimbangkan keseluruhan metrik pengujian, bukan berdasarkan satu metrik secara terpisah. Index dipandang memberikan manfaat terhadap workload penelitian apabila peningkatan read performance menunjukkan hasil yang konsisten pada skenario pengujian dan didukung oleh perubahan execution plan yang sesuai dengan tujuan optimasi, dengan write overhead tetap diperhitungkan sebagai biaya tambahan dari penerapan index. Apabila peningkatan read performance hanya terjadi pada sebagian skenario atau disertai write overhead yang relatif tinggi pada kondisi tertentu, hasil tersebut tidak langsung dikategorikan sebagai kegagalan. Hasil tersebut dianalisis berdasarkan karakteristik workload dan volume data untuk menentukan apakah index tetap memberikan manfaat pada workload yang diuji atau memerlukan peninjauan kembali pada tahap Index Design. Dengan demikian, evaluasi trade-off dalam penelitian ini bersifat kontekstual dan deskriptif. Kesimpulan efektivitas index ditentukan berdasarkan kombinasi Execution Time, Query Cost, Scan Method, Rows Removed by Filter, write overhead, serta konsistensi hasil pada setiap skenario, dengan interpretasi yang dibatasi pada workload dan lingkungan pengujian yang digunakan dalam penelitian.  2.4.6. Interpretasi Hasil Eksperimen Hasil eksperimen dianalisis dengan membandingkan kondisi baseline dan optimized pada setiap skenario volume data, mempertimbangkan Execution Time, Query Cost, Scan Method, Planning Time, dan Rows Removed by Filter. Hasil read performance dan write performance dianalisis bersama untuk melihat dampak penerapan index terhadap keseimbangan operasi baca-tulis. Interpretasi hasil dibatasi pada workload dan lingkungan pengujian database Koladi sehingga tidak digeneralisasikan secara langsung ke sistem SaaS multi-tenant lainnya.

BAB III METODOLOGI

3.1. Alat dan Bahan
Alat dan bahan digunakan untuk mendukung proses penelusuran workload,
analisis execution plan, perancangan index, implementasi index, serta pengujian
performa database.
3.1.1. Alat
Alat yang digunakan untuk mendukung penelitian ini adalah sebagai
berikut: 

Tabel 2. Alat Penelitian

| Alat | Fungsi |
| :--- | :--- |
| Laptop atau komputer | Menjalankan aplikasi, database, dan perangkat pendukung penelitian. |
| Visual Studio Code | Menelusuri source code dan query database Koladi. |
| Docker | Menyediakan lingkungan pengujian aplikasi dan database. |
| Laravel | Framework aplikasi dan sumber penelusuran query database. |
| PostgreSQL | Menyediakan database untuk implementasi dan pengujian index. |
| PostgreSQL Command Line Interface (psql) atau perangkat administrasi PostgreSQL | Menjalankan query, membuat index, dan EXPLAIN ANALYZE. |
| Git | Mengelola dan melacak perubahan source code. |
| Web browser | Memeriksa aplikasi, fitur, dan endpoint Koladi. |

Spesifikasi perangkat keras dan perangkat lunak yang digunakan dicatat
sebagai bagian dari dokumentasi lingkungan pengujian agar kondisi eksperimen
dapat dijelaskan dan direplikasi.
3.1.2. Bahan
Bahan yang digunakan dalam penelitian ini adalah sebagai berikut:

### Tabel 3. Bahan Penelitian[cite: 2]

| Bahan | Fungsi |
| :--- | :--- |
| Source code aplikasi Koladi | Menelusuri fitur, endpoint, controller, model, dan query database. |
| Skema database PostgreSQL Koladi | Mengetahui struktur tabel, kolom, relasi, dan existing index. |
| Daftar route dan endpoint Koladi | Memetakan fitur yang menghasilkan operasi database. |
| Query hasil *Application-Level Query Tracing* | Membentuk workload penelitian. |
| *Seeded dataset* | Data pengujian dengan volume Small (1.000 baris), Medium (10.000 baris), dan Large (100.000 baris). |
| Hasil EXPLAIN ANALYZE | Menganalisis execution plan dan performa query sebelum dan sesudah index. |
| Rancangan index | Merancang composite index dan partial index berdasarkan workload dan execution plan. |
| Literatur penelitian | Menjadi dasar teori dan metodologi penelitian. |
```[cite: 2]

3.2. Jalannya Penelitian
Jalannya penelitian menggambarkan urutan kegiatan yang dilakukan mulai
dari pembentukan workload hingga evaluasi hasil optimasi. Penelitian
menggunakan lima fase utama Workload-Based Index Optimization, yaitu
Workload Analysis, Execution Plan Analysis, Index Design, Index Implementation,
dan Performance Evaluation.

Gambar 1. Tahapan Penelitian

| Fase | Tahapan |
| :--- | :--- |
| Phase 1 | Workload analysis |
| Phase 2 | Execution plan analysis |
| Phase 3 | Index design |
| Phase 4 | Index implementation |
| Phase 5 | Performance evaluation |
```[cite: 3]

3.2.1. Workload Analysis
Tahap Workload Analysis dilakukan untuk memperoleh query yang
membentuk workload database Koladi. Query diperoleh melalui penelusuran pada
source code aplikasi sehingga workload yang digunakan dalam penelitian berasal
dari pola akses database yang terdapat pada aplikasi. Kegiatan pada tahap ini
meliputi identifikasi fitur Koladi yang relevan dengan penelitian, pemetaan route
menggunakan php artisan route:list, penelusuran controller dan model, serta
identifikasi operasi database yang menggunakan Eloquent maupun raw SQL. Query
yang ditemukan kemudian ditelusuri hingga diperoleh bentuk SQL yang dapat
digunakan untuk analisis pada PostgreSQL. Query yang memiliki pola SQL yang
sama dikelompokkan untuk menghindari pengujian berulang terhadap query yang
memiliki karakteristik akses data yang sama. Query yang telah dikelompokkan
tersebut kemudian disaring menjadi dominant query menggunakan tiga kriteria,
yaitu core feature relevance, critical column usage, dan large-table access potential,
sebagaimana dijelaskan pada Subbab 3.4.2. Query yang memenuhi ketiga kriteria
dikategorikan sebagai prioritas utama (highest priority), sedangkan query yang
memenuhi tepat dua dari tiga kriteria dikategorikan sebagai prioritas kedua (second
priority). Hasil tahap ini berupa kumpulan dominant query yang membentuk
workload penelitian dan menjadi input untuk tahap Execution Plan Analysis.
3.2.2. Execution Plan Analysis
Tahap Execution Plan Analysis dilakukan untuk mengetahui bagaimana
PostgreSQL menjalankan query yang diperoleh dari workload. Analisis dilakukan
menggunakan EXPLAIN ANALYZE pada tiga skenario volume data, yaitu Small,
Medium, dan Large. Pada setiap skenario, query dijalankan dan hasil execution plan
dicatat. Parameter yang diamati meliputi Execution Time, Planning Time, Query
Cost, Scan Method, dan Rows Removed by Filter. Analisis dilakukan untuk
mengetahui access path yang digunakan PostgreSQL, penggunaan existing index,
serta efisiensi proses filtering dan pengambilan data. Analisis execution plan
dilakukan terutama terhadap dominant query berprioritas utama (highest priority),
dengan dominant query berprioritas kedua (second priority) dianalisis sebagai pola
pendukung apabila diperlukan. Dominant query yang pada hasil EXPLAIN
ANALYZE menunjukkan indikasi inefisiensi, seperti penggunaan Sequential Scan
pada tabel dengan volume data besar, jumlah Rows Removed by Filter yang tinggi,
atau penggunaan existing index yang belum mendukung pola filtering dan sorting
secara optimal, dipertimbangkan sebagai kandidat optimasi index dan menjadi
dasar dalam tahap Index Design.
3.2.3. Index Design
Tahap Index Design dilakukan berdasarkan karakteristik workload dan hasil
Execution Plan Analysis. Perancangan diarahkan pada query yang menunjukkan
pola akses yang dapat didukung oleh struktur index tambahan. Kolom yang
digunakan dalam filtering dan sorting dianalisis untuk menentukan struktur index
yang sesuai. Index yang dirancang menggunakan B-Tree dengan pendekatan
composite index atau partial index sesuai dengan karakteristik query. Setiap
rancangan index didokumentasikan berdasarkan tabel target, kolom yang
digunakan, urutan kolom, kondisi partial apabila digunakan, serta query yang
menjadi target optimasi.
3.2.4. Index Implementation
Tahap Index Implementation dilakukan dengan menerapkan rancangan
index pada database PostgreSQL Koladi. Index dibuat menggunakan perintah SQL
sesuai dengan rancangan yang telah ditentukan. Setelah index diterapkan, struktur
index diperiksa untuk memastikan index berhasil dibuat pada tabel dan kolom yang
sesuai. Query yang menjadi target kemudian dijalankan kembali menggunakan
EXPLAIN ANALYZE untuk mengamati perubahan execution plan dan
penggunaan index yang telah diterapkan. Implementasi index dilakukan tanpa
mengubah fungsi utama aplikasi maupun logika query yang menjadi workload
penelitian.
Sebelum hasil pengujian pada kondisi optimized digunakan dalam tahap
Performance Evaluation, dilakukan verifikasi correctness untuk memastikan bahwa
penerapan index tidak mengubah semantik maupun hasil query. Verifikasi
dilakukan dengan menjalankan query yang sama pada kondisi baseline dan
optimized menggunakan dataset dan parameter yang identik. Hasil dari kedua
kondisi kemudian dibandingkan secara otomatis berdasarkan jumlah baris (row
count) dan kesamaan result set. Pada query yang menggunakan klausa ORDER BY,
verifikasi dilakukan untuk memastikan bahwa hasil tetap memenuhi urutan yang
ditentukan oleh query. Selanjutnya, fungsi atau endpoint aplikasi yang berkaitan
dengan query tersebut diperiksa untuk memastikan bahwa penerapan index tidak
mengubah perilaku fungsional sistem. Hasil pengujian optimized hanya digunakan
dalam analisis performa apabila seluruh proses verifikasi menunjukkan hasil yang
sesuai dengan kondisi baseline.
Index dinyatakan valid dan dapat dilanjutkan ke tahap Performance
Evaluation apabila hasil query pada kondisi optimized sama dengan kondisi
baseline, baik dari sisi jumlah baris maupun isi result set. Untuk query yang
menggunakan klausa ORDER BY, hasil juga harus tetap memenuhi urutan yang
ditentukan oleh query. Apabila ditemukan perbedaan hasil atau perubahan pada
perilaku fungsional aplikasi, index yang bersangkutan akan ditinjau kembali pada
tahap Index Design sebelum pengujian performa dilanjutkan.
3.2.5. Performance Evaluation
Tahap Performance Evaluation hanya dilakukan terhadap index yang telah
dinyatakan valid melalui Verifikasi Correctness pada Subbab 3.2.4. Tahap
Performance Evaluation dilakukan untuk membandingkan kondisi baseline dan
optimized setelah index diterapkan. Pengujian dilakukan menggunakan query dan
skenario volume data yang sama sehingga perubahan hasil dapat dibandingkan
secara langsung. Evaluasi dibagi menjadi tiga aspek: pertama, read performance,
yang dianalisis menggunakan Execution Time dan Query Cost serta perubahan
Scan Method dan Rows Removed by Filter; kedua, write performance, yang
dianalisis melalui perubahan waktu eksekusi operasi INSERT, UPDATE, dan
DELETE setelah penerapan index; ketiga, read-write trade-off, yang digunakan
untuk melihat keseimbangan antara peningkatan performa query target dan
tambahan write overhead. Hasil pengujian kemudian dianalisis menggunakan
Improvement (%) dan Cost Reduction Ratio.

3.3. Tahapan Pengembangan Aplikasi
Sistem Koladi yang digunakan dalam penelitian merupakan aplikasi yang
telah tersedia dan menjadi objek studi kasus. Penelitian ini melakukan aktivitas
rekayasa pada struktur index database sebagai bagian dari optimasi sistem yang
sudah berjalan. Penelitian ini tidak mencakup pengembangan fitur baru, perubahan
alur fungsional, maupun perancangan ulang antarmuka aplikasi, sehingga metode
pengembangan perangkat lunak seperti Waterfall, Agile, RAD, atau metode lainnya
tidak diterapkan sebagai tahapan utama penelitian.
Meskipun tidak menerapkan metodologi pengembangan perangkat lunak
konvensional, penelitian ini tetap merupakan aktivitas rekayasa terhadap sistem
Koladi yang sudah berjalan. Aktivitas tersebut mencakup pemahaman arsitektur
dan skema database, penelusuran workload dari source code, identifikasi query
kandidat, analisis execution plan, perancangan dan penerapan index, pengujian
performa, verifikasi correctness, serta evaluasi read performance dan write
performance.
Fokus perubahan berada pada struktur index database yang diperlukan
untuk eksperimen, sedangkan fungsi utama aplikasi dan query yang menjadi
workload penelitian dipertahankan. Dengan demikian, tahapan pada Subbab 3.2
menjadi alur utama pelaksanaan Tugas Akhir, sedangkan Koladi berfungsi sebagai
objek dan lingkungan studi kasus.
Perubahan pada database dibatasi pada penerapan index yang telah
dirancang berdasarkan workload dan hasil execution plan. Setelah index diterapkan,
query yang sama digunakan kembali pada kondisi optimized untuk memperoleh
hasil yang dapat dibandingkan dengan kondisi baseline. Dengan demikian, tahapan
penelitian pada Subbab 3.2 menjadi alur utama pelaksanaan Tugas Akhir,
sedangkan aplikasi Koladi berfungsi sebagai objek dan lingkungan studi kasus.

3.4. Rencana Perancangan Sistem
Perancangan pada penelitian ini tidak berfokus pada pembangunan sistem
aplikasi baru, tetapi pada penyiapan lingkungan dan komponen yang diperlukan
untuk melakukan optimasi dan pengujian database Koladi. Rancangan meliputi
gambaran umum sistem, workload, dataset, index, dan lingkungan pengujian.
3.4.1. Gambaran Umum Sistem
Koladi merupakan aplikasi Digital Office berbasis web yang menggunakan
Laravel sebagai framework aplikasi dan PostgreSQL sebagai database. Sistem
menggunakan pendekatan SaaS multi-tenant sehingga beberapa tenant
menggunakan aplikasi dan infrastruktur yang sama dengan pemisahan data
berdasarkan konteks masing-masing tenant. Alur akses data yang menjadi fokus
penelitian digambarkan pada Gambar 2.

| No. | Komponen / Entitas | Deskripsi / Sub-komponen |
| :---: | :--- | :--- |
| 1 | User | Pengguna sistem |
| 2 | Aplikasi Koladi | Laravel |
| 3 | Eloquent / Raw SQL | Application-Level Query Tracing |
| 4 | PostgreSQL | Tables, Indexes (Data terpisah berdasarkan tenant) |
| 5 | Query Execution Plan | EXPLAIN ANALYZE |

Gambar 2. Gambaran Umum Sistem dan Lingkungan Penelitian
Pada penelitian ini, fokus optimasi berada pada bagian database
PostgreSQL, khususnya struktur index yang digunakan untuk mendukung pola
akses query. Source code aplikasi digunakan sebagai sumber untuk melakukan
Application-Level Query Tracing, sedangkan PostgreSQL digunakan sebagai
lingkungan eksperimen untuk mengukur perubahan execution plan dan performa
query.
3.4.2. Rancangan Workload
Penelusuran query dari source code menghasilkan kumpulan Eloquent
query candidates, yaitu seluruh query yang teridentifikasi pada tahap tracing
sebelum disaring lebih lanjut. Dari kumpulan tersebut, query ditetapkan sebagai
dominant query, yaitu query yang layak membentuk workload penelitian, apabila
memenuhi minimal dua dari tiga kriteria berikut, sebagaimana dirangkum pada
Tabel 4.
Tabel 4. Kriteria Penentuan Dominant Query


| Kriteria | Penjelasan |
| :--- | :--- |
| Core Feature Relevance | Query berasal dari fitur inti Koladi yang rutin digunakan, seperti manajemen tugas, dokumen, dan pesan. |
| Critical Column Usage | Query menggunakan kolom kritis untuk filtering, seperti `workspace_id`, `company_id`, `visibility`, dan `deleted_at`. |
| Large-Table Access Potential | Query mengakses tabel berskala besar yang berpotensi mengalami Sequential Scan tanpa index yang sesuai. |
```[cite: 5]

Query yang memenuhi ketiga kriteria sekaligus dikategorikan sebagai
prioritas utama (highest priority) dan menjadi fokus utama pada tahap Execution
Plan Analysis dan Index Design, sedangkan query yang memenuhi tepat dua dari
tiga kriteria dikategorikan sebagai prioritas kedua (second priority) dan berfungsi
sebagai pola workload pendukung. Query yang hanya memenuhi satu kriteria atau
kurang tidak disertakan sebagai dominant query. Skema prioritas ini digunakan
untuk menyaring query yang membentuk workload penelitian. Penetapan suatu
query sebagai kandidat optimasi index tidak ditentukan pada tahap ini, tetapi
dilakukan setelah Execution Plan Analysis berdasarkan karakteristik access path
dan indikasi inefisiensi yang ditemukan.
Workload penelitian dirancang berdasarkan dominant query yang diperoleh
melalui proses tersebut. Query dengan pola SQL yang identik dihitung satu kali
sebagai representatif untuk menghindari pengujian berulang terhadap karakteristik
akses yang sama. Informasi workload yang didokumentasikan meliputi identitas
query, fitur atau endpoint asal, tabel yang digunakan, pola filtering, pola sorting,
serta kategori prioritas (highest priority/second priority). Contoh struktur
dokumentasi workload disajikan pada Tabel 5.
Tabel 5. Rancangan Dokumentasi Workload
Query Fitur/Endpoint Tabel Pola Filtering Pola Sorti

| Query | Fitur/Endpoint | Tabel | Pola Filtering | Pola Sorting | Prioritas |
| :---: | :--- | :--- | :--- | :--- | :--- |
| Q12 | Pesan | messages | `conversation_id` | `created_at` DESC | Second Priority |
| Q27 | File | files | `workspace_id` dan `visibility` | `uploaded_at` DESC | Highest Priority |
```[cite: 6]


Tabel workload lengkap diperoleh setelah proses tracing dilakukan terhadap
source code dan dapat disajikan pada lampiran sebagai dokumentasi hasil
penelitian.
3.4.3. Rancangan Dataset
Pemilihan tiga skenario volume data tidak dimaksudkan untuk
merepresentasikan satu ukuran dataset tunggal, melainkan untuk mengamati
kecenderungan (trend) perubahan performa query seiring pertumbuhan volume
data. Perubahan tersebut tidak dapat diamati hanya dengan satu titik pengukuran,
karena pengujian pada volume tunggal berisiko memberikan kesimpulan yang
kebetulan berlaku pada ukuran data tertentu saja (Wohlin et al., 2024). Pengujian
pada beberapa skala volume data juga merupakan praktik umum pada penelitian
evaluasi performa index (Abbasi et al., 2024c; Liu et al., 2025). Ketiga skenario
dirancang untuk merepresentasikan tahapan pertumbuhan data yang realistis pada
konteks tenant Koladi, sebagaimana disajikan pada Tabel 6.


### Tabel 6. Rancangan Dataset Pengujian

| Skenario | Volume Data | Representasi Konteks | Tujuan Pengujian |
| :--- | :--- | :--- | :--- |
| Small | 1.000 baris | Volume data rendah | Mengukur kondisi awal performa query pada volume data rendah |
| Medium | 10.000 baris | Volume data menengah | Mengamati perubahan performa query pada volume data menengah |
| Large | 100.000 baris | Volume data tinggi | Mengamati kecenderungan performa query dan write overhead pada volume data yang lebih besar |
```[cite: 7]


Kenaikan volume antar skenario (1.000 → 10.000 → 100.000) mengikuti
rasio kenaikan yang seragam, yaitu sepuluh kali lipat pada setiap tahap, sehingga
perubahan performa dapat dibandingkan secara proporsional antar skenario tanpa
bias akibat interval volume yang tidak merata. Pendekatan rasio seragam ini sejalan
dengan skema scale factor yang digunakan pada penelitian evaluasi index berskala
besar, misalnya pengujian pada skala 1 GB, 10 GB, dan 100 GB dengan rasio
kenaikan yang sama (Abbasi et al., 2024c). Skala 100.000 baris dipilih sebagai
batas atas pengujian pada penelitian ini karena memberikan margin yang lebih
meyakinkan untuk mengamati kecenderungan write overhead maupun potensi
penurunan manfaat index secara terukur, dibandingkan skala yang hanya berjarak
dua kali lipat dari skenario Medium; semakin besar volume data yang diuji, semakin
jelas pula kecenderungan tersebut teramati, sejalan dengan temuan bahwa manfaat
teknik indexing cenderung semakin terlihat seiring pertumbuhan volume data
(Abbasi et al., 2024c). Skala ini tidak diklaim sebagai representasi universal seluruh
sistem SaaS multi-tenant, melainkan spesifik terhadap konteks dan batasan waktu
penelitian ini.
Perlu ditegaskan bahwa dataset pada ketiga skenario merupakan seeded data
yang dihasilkan menggunakan skrip seeding yang sama dengan proporsi distribusi
yang dijaga konsisten antar skenario (misalnya proporsi data per tenant/workspace,
proporsi data dengan status visibility tertentu, dan proporsi data yang soft-deleted),
bukan berasal dari production query logs. Dengan demikian, kecenderungan hasil
yang diperoleh mencerminkan pola workload yang telah diidentifikasi pada tahap
Workload Analysis, namun belum tentu mencerminkan seluruh variasi dan skew
data yang mungkin terjadi pada kondisi produksi riil. Dataset yang digunakan pada
kondisi baseline dan optimized dibuat identik pada setiap skenario volume data;
perbedaan kedua kondisi terletak semata pada konfigurasi index, yaitu baseline
menggunakan existing index sebelum optimasi, sedangkan optimized
menggunakan index tambahan yang telah dirancang pada tahap Index Design.
3.4.4. Rancangan Index
Rancangan index disusun berdasarkan hasil Execution Plan Analysis. Index
tidak ditambahkan secara umum pada seluruh kolom query, tetapi diarahkan pada
pola akses yang menunjukkan kebutuhan optimasi. Struktur rancangan index
menggunakan B-Tree dengan pendekatan composite index atau partial index.

### Tabel 7. Contoh Rancangan Index Awal

| ID | Tabel | Struktur Index | Jenis | Target Query |
| :---: | :--- | :--- | :--- | :---: |
| I01 | files | `workspace_id`, `uploaded_at` | Partial Composite B-Tree | Q27 |
| I02 | files | `workspace_id`, `uploaded_by`, `uploaded_at` | Composite B-Tree | Q27 |
```[cite: 8]

Rancangan tersebut menjadi dasar implementasi index pada database
PostgreSQL. Apabila hasil Execution Plan Analysis menunjukkan bahwa existing
index telah mencukupi untuk suatu query, query tersebut tidak harus memperoleh
index tambahan.
3.4.5. Rancangan Lingkungan Pengujian
Lingkungan pengujian menggunakan aplikasi Koladi dan database
PostgreSQL yang dijalankan pada lingkungan terkontrol. Aplikasi Laravel
digunakan untuk menyediakan source code dan struktur query, sedangkan
PostgreSQL digunakan untuk menjalankan query dan mengukur execution plan.

### Gambar 3. Lingkungan Pengujian

| Tahap / Komponen | Sub-Komponen / Detail | Deskripsi / Aktivitas |
| :--- | :--- | :--- |
| **Perangkat Pengujian** | Laptop/PC (Host OS) | Lingkungan host pengujian |
| **Docker** | • Laravel Application<br>• Nginx (Web Server)<br>• PostgreSQL (Database) | Kontainerisasi lingkungan aplikasi dan database |
| **Seeded PostgreSQL** | • Dataset Kecil (*Small Dataset*)<br>• Dataset Sedang (*Medium Dataset*)<br>• Dataset Besar (*Large Dataset*) | Populasi data PostgreSQL untuk pengujian |
| **EXPLAIN ANALYZE** | — | Menganalisis: Execution Plan dan Performa Query PostgreSQL |
```[cite: 9]


Agar eksperimen dapat direplikasi secara konsisten dan pengaruh
perubahan lingkungan terhadap hasil pengujian dapat diminimalkan, seluruh
komponen lingkungan pengujian ditetapkan pada versi dan spesifikasi tertentu serta
didokumentasikan secara rinci sebagaimana pada Tabel 8.

### Tabel 8. Spesifikasi Lingkungan Pengujian

| Komponen | Spesifikasi |
| :--- | :--- |
| CPU | Intel(R) Core(TM) i5-8250U CPU @ 1.60GHz |
| RAM | 12 GB |
| Storage | 256 GB SSD (Samsung MZVLB256HAHQ) |
| Sistem Operasi (host) | Windows 10 Pro |
| Docker | Version 29.1.3 |
| PostgreSQL | Version 15.15 (run via Docker) |
| Laravel | Version 12.43.1 |
| PHP | Version 8.2.32 |
```[cite: 10]

Seluruh spesifikasi dan versi pada Tabel 8 ditetapkan pada nilai yang tetap
dan tidak diubah selama rangkaian pengujian baseline maupun optimized
berlangsung. Selama pengujian berlangsung, aplikasi dan proses lain yang tidak
diperlukan pada perangkat pengujian diminimalkan untuk mengurangi potensi
gangguan terhadap hasil pengukuran. Pengujian baseline dan optimized dilakukan
pada environment, dataset, dan parameter query yang sama persis, dengan satu-
satunya perbedaan terletak pada konfigurasi index yang diterapkan, sehingga
perubahan Execution Time yang diamati dapat dibandingkan dengan lebih
terkontrol dan perbedaan konfigurasi utama antara kedua kondisi berasal dari
penerapan index.
3.5. Rencana Analisis
Rencana analisis mengolah hasil pengujian baseline dan optimized yang
diperoleh pada tahap Performance Evaluation (Subbab 3.2.5), menggunakan
parameter, teknik pengukuran, formula, dan kriteria evaluasi yang telah dijelaskan
pada Subbab 2.4.
3.5.1. Analisis Baseline
Query yang dipilih dari workload dijalankan pada tiga skenario volume data
(Small, Medium, Large) menggunakan EXPLAIN ANALYZE sebelum index
tambahan diterapkan. Setiap query dijalankan sebanyak 1 kali warm-up (tidak
dicatat) dan 10 kali pengukuran sesuai Subbab 2.4.2. Hasil yang dicatat meliputi
nilai median Execution Time, Planning Time, Query Cost, Scan Method, dan Rows
Removed by Filter, sebagai kondisi awal pembanding terhadap kondisi optimized.
3.5.2. Analisis Optimized
Setelah index diterapkan pada database PostgreSQL Koladi, query yang
sama dijalankan kembali menggunakan dataset dan skenario volume data yang
sama, dengan parameter yang dicatat identik dengan kondisi baseline untuk
menjaga keterbandingan hasil. Sebagaimana pada kondisi baseline, setiap query
dijalankan dengan 1 kali warm-up dan 10 kali pengukuran mengikuti Subbab 2.4.2,
dengan nilai median dicatat sebagai hasil akhir pengukuran pada kondisi optimized.
3.5.3. Analisis Read Performance
Read performance dianalisis dengan membandingkan hasil baseline dan
optimized menggunakan Improvement (%) dan Cost Reduction Ratio sesuai
persamaan (1) dan (2) pada Subbab 2.4.3. Perubahan Scan Method turut dianalisis,
khususnya transisi dari Sequential Scan menuju access path berbasis index (Index
Scan, Index Only Scan, atau Bitmap Heap Scan), dengan Rows Removed by Filter
sebagai indikator pendukung efisiensi filtering.
3.5.4. Analisis Write Performance
Write performance dianalisis dengan membandingkan waktu eksekusi
operasi INSERT, UPDATE, dan DELETE sebelum dan sesudah penerapan index,
menggunakan persamaan Write Overhead (%) pada Subbab 2.4.4, untuk setiap
tabel yang menerima index tambahan (files, tasks, messages). Pada operasi
DELETE yang menggunakan mekanisme soft delete pada Koladi, pengukuran
merepresentasikan perubahan nilai deleted_at sesuai mekanisme penghapusan data
yang digunakan oleh aplikasi. Setiap pengulangan dilakukan pada kondisi dataset
yang setara dengan mengembalikan dataset ke keadaan awal sebelum pengukuran
berikutnya.
3.5.5. Analisis Read-Write Trade-off
Hasil read performance dan write performance dibandingkan menggunakan
pendekatan interpretasi trade-off kontekstual yang telah dijelaskan pada Subbab
2.4.5. Analisis dilakukan dengan mempertimbangkan konsistensi perubahan
Execution Time, peningkatan performa baca, write overhead, perubahan Scan
Method, Query Cost, dan Rows Removed by Filter pada setiap tabel dan skenario
volume data. Interpretasi efektivitas index dilakukan berdasarkan keseluruhan hasil
pengujian dan konsistensi hasil pada setiap skenario, tanpa menggunakan batas
persentase tertentu sebagai kriteria penerimaan atau penolakan index.
3.5.6. Interpretasi Hasil
Hasil baseline dan optimized dibandingkan pada setiap skenario volume
data, mempertimbangkan Execution Time, Query Cost, Scan Method, Planning
Time, dan Rows Removed by Filter, untuk menentukan efektivitas penerapan
Workload-Based Index Optimization pada workload Koladi. Kesimpulan dibatasi
pada lingkungan dan skenario pengujian yang digunakan, sehingga tidak
digeneralisasikan secara langsung ke seluruh sistem SaaS multi-tenant lainnya.