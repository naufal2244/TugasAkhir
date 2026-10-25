Berikut adalah analisis mendalam mengenai siklus hidup refaktorisasi (*Refactoring Lifecycle / Activities*) berdasarkan artikel survei Tom Mens & Tom Tourwé (2004) berjudul *"A Survey of Software Refactoring"*, yang dirancang untuk kebutuhan **Subbab 2.3 Metode Pengembangan Produk** pada proposal Tugas Akhir proyek sistem **Koladi**.

---

### 1. DEFINISI FORMAL REFACTORING

#### Definisi Formal Menurut Mens & Tourwé
Dalam Seksi 1 (Hal. 127), Mens & Tourwé mengadopsi definisi dasar *refactoring* dari William Opdyke (1992) dan Martin Fowler (1999) sebagai varian berorientasi objek dari *restructuring*:

> *"Refactoring is basically the object-oriented variant of restructuring: 'the process of changing a [object-oriented] software system in such a way that it does not alter the external behavior of the code, yet improves its internal structure'"*.

Mens & Tourwé juga merujuk pada taksonomi Chikofsky & Cross (1990) mengenai *restructuring*, yaitu transformasi dari satu bentuk representasi ke bentuk lain pada tingkat abstraksi yang sama tanpa mengubah perilaku eksternal sistem (fungsionalitas dan semantik).

#### Tujuan Utama: Struktur Internal vs. Perilaku Eksternal
* **Perilaku Eksternal (*External Behavior*)**: Harus **tetap dipertahankan secara mutlak (*behavior-preserving*)**. Artinya, untuk himpunan nilai masukan (*input*) yang sama, sistem harus menghasilkan himpunan nilai keluaran (*output*) dan semantik yang identik sebelum dan sesudah refaktorisasi.
* **Struktur Internal (*Internal Structure*)**: Diperbaiki dan ditingkatkan kualitasnya (*internal quality attributes*). Tujuan utamanya adalah mendistribusikan ulang kelas, variabel, dan metode, mengurangi kompleksitas kode, serta meningkatkan kemudahan pemeliharaan (*maintainability*), keterbacaan (*understandability*), kemudahan pengembangan (*extensibility*), efisiensi/kinerja (*efficiency/performance*), dan kebolehulangan penggunaan (*reusability*).

---

### 2. TAHAPAN BAKU SIKLUS HIDUP REFACTORING (*THE REFACTORING ACTIVITIES*)

Sesuai Seksi 3 (Hal. 127–129), Mens & Tourwé mengidentifikasi **6 aktivitas baku** dalam proses refaktorisasi:

#### a. *Identify where the software should be refactored*
* **Penjelasan**: Mengidentifikasi bagian kode atau komponen sistem yang mengalami penurunan kualitas, memiliki kelemahan struktural, atau menjadi *bottleneck*.
* **Mekanisme**: Umumnya dilakukan dengan mendeteksi *bad code smells* (seperti *duplicated code*, *long method*, *god class*), menganalisis invariansi program dinamis, menggunakan metrik perangkat lunak (seperti metrik *cohesion* dan *coupling*), atau bantuan visualisasi perangkat lunak.

#### b. *Determine which refactoring(s) should be applied*
* **Penjelasan**: Memilih dan merancang teknik transformasi kode (*refactoring pattern/operation*) yang tepat untuk mengeliminasi kelemahan atau *bad smell* yang telah teridentifikasi.
* **Mekanisme**: Memetakan *bad smell* ke kandidat *primitive refactoring* (seperti *Move Method*, *Extract Method*, *Pull Up Method*) atau *composite refactoring* (seperti penerapan *Design Pattern* misal *Visitor Pattern*).

#### c. *Guarantee that the applied refactoring preserves behavior*
* **Penjelasan**: Menjamin secara ketat bahwa transformasi yang diterapkan tidak merubah perilaku atau fungsi eksternal sistem.
* **Mekanisme**: Dapat dicapai melalui pemeriksaan *preconditions* dan *postconditions* formal, pengetesan yang ketat (*rigorous unit testing discipline*), atau penjaminan tingkat bahasa seperti *type constraints*, *call preservation*, dan *access preservation*.

#### d. *Apply the refactoring*
* **Penjelasan**: Mengeksekusi transformasi kode pada basis kode (*codebase*).
* **Mekanisme**: Eksekusi dapat dilakukan secara manual, semi-otomatis melalui bantuan *refactoring browser* pada IDE, atau otomatis penuh. Penggunaan *composite refactoring* memungkinkan penggabungan urutan *primitive refactoring* agar eksekusi lebih efisien dan terstruktur.

#### e. *Assess the effect of refactoring on quality characteristics*
* **Penjelasan**: Menilai dan mengukur dampak dari refaktorisasi terhadap karakteristik kualitas internal dan eksternal sistem maupun proses pengembangannya.
* **Mekanisme**: Menggunakan pengukuran kuantitatif seperti metrik perangkat lunak (kompleksitas, keterikatan/keterpaduan), pengujian empiris (*empirical measurements*), eksperimen terkontrol, serta pengukuran performa/efisiensi eksekusi sebelum dan sesudah refaktorisasi.

#### f. *Maintain consistency between refactored code and other artifacts*
* **Penjelasan**: Menjaga dan memelihara konsistensi antara kode yang telah di-refaktor dengan artefak perangkat lunak lainnya.
* **Mekanisme**: Menerapkan propagasi perubahan (*change propagation*) pada model desain (UML), dokumen spesifikasi kebutuhan, *test suite/unit test*, dokumentasi API, serta konfigurasi lingkungan eksekusi.

---

### 3. RELEVANSI DAN PEMETAAN DENGAN PROYEK KOLADI

Berikut adalah pemetaan ke-6 tahapan siklus hidup refaktorisasi Mens & Tourwé (2004) ke dalam kegiatan teknis operasional pada proposal Tugas Akhir sistem **Koladi** (Subbab 2.3 Metode Pengembangan Produk):

| Tahapan Refaktorisasi (Mens & Tourwé, 2004) | Aktivitas Teknis Nyata pada Proyek Koladi | Poin Penekanan Metodologis (Proposal TA) |
| :--- | :--- | :--- |
| **a. Identify Refactoring Target** | **Profiling Query Bottleneck & N+1 Problem** | Menggunakan **Laravel Telescope** untuk memantau waktu eksekusi query SQL, menganalisis *redundant query* (masalah N+1 pada Eloquent ORM), serta mengidentifikasi alokasi memori puncak saat kalkulasi metrik DSS pada database PostgreSQL 15. |
| **b. Determine Refactoring Technique** | **Perancangan Arsitektur Optimasi ORM & Cache** | Merancang strategi *selective eager loading* (`with()`), *batch retrieval*, penerapan **Redis Cache-Aside pattern** untuk query DSS berbiaya komputasi tinggi, dan pembentukan **Eloquent Observers** untuk *cache invalidation/refresh* terotomasi. |
| **c. Guarantee Behavior Preservation** | **Pengujian Regresi & Ekuivalensi Fungsional** | Menjalankan *functional equivalence test* dan *automated unit/integration test* untuk menjamin bahwa **22 metrik Decision Support System (DSS)** Koladi menghasilkan nilai kalkulasi presisi yang 100% sama antara *pre-refactoring* dan *post-refactoring*. |
| **d. Apply Refactoring** | **Refaktorisasi Codebase Laravel 12** | Menerapkan transformasi kode secara aktual pada *Service Layer* dan *Eloquent Query Builder* Laravel 12, mengintegrasikan Redis client driver, serta merestrukturisasi method akses data yang lambat. |
| **e. Assess Quality Effect** | **Benchmarking Performa *Before-After*** | Mengukur dan membandingkan metrik performa secara kuantitatif sebelum dan sesudah refaktorisasi, meliputi *response time* (ms), *query count* ke PostgreSQL, dan *memory consumption* (MB) saat beban kerja puncak. |
| **f. Maintain Artifact Consistency** | **Integrasi Docker Compose & Dokumentasi** | Memperbarui spesifikasi arsitektur deployment pada **Docker Compose** (layanan Redis, App, PostgreSQL), menyelaraskan skenario *test suite*, serta memperbarui dokumentasi teknis API (OpenAPI/Swagger) dan dokumentasi kode. |

---

#### Kutipan Kalimat Kunci dari Naskah PDF (Mens & Tourwé, 2004) untuk Acuan Sitasi Proposal:
1. **Definisi Refactoring** (Seksi 1, Hal. 127):
   > *"Refactoring is basically the object-oriented variant of restructuring: 'the process of changing a [object-oriented] software system in such a way that it does not alter the external behavior of the code, yet improves its internal structure'"*.
2. **Preservasi Perilaku** (Seksi 3.2, Hal. 128):
   > *"The original definition of behavior preservation as suggested by Opdyke states that, for the same set of input values, the resulting set of output values should be the same before and after the refactoring"*.
3. **Dampak terhadap Performa/Kualitas** (Seksi 3.3, Hal. 128):
   > *"An important software quality characteristic that can be affected by refactoring is performance. It is a common misconception that improving the program structure has a negative effect on the program performance"*.
4. **Konsistensi Artefak** (Seksi 3.4, Hal. 128–129):
   > *"If we refactor any of these software artifacts, we need mechanisms to maintain their consistency with other related software artifacts"*.

---

💡 **Saran Langkah Selanjutnya:** Jika Anda membutuhkan draf narasi paragraf utuh dalam bahasa Indonesia yang siap dimasukkan langsung ke dalam naskah Subbab 2.3 proposal Tugas Akhir Anda, saya dapat bantu menyusunnya.