# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 3)
**File:** `log/Log3.md`  
**Waktu Pembaruan:** 2026-10-25  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan, verifikasi literatur, penegasan posisi teoritis, serta perbaikan naskah proposal pada Sesi 3 agar sesi selanjutnya dapat melanjutkan tanpa kehilangan detail argumen dan keputusan ilmiah yang telah disepakati.

---

## 1. PENYELARASAN METODOLOGI RISET (BAB 2.4 & SUBBAB TERKAIT)

### A. Duduk Perkara Wohlin et al. (2000) vs. Henning & Hasselbring (2022)
* **Kekhawatiran Awal:** Timbul kebingungan mengapa ada dua rujukan metodologi utama (Wohlin dan Henning) yang terkesan "dipadukan seenak jidat" tanpa ada satu paper pun di literatur review yang secara eksplisit memadukan keduanya.
* **Hasil Penelusuran Literatur:**
  1. Di dalam literatur review tidak pernah ada paper yang berjudul atau mengklaim "memadukan metode Wohlin dan Henning".
  2. **Wohlin et al. (2000; 2012)** adalah **Buku Teks Metodologi Riset** (*Experimental Software Engineering*) yang menjadi standar akademik kampus untuk penelitian eksperimental rekayasa perangkat lunak dengan desain *before-after* (juga digunakan pada proposal Dzakwan).
  3. **Henning & Hasselbring (2022)** adalah **Paper Jurnal Praktik Benchmarking** (*Empirical Software Engineering*, Springer) yang membahas protokol teknis pengujian performa sistem cloud/web (standarisasi kontainer Docker dan prosedur membuang 3 *warm-up requests* untuk mengeliminasi bias *cold-start*).
* **Keputusan & Perbaikan:**
  * Menghapus frasa *"penelitian ini memadukan kerangka kerja Wohlin dengan rancangan Henning"* pada kalimat pembuka Subbab 2.4.
  * **Wohlin et al. (2000; 2012)** berdiri murni sebagai **kerangka metodologi riset utama** di Subbab 2.4.
  * **Henning & Hasselbring (2022)** ditempatkan pada proporsi aslinya sebagai **rujukan protokol teknis pengujian server** pada Subbab 2.4.1 (isolasi Docker) dan Subbab 2.4.3 (prosedur 3 *warm-up requests* yang dibuang).

### B. Dekonstruksi & Pemahaman 4 Langkah Analisis Data (Subbab 2.4.4)
* **Statistik Deskriptif:** Menggunakan *Mean* untuk ukuran pemusatan utama, serta *Min*, *Max*, dan *Standard Deviation* (SD) untuk mengevaluasi stabilitas dan konsistensi server antar-repetisi (Japke et al., 2025).
* **Persentase Perbaikan (*Practical Improvement*):** Mengukur magnitudo perbaikan dengan formula:
  $$\text{Improvement (\%)} = \left(\frac{\text{Mean Baseline} - \text{Mean Optimized}}{\text{Mean Baseline}}\right) \times 100\%$$
  Perubahan nilai di bawah 3% dikategorikan secara hati-hati sebagai variasi wajar (*measurement noise*) sistem web dan tidak diklaim sebagai hasil optimasi murni (Japke et al., 2025).
* **Uji Hipotesis Komparatif (*Paired-Sample t-Test*):** Digunakan untuk membuktikan bahwa perbedaan performa bukan faktor kebetulan, melainkan signifikan secara statistik pada tingkat kepercayaan 95% ($\alpha = 0,05$) (Uddin & Lu, 2024). Valid karena pasangan observasi memiliki skala data, endpoint, kondisi cache, dan urutan repetisi yang identik.
* **Kaidah Khusus Data Deterministik ($p < 0,001$) & Evaluasi Cache:**
  * *Mengapa muncul $p < 0,001$?* Pada metrik *SQL Query Count*, angkanya bernilai pasti/konstan di setiap repetisi sehingga variansi selisih antar-pasangan adalah nol ($SD = 0$). Di kalkulator komputer, rumus t-test menghasilkan nilai $t$ ekstrem dan nilai $p = 0.000000$. Kaidah pelaporan ilmiah internasional melarang penulisan $p = 0$ mutlak, melainkan wajib dilaporkan sebagai **$p < 0,001$** dengan penekanan analisis pada reduksi absolut kueri (Japke et al., 2025; Uddin & Lu, 2024).
  * *Evaluasi Cache Hit Ratio:* Dianalisis bersamaan (*joint interpretation*) dengan *SQL Query Count* untuk membuktikan apakah status *cache hit* benar-benar bertindak sebagai *full database bypass* atau masih menyisakan kueri SQL tersembunyi (*partial cache hit*, An & Cao, 2022).
* **Hubungan Wohlin dengan Sitasi Lain di 2.4.4:**
  Wohlin et al. (2000) menyediakan **alur tahapan eksperimen umum** (Deskriptif $\rightarrow$ Ukuran Efek $\rightarrow$ Uji Hipotesis), sedangkan Japke, Uddin & Lu, dan An & Cao mengisi **standar parameter dan aturan teknis spesifik** pada domain *web/database benchmarking*.

---

## 2. PENYELARASAN METODE PENGEMBANGAN PRODUK (BAB 2.3)

### A. Hubungan Teoretis Mens & Tourwé (2004) dan Martin Fowler (2018)
* **Temuan Literatur:** Di paper aslinya (*A Survey of Software Refactoring*, IEEE TSE 2004, hal. 127), **Mens & Tourwé secara eksplisit mengutip dan mengadopsi definisi Martin Fowler** mengenai restrukturisasi berorientasi objek yang mempertahankan perilaku eksternal (*behavior-preserving*).
* **Keputusan:** Menghilangkan kata "dipadukan", dan menegaskan bahwa kerangka *Refactoring Lifecycle* Mens & Tourwé (2004) memang berlandaskan pada prinsip dasar refaktorisasi Martin Fowler (2018).

### B. Pemahaman Tahap 2.3.3: Penjaminan Preservasi Perilaku (*Guarantee Behavior Preservation*)
* Tujuannya memastikan bahwa optimasi kueri dan Redis tidak merusak logika kalkulasi bisnis DSS.
* Dibuktikan melalui penyusunan pengujian ekuivalensi otomatis (*automated equivalence test*) yang membandingkan respons data JSON dari **22 metrik performa tim dan 5 rekomendasi manajerial** sebelum dan sesudah refaktorisasi hingga terbukti 100% identik.

### C. Pemahaman & Dekonstruksi Tahap 2.3.6: Pemeliharaan Konsistensi Sistem (*Maintain Consistency*)
* **Kewajiban 6 Langkah:** Dalam kerangka Mens & Tourwé (2004), siklus refaktorisasi secara baku memang terdiri atas 6 aktivitas (a s.d. f), sehingga tahap ke-6 wajib ada agar metodologi tidak terpotong.
* **Tiga Dimensi Konsistensi Operasional (Bukan Uji Seluruh Modul Koladi):**
  1. *Konsistensi Lingkungan (Infrastruktur):* Memverifikasi orkestrasi kontainer Docker Compose (`koladi_net`) berjalan stabil (trio layanan Laravel 12, PostgreSQL 15, dan Redis 7 berstatus *Up/Healthy*).
  2. *Ketiadaan Efek Samping (Aplikasi):* Menjamin penambahan `TaskObserver` dan `TaskAssignmentObserver` untuk invalidasi cache tidak merusak siklus hidup tugas (*task lifecycle*: create, update, delete) pada modul tugas (Kanban). Mahasiswa **tidak perlu** membuat instrumen pengujian terpisah untuk seluruh aplikasi dan **tidak perlu** membuat tabel Pass/Fail 50 fitur lain.
  3. *Konsistensi Dokumentasi (Artefak Non-Kode):* Menjawab definisi *"other related software artifacts"* dari Mens & Tourwé melalui pembaruan parameter Redis pada file konfigurasi (`.env.example`), diagram arsitektur sistem, dan spesifikasi kontrak antarmuka API DSS.

---

## 3. PENYELARASAN SITASI LANDASAN TEORI (BAB 2.2)

1. **Subbab 2.2.1 (DSS vs. Response Time):**
   * Kalimat *"Karena modul DSS harus mengeksekusi pemrosesan analitis intensif... response time menjadi metrik utama (Putra et al., 2025; Alnuhait et al., 2024)"* telah diluruskan.
   * **Fakta:** Putra et al. dan Alnuhait et al. meneliti performa web secara umum, bukan DSS. Pihak yang membahas DSS adalah **Cai & Chen (2021)** dan **Koornneef et al. (2020)**.
   * **Solusi:** Sitasi dipisahkan: Cai & Chen (2021) mengawal kebutuhan kecepatan analitis DSS, sedangkan Alnuhait et al. (2024) dan Putra et al. (2025) mengawal definisi standar metrik *response time* pada aplikasi web.
2. **Subbab 2.2.2 (Sumber Latensi RDBMS Disk):**
   * Pernyataan bahwa akses disk RDBMS adalah sumber utama latensi terbukti berdasar kuat pada **Privalov & Stupina (2024)** (disk I/O, filtering, sorting kueri RDBMS sebagai bottleneck utama) dan **Taipalus (2024)** (latensi RAM sekitar 4 orde magnitudo / 10.000x lebih cepat daripada disk).
3. **Subbab 2.2.3 (Pola Active Record & Abstraksi ORM):**
   * Pola *Active Record* berakar pada **Martin Fowler (2002)** (*Patterns of Enterprise Application Architecture*).
   * Kritik bahwa abstraksi ORM menyembunyikan biaya eksekusi kueri SQL di belakang layar berdasar kuat pada premis utama paper **Alshemaimri et al. (2021)** dan **Turcotte et al. (2022)**.

---

## 4. PENYELARASAN SIMETRI 1-TO-1 RUMUSAN MASALAH (1.2) & TUJUAN (1.3)

Untuk memastikan penelitian bersifat kuantitatif terukur (*measurable*) dan tidak terkesan kualitatif, kata tanya *"Bagaimana"* pada Poin 2 dan 3 diubah menjadi kata tanya yang menuntut ukuran magnitudo dan uji signifikansi:

### A. Rumusan Masalah (1.2):
1. **Bagaimana merancang dan menerapkan** strategi Redis *Cache-Aside* serta optimasi kueri Eloquent ORM (*selective eager loading* dan *in-memory aggregation*) pada *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data` modul DSS Koladi? *(Ranah Rekayasa)*
2. **Seberapa besar peningkatan kinerja** modul DSS Koladi pada kondisi sesudah optimasi (*optimized*) dibandingkan kondisi sebelum optimasi (*baseline*), ditinjau dari parameter *response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*? *(Ranah Kuantitatif Terukur)*
3. **Seberapa besar pengaruh variasi skala volume dataset sintetis** (*Small*, *Medium*, *Large*) terhadap efektivitas optimasi tersebut, serta **apakah peningkatan performa yang dihasilkan terbukti signifikan secara statistik** melalui *paired-sample t-test*? *(Ranah Skalabilitas & Uji Hipotesis)*

### B. Tujuan Penelitian (1.3):
1. **Menerapkan** strategi Redis *Cache-Aside* serta optimasi kueri Eloquent ORM (*selective eager loading* dan *in-memory aggregation*) pada *endpoint* `GET /statistik/suggestions` dan `GET /statistik/modal-data` modul DSS Koladi.
2. **Membandingkan kinerja modul DSS sebelum (baseline) dan sesudah optimasi (optimized)** berdasarkan parameter *response time*, *SQL query count*, *memory utilization*, dan *cache hit ratio*.
3. **Menganalisis pengaruh intervensi simultan tersebut di bawah variasi volume dataset sintetis** (*Small*, *Medium*, *Large*) serta **mengevaluasi signifikansi perbedaannya** melalui *paired-sample t-test* dan persentase perbaikan performa.

---

## 5. STATUS BERKAS UTAMA (`proposal.md`)

Seluruh penyesuaian dan pemurnian ilmiah hasil diskusi Sesi 3 telah **100% diaplikasikan dan tersinkronisasi** secara langsung ke dalam [proposal.md](file:///c:/xampp/htdocs/TugasAkhir/proposal.md):

* **Bab I (Pendahuluan):**
  * **Subbab 1.1 (Latar Belakang):** Narasi *lazy loading* relasi *checklists* yang menimbulkan *N+1 query problem* telah diperjelas (didukung sitasi Turcotte et al., 2022).
  * **Subbab 1.2 & 1.3 (Rumusan Masalah & Tujuan Penelitian):** Telah diubah menjadi 3 butir terstruktur dengan korespondensi 1-ke-1 yang simetris, menggunakan kata tanya kuantitatif terukur (*"Seberapa besar..."* alih-alih *"Bagaimana"* pada ranah pengukuran performa dan pengujian hipotesis).
* **Bab II (Tinjauan Pustaka & Landasan Teori):**
  * **Subbab 2.2.1 (DSS Berbasis Web):** Sitasi telah dipisahkan secara akurat: Cai & Chen (2021) dan Koornneef et al. (2020) untuk komputasi analitis & ketepatan waktu DSS; Alnuhait et al. (2024) dan Putra et al. (2025) untuk metrik standar *response time* aplikasi web; Taipalus (2024) untuk dampak latensi disk I/O.
  * **Subbab 2.2.2 (Arsitektur Caching & Redis):** Pernyataan sumber latensi utama akses disk RDBMS telah diperkuat dengan rujukan Privalov & Stupina (2024).
  * **Subbab 2.2.3 (Teori ORM):** Kritik terhadap abstraksi ORM yang menyembunyikan biaya eksekusi kueri SQL di balik layar telah dilengkapi sitasi Alshemaimri et al. (2021) dan Turcotte et al. (2022).
  * **Subbab 2.3 (Metode Pengembangan Produk):**
    * Kalimat pembuka menegaskan adopsi *Refactoring Lifecycle* Mens dan Tourwé (2004) dengan berlandaskan pada prinsip dasar refaktorisasi Martin Fowler (2018).
    * Subbab 2.3.6 (*Maintain Consistency*) telah disempurnakan menjadi 3 tindakan terpadu yang realistis dan fokus: (1) stabilitas orkestrasi Docker Compose (`koladi_net`), (2) verifikasi ketiadaan efek samping pada siklus hidup tugas (*Kanban*) akibat integrasi `TaskObserver` dan `TaskAssignmentObserver`, serta (3) pembaruan artefak nonkode (`.env.example`, diagram arsitektur, dokumentasi teknis API DSS).
  * **Subbab 2.4 (Metode Analisis yang Digunakan):**
    * Judul `2.4. Metode Analisis yang Digunakan` telah dikembalikan secara benar.
    * Kalimat pembuka merujuk murni pada kerangka *Experimental Software Engineering* (Wohlin et al., 2000, 2012).
    * Subbab 2.4.3: Empat instrumen pengujian telah dinomori secara runtut (1. Middleware Instrumentasi Laravel, 2. Node.js Benchmark Runner, 3. CSV Validator, 4. Laravel Telescope).
    * Subbab 2.4.4: Empat tahapan analisis data dinomori dengan rapi (1. Statistik Deskriptif, 2. Persentase Perbaikan, 3. Paired-Sample t-Test, 4. Analisis Khusus Kueri SQL dan Cache) dengan blok formula matematika LaTeX `$$ \text{Improvement (\%)} = \left( \frac{\text{Mean Baseline} - \text{Mean Optimized}}{\text{Mean Baseline}} \right) \times 100 $$` yang bersih dan rapi.
* **Kondisi Naskah Saat Ini:** Terdiri atas 249 baris terstruktur rapi, bebas dari glitch duplikasi/pemotongan karakter, dan siap digunakan sebagai naskah Bab I dan Bab II yang kokoh secara metodologis.

---

## 6. AGENDA & LANGKAH SELANJUTNYA
1. Melanjutkan penyusunan Bab III (Metodologi Penelitian, Perancangan Eksperimen Terperinci, dan Prosedur Pelaksanaan Pengujian).
2. Mempersiapkan implementasi instrumen teknis pengujian (*script* Node.js Benchmark Runner, *middleware* pencatat metrik, *seeder* dataset sintetis, dan *validator* CSV).
3. Melakukan pemeriksaan akhir format penulisan proposal sesuai buku pedoman Tugas Akhir Politeknik Negeri Batam sebelum pengajuan sidang proposal.
