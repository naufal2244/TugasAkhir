Tentu, berikut adalah penyusunan sintesis akademis berdasarkan buku karya **Martin Fowler** (*Refactoring: Improving the Design of Existing Code*) untuk bahan penyusunan **Subbab 2.3 Metode Pengembangan Produk** pada proposal Tugas Akhir Anda.

---

### 1. Definisi Refactoring & Aturan "The Two Hats"

#### A. Definisi Refactoring menurut Fowler
Fowler menjelaskan bahwa istilah *refactoring* memiliki dua definisi formal tergantung konteks penggunaannya:

* **Sebagai Kata Benda (*Noun*)**:
  > *"A change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior."*  
  *(Suatu perubahan yang dilakukan pada struktur internal perangkat lunak untuk membuatnya lebih mudah dipahami dan lebih murah untuk dimodifikasi tanpa mengubah perilaku teramatinya)*. Contohnya adalah penerapan pola seperti *Extract Method* atau *Move Field*.
* **Sebagai Kata Kerja (*Verb*)**:
  > *"To restructure software by applying a series of refactorings without changing its observable behavior."*  
  *(Meresrukturisasi perangkat lunak dengan menerapkan serangkaian refaktorisasi tanpa mengubah perilaku teramatinya)*.

Secara umum, *refactoring* adalah proses mengubah sistem perangkat lunak sedemikian rupa sehingga tidak mengubah perilaku eksternal kode, namun meningkatkan struktur internalnya setelah kode tersebut ditulis.

#### B. Metafora "The Two Hats" (Dua Topi)
Fowler mengutip metafora Kent Beck mengenai **"The Two Hats"**, yaitu pemisahan tegas antara dua aktivitas utama dalam pengembangan perangkat lunak:

1. **Topi Menambah Fungsi (*Adding Function*)**: Fokus utama adalah menambahkan kapabilitas/fitur baru. Pada tahap ini, pengembang tidak boleh mengubah kode yang sudah ada, melainkan hanya menambahkan fungsi baru dan memastikan uji otomatis (*tests*) bekerja.
2. **Topi Refaktorisasi (*Refactoring*)**: Fokus utama adalah merestrukturisasi kode tanpa menambahkan fitur baru. Pengembang tidak menambahkan pengujian baru (kecuali jika menemukan kasus uji yang terlewat) dan hanya mengubah pengujian jika antarmuka (*interface*) mengalami perubahan.

**Mengapa Keduanya Tidak Boleh Dicampuradukkan?**  
Pengembang dapat berpindah topi secara bergantian dalam hitungan menit. Namun, mencampuradukkan kedua aktivitas dalam satu langkah kerja akan meningkatkan risiko kesalahan: ketika *bug* terjadi, pengembang akan kesulitan mengidentifikasi apakah *bug* disebabkan oleh logika fitur baru atau oleh perubahan struktur kode lama. Dengan memisahkan kedua topi ini, kemajuan pengembangan menjadi terukur dan risiko regresi dapat diminimalkan.

---

### 2. Refactoring dan Performa Sistem (*Refactoring and Performance*)

#### A. Bagian Khusus dalam Buku Fowler
Topik ini dibahas secara khusus pada **Chapter 2** dalam subbab **"Refactoring and Performance"**.

#### B. Pandangan Fowler tentang Optimasi Performa yang Benar
Fowler menekankan bahwa kunci menghasilkan perangkat lunak yang cepat adalah **menulis perangkat lunak yang mudah di-tuning terlebih dahulu (*tunable software*), kemudian melakukan tuning untuk mencapai kecepatan yang cukup**.

* **Dilarang Menebak Bottleneck (*Don't Speculate*)**:  
  Fowler dan Ron Jeffries menegaskan bahwa sejauh apa pun pengembang mengenal sistemnya, pengembang tidak boleh menebak sumber masalah performa. Fowler mengutip fakta bahwa sebagian besar program menghabiskan **90% waktunya hanya pada sebagian kecil kode (hot spots)**. Optimasi yang dilakukan secara asal/merata akan membuang 90% usaha pada kode yang jarang dieksekusi.
* **Pentingnya Pengukuran Menggunakan Profiler**:  
  Pendekatan optimasi yang tepat adalah menjalankan program di bawah *profiler* yang memantau konsumsi waktu dan memori. *Profiler* akan menuntun pengembang tepat pada titik panas (*performance hot spots*). Pengembang kemudian melakukan optimasi secara bertahap (*small steps*), mengompilasi, menguji, dan mengukur ulang dengan *profiler*. Jika langkah optimasi tidak meningkatkan performa, perubahan tersebut harus dibatalkan (*backed out*).
* **Bagaimana Refactoring Mempermudah Tuning Performa**:  
  1. *Hemat Waktu*: Kode yang terstruktur baik (*well-factored*) memungkinkan penambahan fitur lebih cepat, sehingga memberikan sisa waktu lebih banyak untuk fokus pada *performance tuning*.
  2. *Granularitas Lebih Halus*: Kode hasil refaktorisasi memiliki fungsi-fungsi yang lebih kecil dan jelas, sehingga *profiler* dapat menunjuk komponen kode yang sangat spesifik dan mudah dianalisis serta dioptimasi.

---

### 3. Jaminan Preservasi Perilaku & *Self-Testing Code*

#### A. Pentingnya *Self-Testing Code*
Fowler menyatakan dalam **Chapter 4 ("Building Tests")** bahwa syarat mutlak sebelum melakukan refaktorisasi adalah memiliki rangkaian pengujian otomatis yang andal (*solid suite of self-checking tests*).

* **Mekanisme Otomatis (*Self-Checking*)**: Pengujian harus memverifikasi hasilnya sendiri secara otomatis (menghasilkan status "OK" atau daftar kegagalan) tanpa memerlukan inspeksi manual oleh manusia.
* **Detektor Bug yang Kuat**: Rangkaian tes otomatis yang dijalankan secara berkala bertindak sebagai detektor *bug* yang efektif. Karena perubahan kode dilakukan dalam langkah-langkah kecil (*small steps*), jika sebuah pengujian gagal, pengembang langsung mengetahui bahwa letak kesalahan berada pada potongan kode kecil yang baru saja diubah.

#### B. Mengapa Preservasi *Observable Behavior* adalah Syarat Mutlak
Preservasi perilaku yang teramati (*observable behavior*) adalah batasan mendasar yang membedakan *refactoring* dari aktivitas pemrograman lainnya. Pengguna sistem (baik *end-user* maupun modul/program lain yang memanggil kode tersebut) tidak boleh merasakan perubahan keluaran fungsional. Tanpa jaminan ini, restrukturisasi internal berisiko tinggi menyisipkan cacat (*bugs*) baru ke dalam sistem yang sudah berjalan.

---

### 4. Relevansi dengan Proyek Optimasi Modul DSS Koladi

Prinsip-prinsip Fowler di atas memberikan dasar teoritis yang kokoh untuk Subbab 2.3 proposal Anda:

#### A. Justifikasi Verifikasi Identik pada 22 Metrik Produktivitas DSS
* **Landasan Teoritis**: Mengacu pada prinsip bahwa *refactoring* dan optimasi **tidak boleh mengubah *observable behavior***. Pada modul Decision Support System (DSS), keluaran numerik dari 22 metrik produktivitas merupakan perilaku teramati (*observable behavior*) utama dari *domain logic* sistem.
* **Penerapan Metodologis**: Rangkaian unit/integration test otomatis yang mengevaluasi 22 metrik tersebut sebelum dan sesudah optimasi berperan sebagai ***Self-Testing Code***. Verifikasi bahwa nilai numerik sebelum dan sesudah optimasi menghasilkan nilai yang identik (*100% match*) membuktikan secara ilmiah bahwa optimasi struktur/kueri yang dilakukan bersifat *behavior-preserving* dan tidak merusak keabsahan formula DSS.

#### B. Justifikasi Penggunaan Laravel Telescope sebagai Profiler
* **Landasan Teoritis**: Mengacu pada prinsip Fowler **"measure performance, don't speculate"** dan pemanfaatan *profiler* untuk menemukan *hot spots*.
* **Penerapan Metodologis**: Penggunaan Laravel Telescope sebagai *profiler* pada ekosistem Laravel selaras dengan ajaran Fowler untuk mengukur konsumsi *query execution time*, deteksi kueri *N+1*, dan *memory allocation* secara nyata. Tanpa *profiler*, pengembang berisiko melakukan optimasi spekulatif pada kode yang bukan menjadi hambatan utama. Telescope memandu pengembang untuk fokus melakukan refaktorisasi kueri (seperti *eager loading*) tepat pada lokasi *bottleneck* yang terbukti secara empiris.

---

### Kutipan Kalimat Kunci dari Buku Fowler (Untuk Dikutip dalam Proposal)

Anda dapat mengutip kalimat-kalimat berikut langsung pada draf proposal Anda:

1. **Definisi Refactoring**:
   > *"Refactoring (noun): a change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior."* (Fowler et al., 2002, hlm. 46).

2. **Prinsip Optimasi Performa**:
   > *"The secret to fast software, in all but hard real-time contexts, is to write tunable software first and then to tune it for sufficient speed."* (Fowler et al., 2002, hlm. 59).

3. **Prinsip Pengukuran/Profiling (Don't Speculate)**:
   > *"Even if you know exactly what is going on in your system, measure performance, don't speculate."* (Fowler et al., 2002, hlm. 58).

4. **Prinsip Pengujian Otomatis**:
   > *"Before you start refactoring, check that you have a solid suite of tests. These tests must be self-checking."* (Fowler et al., 2002, hlm. 17).

---

💡 **Saran Langkah Selanjutnya:**  
Apakah Anda ingin saya membantu menyusunkan draf paragraf naskah ilmiah untuk Subbab 2.3 berdasarkan poin-poin di atas, atau mengelaborasikan bagian pengujian integrasi kueri Laravel secara lebih spesifik?