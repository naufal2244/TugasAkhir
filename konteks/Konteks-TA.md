# MASTER CONTEXT — TUGAS AKHIR NAUFAL FADHILAH ARDHANI

## 1. Identitas dan Konteks Akademik

Nama: Naufal Fadhilah Ardhani  
NIM: 4342301061  
Program Studi: Teknologi Rekayasa Perangkat Lunak  
Institusi: Politeknik Negeri Batam

Saya sedang menyusun proposal Tugas Akhir (TA) dengan fokus pada evaluasi kinerja teknis modul Decision Support System (DSS) dalam aplikasi Koladi. Saya membutuhkan bantuan AI untuk menyusun bagian proposal secara akademik, logis, konsisten, dan berbasis sumber penelitian yang benar-benar tersedia.

AI harus berperan sebagai asisten penulisan akademik dan mitra analisis literatur, bukan sekadar generator teks. Setiap bagian harus mempunyai argumentasi yang jelas, mendukung tujuan penelitian, serta konsisten dengan ruang lingkup dan metode penelitian.

## 2. Judul Penelitian

"The System Knows Your Team's Performance, But Takes Forever to Tell You": A Comparative Study of Caching Strategy and Query Optimization on a Decision Support System in a Collaborative Virtual Office Application

Judul ini berfokus pada perbandingan kondisi sistem sebelum dan sesudah penerapan strategi caching dan optimasi kueri ORM pada jalur eksekusi DSS di Koladi.

## 3. Tujuan Utama Penelitian

Penelitian bertujuan mengevaluasi perubahan kinerja teknis modul DSS Koladi setelah penerapan Redis Cache-Aside dan optimasi kueri ORM secara bersamaan.

Evaluasi dilakukan melalui empat metrik:
1. Response time, dalam milidetik.
2. SQL query count, dalam jumlah kueri per permintaan.
3. Memory utilization, dalam megabita.
4. Cache hit ratio, dalam persentase.

Penelitian tidak bertujuan mengukur persepsi pengguna, kepuasan pengguna, user experience, perilaku pengguna, atau kualitas keputusan manajerial. Kesimpulan harus dibatasi pada kinerja teknis yang dapat diukur secara instrumental.

## 4. Objek dan Lingkungan Penelitian

Objek penelitian adalah Koladi, aplikasi collaborative virtual office berbasis Software as a Service (SaaS) dengan karakteristik teknis berikut:
- Laravel 12.x.
- PHP 8.2 atau kompatibel dengan persyaratan proyek.
- PostgreSQL 15.
- Eloquent ORM.
- Redis 7.x pada konfigurasi yang dioptimasi.
- Docker Compose.
- Arsitektur aplikasi multi-tenant.

Modul DSS mengolah data aktivitas tim yang saling berelasi menjadi indikator produktivitas workspace. Pipeline kalkulasi melibatkan DSSService, SnapshotManager, MetricsCalculator, TrendAnalyzer, dan RecommendationEngine. Sejumlah data tambahan untuk modal-data juga diolah melalui fungsi getUrgentTasks() dan getWorkloadDistribution().

Modul ini menghitung 22 metrik produktivitas mentah dan menggunakannya dalam proses analisis serta rekomendasi kinerja workspace.

## 5. Ruang Lingkup Penelitian

Penelitian berfokus pada lapisan DSS Modal, bukan seluruh aplikasi Koladi.

Dua endpoint yang diteliti:
- GET /statistik/suggestions
- GET /statistik/modal-data

Endpoint tersebut berkaitan dengan penyajian rekomendasi dan informasi tindak lanjut kinerja workspace.

Hasil penelitian tidak boleh digeneralisasikan sebagai bukti bahwa seluruh modul atau keseluruhan aplikasi Koladi mengalami peningkatan kinerja yang sama.

## 6. Kondisi Baseline dan Permasalahan Teknis

Pemeriksaan kode sumber dan profiling menggunakan Laravel Telescope mengidentifikasi inefisiensi pada mekanisme caching dan pemrosesan data melalui ORM.

### 6.1. Mekanisme caching berbasis basis data

Pada kondisi baseline, hasil kalkulasi DSS disimpan melalui mekanisme snapshot dalam tabel workspace_performance_snapshots.

Pemeriksaan validitas snapshot tetap membutuhkan akses SQL. Dengan demikian, mekanisme tersebut tidak sepenuhnya menghilangkan akses ke PostgreSQL ketika hasil kalkulasi sebelumnya tersedia.

Pada endpoint modal-data juga terdapat partial cache hit: metrik utama dapat diambil dari snapshot, tetapi informasi tugas mendesak dan distribusi beban kerja masih dihitung ulang.

Akibatnya, pengambilan snapshot tidak selalu menghilangkan biaya pemrosesan dan kueri basis data pada setiap permintaan.

### 6.2. Pengambilan kolom ORM yang berlebihan

Fungsi getTasksInPeriod() menggunakan eager loading terhadap relasi assignedUsers tanpa membatasi kolom.

Padahal, kalkulasi distribusi beban kerja hanya memerlukan kolom id dari pengguna yang terkait.

Pola ini merupakan Unnecessary Data Retrieval, yaitu pengambilan data persisten yang tidak diperlukan oleh logika aplikasi. Pengambilan kolom secara berlebihan berpotensi meningkatkan biaya materialisasi objek dan penggunaan memori server.

### 6.3. N+1 query pada relasi checklists

Relasi checklists tidak dimuat terlebih dahulu pada proses getTasksInPeriod().

Ketika calculateProgressMetrics() memanggil getProgressPercentage() pada setiap task, akses terhadap relasi checklists memicu lazy loading. Akibatnya, satu kueri tambahan dapat muncul untuk setiap task yang diproses.

Pola ini merupakan masalah N+1 query dan dapat membuat jumlah kueri meningkat seiring bertambahnya jumlah tugas.

### 6.4. Kueri berulang di dalam perulangan

Fungsi getWorkloadDistribution() menjalankan Task::whereHas() di dalam perulangan anggota workspace.

Pola query-in-loop ini membuat jumlah operasi basis data berpotensi meningkat mengikuti jumlah anggota yang diproses.

Solusi yang direncanakan adalah mengambil data secara batch dan melakukan agregasi di dalam memori, tanpa mengubah hasil perhitungan bisnis yang seharusnya dihasilkan.

## 7. Intervensi Optimasi

Penelitian mengevaluasi dua kelompok intervensi yang diterapkan secara bersamaan.

### 7.1. Redis Cache-Aside

Mekanisme snapshot berbasis basis data diganti dengan Redis Cache-Aside.

Alur umum:
1. Sistem memeriksa keberadaan hasil dalam Redis.
2. Jika cache hit, hasil yang tersedia digunakan tanpa menjalankan ulang seluruh pipeline DSS.
3. Jika cache miss, sistem menjalankan proses kalkulasi yang diperlukan.
4. Hasil kalkulasi disimpan ke Redis dan dikembalikan kepada pemanggil.

Implementasi juga perlu memperhatikan cakupan data yang di-cache, khususnya pada modal-data, yang memiliki data tambahan selain metrik utama.

### 7.2. Optimasi Eloquent ORM

Optimasi ORM mencakup:
- Membatasi kolom relasi assignedUsers menjadi kolom id yang diperlukan.
- Memuat relasi checklists melalui eager loading dengan kolom yang dibutuhkan, termasuk id, task_id, dan is_done.
- Mengganti kueri berulang di dalam perulangan getWorkloadDistribution() dengan batch retrieval dan in-memory aggregation.

Contoh perubahan eager loading:

Sebelum:
with(['assignedUsers'])

Sesudah:
with(['assignedUsers:id', 'checklists:id,task_id,is_done'])

Optimasi ini diarahkan untuk mengurangi pengambilan data yang tidak diperlukan dan menghilangkan pola N+1 pada relasi checklists.

Logika bisnis, formula metrik produktivitas, dan kontrak API publik harus tetap dipertahankan.

## 8. Hubungan Antara Kedua Intervensi

Redis Cache-Aside dan optimasi ORM saling melengkapi.

Optimasi ORM mengurangi biaya pemrosesan ketika permintaan mengalami cache miss dan harus menjalankan pipeline DSS.

Redis Cache-Aside mengurangi frekuensi pipeline tersebut perlu dijalankan ulang pada permintaan berulang.

Oleh karena itu, dampak keduanya dievaluasi sebagai kombinasi intervensi. Penelitian tidak boleh mengklaim telah mengisolasi kontribusi kausal masing-masing teknik secara independen apabila desain eksperimennya hanya membandingkan kondisi baseline dan kondisi optimasi gabungan.

## 9. Empat Metrik Evaluasi

### 9.1. Response time

Mengukur waktu yang diperlukan sistem untuk memproses dan merespons permintaan endpoint. Satuan: milidetik.

### 9.2. SQL query count

Mengukur jumlah kueri SQL yang dieksekusi dalam satu permintaan. Metrik ini membantu mengidentifikasi beban akses basis data yang tidak selalu terlihat dari response time saja.

### 9.3. Memory utilization

Mengukur penggunaan memori server selama pemrosesan permintaan. Metrik ini relevan dengan biaya materialisasi objek ORM dan perubahan pemrosesan data.

### 9.4. Cache hit ratio

Mengukur proporsi permintaan atau pemeriksaan cache yang menghasilkan cache hit sesuai definisi operasional pengukuran yang digunakan.

Cache hit ratio harus diinterpretasikan bersama SQL query count. Rasio hit yang tinggi tidak otomatis membuktikan bahwa akses ke PostgreSQL telah dihilangkan, khususnya pada mekanisme snapshot berbasis basis data.

Definisi operasional, pembilang, penyebut, dan cara pengukuran setiap metrik harus konsisten antara kondisi baseline dan optimasi.

## 10. Rancangan Eksperimen

Penelitian menggunakan pendekatan kuantitatif dengan desain eksperimental komparatif before-after.

Karakteristik rancangan yang direncanakan:
- Dua konfigurasi: baseline dan optimized.
- Dua endpoint: suggestions dan modal-data.
- Tiga ukuran dataset: small, medium, dan large.
- Dua kondisi cache: cache hit dan cache miss.
- Sepuluh pengulangan valid untuk setiap kombinasi.
- Tiga iterasi warm-up yang tidak dimasukkan ke analisis.

Jumlah pengukuran valid:
3 ukuran dataset × 2 endpoint × 2 konfigurasi × 2 kondisi cache × 10 pengulangan = 240 pengukuran.

Benchmark dilakukan secara sekuensial menggunakan runner berbasis Node.js. Penelitian ini bukan load testing, stress testing, concurrency testing, atau throughput testing.

Dataset sintetis dihasilkan melalui Laravel Seeder dan Factory untuk mendukung pengujian terkontrol serta replikasi eksperimen.

Instrumen pengukuran yang relevan meliputi Laravel Telescope untuk profiling, pencatatan kueri SQL, memory_get_peak_usage(), Redis INFO STATS, Redis INFO MEMORY, dan runner benchmark.

Rencana analisis statistik menggunakan paired-sample t-test dengan α = 0,05 dan perhitungan persentase perubahan atau peningkatan. Kesesuaian pasangan observasi, asumsi uji statistik, dan definisi perhitungan improvement harus diverifikasi sebelum analisis final.

## 11. Batasan Penelitian

Penelitian tidak mencakup:
- Pengujian kepuasan atau persepsi pengguna.
- Evaluasi UX.
- Evaluasi kualitas keputusan strategis.
- Perubahan formula perhitungan metrik DSS.
- Perubahan kontrak API publik.
- Generalisasi hasil ke seluruh modul Koladi.
- Load testing, stress testing, concurrency testing, throughput testing, atau latensi P95.

Penelitian juga tidak mengklaim pengukuran independen terhadap kontribusi masing-masing teknik optimasi jika keduanya hanya diuji sebagai satu konfigurasi gabungan.

## 12. Konteks Bab I yang Sudah Disusun

Bab I mencakup:
- 1.1 Latar Belakang.
- 1.2 Rumusan Masalah.
- 1.3 Tujuan Penelitian.
- 1.4 Batasan Masalah.
- 1.5 Manfaat Penelitian.

Rumusan masalah utama:

Sejauh mana penerapan Redis Cache-Aside dan optimasi kueri ORM secara bersamaan memengaruhi kinerja modul DSS pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data, ditinjau dari response time, SQL query count, memory utilization, dan cache hit ratio?

Tujuan penelitian:
1. Menerapkan Redis Cache-Aside dan optimasi kueri ORM pada dua endpoint DSS tersebut.
2. Membandingkan kinerja sebelum dan sesudah optimasi berdasarkan empat metrik.
3. Menganalisis perubahan keempat metrik untuk mengevaluasi dampak kombinasi optimasi terhadap kinerja modul DSS.

Bab I menjadi landasan utama bagi penyusunan Bab II. Bab II harus menjelaskan teori dan bukti penelitian terdahulu yang relevan dengan masalah, teknik optimasi, metrik evaluasi, serta posisi kontribusi penelitian ini.

## 13. File Sumber yang Akan Disediakan kepada AI Agent

Saya akan menyediakan empat file utama:

### A. proposal.md

Berisi isi Bab I yang telah disusun.

Fungsi:
- Menjadi sumber utama untuk memahami narasi, masalah, tujuan, batasan, dan arah penelitian.
- Menjadi acuan untuk menjaga kesinambungan antara seluruh bab
- Menentukan istilah, ruang lingkup, dan formulasi masalah yang harus dipertahankan.

Jika terdapat perbedaan antara ringkasan konteks ini dan isi terbaru proposal.md, prioritaskan proposal.md untuk redaksi dan ruang lingkup proposal yang sedang dikerjakan.

### B. Lit Review 1.md
### C. Lit Review 2.md
### D. Lit Review 3.md

Ketiga file tersebut berisi hasil kajian terhadap lima penelitian terdahulu yang sudah dipilih dan dianalisis.

Fungsi:
- Menjadi sumber utama untuk menyusun Subbab 2.1 Penelitian Terkait.
- Menjadi dasar pengambilan informasi tentang tujuan, metode, objek, teknologi, metrik, hasil, keterbatasan, dan relevansi setiap studi.
- Menjadi sumber untuk membandingkan penelitian terdahulu dan menyusun sintesis research gap.

Jangan berasumsi bahwa setiap file berisi satu penelitian yang berbeda. Baca ketiganya sebagai satu kumpulan materi kajian, lalu identifikasi lima studi yang memang telah dipilih.

Jangan menambah penelitian keenam atau mengganti studi yang telah dipilih tanpa instruksi eksplisit.

## 14. Kerangka Argumentasi Literatur yang Diharapkan

Lima penelitian yang dipilih diarahkan untuk membangun alur argumentasi berikut. Alur ini merupakan panduan analitis, bukan kewajiban bahwa setiap studi hanya boleh masuk ke satu kategori.

1. Penelitian mengenai optimasi performa aplikasi Laravel.
   Tujuan: menjelaskan konteks teknologi yang paling dekat dengan Koladi dan bukti optimasi performa aplikasi berbasis Laravel.

2. Penelitian mengenai Redis caching.
   Tujuan: menjelaskan mekanisme caching, alasan teknis penggunaan Redis, serta dampaknya terhadap akses basis data dan response time.

3. Penelitian mengenai masalah N+1 dan optimasi ORM.
   Tujuan: menjelaskan bagaimana pola pengambilan relasi dan kueri yang tidak efisien dapat meningkatkan jumlah operasi basis data.

4. Penelitian mengenai biaya memori akibat penggunaan ORM.
   Tujuan: menjelaskan hubungan antara pengambilan kolom yang berlebihan, materialisasi objek, dan penggunaan memori server.

5. Penelitian yang paling dekat dengan integrasi caching dan optimasi ORM.
   Tujuan: menjadi pembanding utama untuk menilai persamaan, perbedaan, dan posisi kontribusi penelitian Koladi.

Kelima studi harus disintesis secara komparatif. Jangan membuat lima ringkasan yang berdiri sendiri tanpa hubungan argumentatif.

## 15. Prinsip Penyusunan Subbab 2.1 Penelitian Terkait

Subbab 2.1 harus membantu menjawab:
- Apa yang telah diteliti oleh penelitian sebelumnya?
- Apa metode dan teknologi yang digunakan?
- Apa hasil utama yang benar-benar dilaporkan?
- Apa relevansinya dengan permasalahan teknis Koladi?
- Apa persamaan dan perbedaannya dengan penelitian ini?
- Apa keterbatasan atau ruang yang belum dijawab oleh studi-studi tersebut?
- Bagaimana sintesisnya mendukung posisi penelitian ini?

Perbandingan harus mencakup aspek yang memang tersedia dalam sumber, misalnya objek sistem, teknologi, strategi caching, teknik ORM, metrik pengukuran, desain eksperimen, hasil kuantitatif, dan batasan penelitian.

Tidak semua aspek harus dipaksakan jika informasinya tidak tersedia.

Research gap harus dirumuskan dari perbandingan konkret kelima studi. Hindari klaim absolut seperti "belum pernah ada penelitian" jika sumber yang tersedia hanya menunjukkan bahwa studi semacam itu tidak ditemukan dalam kumpulan literatur yang ditinjau.

Penelitian ini memiliki fokus khusus pada kombinasi Redis Cache-Aside dan optimasi kueri ORM dalam jalur eksekusi DSS yang melakukan agregasi data aktivitas tim secara relasional, dengan empat metrik evaluasi dalam eksperimen terkontrol.

## 16. Aturan Keakuratan Akademik

1. Gunakan isi file sumber sebagai dasar utama.
2. Jangan mengarang judul, penulis, tahun, nama jurnal, DOI, metode, hasil, persentase peningkatan, atau keterbatasan penelitian.
3. Jangan menganggap angka dari suatu studi berlaku pada sistem Koladi.
4. Bedakan hasil empiris penelitian terdahulu dari interpretasi penulis proposal.
5. Jangan mengubah makna temuan hanya agar terlihat mendukung penelitian ini.
6. Jika suatu informasi tidak ditemukan, nyatakan bahwa informasi tersebut tidak tersedia dalam sumber.
7. Jika terdapat ketidakkonsistenan antardokumen, tandai dan minta klarifikasi atau gunakan sumber yang lebih otoritatif sesuai konteks.
8. Jangan menambahkan referensi eksternal tanpa kebutuhan dan izin yang jelas.
9. Pertahankan sitasi yang dapat ditelusuri ke sumbernya.
10. Jangan menggunakan sumber yang hanya disebut secara sekilas seolah-olah sudah dibaca dan diverifikasi secara penuh.
11. Hindari klaim kausal yang lebih kuat daripada desain penelitian yang dilaporkan.
12. Gunakan bahasa Indonesia akademik yang alami, lugas, dan tidak bertele-tele.

## 17. Gaya Penulisan yang Diinginkan

Gunakan bahasa Indonesia akademik untuk proposal, dengan ketentuan:
- Formal, jelas, dan logis.
- Mengutamakan sintesis daripada repetisi.
- Satu paragraf memiliki gagasan utama yang jelas.
- Transisi antarparagraf menunjukkan hubungan argumentasi.
- Istilah teknis dipertahankan secara konsisten.
- Tidak memakai kalimat generik yang sekadar menyatakan bahwa teknologi tertentu penting atau berkembang pesat tanpa dukungan sumber.
- Tidak menambahkan klaim normatif yang tidak diperlukan.
- Tidak mengulang isi Bab I secara panjang lebar.
- Tidak menggunakan gaya promosi untuk aplikasi Koladi.
- Tidak menyamakan peningkatan performa teknis dengan peningkatan kepuasan pengguna.

## 18. Instruksi Umum untuk AI Agent

Sebelum menyusun suatu subbab:
1. Identifikasi bagian proposal yang sedang dikerjakan dan fungsinya.
2. Baca file yang relevan secara menyeluruh atau pada bagian yang benar-benar dibutuhkan.
3. Petakan bukti yang tersedia sebelum menulis.
4. Periksa kesesuaian antara sumber literatur dan narasi Bab I.
5. Susun argumentasi yang menjawab kebutuhan akademik subbab tersebut.
6. Periksa kembali akurasi sitasi, angka, terminologi, dan konsistensi metodologis.

Jangan langsung menulis hanya berdasarkan ringkasan master context jika file sumber tersedia. Master context ini berfungsi sebagai peta penelitian, sedangkan file sumber tetap menjadi acuan untuk fakta dan bukti terperinci.

Fokus pekerjaan saat ini adalah menyusun Subbab 2.1 Penelitian Terkait berdasarkan lima studi yang sudah dikaji dalam Lit Review 1.md sampai Lit Review 3.md, dengan menjaga kesinambungan terhadap Bab I dalam proposal.md.