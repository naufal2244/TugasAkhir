# RINGKASAN KONTEKS & LOG SESI PENELITIAN TUGAS AKHIR (SESI 7)
**File:** `TugasAkhir/log/Log7.md`  
**Waktu Pembaruan:** 2026-10-27  
**Peneliti:** Naufal Fadhilah Ardhani (NIM: 4342301061)  
**Program Studi:** D4 Teknologi Rekayasa Perangkat Lunak (TRPL), Politeknik Negeri Batam  
**Tujuan Dokumen:** Menjadi rangkuman komprehensif seluruh pembahasan pada Sesi 7 mengenai perumusan dan penyederhanaan naskah **Subbab 3.5. Rencana Analisis** pada proposal Tugas Akhir, klarifikasi metodologis peran Laravel Telescope vs. Benchmark Baseline, pemahaman operasional *Cache Hit Ratio*, serta teks draf final yang siap disematkan ke `proposal.md`.

---

## 1. KEPUTUSAN STRUKTURAL & METODOLOGIS SUBBAB 3.5 (RENCANA ANALISIS)

### A. Mengikuti Pola Penulisan Dzakwan (`proposalDzakwan.md`)
* Bagian 3.5 disusun secara **singkat, padat, lugas, dan berfokus pada rujukan silang (*cross-referencing*)** ke konsep dan formula yang sudah dibahas pada Bab II (Subbab 2.2 Landasan Teori dan Subbab 2.4 Metode Analisis).
* **Keputusan Eliminasi Tabel:** Tabel matriks desain dan tabel parameter metrik diputuskan **tidak dimasukkan ke dalam Subbab 3.5**. Subbab 3.5 disajikan murni dalam bentuk paragraf naratif mengalir (terdiri atas 6 sub-subbab) sebagaimana preseden sukses pada `proposalDzakwan.md` (baris 402–453).

### B. Klarifikasi Krusial: Peran Laravel Telescope vs. Pengujian Baseline
Sempat muncul kerancuan redaksi kalimat pada Subbab 3.5.1 mengenai *"memverifikasi keberadaan inefisiensi"*. Hal ini diluruskan dengan batas peran yang tegas:
1. **Laravel Telescope (Subbab 3.2.1 - Diagnosis/Profiling Awal):**
   * Berfungsi sebagai alat inspeksi kualitatif untuk **mendeteksi dan menemukan titik hambatan** (membongkar kueri N+1 pada `checklists`, perulangan kueri pada `getWorkloadDistribution()`, dan *partial cache hit* pada `modal-data`).
   * Dijalankan pada awal penelitian dan dinonaktifkan saat pengujian benchmark agar tidak membebani sistem.
2. **Pengujian Baseline (Subbab 3.2.5 & 3.5.1 - Analisis Data):**
   * Berfungsi sebagai pengukuran kuantitatif untuk **menghasilkan angka tolok ukur awal** (*baseline reference*: Response Time, SQL Query Count, Memory, Cache Hit Ratio).
   * Angka ini murni berfungsi sebagai pembanding dasar terhadap kondisi *optimized*, bukan untuk mencari-cari bug kembali.

### C. Penyederhanaan Bahasa dan Pemahaman Teknis
Bahasa narasi disederhanakan dari istilah akademik yang rumit menjadi kalimat yang wajar, logis, dan mudah dipahami oleh mahasiswa maupun penguji:
* **Efisiensi Kueri & Memori (3.5.3):** Menghilangkan istilah rumit seperti *"deterministik"*, *"kompleksitas $O(N)$ ke $O(1)$"*, dan *"object graph materialization expense"*. Fokus langsung pada pembuktian bahwa jumlah kueri berkurang drastis (eliminasi N+1 dan perulangan kueri) serta RAM server lebih hemat karena hanya kolom yang dibutuhkan yang diambil.
* **Cache Hit Ratio & Joint Cache Analysis (3.5.5):** 
  * Istilah internal Redis *keyspace hits* (data ketemu di RAM) dan *keyspace misses* (data belum ada di RAM) diterjemahkan ke bahasa operasional yang mudah: perbandingan antara permintaan yang berhasil dilayani dari memori Redis terhadap total pemeriksaan cache.
  * Analisis keterpaduan (*Joint Cache Analysis*) bertujuan memastikan bahwa saat terjadi *Cache Hit*, basis data PostgreSQL benar-benar menganggur (0 kueri komputasi DSS) dan data analitis `modal-data` tersimpan utuh di memori tanpa menyisakan kueri live (*eliminasi partial cache hit*).
* **Uji Hipotesis & Batasan (3.5.6):** Menggunakan *Paired-Sample t-Test* ($\alpha = 0{,}05$) untuk memastikan perbaikan kinerja terbukti secara statistik, dengan batasan bahwa hasil penelitian berlaku spesifik pada modul DSS Koladi dan lingkungan pengujian yang digunakan.

---

## 2. NASKAH FINAL SUBBAB 3.5 (SIAP DISALIN KE `PROPOSAL.MD`)

Berikut adalah teks final Subbab 3.5 yang telah disepakati dan siap ditempelkan langsung ke berkas `proposal.md`:

```markdown
3.5. Rencana Analisis

Rencana analisis mengolah hasil pengujian baseline dan optimized yang diperoleh pada tahap Penilaian Efek Kualitas (Subbab 3.2.5), menggunakan parameter, instrumen pengukuran, formula, dan kriteria evaluasi yang telah dijelaskan pada Subbab 2.2 dan Subbab 2.4.

3.5.1. Analisis Baseline

Dua endpoint DSS (/suggestions dan /modal-data) dijalankan pada tiga skenario volume data sintetis (Small, Medium, Large) sebelum optimasi diterapkan, mencakup kondisi Cache Miss dan Cache Hit. Sebagaimana dirumuskan pada Subbab 2.4.3, setiap pengujian diawali dengan 3 kali warm-up (tidak dimasukkan ke analisis) dan 10 kali pengukuran valid, menghasilkan total 120 data observasi baseline. Hasil yang dicatat meliputi Response Time, SQL Query Count, Memory Utilization, dan Cache Hit Ratio yang dihitung nilai rata-rata (Mean) dan standar deviasinya sesuai Subbab 2.4.4, sebagai kondisi awal pembanding terhadap kondisi optimized.

3.5.2. Analisis Optimized

Setelah Redis Cache-Aside dan optimasi kueri Eloquent ORM diterapkan pada modul DSS Koladi, kedua endpoint diuji kembali menggunakan dataset, skenario volume data, kondisi cache, dan parameter yang sama persis dengan kondisi baseline untuk menjaga keterbandingan hasil. Sebagaimana pada kondisi baseline, setiap pengujian dijalankan dengan 3 kali warm-up dan 10 kali pengukuran valid mengikuti Subbab 2.4.3, menghasilkan 120 data observasi optimized. Parameter yang dicatat identik dengan kondisi baseline untuk melihat stabilitas performa sistem setelah optimasi diterapkan.

3.5.3. Analisis Efisiensi Kueri SQL dan Penggunaan Memori

Efisiensi penarikan data Eloquent ORM dianalisis dengan membandingkan jumlah kueri SQL (SQL Query Count) dan penggunaan memori (Memory Utilization) antara kondisi baseline dan optimized, terutama pada kondisi Cache Miss saat modul DSS memproses data secara penuh. 

Pengurangan jumlah kueri SQL dianalisis untuk melihat keberhasilan eliminasi masalah N+1 pada relasi checklists dan kueri berulang di dalam perulangan anggota tim (Subbab 2.2.3). Sementara itu, penggunaan memori dianalisis menggunakan persentase perbaikan (Improvement %) sesuai Subbab 2.4.4, untuk melihat seberapa besar penghematan RAM server setelah pengambilan kolom dibatasi hanya pada data yang diperlukan.

3.5.4. Analisis Response Time dan Skalabilitas Kinerja

Response Time dianalisis dengan membandingkan waktu respons antara kondisi baseline dan optimized menggunakan formula Improvement (%) sesuai Subbab 2.4.4. Analisis dipisahkan antara kondisi Cache Miss (untuk melihat dampak optimasi kueri ORM) dan kondisi Cache Hit (untuk melihat kecepatan pembacaan dari memori Redis dibandingkan basis data PostgreSQL sesuai Subbab 2.2.2). Selain itu, perubahan Response Time pada dataset Small, Medium, dan Large dianalisis untuk melihat apakah sistem tetap stabil dan tidak mengalami perlambatan drastis saat jumlah data bertambah (Subbab 2.2.1).

3.5.5. Analisis Keterpaduan Cache Hit dan Akses Basis Data (Joint Cache Analysis)

Hasil Cache Hit Ratio dan SQL Query Count dianalisis secara bersamaan menggunakan pendekatan Joint Cache Analysis (Subbab 2.4.4). Pada konfigurasi optimized, Cache Hit Ratio mengukur keberhasilan pengambilan data langsung dari memori Redis. Analisis keterpaduan ini dilakukan untuk memastikan apakah kondisi Cache Hit benar-benar membebaskan basis data dari pemanggilan kueri SQL (mencapai 0 kueri komputasi). Selain itu, analisis ini membuktikan terselesaikannya masalah partial cache hit pada endpoint /modal-data, di mana seluruh data analitis (termasuk tugas mendesak dan sebaran beban kerja) kini tersaji utuh dari memori Redis tanpa menyisakan kueri tambahan ke PostgreSQL (Subbab 2.2.4).

3.5.6. Interpretasi Hasil dan Uji Hipotesis

Hasil baseline dan optimized dibandingkan pada setiap skenario pengujian dengan mempertimbangkan Response Time, SQL Query Count, Memory Utilization, dan Cache Hit Ratio untuk menentukan efektivitas optimasi pada modul DSS Koladi. Uji Paired-Sample t-Test pada taraf signifikansi α = 0,05 digunakan untuk memastikan apakah perbedaan performa yang diperoleh terbukti signifikan secara statistik (Subbab 2.4.4). 

Kesimpulan dari penelitian ini dibatasi pada lingkungan pengujian dan jalur eksekusi modul DSS Koladi (endpoint /suggestions dan /modal-data), sehingga tidak digeneralisasikan secara langsung ke seluruh fitur aplikasi Koladi maupun sistem SaaS lainnya (Subbab 2.2.4).
```

---

## 3. STATUS DOKUMEN & RENCANA TINDAK LANJUT

### A. Status Saat Ini
1. **Bab I:** Selesai 100% dan terverifikasi di `proposal.md`.
2. **Bab II:** Selesai 100% (Subbab 2.1 s.d. 2.4) dan terverifikasi di `proposal.md`.
3. **Bab III:**
   * Subbab 3.1 (Alat dan Bahan): Selesai.
   * Subbab 3.2 (Jalannya Penelitian): Selesai (6 tahapan Mens & Tourwé).
   * Subbab 3.3 (Tahapan Pengembangan Aplikasi): Selesai (Penegasan penolakan SDLC konvensional & adopsi Refactoring Lifecycle).
   * Subbab 3.4 (Rencana Perancangan Sistem): Selesai (3.4.1 s.d. 3.4.5).
   * **Subbab 3.5 (Rencana Analisis): Telah selesai dirumuskan dan disederhanakan secara final di dalam `Log6.md` ini**.

### B. Agenda Sesi Berikutnya
1. Memasukkan teks draf final Subbab 3.5 dari `Log6.md` ke bagian akhir `proposal.md`.
2. Melakukan audit nomor tabel dan gambar di Bab III agar berurutan rapi.
3. Melakukan pemeriksaan menyeluruh keterkaitan Bab I, II, dan III sebelum persiapan cetak/sidang proposal.
