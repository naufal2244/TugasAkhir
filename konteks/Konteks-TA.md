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

Instrumen pengukuran yang digunakan terdiri atas rangkaian instrumen terpadu:
1. Middleware Instrumentasi Laravel (pencatatan kueri SQL via DB::listen, memory peak usage via memory_get_peak_usage, dan rasio cache Redis).
2. Node.js Benchmark Runner (eksekusi HTTP sekuensial terkontrol, protokol 3 warm-up dibuang + 10 repetisi valid, dan pencatatan response time end-to-end via process.hrtime).
3. CSV Validator (skrip verifikasi otomatis keutuhan 240 baris matriks data berpasangan).
4. Laravel Telescope (diisolasi khusus untuk diagnosis/profiling awal mendeteksi bottleneck kueri sebelum eksperimen).

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

Bab I telah tuntas disusun pada `proposal.md` mencakup:
- 1.1 Latar Belakang.
- 1.2 Rumusan Masalah.
- 1.3 Tujuan Penelitian.
- 1.4 Batasan Masalah.
- 1.5 Manfaat Penelitian.

Rumusan masalah (3 butir berstruktur simetris):
1. Bagaimana merancang dan menerapkan strategi Redis Cache-Aside serta optimasi kueri Eloquent ORM (selective eager loading dan in-memory aggregation) pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data modul DSS Koladi? *(Ranah Rekayasa)*
2. Seberapa besar peningkatan kinerja modul DSS Koladi pada kondisi sesudah optimasi (optimized) dibandingkan kondisi sebelum optimasi (baseline), ditinjau dari parameter response time, SQL query count, memory utilization, dan cache hit ratio? *(Ranah Kuantitatif Terukur)*
3. Seberapa besar pengaruh variasi skala volume dataset sintetis (Small, Medium, Large) terhadap efektivitas optimasi tersebut, serta apakah peningkatan performa yang dihasilkan terbukti signifikan secara statistik melalui paired-sample t-test? *(Ranah Skalabilitas & Uji Hipotesis)*

Tujuan penelitian (korespondensi 1-ke-1 simetris):
1. Menerapkan strategi Redis Cache-Aside serta optimasi kueri Eloquent ORM (selective eager loading dan in-memory aggregation) pada endpoint GET /statistik/suggestions dan GET /statistik/modal-data modul DSS Koladi.
2. Membandingkan kinerja modul DSS sebelum (baseline) dan sesudah optimasi (optimized) berdasarkan parameter response time, SQL query count, memory utilization, dan cache hit ratio.
3. Menganalisis pengaruh intervensi simultan tersebut di bawah variasi volume dataset sintetis (Small, Medium, Large) serta mengevaluasi signifikansi perbedaannya melalui paired-sample t-test dan persentase perbaikan performa.

Batasan Masalah (1.4):
1. Objek penelitian difokuskan secara eksklusif pada modul DSS Koladi (dua endpoint: GET /statistik/suggestions dan GET /statistik/modal-data).
2. Intervensi optimasi dibatasi pada Redis Cache-Aside dan optimasi ORM (selective eager loading & batch in-memory aggregation).
3. Formula 22 metrik dan kontrak respons API publik tidak diubah (behavior-preserving).
4. Evaluasi dibatasi pada 4 metrik teknis kuantitatif: response time, SQL query count, memory utilization, dan cache hit ratio.
5. Skenario pengujian menggunakan dataset sintetis Small (50/5), Medium (200/15), Large (500/30) dieksekusi secara sekuensial menghasilkan 240 observasi valid.
6. Tidak mencakup concurrent load testing, stress testing, throughput, maupun persepsi/kepuasan pengguna (UX).

## 12.B. Konteks Bab II yang Sudah Selesai Disusun

Bab II telah selesai disusun 100% pada `proposal.md` (249 baris) dengan struktur metodologis yang kokoh:
- **2.1. Penelitian Terkait:** Sintesis komparatif dari 5 studi utama mengenai optimasi Laravel, Redis caching, N+1 query ORM, memori ORM, dan benchmark sistem web.
- **2.2. Landasan Teori:** Teori DSS berbasis web (Cai & Chen, 2021; Koornneef et al., 2020), Arsitektur Caching dan Redis (Privalov & Stupina, 2024; Taipalus, 2024; An & Cao, 2022), Teori Active Record & Inefisiensi Eloquent ORM (Alshemaimri et al., 2021; Turcotte et al., 2022; Yusmita et al., 2025), serta Profil Objek Penelitian Koladi.
- **2.3. Metode Pengembangan Produk:** Mengadopsi kerangka kerja *Refactoring Lifecycle* (Mens & Tourwé, 2004) berlandaskan prinsip dasar *behavior-preserving* (Fowler, 2018), mencakup 6 langkah terstruktur: (2.3.1) Identifikasi Lokasi, (2.3.2) Penentuan Teknik, (2.3.3) Penjaminan Preservasi Perilaku via *automated equivalence tests* untuk 22 metrik & 5 rekomendasi, (2.3.4) Penerapan Refaktorisasi, (2.3.5) Evaluasi Efek Kualitas, dan (2.3.6) Pemeliharaan Konsistensi Sistem (Docker Compose stability, verifikasi ketiadaan efek samping pada modul Kanban, dan penyelarasan artefak nonkode).
- **2.4. Metode Analisis yang Digunakan:** Mengadopsi *Experimental Software Engineering* (Wohlin et al., 2000, 2012) dengan rancangan *Controlled Before-After Benchmarking* sekuensial terisolasi. Meliputi 4 variabel terikat, 4 instrumen pengujian terpadu (Middleware Laravel, Node.js Runner, CSV Validator, Telescope), protokol 3 warm-up dibuang + 10 repetisi = 240 observasi valid, serta 4 tahapan analisis data (Deskriptif, Persentase Perbaikan, Paired-Sample t-Test α=0,05 dengan pelaporan $p < 0,001$ untuk data deterministik, dan Joint Cache Analysis).

## 12.C. Konteks Bab III yang Sudah Selesai Disusun

Bab III telah selesai disusun secara komprehensif pada `proposal.md` dengan keselarasan penuh terhadap metodologi Bab II:
- **3.1. Alat dan Bahan:** 
  - *Alat (Tabel 2):* Perangkat keras komputasi (Apple M2 Pro, RAM 16 GB, SSD 1 TB, macOS Sequoia); lingkungan terkontainerisasi Docker Compose (`koladi_net`) mencakup PHP 8.2 / Laravel 12, PostgreSQL 15, dan Redis 7; serta rangkaian instrumen benchmark terpadu (Middleware `BenchmarkInstrumentation.php`, Node.js Benchmark Runner `benchmark-dss.js`, skrip `validate-csv.js`, dan Laravel Telescope untuk profiling kualitatif awal).
  - *Bahan (Tabel 3):* Kode sumber modul DSS Koladi, rute dua endpoint analitis (`/suggestions` dan `/modal-data`), skema relasional basis data, dataset sintetis bertingkat (`ResearchDataSeeder.php`), data mentah CSV 240 observasi, rancangan refaktorisasi kode, skrip uji ekuivalensi, serta literatur pendukung.
- **3.2. Jalannya Penelitian:** 
  Mengadopsi enam aktivitas baku dalam siklus hidup refaktorisasi (*Refactoring Lifecycle*) Mens & Tourwé (2004) dan Fowler (2018):
  1. *3.2.1. Identifikasi Titik Hambatan:* Profiling empiris kualitatif via Laravel Telescope mendeteksi kueri berulang pada tabel snapshot, partial cache hit modal-data, column over-fetching, N+1 checklists, dan query-in-loop per anggota (tercatat di Lampiran A).
  2. *3.2.2. Penentuan Strategi Optimasi:* Perancangan Redis Cache-Aside menggantikan snapshot basis data, selective eager loading (`with(['assignedUsers:id', 'checklists:id,task_id,is_done'])`), dan batch retrieval dengan in-memory aggregation.
  3. *3.2.3. Penjaminan Preservasi Perilaku:* Uji regresi otomatis memastikan output payload JSON 22 metrik dan 5 rekomendasi menghasilkan nilai 100% identik sebelum kode diubah.
  4. *3.2.4. Penerapan Refaktorisasi:* Eksekusi transformasi kode pada `DSSService`, `SnapshotManager`, controller, dan aktivasi driver Redis 7.
  5. *3.2.5. Penilaian Efek Kualitas:* Pengujian komparatif controlled benchmarking sekuensial terisolasi dalam Docker `koladi_net`, mengeksekusi 3 warm-up requests (dibuang) + 10 repetisi valid menghasilkan 240 data observasi valid (120 baseline dan 120 optimized).
  6. *3.2.6. Pemeliharaan Konsistensi Sistem:* Verifikasi orkestrasi kontainer Docker, uji regresi fungsional ketiadaan efek samping pada modul Kanban, dan pembaruan artefak nonkode (.env, arsitektur, dokumentasi API).
  *(Catatan: Mekanisme TaskObserver/invalidasi event berbasis event telah 100% dieliminasi dari naskah karena tidak pernah diukur before-after-nya pada pengujian read path).*
- **3.3. Tahapan Pengembangan Aplikasi:**
  Menegaskan penolakan metodologi SDLC konvensional (Waterfall, Agile/Scrum, RAD) karena penelitian bukan membangun aplikasi dari awal (*greenfield*), melainkan evolusi/refaktorisasi sistem berjalan (*brownfield software engineering*). Metodologi resmi yang diadopsi adalah *Refactoring Lifecycle* Mens & Tourwé (2004) dengan prinsip *behavior-preserving* (Fowler, 2018).
- **3.4. Rencana Perancangan Sistem:**
  - *3.4.1. Gambaran Umum Sistem:* Diagram 5 kotak arsitektur target (Gambar 2): Pengguna $\rightarrow$ Aplikasi Laravel 12 $\rightarrow$ Modul DSS $\rightarrow$ Redis 7 Cache-Aside $\leftrightarrow$ PostgreSQL 15.
  - *3.4.2. Rancangan Alur Komputasi Modul DSS dan Cache-Aside:* Diagram sekuensial UML (Gambar 3) yang memodelkan alur *Cache Hit* (langsung dari RAM Redis, 0 kueri komputasi DSS) dan alur *Cache Miss* (eksekusi kueri ORM teroptimasi, kalkulasi 22 metrik, batch aggregation, dan penyimpanan payload utuh via `SETEX` berdurasi 3.600 detik).
  - *3.4.3. Rancangan Dataset Pengujian:* Spesifikasi rancangan dataset sintetis deterministik via Laravel Seeder/Factory: Small (50 tugas/5 anggota), Medium (200 tugas/15 anggota), Large (500 tugas/30 anggota) dengan proporsi status 40% completed, 35% in progress, 25% to-do guna menguji batas beban N+1 checklists dan query-in-loop.
  - *3.4.4. Rancangan Transformasi Kueri ORM dan Skema Cache Redis:* Dokumentasi transformasi sintaks Eloquent sebelum vs sesudah, serta skema kunci Redis hierarkis (`dss:{endpoint}:workspace_{id}:start_{date}:end_{date}`) untuk menjamin isolasi multi-tenant dan menyelesaikan masalah *partial cache hit* pada endpoint `modal-data`.
  - *3.4.5. Rancangan Lingkungan Pengujian dan Instrumentasi:* Topologi jaringan Docker `koladi_net` (Gambar 5), arsitektur dual-layer instrumentation (Client Node.js runner dengan `process.hrtime.bigint()` dan Server Middleware dengan `DB::listen` serta `memory_get_peak_usage(true)`), dan spesifikasi statis komputer pengujian (Tabel 6).
- **3.5. Rencana Analisis:**
  Telah dirumuskan secara final dalam bentuk narasi padat tanpa tabel ekstra (mengikuti preseden Dzakwan) yang terbagi dalam 6 sub-subbab:
  1. *3.5.1. Analisis Baseline:* Pengolahan 120 data observasi awal (Mean, SD, Min, Max) sebagai tolok ukur acuan pembanding dasar (*baseline reference*).
  2. *3.5.2. Analisis Optimized:* Pengolahan 120 data observasi teroptimasi pada kondisi identik untuk menilai stabilitas performa sistem.
  3. *3.5.3. Analisis Efisiensi Kueri SQL dan Penggunaan Memori:* Evaluasi eliminasi kueri N+1 checklists dan query-in-loop per anggota (deterministik $p < 0{,}001$), serta evaluasi penghematan alokasi RAM server PHP akibat selective eager loading.
  4. *3.5.4. Analisis Response Time dan Skalabilitas Kinerja:* Evaluasi persentase perbaikan latensi (*Improvement %*) pada kondisi Cache Miss dan Cache Hit (kecepatan RAM vs disk), serta kurva ketahanan skalabilitas terhadap pertumbuhan volume data (Small $\rightarrow$ Medium $\rightarrow$ Large).
  5. *3.5.5. Analisis Keterpaduan Cache Hit dan Akses Basis Data (Joint Cache Analysis):* Evaluasi terpadu antara Cache Hit Ratio dan SQL Query Count untuk membuktikan tercapainya *full database bypass* (0 kueri komputasi) pada Redis Cache-Aside dan tuntasnya masalah *partial cache hit* pada `modal-data`.
  6. *3.5.6. Interpretasi Hasil dan Uji Hipotesis:* Uji signifikansi berpasangan *Paired-Sample t-Test* ($\alpha = 0{,}05$), ambang batas variasi acak (*measurement noise*) 3%, dan penegasan batasan generalisasi khusus pada modul DSS Koladi dan lingkungan pengujian terkontainerisasi lokal.

## 13. File Sumber yang Akan Disediakan kepada AI Agent

Saya akan menyediakan file utama:

### A. proposal.md

Berisi naskah utuh proposal Tugas Akhir mencakup Bab I, Bab II, dan Bab III (Subbab 3.1 s.d. 3.4.5 lengkap, dan draf Subbab 3.5 siap disematkan).

Fungsi:
- Menjadi sumber utama untuk memahami narasi, masalah, tujuan, batasan, teori, metodologi refaktorisasi, perancangan sistem, dan metode analisis.
- Menjadi acuan untuk menjaga kesinambungan antar-bab.
- Menentukan istilah baku, ruang lingkup, dan formulasi masalah yang harus dipertahankan.

Jika terdapat perbedaan antara ringkasan konteks ini dan isi terbaru proposal.md, prioritaskan proposal.md untuk redaksi dan ruang lingkup proposal yang sedang dikerjakan.

### B. Berkas Log Sesi Penelitian (`TugasAkhir/log/`)
- `Log1.md` s.d. `Log4.md`: Dokumentasi awal perumusan latar belakang, literatur review, dan draf Bab I-II.
- `Log5.md`: Rangkuman pembersihan konsep Observer/Invalidasi event, penegasan 6 tahap Mens & Tourwé, dan audit berkas benchmark Git.
- `Log6.md`: Dokumentasi penyusunan teknis Subbab 3.4.3 s.d. 3.4.5 (dataset sintetis, skema Redis, dual-layer instrumentation, dan spesifikasi hardware/software).
- `Log7.md`: Dokumentasi perumusan dan penyederhanaan Subbab 3.5 (Rencana Analisis), klarifikasi peran Telescope vs. Benchmark Baseline, pemahaman Cache Hit Ratio, serta teks final Subbab 3.5 yang disepakati.

### C. Lit Review 1.md, Lit Review 2.md, Lit Review 3.md

Ketiga file tersebut berisi hasil kajian terhadap lima penelitian terdahulu yang sudah dipilih dan dianalisis untuk Subbab 2.1 Penelitian Terkait.

## 14. Kerangka Argumentasi Literatur yang Diharapkan

Lima penelitian yang dipilih diarahkan untuk membangun alur argumentasi berikut:
1. Penelitian mengenai optimasi performa aplikasi Laravel (Putra et al., 2025).
2. Penelitian mengenai Redis caching dan latensi basis data (Privalov & Stupina, 2024; Taipalus, 2024; An & Cao, 2022).
3. Penelitian mengenai masalah N+1 dan optimasi ORM (Alshemaimri et al., 2021; Turcotte et al., 2022).
4. Penelitian mengenai biaya memori akibat penggunaan ORM (Yusmita et al., 2025).
5. Penelitian komparatif kombinasi caching dan optimasi ORM (Kundan & Makwana, 2025).

## 15. Prinsip Penyusunan Subbab 2.1 Penelitian Terkait

Perbandingan harus mencakup objek sistem, teknologi, strategi caching, teknik ORM, metrik pengukuran, desain eksperimen, hasil kuantitatif, dan batasan penelitian tanpa membuat klaim absolut yang tidak didukung literatur.

## 16. Aturan Keakuratan Akademik

1. Gunakan isi file sumber sebagai dasar utama.
2. Jangan mengarang judul, penulis, tahun, nama jurnal, DOI, metode, hasil, persentase peningkatan, atau keterbatasan penelitian.
3. Bedakan hasil empiris penelitian terdahulu dari interpretasi penulis proposal.
4. Jangan menambahkan referensi eksternal tanpa kebutuhan dan izin yang jelas.
5. Pertahankan sitasi yang dapat ditelusuri ke sumbernya.
6. Hindari klaim kausal yang lebih kuat daripada desain penelitian yang dilaporkan.
7. Gunakan bahasa Indonesia akademik yang alami, lugas, dan tidak bertele-tele.

## 17. Gaya Penulisan yang Diinginkan

- Formal, jelas, dan logis.
- Mengutamakan sintesis daripada repetisi.
- Satu paragraf memiliki gagasan utama yang jelas.
- Transisi antarparagraf menunjukkan hubungan argumentasi.
- Istilah teknis dipertahankan secara konsisten.
- Tidak memakai kalimat generik yang sekadar menyatakan bahwa teknologi tertentu penting tanpa sumber.
- Tidak mengulang isi Bab I secara panjang lebar.
- Tidak menyamakan peningkatan performa teknis dengan peningkatan kepuasan pengguna (UX).

## 18. Instruksi Umum untuk AI Agent

Sebelum menyusun atau mereview suatu subbab:
1. Identifikasi bagian proposal yang sedang dikerjakan dan fungsinya.
2. Baca file yang relevan secara menyeluruh.
3. Petakan bukti empiris yang tersedia sebelum menulis.
4. Periksa kesesuaian antara sumber literatur dan narasi bab-bab sebelumnya.
5. Periksa kembali akurasi sitasi, angka, terminologi, dan konsistensi metodologis.

**Status Saat Ini:**
- **Bab I:** Selesai 100% dan terverifikasi di `proposal.md`.
- **Bab II:** Selesai 100% (Subbab 2.1 s.d. 2.4) dan terverifikasi di `proposal.md`.
- **Bab III:** 
  - Subbab 3.1, 3.2, 3.3, dan 3.4 telah selesai disusun secara lengkap di `proposal.md`.
  - Subbab 3.5 (Rencana Analisis) telah selesai dirumuskan dan disederhanakan secara final di `Log7.md`.
- **Fokus Pekerjaan Selanjutnya:** 
  1. Menyematkan naskah final Subbab 3.5 dari `Log7.md` ke dalam `proposal.md`.
  2. Melakukan audit nomor urut tabel dan gambar pada Bab III agar terstruktur rapi.
  3. Menyusun Lampiran A (bukti profiling Laravel Telescope).
  4. Pemeriksaan akhir konsistensi proposal sebelum pengujian sidang proposal.