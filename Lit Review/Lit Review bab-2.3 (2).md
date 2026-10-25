# Metode **software evolution iteratif** paling tepat untuk **refactoring** dan **optimasi kinerja** pada sistem Koladi yang sudah berjalan.

Untuk konteks **brownfield optimization** seperti Koladi, pertanyaannya terbagi menjadi tiga bagian: metode utama yang paling tepat, alternatif metode yang masih layak secara akademik, dan pemetaan tahap-ke-aktivitas yang bisa langsung dipakai dalam Subbab 2.3 proposal. Untuk proyek yang berfokus pada perubahan pada sistem yang sudah berjalan, refactoring yang mempertahankan perilaku, dan pengukuran kinerja sebelum-sesudah, pendekatan yang paling kuat adalah **software evolution / iterative enhancement yang diperkaya praktik performance engineering dan refactoring evaluation** (Mens & Tourwé, 2004; Rajeeb & Mantri, 2026; Eramo et al., 2023).

**Fig 1.** Methodology for developing the proposed refactoring taxonomy. (Almogahed et al., 2026)

*How to read: Diagram ini memperlihatkan alur metodologi bertahap untuk mengembangkan taksonomi refactoring. Baca dari fase awal eksplorasi, lanjut ke eksperimen, lalu ke analisis lintas kasus.*

## Rekomendasi Utama

Metode yang paling sesuai adalah **iterative software evolution / iterative enhancement yang diadaptasi untuk refactoring dan performance engineering**. Pilihan ini paling cocok karena proyek Anda bukan membangun sistem baru, melainkan memodifikasi perangkat lunak setelah rilis untuk memenuhi kebutuhan operasional baru, sambil menjaga kualitas dan menekan rework (Rajeeb & Mantri, 2026; Alseadoon, 2021).

Literatur refactoring menegaskan bahwa refactoring bertujuan **memperbaiki struktur internal tanpa mengubah perilaku eksternal**, dan dalam konteks software evolution digunakan untuk meningkatkan maintainability, modularity, reusability, complexity, hingga efficiency (Mens & Tourwé, 2004; Motogna et al., 2024). Literatur performance engineering juga menekankan bahwa optimasi tidak sebaiknya diperlakukan sebagai aktivitas akhir, tetapi dievaluasi berulang sepanjang siklus pengembangan melalui profiling, pengukuran, tuning, dan pengujian ulang (Mattoo, 2023).

- **Kelebihan utama**: paling selaras dengan proyek evolusi sistem berjalan, karena perubahan dilakukan inkremental, terukur, dan berbasis evaluasi ulang (Alseadoon, 2021; Cervantes & Jiménez, 2024).
- Mendukung **behavior-preserving refactoring**, yang krusial karena 22 metrik DSS harus tetap identik setelah optimasi (Mens & Tourwé, 2004; Noei et al., 2023).
- Cocok untuk **before–after performance evaluation**, karena studi continuous/performance engineering banyak memakai performance testing, benchmarking, dan model/evaluation pada sistem yang sudah berjalan (Eramo et al., 2023; Kumar & Mohd, 2026).

## Metode Alternatif

Tiga alternatif yang paling layak adalah **lifecycle refactoring 3 tahap**, **metodologi refactoring 4 tahap berbasis evaluasi ulang**, dan **performance engineering lifecycle**. Ketiganya diakui dalam literatur, tetapi masing-masing memiliki fokus yang lebih sempit daripada kebutuhan proyek Anda (Smiari et al., 2021; Cervantes & Jiménez, 2024; Haque, 2025).

- **Refactoring process 3 tahap**: planning, design, evaluation. Kuat untuk proyek refactoring terstruktur, tetapi kurang eksplisit pada deployment dan pengukuran baseline-vs-optimized (Smiari et al., 2021).
- **Metodologi refactoring 4 tahap**: evaluation, analysis and prioritization, refactoring, re-evaluation. Sangat dekat dengan proyek Anda karena menekankan evaluasi awal dan evaluasi ulang, tetapi cakupannya dikembangkan untuk atribut deployability, sehingga perlu adaptasi bila dipakai untuk performa backend dan kesetaraan hasil DSS (Cervantes & Jiménez, 2024).
- **Performance engineering lifecycle**: kuat untuk requirement kinerja, profiling, tuning, dan monitoring berulang, tetapi tidak secara inheren cukup untuk menjelaskan refactoring yang harus menjaga perilaku bisnis tetap sama (Mattoo, 2023).

## Pemetaan Tahap Koladi

Metode yang paling direkomendasikan untuk ditulis di proposal adalah **Adapted Iterative Software Evolution Method for Performance Refactoring**. Secara akademik, bentuk ini dapat dibenarkan sebagai adaptasi dari software evolution yang inkremental, refactoring yang behavior-preserving, dan performance engineering yang iteratif (Rajeeb & Mantri, 2026; Mens & Tourwé, 2004; Mattoo, 2023; Mumtaz et al., 2022).

| Stage | Theoretical Description | Concrete Activities in Koladi | Output/Deliverable |
|---|---|---|---|
| Identifikasi & Baseline | Memahami sistem existing, kebutuhan perubahan, dan bottleneck awal pada software yang telah dirilis (Alseadoon, 2021; Mattoo, 2023) | Profiling modul DSS dengan Laravel Telescope; mencatat response time, query count, memory usage; mengidentifikasi query-in-loop, N+1 pada relasi checklists, dan cache database yang lambat | Dokumen baseline kinerja, daftar bottleneck, target optimasi |
| Analisis & Prioritisasi | Menentukan area perubahan yang paling bernilai dan memilih refactoring yang tepat secara selektif (Mattoo, 2023; Tourwé & Mens, 2003) | Menetapkan prioritas intervensi: Redis cache-aside, invalidasi berbasis observer, selective eager loading, eliminasi N+1, dan batch retrieval untuk workload distribution | Matriks prioritas optimasi, rancangan perubahan teknis |
| Implementasi Refactoring | Menerapkan perubahan struktur/internal code tanpa mengubah perilaku eksternal sistem (Mens & Tourwé, 2004; Alseadoon, 2021; Mumtaz et al., 2022) | Implementasi Redis cache-aside; observer pada Task dan TaskAssignment; pembatasan kolom eager loading; refactor getWorkloadDistribution() menjadi batch retrieval dan agregasi in-memory | Kode hasil refactoring/optimasi, branch implementasi |
| Verifikasi Fungsional | Memastikan hasil bisnis tetap sama setelah refactoring melalui evaluasi dan pengujian (Smiari et al., 2021; Motogna et al., 2024; Nyirongo et al., 2024) | Menjalankan regression/characterization tests; membandingkan keluaran 22 metrik DSS sebelum dan sesudah; memverifikasi tidak ada perubahan perilaku | Laporan kesetaraan hasil 22 metrik, hasil uji regresi |
| Re-measurement Kinerja | Mengukur ulang performa setelah intervensi untuk menilai dampak optimasi secara empiris (Cervantes & Jiménez, 2024; Eramo et al., 2023; Haque, 2025) | Menjalankan skenario uji yang sama dengan baseline; membandingkan response time, query count, dan memory usage; menganalisis gain dan trade-off | Tabel perbandingan before–after, analisis peningkatan |
| Integrasi & Deployment | Mengintegrasikan perubahan ke sistem berjalan dan menyiapkan pemantauan pascadeploy sebagai bagian dari evolusi berkelanjutan (Mattoo, 2023; Kumar & Mohd, 2026; Arshad et al., 2025) | Merge ke branch utama, deployment bertahap, validasi cache invalidation di lingkungan integrasi/produksi, monitoring regresi performa | Sistem terintegrasi, catatan deployment, hasil monitoring awal |

**Figure 22.** Output measures / indices - Response time (Eramo et al., 2023)

*How to read: Gambar ini merangkum ukuran keluaran yang dipakai dalam studi performance engineering. Baca jenis metrik yang dipetakan, dengan perhatian khusus pada response time sebagai salah satu ukuran utama.*

## Rumusan Siap Pakai

Untuk Subbab 2.3, formulasi yang paling aman adalah menyatakan bahwa penelitian ini menggunakan **metode software evolution iteratif yang diadaptasi untuk refactoring dan performance engineering**. Dasarnya kuat karena software evolution membahas perubahan pada sistem setelah rilis (Rajeeb & Mantri, 2026), refactoring mempertahankan perilaku sambil memperbaiki struktur internal (Mens & Tourwé, 2004), dan performance engineering menuntut evaluasi kinerja berulang melalui profiling, tuning, dan re-measurement (Mattoo, 2023).

Secara praktis, untuk proyek Koladi, metode ini paling tepat karena menggabungkan **analisis bottleneck, perubahan kode terukur, pengujian kesetaraan hasil, dan evaluasi before–after** dalam satu alur yang cocok untuk sistem existing, bukan greenfield (Alseadoon, 2021; Eramo et al., 2023).

## Referensi Singkat

- Almogahed, A., Mahdin, H., Badreddine, S., Omar, M., Alawadhi, A., Barraood, S. O., Othman, M., Shaharudin, S. M., & Yaacob, S. S. (2026). *Towards an effective refactoring taxonomy for sustainable software systems*. PLOS One.  
- Alseadoon, I. (2021). *Motivations, challenges, and process support for the evolution of existing software to mobile computing platforms*. International Journal of Advanced and Applied Sciences.  
- Cervantes, H., & Jiménez, R. (2024). *Towards a Systematic Refactoring Method to Support Deployability*. 2024 12th International Conference in Software Engineering Research and Innovation (CONISOFT).  
- Eramo, R., Tucci, M., Di Pompeo, D., Cortellessa, V., Marco, A., & Taibi, D. (2023). *Architectural Support for Software Performance in Continuous Software Engineering: A Systematic Mapping Study*. Journal of Systems and Software.  
- Haque, W. (2025). *Designing a Software Performance Engineering Laboratory*. Journal of Software Engineering and Applications.  
- Mattoo, K. (2023). *Performance Engineering: The Crucial Step for Optimized System*. International Journal of Scientific Research in Engineering and Management.  
- Mens, T., & Tourwé, T. (2004). *A survey of software refactoring*. IEEE Transactions on Software Engineering.  
- Mens, T., Tourwé, T., & Munoz, F. (2003). *Beyond the refactoring browser: advanced tool support for software refactoring*. Proceedings of the Sixth International Workshop on Principles of Software Evolution.  
- Mumtaz, M., Ahmad, N., Ashraf, M., Alshaflut, A., Alourani, A., & Anjum, H. J. (tahun terbit sudah tersedia, detail volume/issue tidak lengkap pada data). *Modeling Iteration’s perspectives in Software Engineering*. IEEE Access.  
- Smiari, P., Bibi, S., Ampatzoglou, A., & Arvanitou, E.-M. (2021). *Refactoring embedded software: A study in healthcare domain*. Information and Software Technology.  

Jika diinginkan, jawaban ini bisa diubah lagi menjadi **narasi Subbab 2.3 siap-tempel** dalam format proposal kampus Anda.
 
_These search results were found and analyzed using Consensus, an AI-powered search engine for research. Try it at https://consensus.app. © 2026 Consensus NLP, Inc. Personal, non-commercial use only; redistribution requires copyright holders’ consent._
 
## References
 
R., & Mantri, P. (2026). Software Evolution Process Modeling and Optimization Applying Colored Petri Nets. *i-manager's Journal on Software Engineering*. https://doi.org/10.26634/jse.20.3.1362
 
Almogahed, A., Mahdin, H., Badreddine, S., Omar, M., Alawadhi, A., Barraood, S. O., Othman, M., Shaharudin, S. M., & Yaacob, A. S. S. (2026). Towards an effective refactoring taxonomy for sustainable software systems. *PLOS One, 21*. https://doi.org/10.1371/journal.pone.0336296
 
Alseadoon, I. (2021). Motivations, challenges, and process support for the evolution of existing software to mobile computing platforms. *International Journal of ADVANCED AND APPLIED SCIENCES*. https://doi.org/10.21833/ijaas.2021.05.011
 
Arshad, N., Butt, T., & Iqbal, M. (2025). A Comprehensive Framework for Intelligent, Scalable, and Performance-Optimized Software Development. *IEEE Access, 13*, 74062-74077. https://doi.org/10.1109/access.2025.3564139
 
Cervantes, H., & Jiménez, R. (2024). Towards a Systematic Refactoring Method to Support Deployability. *2024 12th International Conference in Software Engineering Research and Innovation (CONISOFT)*, 38-47. https://doi.org/10.1109/conisoft63288.2024.00016
 
Eramo, R., Tucci, M., Di Pompeo, D., Cortellessa, V., Marco, A., & Taibi, D. (2023). Architectural Support for Software Performance in Continuous Software Engineering: A Systematic Mapping Study. *J. Syst. Softw., 207*, 111833. https://doi.org/10.1016/j.jss.2023.111833
 
Haque, W. (2025). Designing a Software Performance Engineering Laboratory. *Journal of Software Engineering and Applications*. https://doi.org/10.4236/jsea.2025.183006
 
Kumar, A., & Mohd, W. G. (2026). An Automated Architectural Feedback Framework for Performance Optimization in Cloud-Native Software Systems. *International Journal of Drug Delivery Technology*. https://doi.org/10.25258/ijddt.16.59s.184
 
Mattoo, K. (2023). Performance Engineering: The Crucial Step for Optimized System. *INTERANTIONAL JOURNAL OF SCIENTIFIC RESEARCH IN ENGINEERING AND MANAGEMENT*. https://doi.org/10.55041/ijsrem24961
 
Mens, T., & Tourwé, T. (2004). A survey of software refactoring. *IEEE Transactions on Software Engineering, 30*, 126-139. https://doi.org/10.1109/tse.2004.1265817
 
Motogna, S., Berciu, L., & Moldovan, V. (2024). Artificial Intelligence Methods in Software Refactoring: A Systematic Literature Review. *2024 50th Euromicro Conference on Software Engineering and Advanced Applications (SEAA)*, 309-316. https://doi.org/10.1109/seaa64295.2024.00055
 
Mumtaz, M., Ahmad, N., Ashraf, M., Alshaflut, A., Alourani, A., & Anjum, H. J. (2022). Modeling Iteration’s perspectives in Software Engineering. *IEEE Access, PP*, 1-1. https://doi.org/10.1109/access.2022.3150878
 
Noei, S., Li, H., Georgiou, S., & Zou, Y. (2023). An Empirical Study of Refactoring Rhythms and Tactics in the Software Development Process. *IEEE Transactions on Software Engineering, 49*, 5103-5119. https://doi.org/10.1109/tse.2023.3326775
 
Nyirongo, B., Jiang, Y.-J., Jiang, H., & Liu, H. (2024). A Survey of Deep Learning Based Software Refactoring. *ACM Computing Surveys*. https://doi.org/10.1145/3849693
 
Smiari, P., Bibi, S., Ampatzoglou, A., & Arvanitou, E.-M. (2021). Refactoring embedded software: A study in healthcare domain. *Inf. Softw. Technol., 143*, 106760. https://doi.org/10.1016/j.infsof.2021.106760
 
Tourwé, T., & Mens, T. (2003). Identifying refactoring opportunities using logic meta programming. *Seventh European Conference onSoftware Maintenance and Reengineering, 2003. Proceedings.*, 91-100. https://doi.org/10.1109/csmr.2003.1192416
 
