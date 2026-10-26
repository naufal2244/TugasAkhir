Berdasarkan sumber-sumber ilmiah yang tersedia di dalam *notebook*, analisis mendalam untuk menyusun Subbab **"Rancangan Dataset Pengujian"** (*Test Dataset Design*) dijabarkan sebagai berikut:

---

### **1. Perancangan dan Logika Ilmiah Pemilihan Ukuran Dataset Bertingkat (*Multi-Tiered Workloads*)**

#### **A. Perancangan Skenario Volume Data**
Dataset pengujian dirancang ke dalam tiga strata volume data bertingkat, yaitu:
* **Small:** 50 *tasks* dan 5 anggota (*members*), merepresentasikan lingkungan *workspace* skala kecil.
* **Medium:** 200 *tasks* dan 15 anggota, merepresentasikan *workspace* skala menengah.
* **Large:** 500 *tasks* dan 30 anggota, merepresentasikan *workspace* skala besar.

#### **B. Logika Ilmiah di Balik Pemilihan Volume Bertingkat**
1. **Bukan Beban Trafik Paralel (*Non-Concurrent Workload Scaling*):**  
   Peningkatan skala data dari *Small*, *Medium*, hingga *Large* secara tegas **tidak dimaksudkan sebagai *load testing*** atau peningkatan trafik konkuren (karena jumlah *request* paralel tidak dinaikkan). Yang dinaikkan secara bertahap adalah **jumlah entitas data relasional yang diproses dalam satu siklus *request* terkontrol**. Pendekatan ini mengikuti kaidah eksperimen performa perangkat lunak yang menaikkan ukuran dataset secara bertingkat guna mengamati perubahan biaya eksekusi, pertumbuhan jumlah kueri (*query count*), dan degradasi performa sistem (An & Cao, 2022; Sasmoko dkk., 2024; Turcotte dkk., 2022; Yusmita dkk., 2025).
2. **Pemicu Eksaserbasi Antipola Kueri Spesifik (*Exercising Codebase Bottlenecks*):**  
   Penetapan rasio entitas ini memiliki dasar operasional yang kontekstual terhadap arsitektur kode internal:
   * **Peningkatan jumlah *tasks* (50 \\(\rightarrow\\) 200 \\(\rightarrow\\) 500):** Dipilih secara sengaja untuk menguji dan memicu eksaserbasi **masalah *N+1 query* pada relasi `checklists`** saat pemanggilan fungsi `getProgressPercentage()`. Pada kondisi *baseline*, pemuatan *lazy loading* menyebabkan kueri SQL membengkak secara linier mengikuti pertambahan tugas.
   * **Peningkatan jumlah anggota/anggota *workspace* (5 \\(\rightarrow\\) 15 \\(\rightarrow\\) 30):** Dipilih secara sengaja untuk menguji dan memicu eksaserbasi **pola *query-in-loop*** pada fungsi `getWorkloadDistribution()`, di mana pemanggilan kueri `Task::whereHas()` bertambah secara linier mengikuti jumlah anggota aktif.
3. **Pembuktian Penurunan Kompleksitas Algoritmik (\\(O(N) \rightarrow O(1)\\)):**  
   Melalui tiga skala bertingkat ini, peneliti dapat membuktikan secara empiris apakah refaktorisasi ORM (*selective eager loading* dan *batch in-memory aggregation*) berhasil mengubah pertumbuhan kueri dari skala linier terhadap data (\\(O(N)\\)) menjadi kueri konstan (\\(O(1)\\)) di seluruh ukuran dataset.

---

### **2. Alasan Metodologis Pemilihan Dataset Sintetis Terstruktur (Seeder/Factory) vs. Data Riil**

Peneliti memilih membangkitkan data sintetis terstruktur menggunakan perkakas bawaan *framework* (Laravel *Seeder* dan *Factory*) dibandingkan menggunakan data produksi riil karena pertimbangan metodologis berikut:

1. **Kontrol Eksperimental Presisi (*Experimental Control*):**  
   Data riil produksi bersifat dinamis, tidak seimbang, dan sering kali memiliki struktur relasi yang acak. Sebaliknya, *Seeder* dan *Factory* memberikan kendali penuh dan presisi terhadap volume data, rasio status tugas, kepadatan relasi tugas-pengguna (*task-member relationship density*), serta parameter jatuh tempo (*due date*).
2. **Determinisme dan Perbandingan Berkeadilan (*Fair Comparison*):**  
   Karakteristik deterministik menjamin bahwa konfigurasi *baseline* (sebelum optimasi) dan konfigurasi *optimized* (sesudah optimasi) diuji pada kondisi data yang persis sama (*identical fair setup*) (An & Cao, 2022; Putra et al., 2025; Sasmoko et al., 2024; Yusmita et al., 2025).
3. **Replikasi dan Reprodusibilitas Eksperimen (*Repeatability & Reproducibility*):**  
   Dalam rekayasa perangkat lunak eksperimental (*Experimental Software Engineering*), eksperimen harus dapat diulang (*repeatable*) dan direplikasi oleh peneliti lain (Gómez-Pantoja et al., 2025; Wohlin et al., 2000; Yusmita et al., 2025). Skrip *Seeder/Factory* memungkinkan *database state* dibersihkan dan dibangkitkan ulang ke kondisi awal secara identik sebelum siklus pengujian dijalankan.
4. **Isolasi Efek Kausalitas (*Attribution to Optimization*):**  
   Jika data pengujian mengalami perubahan atau fluktuasi selama pengujian, variasi performa yang teramati tidak dapat disimpulkan apakah berasal dari perbaikan algoritma/infrastruktur atau dari anomali data. Dengan dataset sintetis yang terkunci sama, perbedaan pada *response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio* dapat secara meyakinkan **diatribusikan langsung pada intervensi teknis** (Redis Cache-Aside dan optimasi ORM), bukan akibat perbedaan data.

---

### **3. Pentingnya Menjaga Distribusi Data Konstan dan Identik Antara Baseline dan Optimized**

Menjaga distribusi data tetap konstan antara kondisi *baseline* dan *optimized* merupakan keharusan mutlak dalam desain *controlled before-after benchmarking* karena:

1. **Mengontrol Jalur Eksekusi Logika Bisnis (*Control Flow Integrity*):**  
   Pada modul analitis seperti DSS, setiap status tugas mengaktifkan alur kalkulasi yang berbeda:
   * Dalam penelitian ini, distribusi status tugas dikunci secara presisi pada rasio: **40% *done***, **35% *in progress***, dan **25% *todo***.
   * Sebagian entri tugas secara sengaja disetel memiliki tanggal jatuh tempo yang telah lewat (*past due date*).
   * Jika proporsi ini berubah antara *baseline* dan *optimized*, formula metrik agregasi internal—seperti `overdueRate`, `urgentTaskRatio`, dan `criticalTaskRatio`—akan mengeksekusi jumlah iterasi dan kalkulasi pembobotan yang berbeda, sehingga membiaskan pengukuran waktu pemrosesan dan penggunaan memori.
2. **Mengeliminasi Variabel Perancu (*Confounding Variables*):**  
   Sebagaimana ditekankan dalam literatur pengujian sistem web, perubahan konfigurasi perangkat lunak harus diuji dengan mengisolasi domain pengujian dari gangguan luar (*isolation testing* dan *configuration testing*) (Alnuhait et al., 2024; Gómez-Pantoja et al., 2025). Jika distribusi data berubah, variasi data tersebut menjadi variabel perancu yang merusak validitas internal eksperimen.
3. **Memenuhi Asumsi Dependensi *Paired-Sample t-Test*:**  
   Metode evaluasi statistik menggunakan uji parametrik *paired-sample t-test* (\\(\alpha = 0{,}05\\)). Uji ini menuntut dependensi struktural yang ketat: setiap satu observasi *baseline* harus memiliki pasangan langsung (*direct pair*) pada *optimized* dengan skala volume data, *endpoint*, kondisi *cache*, parameter *request*, serta karakteristik data yang identik (Uddin & Lu, 2024).

---

### **4. Ekstraksi Kutipan Langsung dan Sitasi Formal untuk Naskah Proposal**

Kutipan-kutipan langsung berikut dapat digunakan secara formal di Bab Metodologi:

#### **A. Mengenai Penggunaan Dataset Sintetis, Kontrol Variabel, dan Reprodusibilitas**
> *"Seluruh eksperimen menggunakan data sintetis yang dihasilkan melalui Laravel Seeder dan Factory, bukan data produksi, agar volume data, distribusi status, dan kepadatan relasi task-anggota dapat dikontrol secara presisi. Pendekatan ini menghasilkan dataset yang deterministik dan reproducible, sehingga konfigurasi baseline dan optimized dapat dibandingkan pada kondisi data yang sama (Yusmita dkk., 2025; Sasmoko dkk., 2024). Dengan menggunakan dataset yang sama, perbedaan response time, query count, memory utilization, dan cache hit ratio dapat diatribusikan pada intervensi optimasi, bukan pada perubahan data."*  
> — **(Introduction.docx, Bagian Metodologi)**

> *"Dataset sintetis yang dihasilkan melalui seeder atau factory digunakan untuk menjaga kontrol volume data, distribusi status, dan reprodusibilitas eksperimen (Sasmoko et al., 2024; Yusmita et al., 2025)."*  
> — **(Introduction.docx, Bagian Kajian Pustaka)**

---

#### **B. Mengenai Logika Skala Volume Bertingkat (*Workload Scaling*)**
> *"Pembagian skenario ini tidak dimaksudkan sebagai load testing karena jumlah request paralel tidak dinaikkan; yang dinaikkan adalah jumlah entitas yang diproses dalam request terkontrol. Pendekatan ini sejalan dengan eksperimen performa yang menaikkan ukuran dataset atau jumlah entitas secara bertahap untuk mengamati perubahan biaya eksekusi, query count, dan performa sistem (Sasmoko dkk., 2024; An dan Cao, 2022; Turcotte dkk., 2022; Yusmita dkk., 2025)... Peningkatan jumlah task relevan untuk mengamati dampak N+1 query pada relasi checklists, sedangkan peningkatan jumlah anggota relevan untuk mengamati dampak query loop pada getWorkloadDistribution()."*  
> — **(Introduction.docx, Bagian Metodologi)**

---

#### **C. Mengenai Kontrol Distribusi Data yang Konstan**
> *"Setiap task di dalam workspace memiliki distribusi status yang dikontrol, yaitu 40% done, 35% in progress, dan 25% todo, dengan sebagian entri memiliki tanggal jatuh tempo yang telah lewat untuk menguji metrik berbasis waktu seperti overdueRate dan urgentTaskRatio."*  
> — **(Introduction.docx, Bagian Metodologi)**

---

#### **D. Mengenai *Controlled Benchmarking* & *Isolation Testing***
> *"Configuration testing is carried out to determine the impact of changes to a system's configuration on its performance, while isolation testing is used to isolate and confirm fault domains."*  
> — **Alnuhait, H., Alzyadat, W., Althunibat, A., Kahtan, H., Zaqaibeh, B., & Al-Khawaja, H. A. (2024)**, *International Journal of Advanced and Applied Sciences*, 11(9), hlm. 219.

> *"To isolate the impact of this architectural segmentation, we intentionally employed two fundamental and widely adopted replacement policies: LRU and LFU... While this approach allows for controlled, repeatable evaluations, it does not capture all aspects of a live deployment."*  
> — **Gómez-Pantoja, C., Baeza-Rocha, D., & Inostrosa-Psijas, A. (2025)**, *Future Internet*, 17(9), hlm. 5, 16.

---

### **Daftar Pustaka Formal (Format APA 7th):**

* **Alnuhait, H., Alzyadat, W., Althunibat, A., Kahtan, H., Zaqaibeh, B., & Al-Khawaja, H. A. (2024).** Web application performance assessment: A study of responsiveness, throughput, and scalability. *International Journal of Advanced and Applied Sciences*, 11(9), 214–226. https://doi.org/10.21833/ijaas.2024.09.023
* **An, S., & Cao, Y. (2022).** Making cache monotonic and consistent. *Proceedings of the VLDB Endowment*, 16(4), 891–904. https://doi.org/10.14778/3574245.3574271
* **Gómez-Pantoja, C., Baeza-Rocha, D., & Inostrosa-Psijas, A. (2025).** A behavior-aware caching architecture for web applications using static, dynamic, and burst segmentation. *Future Internet*, 17(9), 429. https://doi.org/10.3390/fi17090429
* **Sasmoko, Indrianti, Y., Manalu, S. R., & Danaristo, J. (2024).** Analyzing database optimization strategies in Laravel for an enhanced learning management. *Procedia Computer Science*, 234, 799–804. https://doi.org/10.1016/j.procs.2024.10.306
* **Turcotte, A., Aldrich, M. W., & Tip, F. (2022).** Reformulator: Automated refactoring of the N+1 problem in database-backed applications. In *ACM International Conference Proceeding Series*. Association for Computing Machinery. https://doi.org/10.1145/3551349.3556911
* **Uddin, M. S., & Lu, H. (2024).** An empirical study on performance optimization and evaluation in software systems.
* **Wohlin, C., Runeson, P., Höst, M., Ohlsson, M. C., Regnell, B., & Wesslén, A. (2000).** *Experimentation in software engineering: An introduction*. Kluwer Academic Publishers.
* **Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025).** Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM. *Procedia Computer Science*, 258, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061

Berdasarkan penelusuran terhadap literatur ilmiah dan dokumen eksperimental yang tersedia di dalam *notebook*, berikut adalah analisis mendalam mengenai perancangan dan penentuan dataset pengujian untuk eksperimen komparatif *before-after*:

---

### **1. Perancangan dan Penentuan Ukuran Dataset (Logika Skala Bertingkat: Small, Medium, Large)**

Dalam literatur *database benchmarking* dan rekayasa perangkat lunak, perancangan ukuran dataset tidak ditentukan secara arbitrer, melainkan disusun berdasarkan logika ilmiah berikut:

1. **Menguji Kompleksitas Algoritma dan Titik Degradasi (*Scalability Stress Test*)**:
   * Pengujian performa kueri basis data membutuhkan variasi volume bertingkat (*multi-tiered workloads*) untuk melihat bagaimana waktu respons dan jumlah kueri bereaksi terhadap pertumbuhan data. 
   * Sasmoko et al. (2024) menguji skala 1.000, 10.000, dan 100.000 data guna mengevaluasi efek skalabilitas data (*data scaling analysis*) terhadap strategi pengindeksan dan metode kueri ORM.
   * Turcotte et al. (2022) merancang pengujian menggunakan tiga skala basis data bertingkat: *10 scale*, *100 scale*, dan *1.000 scale*. Tujuannya adalah untuk mendemonstrasikan secara empiris bahwa keuntungan performa dari eliminasi antipola kueri (seperti \\(N+1\\)) akan semakin membesar secara eksponensial seiring bertambahnya ukuran data (menghasilkan *speedup* dari 3,05x pada skala 10 menjadi 38,58x pada skala 1.000).
2. **Menstimulasi Sensitivitas Antipola Kueri Tertarget**:
   * Dalam konteks modul analitis/DSS (seperti Koladi), ukuran dataset sengaja dirancang berjenjang—*Small* (50 tugas, 5 anggota), *Medium* (200 tugas, 15 anggota), dan *Large* (500 tugas, 30 anggota)—agar setiap dimensi skala memicu hambatan (*bottleneck*) komputasi yang berbeda:
     * **Skalasi Tugas (*Tasks*)**: Dirancang untuk mengeksploitasi dan menekan antipola **\\(N+1\\) query** pada pemuatan relasi bertingkat (misalnya relasi `checklists`). Pada kondisi *baseline*, penambahan jumlah entitas utama akan melipatgandakan *database round-trips* secara linier/eksponensial.
     * **Skalasi Anggota (*Members*)**: Dirancang untuk menguji inefisiensi **kueri di dalam perulangan (*query-in-loop*)** pada kalkulasi agregasi distribusi beban kerja (`getWorkloadDistribution()`).

---

### **2. Alasan Memilih Dataset Sintetis Terstruktur (Seeder/Factory) daripada Data Riil**

Peneliti dalam literatur memilih data sintetis yang dibangkitkan via *data generation tools* (seperti Faker, Seeder, dan Factory) karena beberapa keunggulan metodologis:

1. **Determinisme dan Kontrol Penuh terhadap Variabel (*Experimental Control*)**:
   * Data produksi riil sering kali mengandung data yang tidak lengkap (*missing values*), anomali, atau bias historis yang tidak terkontrol.
   * Sebaliknya, generator data sintetis memungkinkan peneliti mengonfigurasi properti statistik secara presisi dan deterministik. Hal ini mengeliminasi variabel pengganggu (*confounding factors*) sehingga perbedaan performa yang terukur murni disebabkan oleh intervensi teknis yang diuji.
2. **Replikasi Eksperimen (*Bit-for-Bit Reproducibility*)**:
   * Standarisasi skrip pembangkitan data (misalnya berkas seeder dengan konfigurasi terstandarisasi) memungkinkan peneliti lain mereplikasi basis data dengan struktur, volume, dan kardinalitas relasi yang persis sama. Yusmita et al. (2025) menekankan bahwa kombinasi kontainerisasi dengan pembangkitan data sintetis mendukung eksperimen yang dapat diulang secara deterministik (*repeatable side-by-side comparison*).
3. **Fleksibilitas Skalabilitas Bebas (*Arbitrary Size Generation*)**:
   * Pustaka *dummy data factory* (seperti *built-in fake data factory* pada Laravel) memudahkan pembuatan populasi data dalam volume masif (puluhan hingga ratusan ribu baris) sesuai kebutuhan skenario uji tanpa terikat pada keterbatasan volume data historis produksi.
4. **Kepatuhan Privasi dan Etika Penelitian (*Data Privacy*)**:
   * Data produksi mengandung data sensitif atau informasi pribadi pengguna. Penggunaan data sintetis menjamin kepatuhan etika riset tanpa risiko kebocoran data.

---

### **3. Pentingnya Menjaga Distribusi Data Tetap Konstan dan Identik (Baseline vs. Optimized)**

Menjaga distribusi data (seperti status data, proporsi relasi, dan kardinalitas entitas) tetap seragam antara kondisi *baseline* dan *optimized* merupakan prinsip krusial dalam metodologi eksperimental:

1. **Menjamin Keabsahan Atribusi Kausalitas (*Causal Attribution*)**:
   * Jika pada kondisi *baseline* sebagian besar tugas berstatus *in-progress* (yang membutuhkan pemrosesan kalkulasi agregat lebih berat), sementara pada kondisi *optimized* komposisinya berubah menjadi tugas yang sudah selesai (*completed*), maka penurunan latensi bisa saja terjadi akibat ringannya komputasi data, bukan karena optimasi kueri atau *caching*.
   * Dengan mempertahankan densitas relasi (misalnya jumlah *checklist* per tugas dan rasio tugas per anggota tim) yang persis sama di kedua perlakuan, peneliti memastikan bahwa beban kerja komputasional (*computational workload*) di lapisan basis data berada pada titik ekuivalen.
2. **Menghindari Bias *Workload Skewness***:
   * Taipalus (2024) dalam tinjauan sistematisnya mencatat bahwa perilaku DBMS sangat sensitif terhadap *skewness* data dan rasio operasi. Agar perbandingan performa *fair* dan tidak bias, lingkungan uji harus mengevaluasi kueri pada skema dan sebaran data yang identik.
3. **Mendukung Uji Statistik Komparatif (*Paired Observations*)**:
   * Desain perbandingan berpasangan (*paired-sample analysis*) mensyaratkan bahwa setiap observasi pada konfigurasi teroptimasi memiliki padanan yang setara pada konfigurasi dasar. Keseragaman distribusi data menjamin bahwa variasi yang diuji hanyalah arsitektur kueri dan *caching*-nya.

---

### **4. Kutipan Langsung dan Sitasi Formal untuk Naskah Tugas Akhir**

Berikut adalah kutipan langsung yang dapat digunakan untuk memperkuat argumen metodologi Anda:

#### **A. Mengenai Dataset Sintetis, Determinisme, dan Reproducibility**
*   **Yusmita et al. (2025)**:
    > *"In addition to using containers, synthetic data generation (often called seeding) allow researchers to create datasets of arbitrary size with controlled statistical properties, yielding deterministic and scalable benchmarks... By combining containerized environments with controlled synthetic datasets, our methodology supports repeatable and measurable side-by-side comparisons of raw SQL and ORM frameworks on PostgreSQL. This setup isolates the performance impact of the ORM abstraction layer."*
    *   **Sitasi**: Yusmita, J. C., Arya, R., Wijaya, J. M., Suryaningrum, K. M., & Siswanto, R. R. (2025). Optimizing database access strategy: A performance analysis comparison of Raw SQL and Prisma ORM. *Procedia Computer Science*, 269, 1201–1210. https://doi.org/10.1016/j.procs.2025.09.061

*   **Henning & Hasselbring (2022)**:
    > *"In order to obtain reproducible results, measurements should therefore be repeated and the confidence in the final results should be quantified... Using containerization tools such as Docker provides bit-for-bit reproducibility with minimal overhead..."*
    *   **Sitasi**: Henning, S., & Hasselbring, W. (2022). A configurable method for benchmarking scalability of cloud-native applications. *Empirical Software Engineering*, 27(6), Article 143. https://doi.org/10.1007/s10664-022-10162-1

*   **Privalov & Stupina (2024)**:
    > *"Next, the tables were populated with big amount of data. For this purpose, the built-in fake data factory of the Laravel framework was used. Thus, the articles table was loaded with 121,000 records..."*
    *   **Sitasi**: Privalov, M. V., & Stupina, M. V. (2024). Improving web-oriented information systems efficiency using Redis caching mechanisms. *Indonesian Journal of Electrical Engineering and Computer Science*, 33(3), 1667–1675. https://doi.org/10.11591/ijeecs.v33.i3.pp1667-1675

#### **B. Mengenai Pengujian Bertingkat dan Skalabilitas Kueri**
*   **Turcotte et al. (2022)**:
    > *"In this case study, we created three databases of size 10, 100, and 1000 (henceforth referred to as the '10 scale', '100 scale', and '1000 scale' configurations) so that the HTTP request handler needs to process that much data, and measure the performance of the handlers before and after refactoring at each database size... Additional experiments revealed speedups of up to 38.58x and substantial improvements in scalability by demonstrating that the relative performance improvements grew as the database size was increased."*
    *   **Sitasi**: Turcotte, A., Aldrich, M. W., & Tip, F. (2022). reformulator: Automated refactoring of the N+1 problem in database-backed applications. In *Proceedings of the 37th IEEE/ACM International Conference on Automated Software Engineering* (ASE '22, pp. 1–12). ACM. https://doi.org/10.1145/3551349.3556911

*   **Sasmoko et al. (2024)**:
    > *"The data generation uses a faker to generate a large amount of dummy data. This tool is responsible for the generation of random data into the database and tables that are generated by the Laravel migration. To cover the case of scalability of data, the test will be taken in a total course of 1000, 10000, and 100000 data."*
    *   **Sitasi**: Sasmoko, dkk. (2024). Analyzing database optimization strategies in Laravel. *Procedia Computer Science*, 245, 799–804. https://doi.org/10.1016/j.procs.2024.10.358

#### **C. Mengenai Pentingnya Karakteristik Data Pengujian yang Representatif & Konsisten**
*   **Taipalus (2024)**:
    > *"The credibility of testing results is also related to how well the test database and data therein represent the target environment... Furthermore, in business domains such as online stores, there are typically popular products, and thus the data related to them are targets of a relatively large number of database operations. For generalizable benchmarking results, the benchmark must account for such skewness in database use, rather than, e.g., randomly querying data objects."*
    *   **Sitasi**: Taipalus, T. (2024). Database management system performance comparisons: A systematic literature review. *The Journal of Systems & Software*, 208, Article 111872. https://doi.org/10.1016/j.jss.2023.111872