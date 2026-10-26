Berdasarkan artikel survei Tom Mens & Tom Tourwé (2004) berjudul *"A Survey of Software Refactoring"*, berikut adalah analisis mendalam yang difokuskan pada **Tahap 3 (*Guarantee behavior preservation*)** dan **Tahap 5 (*Assess quality effect*)** guna menjustifikasi metodologi pengujian komparatif dan penjaminan keabsahan luaran sistem Anda:

---

### 1. Definisi Preservasi Perilaku (*Behavior Preservation*) dan Aturan Formal Himpunan Nilai

Dalam Seksi 1 dan Seksi 3.2, Mens & Tourwé menegaskan bahwa hakikat fundamental dari refaktorisasi adalah **transformasi yang tidak mengubah perilaku eksternal sistem (*functionality and semantics*)**, meskipun struktur internalnya diperbaiki secara signifikan.

#### Aturan Formal Nilai Masukan dan Keluaran
Secara formal, Mens & Tourwé merujuk pada definisi awal yang dirumuskan oleh William F. Opdyke (1992):
> **Teks Asli (Seksi 3.2, Halaman 128):**  
> *"The original definition of behavior preservation as suggested by Opdyke states that, for the same set of input values, the resulting set of output values should be the same before and after the refactoring."*

**Penjelasan Aturan Formal:**
* Jika sistem sebelum refaktorisasi direpresentasikan sebagai fungsi \\(f_{\text{before}}\\) dan sesudah refaktorisasi sebagai \\(f_{\text{after}}\\), maka untuk setiap himpunan nilai masukan yang sama (\\(I\\)):
  \\[f_{\text{before}}(I) = f_{\text{after}}(I) = O\\]
* Sistem dinyatakan mematuhi prinsip *behavior preservation* apabila himpunan nilai keluaran (\\(O\\)) yang dihasilkan sebelum dan sesudah transformasi **bersifat identik secara fungsional dan semantik**.
* Untuk menjamin aturan formal ini terpenuhi, Opdyke dan Mens & Tourwé mengemukakan penegakan **prakondisi (*preconditions*)** dan **invarian (*invariants*)** sebelum transformasi dieksekusi, serta pemanfaatan **disiplin pengujian yang ketat (*rigorous testing discipline*)**.

---

### 2. Pengukuran Performa dan Kebutuhan Pengujian Empiris / Eksperimen Terkontrol

Pada Seksi 3.3 (*Assessing the Effect of Refactoring on Quality*, Halaman 128), Mens & Tourwé memaparkan evaluasi karakteristik kualitas eksternal, khususnya efisiensi dan performa komputasi:

#### a. Miskonsepsi dan Performa sebagai Atribut Kualitas Refaktorisasi
Mens & Tourwé membantah anggapan konvensional bahwa restrukturisasi desain perangkat lunak akan memperlambat eksekusi:
> *"An important software quality characteristic that can be affected by refactoring is performance. It is a common misconception that improving the program structure has a negative effect on the program performance."*

Mereka menjelaskan bahwa dalam paradigma fungsional dan logika, transformasi restrukturisasi secara eksplisit ditujukan untuk meningkatkan efisiensi program sembari menjaga semantik aslinya. Demikian pula pada sistem berorientasi objek, restrukturisasi struktural (misalnya penghapusan percabangan kondisional rumit menjadi polimorfisme) terbukti meningkatkan performa karena optimalisasi *runtime compiler*.

#### b. Mengapa Membutuhkan Pengujian Empiris Kuantitatif dan Eksperimen Terkontrol?
Mens & Tourwé menguraikan bahwa dampak refaktorisasi terhadap atribut kualitas internal (seperti *size*, *complexity*, *coupling*, dan *cohesion*) hanya dapat mengestimasi perbaikan kualitas eksternal "hingga batas tertentu" (*to a certain extent*). 

Performa, latensi waktu eksekusi, serta alokasi sumber daya sistem bersifat **dinamis saat *runtime***, sehingga estimasi statis tidaklah mencukupi. Oleh karena itu, Mens & Tourwé menegaskan perlunya instrumen pengukuran langsung:
> *"To measure or estimate the impact of a refactoring on quality characteristics, many different techniques can be used. Examples include, but are not limited to, software metrics, empirical measurements, controlled experiments, and statistical techniques."*

Eksperimen terkontrol (*controlled experiments*) dan pengukuran empiris kuantitatif diperlukan untuk memisahkan dampak transformasi kode dari faktor pengganggu eksternal, sehingga perubahan performa dapat dibuktikan secara statistik dan ilmiah.

---

### 3. Urgensi Penggunaan Kumpulan Data Uji yang Setara (*Equivalent Test Datasets*)

Dalam konteks pengujian komparatif (*before-after benchmark*), penggunaan himpunan data uji yang identik/setara (*equivalent dataset*) menjadi syarat mutlak karena dua alasan metodologis utama yang saling mengikat:

1. **Validitas Komparasi Performa (*Controlling Experimental Variables*)**:  
   Sesuai prinsip eksperimen terkontrol pada Seksi 3.3, penilaian dampak efisiensi memerlukan lingkungan eksperimen di mana satu-satunya variabel bebas yang berubah adalah arsitektur kode/mekanisme *caching*. Jika volume, distribusi, atau parameter data masukan (*input values*) berbeda antara kondisi *pre-refactoring* dan *post-refactoring*, maka variasi metrik performa (seperti *response time* dan konsumsi memori) tidak lagi valid untuk diklaim sebagai hasil dari optimasi sistem.
2. **Penjaminan Preservasi Semantik Sistem (*Validation of Functional Equivalence*)**:  
   Pengujian performa harus berjalan beriringan dengan penjaminan preservasi perilaku. Pendekatan pragmatis melalui *rigorous testing discipline* menuntut pengujian terhadap himpunan kasus uji yang representatif. Dengan menyuplai kumpulan data masukan yang setara (\\(I\\)), peneliti dapat memverifikasi dua hal sekaligus:
   * **Secara fungsional (Tahap 3):** Memastikan nilai luaran (\\(O\\)) sebelum dan sesudah refaktorisasi tidak mengalami deviasi sedikit pun.
   * **Secara non-fungsional (Tahap 5):** Mengukur secara objektif penurunan durasi eksekusi atau beban *query* yang terjadi saat memproses beban data yang sama persis.

---

### 4. Kutipan Kunci Relevan dari Mens & Tourwé (2004) untuk Menjustifikasi Metodologi Penelitian

Kutipan-kutipan berikut dapat dicantumkan langsung sebagai landasan teoritis pada naskah proposal Anda:

1. **Landasan Penjaminan Ekuivalensi Luaran (Seksi 3.2, Halaman 128):**
   > *"The original definition of behavior preservation as suggested by Opdyke states that, for the same set of input values, the resulting set of output values should be the same before and after the refactoring."*
2. **Disiplin Pengujian sebagai Pembuktian Preservasi Perilaku (Seksi 3.2, Halaman 128):**
   > *"One may deal with behavior preservation in a very pragmatic way, for example, by means of a rigorous testing discipline. If we have an extensive set of test cases and all these tests still pass after the refactoring, there is good evidence that the refactoring preserves the program behavior."*
3. **Peningkatan Performa sebagai Dampak Sah Refaktorisasi (Seksi 3.3, Halaman 128):**
   > *"An important software quality characteristic that can be affected by refactoring is performance. It is a common misconception that improving the program structure has a negative effect on the program performance."*
4. **Metodologi Pengukuran Empiris dan Eksperimen Terkontrol (Seksi 3.3, Halaman 128):**
   > *"To measure or estimate the impact of a refactoring on quality characteristics, many different techniques can be used. Examples include, but are not limited to, software metrics, empirical measurements, controlled experiments, and statistical techniques."*

---

💡 *Apakah Anda ingin saya bantu menyusun matriks rancangan pengujian (tabel komparasi sebelum-sesudah) untuk memvalidasi ekuivalensi fungsional dan tolok ukur performa sistem Koladi Anda sesuai kaidah di atas?*

Berdasarkan buku Martin Fowler (*Refactoring: Improving the Design of Existing Code*), berikut adalah sintesis komprehensif konsep-konsep evaluasi performa, pengujian regresi, dan jaminan preservasi perilaku untuk landasan metodologis proposal Anda:

---

### 1. Pandangan Fowler tentang Pengujian dan Optimasi Performa yang Benar

Dalam **Chapter 2** pada subbab **"Refactoring and Performance"**, Fowler menguraikan bahwa rahasia menghasilkan perangkat lunak yang cepat adalah **menulis perangkat lunak yang mudah di-tuning terlebih dahulu (*tunable software*), kemudian melakukan tuning untuk mencapai kecepatan yang memadai**. 

Fowler membandingkan tiga pendekatan optimasi [105–107]:
1. ***Time Budgeting*:** Pembagian kuota sumber daya ketat per komponen, lazim pada *hard real-time systems* (seperti *pacemaker*) namun *overkill* untuk sistem informasi.
2. ***Constant Attention*:** Setiap pengembang berusaha mengoptimasi kecepatan di setiap baris kode sepanjang waktu. Fowler menyebut pendekatan ini **tidak efektif dan kontraproduktif**, karena optimasi mikro yang disebar secara acak justru merusak kejelasan kode dan memperlambat laju pengembangan tanpa menghasilkan peningkatan kecepatan yang signifikan.
3. ***Late Performance Tuning* Berbasis Titik Panas (*Hot Spots*):** Pendekatan yang dianjurkan Fowler dengan memanfaatkan fakta empiris bahwa **sebagian besar program menghabiskan 90% waktu eksekusinya hanya pada sebagian kecil kode (sekitar 10%)**. Mengoptimasi seluruh kode secara merata akan membuang 90% usaha pada kode yang jarang dieksekusi.

#### Mengapa Prinsip *"Measure Performance, Don't Speculate"* Mutlak Diperlukan?
Fowler dan Ron Jeffries (dalam sidebar studi kasus sistem kompensasi Chrysler C3) menegaskan bahwa **intuisi pengembang mengenai letak *bottleneck* hampir selalu keliru** [101–104]. Pada sistem C3, para pengembang senior berspekulasi dan merancang solusi rumit yang mereka duga menjadi penyebab kelambatan, namun saat diukur menggunakan *profiler*, dugaan mereka terbukti salah total; masalah utamanya ternyata hanya pembuatan instansiasi objek tanggal berulang [101–104].

Oleh karena itu, Fowler menetapkan prosedur tuning yang disiplin:
* **Gunakan *profiler*** untuk memantau konsumsi waktu dan ruang memori secara riil guna menemukan *performance hot spots*.
* **Fokuskan optimasi hanya pada *hot spots*** tersebut.
* Terapkan perubahan dalam **langkah-langkah kecil (*small steps*)**, lalu lakukan kompilasi, jalankan rangkaian pengujian (*tests*), dan ukur ulang dengan *profiler*.
* **Batalkan perubahan (*back out the change*)** jika *profiler* menunjukkan bahwa modifikasi kode tidak memberikan peningkatan performa yang nyata.
* Kode yang telah direfaktorisasi rapi (*well-factored*) menyediakan **granularitas analisis yang lebih halus**, sehingga mempermudah *profiler* melacak fungsi-fungsi spesifik yang bermasalah.

---

### 2. Konsep "Self-Testing Code" dalam Menjamin *Observable Behavior*

Dalam **Chapter 1** dan **Chapter 4 ("Building Tests")**, Fowler menegaskan bahwa memiliki rangkaian pengujian otomatis (*test suite*) adalah **prasyarat mutlak (*essential precondition*)** sebelum memodifikasi kode.

#### A. Mekanisme *Self-Testing Code*
* **Pemeriksaan Mandiri (*Self-Checking*):** Pengujian dikatakan *self-testing* jika komputer memverifikasi hasilnya sendiri secara otomatis. Tes membandingkan luaran aktual program dengan nilai rujukan yang telah ditentukan (*expected/reference values*) menggunakan asersi otomatis (seperti `assertEquals`).
* **Umpan Balik Biner yang Cepat:** Pengujian hanya perlu menampilkan status **"OK"** jika seluruh asersi lulus, atau mencetak daftar kegagalan (*failures*) dan galat (*errors*) jika terjadi diskrepansi nilai [33, 140, 151, 153–156]. Hal ini menghilangkan proses manual yang lambat dan rentan salah dalam memeriksa angka di layar.

#### B. Menjamin *Observable Behavior* Tidak Berubah (*Behavior-Preserving*)
* Refaktorisasi didefinisikan sebagai restrukturisasi internal tanpa mengubah perilaku teramati (*observable behavior*)—di mana pengguna akhir maupun komponen pemanggil lainnya tidak boleh menyadari adanya perubahan fungsional [79–81].
* Rangkaian tes otomatis bertindak sebagai **detektor *bug* yang presisi**. Fowler menerapkan ritme kerja: **uji, lakukan perubahan kecil, uji kembali (*test, small change, test*)**. 
* Karena setiap langkah restrukturisasi sangat kecil, jika sebuah pengujian menghasilkan kegagalan (*failure*), pengembang langsung mengetahui bahwa letak galat berada tepat pada baris kode yang baru saja dimodifikasi beberapa menit sebelumnya, sehingga kondisi ekuivalensi fungsional dapat dipulihkan secara instan.

---

### 3. Penyiapan Data Uji Representatif dan Pembuktian Ekuivalensi Formula

Dalam bab 1 dan 4, Fowler menjelaskan metodologi penyiapan data uji (*test fixtures*) untuk membuktikan bahwa logika bisnis tidak terdistorsi [33, 147, 164–167]:

1. **Pembentukan *Test Fixture* Dasar:**
   Fowler mencontohkan pembuatan objek sampel dengan variasi karakteristik lengkap (misalnya kombinasi berbagai jenis sewa dan durasi pada sistem rental video) untuk menghasilkan string rujukan (*reference strings*) yang telah divalidasi kebenarannya.
2. **Eksplorasi Kondisi Batas (*Boundary Conditions*):**
   Fowler memberikan pedoman: *"Think of the boundary conditions under which things might go wrong and concentrate your tests there"*. Pengujian formula tidak boleh hanya menguji kondisi nominal/normal, melainkan wajib mencakup nilai batas awal, nilai batas akhir, batas luar, koleksi kosong, serta kondisi khusus/ekstrem [164–167].
3. **Pembuktian Ekuivalensi Matematis 100% Identik:**
   Pada **Chapter 15**, Fowler menegaskan: *"When you are refactoring, your goal is to leave the code computing exactly the same answers it was when you found it. Nothing more, nothing less"*.  
   Hubungan ilmiahnya: jika himpunan data uji (*test suite*) telah mencakup variasi kasus normal dan kondisi batas, kemudian seluruh luaran numerik sebelum optimasi dijadikan nilai ekspektasi (*baseline oracle*) yang diuji silang menggunakan asersi otomatis (`assertEquals`), maka kelulusan uji tersebut membuktikan secara deterministik bahwa restrukturisasi algoritma/kueri bersifat *behavior-preserving* dan luaran formula kalkulasi 100% identik.

---

### 4. Kutipan Kunci Martin Fowler untuk Proposal Akademik

Berikut adalah kutipan langsung (*direct quotes*) dari buku Fowler yang relevan untuk mendukung metodologi profiling performa dan pengujian ekuivalensi fungsional:

#### A. Profiling dan Evaluasi Performa Terukur
* **Prinsip *Measure, Don't Speculate*:**
  > *"Even if you know exactly what is going on in your system, measure performance, don't speculate. You'll learn something, and nine times out of ten, it won't be that you were right!"*  
  — **Chapter 2 (Principles in Refactoring), hlm. 58**.

* **Strategi *Tunable Software*:**
  > *"The secret to fast software, in all but hard real-time contexts, is to write tunable software first and then to tune it for sufficient speed."*  
  — **Chapter 2 (Principles in Refactoring), hlm. 59**.

* **Penggunaan *Profiler* untuk Menemukan *Hot Spots*:**
  > *"You begin by running the program under a profiler that monitors the program and tells you where it is consuming time and space. This way you can find that small part of the program where the performance hot spots lie."*  
  — **Chapter 2 (Principles in Refactoring), hlm. 59**.

* **Siklus Tuning Terisolasi dan *Rollback*:**
  > *"As in refactoring you make the changes in small steps. After each step you compile, test, and rerun the profiler. If you haven't improved performance, you back out the change."*  
  — **Chapter 2 (Principles in Refactoring), hlm. 59**.

* **Granularitas Hasil Refaktorisasi terhadap Profiling:**
  > *"Second, with a well-factored program you have finer granularity for your performance analysis. Your profiler leads you to smaller parts of the code, which are easier to tune."*  
  — **Chapter 2 (Principles in Refactoring), hlm. 59**.

#### B. Pengujian Otomatis dan Preservasi Perilaku
* **Prasyarat Pengujian Otomatis (*Self-Checking Tests*):**
  > *"Before you start refactoring, check that you have a solid suite of tests. These tests must be self-checking."*  
  — **Chapter 1 (Refactoring, a First Example), hlm. 17**.

* **Otomasi Penuh Hasil Uji:**
  > *"Make sure all tests are fully automatic and that they check their own results."*  
  — **Chapter 4 (Building Tests), hlm. 73**.

* **Kondisi Batas (*Boundary Conditions*):**
  > *"Think of the boundary conditions under which things might go wrong and concentrate your tests there."*  
  — **Chapter 4 (Building Tests), hlm. 82**.

* **Preservasi Perilaku Komputasi Identik:**
  > *"When you are refactoring, your goal is to leave the code computing exactly the same answers it was when you found it. Nothing more, nothing less."*  
  — **Chapter 15 (Putting It All Together), hlm. 334**.

---

💡 **Saran Integrasi Naskah:**  
Apakah Anda ingin saya menyusun draf narasi akademik utuh untuk Subbab 2.3 yang mengintegrasikan kutipan Fowler di atas ke dalam diagram alir pengujian modul DSS Anda (misalnya memetakan *baseline test fixture* dengan 22 metrik dan *profiling loop* dengan Laravel Telescope)?