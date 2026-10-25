# 2.3 Metodologi Pengembangan Produk

Subbab ini menguraikan metodologi rekayasa perangkat lunak yang digunakan dalam penelitian optimasi kinerja sistem Koladi. Mengingat karakteristik proyek ini yang berfokus pada evolusi perangkat lunak (*software evolution*) dan penataan ulang kode (*refactoring*) pada sistem yang sudah berjalan (*brownfield*), maka pendekatan yang dipilih harus mampu menjamin efisiensi performa tanpa mengubah fungsi bisnis yang ada.

### 2.3.1 Pilihan Metodologi Pengembangan

Dalam konteks optimasi sistem *backend* dan refaktorisasi, terdapat beberapa metodologi yang diakui secara akademis. Berikut adalah analisis terhadap metode yang direkomendasikan dan beberapa alternatifnya:

#### 1. Refactoring Lifecycle (Mens & Tourwé) - **Metode yang Direkomendasikan**
Metodologi ini dikemukakan oleh Tom Mens dan Tom Tourwé dalam survei komprehensif mereka mengenai refaktorisasi perangkat lunak [1]. Metode ini merupakan siklus terstruktur yang dirancang khusus untuk mengubah struktur internal perangkat lunak tanpa mengubah perilaku eksternalnya.

*   **Kelebihan:** Sangat sesuai untuk proyek optimasi karena mencakup tahapan eksplisit untuk menjamin reservasi perilaku (*behavior preservation*) dan penilaian efek terhadap kualitas (seperti kinerja) [1]. Tahapannya sejalan dengan aktivitas *profiling* dan *benchmarking*.
*   **Keterbatasan:** Membutuhkan upaya tambahan dalam dokumentasi konsistensi antara kode dan artefak lainnya jika sistem memiliki dokumentasi desain yang sangat kompleks.

#### 2. Software Performance Engineering (SPE) (Smith & Williams)
SPE adalah pendekatan sistematis untuk membangun kinerja ke dalam sistem perangkat lunak sepanjang siklus hidupnya [8], [11]. Fokus utamanya adalah pada model kinerja dan pengukuran kuantitatif.

*   **Kelebihan:** Menyediakan kerangka kerja yang sangat kuat untuk pengukuran metrik kinerja (seperti *response time* dan *throughput*) dan evaluasi solusi arsitektural sebelum diimplementasikan [9], [10].
*   **Keterbatasan:** Seringkali diasumsikan untuk pengembangan sistem baru (*greenfield*), sehingga tahap refaktorisasi kode spesifik kurang mendalam dibandingkan metode Mens & Tourwé.

#### 3. Iterative Enhancement Model (Basili)
Model ini, yang dikembangkan oleh Victor Basili, berfokus pada pengembangan perangkat lunak secara bertahap dengan memulai dari implementasi inti dan melakukan penyempurnaan pada setiap iterasi [13].

*   **Kelebihan:** Sangat fleksibel untuk proyek evolusi perangkat lunak karena memungkinkan pengembang untuk menganalisis hasil pada setiap tahap dan melakukan perbaikan berkelanjutan [12].
*   **Keterbatasan:** Kurang memberikan panduan spesifik mengenai teknik refaktorisasi teknis seperti optimasi ORM atau strategi *caching* dibandingkan dengan siklus hidup refaktorisasi khusus.

#### 4. Performance Refinement and Evolution Model (PREM) (Ho & Williams)
PREM merupakan model evolusi yang berfokus pada pengembangan kinerja perangkat lunak melalui level-level kematangan tertentu (Level 0 hingga Level 3) [5], [7].

*   **Kelebihan:** Sangat relevan untuk proyek yang tujuan utamanya adalah peningkatan performa secara bertahap pada sistem yang sudah ada [5].
*   **Keterbatasan:** Lebih berfokus pada manajemen tingkat kematangan proses kinerja secara organisasi daripada langkah-langkah teknis implementasi refaktorisasi kode.

### 2.3.2 Detail Metodologi: Siklus Hidup Refactoring (Mens & Tourwé)

Berdasarkan perbandingan di atas, penelitian ini menggunakan **Refactoring Lifecycle** oleh Mens & Tourwé [1] sebagai metodologi utama. Metodologi ini dipilih karena secara eksplisit mendukung aktivitas preservasi perilaku melalui pengujian ketat, yang sangat krusial dalam memastikan 22 metrik DSS pada aplikasi Koladi tetap akurat setelah optimasi.

Berikut adalah pemetaan aktivitas konkret dalam pengembangan optimasi Koladi berdasarkan tahapan literatur:

**Tabel 1. Pemetaan Aktivitas Metodologi Refactoring Lifecycle pada Koladi**

| Tahapan (Literasi [1]) | Deskripsi Teoretis | Aktivitas Konkret pada Proyek Koladi | Output / Deliverable |
| :--- | :--- | :--- | :--- |
| **1. Identifikasi Kandidat Refactoring** | Menentukan bagian kode yang memerlukan perbaikan (seperti *code smells* atau *bottleneck*). | *Profiling* performa menggunakan Laravel Telescope untuk menemukan kueri N+1 pada relasi *checklists* dan *bottleneck* pada `getWorkloadDistribution()`. | Laporan Profiling Awal (Baseline Kinerja). |
| **2. Penentuan Strategi Refactoring** | Memilih teknik refaktorisasi yang tepat untuk diterapkan pada bagian yang teridentifikasi. | Perancangan pola *Redis Cache-Aside* dengan *Eloquent Observers*, optimasi kueri ORM (eager loading), dan transformasi *query-in-loop* menjadi *batch retrieval*. | Dokumen Rancangan Optimasi & Arsitektur Cache. |
| **3. Menjamin Preservasi Perilaku** | Memastikan bahwa perubahan tidak mengubah output fungsional sistem. | Pengembangan *characterization tests* dan pemeriksaan ekivalensi hasil untuk 22 metrik DSS guna memastikan data tidak berubah. | *Test Suite* & Hasil Uji Ekivalensi Fungsional. |
| **4. Penerapan Refactoring** | Melakukan transformasi kode sesuai dengan strategi yang telah ditentukan. | Implementasi kode pada Laravel 12: menambahkan Redis cache, mengubah kueri Eloquent, dan memigrasi logika agregasi ke memori. | *Source Code* yang telah dioptimasi. |
| **5. Penilaian Efek Refactoring** | Mengukur dampak perubahan terhadap karakteristik kualitas (kinerja). | Pengukuran ulang metrik kinerja (waktu respons, jumlah kueri, penggunaan memori) setelah optimasi dan membandingkannya dengan data *baseline*. | Laporan Perbandingan Performa (Sebelum vs. Sesudah). |
| **6. Pemeliharaan Konsistensi** | Menyelaraskan kode yang telah direfaktor dengan artefak lain seperti tes dan dokumentasi. | Integrasi kode ke dalam sistem utama Koladi dan memperbarui dokumentasi teknis terkait mekanisme *caching* dan optimasi kueri. | Dokumentasi Teknis & Sistem Terintegrasi. |

Penggunaan metodologi ini menjamin bahwa setiap intervensi teknis, mulai dari penggantian *database-backed cache* hingga optimasi kueri *loop*, dilakukan secara terukur dengan basis data kinerja yang valid [1], [6]. Tahapan pengujian regresi (*regression testing*) juga menjadi bagian integral untuk memastikan stabilitas sistem selama proses evolusi [2], [3], [4].

## References

[1]T. Mens and T. Tourwe, “A survey of software refactoring,” IEEE Transactions on Software Engineering, vol. 30, no. 2, pp. 126–139, Feb. 2004, doi: 10.1109/TSE.2004.1265817.

[2]A. Mehta and G. T. Heineman, “Evolving legacy system features into fine-grained components,” pp. 417–427, May 2002, doi: 10.1145/581339.581391.

[3]M. J. Harrold, “Testing: a roadmap,” pp. 61–72, May 2000, doi: 10.1145/336512.336532.

[4]A. Orso and T. Xie, “BERT: BEhavioral Regression Testing,” pp. 36–42, July 2008, doi: 10.1145/1401827.1401835.

[5]C.-W. Ho and L. Williams, “Developing software performance with the performance refinement and evolution model,” pp. 133–136, Feb. 2007, doi: 10.1145/1216993.1217016.

[6]M. Fowler, “Refactoring: Improving the Design of Existing Code,” Jan. 1999.

[7]C. W. Ho and L. Williams, “Managing software performance engineering activities with the performance refinement and evolution model (prem),” Jan. 2006.

[8]C. U. Smith and L. G. Williams, “Software performance engineering,” pp. 343–365, Jan. 2003, doi: 10.1007/0-306-48738-1_16.

[9]L. G. Williams and C. U. Smith, “The Business Case for Software Performance Engineering,” Jan. 2002.

[10]C. U. Smith, “Performance engineering for software architectures,” pp. 166–167, Aug. 1997, doi: 10.1109/CMPSAC.1997.624783.

[11]C. U. Smith and L. G. Williams, “Best Practices for Software Performance Engineering,” pp. 83–92, Jan. 2003.

[12]V. R. Basili, “Is there a future for empirical software engineering,” pp. 1–1, Sept. 2006, doi: 10.1145/1159733.1159735.

[13]B. Boehm, H. D. Rombach, and M. V. Zelkowitz, “Foundations of Empirical Software Engineering: The Legacy of Victor R. Basili,” Nov. 2010.