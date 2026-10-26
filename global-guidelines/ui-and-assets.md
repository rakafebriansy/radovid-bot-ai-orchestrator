# User Interface (UI) & Asset Integration Standards

> **[PRINSIP UTAMA]** Standar teknis mutlak untuk perancangan antarmuka visual, integrasi aset, evaluasi usabilitas, dan pencegahan estetika generik (*anti-slop*) pada seluruh subsistem frontend aplikasi.

---

## 1. Fondasi Prototipe & Standar Aset Visual

### 1.1 Prototyping Wajib Pre-Coding
1. **[WAJIB]** Sebelum menulis kode antarmuka pada framework utama (Flutter, SwiftUI, Next.js, dll.), rancang sketsa visual di `nodes/[nama-node]/prototypes/`.
2. **[WAJIB]** Baca dan patuhi arsitektur *Rapid HTML-based Previewing* dan *Single-File Policy* pada `nodes/[nama-node]/prototypes/README.md` sebelum menyusun kode UI.

### 1.2 Konversi Aset Visual ke Kode Implementasi
Saat mengonversi aset visual mentah (SVG mentah, templat HTML, XML layout) menjadi kode frontend fungsional:
1. **[WAJIB]** Patuhi konvensi penamaan target framework (contoh: konversi atribut *kebab-case* menjadi *camelCase* pada JSX/TSX).
2. **[WAJIB]** Hapus nilai default usang dari berkas sumber mentah, format ulang parameter penataan gaya (*styling*), dan pastikan seluruh penutup tag valid sesuai standar parser/kompiler.

### 1.3 Tata Kelola Domain Media Eksternal
1. **[WAJIB]** Saat merender media statis atau gambar secara jarak jauh (*remote fetching*), daftarkan alamat domain eksternal ke konfigurasi daftar aman (*whitelist* / *safe origins*) proyek.
2. Pencegahan ini wajib untuk menghindari pemblokiran aset akibat kebijakan CORS atau galat runtime pemuatan gambar.

### 1.4 Skalabilitas Satuan & Dimensi (Aksesibilitas Skala Teks)
1. **[TERLARANG]** Menggunakan unit piksel (`px`) kaku dan absolut pada spesifikasi web/CSS untuk margin, padding, kerangka tata letak, dan ukuran tipografi.
2. **[WAJIB]** Gunakan satuan proporsional dinamis (`rem`, `em`, atau persentase `%`) pada pengembangan Web agar tampilan beradaptasi mulus terhadap preferensi skala font pengguna di tingkat sistem operasi atau peramban.
3. **Pengecualian Native Framework:** Larangan `px` tidak berlaku pada framework native (Flutter, SwiftUI, React Native) yang secara baku telah menggunakan satuan kerapatan independen (*logical pixels*, *points*, atau *density-independent pixels* `dp`).

---

## 2. Standar Anti-Slop UI/UX & Desain Frontend

> **[PRINSIP ANTI-SLOP]** Dilarang mendesain UI agar sekadar terlihat mewah di tangkapan layar. Optimasi antarmuka agar membuat tugas pengguna jelas, cepat, dan mudah diselesaikan. Jika suatu elemen tidak memiliki fungsi nyata, hapus segera.

### 2.1 Larangan Keras Estetika AI SaaS Generik
AI Agent DILARANG KERAS menggunakan resep visual default berikut sebagai template otomatis:
1. **[TERLARANG] Gradien Ungu/Biru Klise:** Latar belakang mesh gradien ungu/biru gelap generik tanpa dasar branding.
2. **[TERLARANG] Dark Navy + Neon Accent:** Palet seragam navy gelap dengan aksen neon mencolok tanpa alasan produk.
3. **[TERLARANG] Glassmorphism Berlebih:** Panel kaca buram (*frosted glass*), blur tebal, dan kartu transparan bertumpuk yang merusak keterbacaan teks.
4. **[TERLARANG] Glowing Borders & Blobs:** Garis tepi menyala (*glow*), partikel mengambang, atau blob dekoratif tanpa arti fungsi.
5. **[TERLARANG] Teks Gradien Dekoratif:** Teks gradien di sembarang tempat hanya demi kesan futuristik semu.
6. **[TERLARANG] Pola Kartu Seragam Berulang:** Mengubah seluruh konten menjadi 6 kartu identik berjejer dengan formula: [ikon kotak warna] + [heading tebal] + [2 baris teks filler].
7. **[TERLARANG] Pills & Radius Ekstrem:** Kapsul/pill raksasa dan sudut melengkung ekstrem bertumpuk pada setiap kontainer.
8. **[TERLARANG] Dashboard & Metrik Hiasan:** Kartu statistik palsu, grafik dekoratif tanpa data, atau metrik pura-pura di layar kerja operasional.
9. **[TERLARANG] Hero Section di Layar Operasional:** Banner hero besar di aplikasi admin, dashboard, POS, atau alat internal tempat pengguna seharusnya langsung bekerja.
10. **[TERLARANG] Teks Filler Generik AI:** Copy hampa makna seperti *"Unlock your potential"*, *"Supercharge your workflow"*, *"AI-powered next-generation experience"*, *"Transform your business"*.
11. **[TERLARANG] Animasi Asal Ramai:** Efek *bounce-on-hover*, animasi memantul tak berkesudahan, atau transisi lambat yang menghambat kecepatan kerja pengguna.

### 2.2 Definisi Niat Desain Sebelum Menulis Kode (Design Intent First)
Sebelum menulis kode antarmuka, AI Agent WAJIB mendefinisikan batasan konkret proyek dalam format terstruktur:
```text
Produk / Domain:
Target Pengguna:
Tugas Utama (Primary Task):
Tugas Pendukung (Secondary Tasks):
Densitas Informasi: [Rendah / Sedang / Tinggi]
Arah Visual & Tone: [Contoh: Utilitarian padat, basis netral hangat, 1 aksen fungsional]
Aksi Utama (Primary CTA):
Batasan Aksesibilitas: [WCAG AA, Touch Targets, Screen Reader]
Perilaku Responsif: [320px fluid, breakpoints, container queries]
```

### 2.3 Hierarki Visual Sebelum Dekorasi (Grayscale First)
Antarmuka yang gagal dalam tampilan hitam-putih (*grayscale*) tidak akan bisa diselamatkan dengan gradien atau animasi. Selesaikan perancangan dengan urutan prioritas mutlak:
1. Hierarki Informasi
2. Tata Letak (*Layout & Flow*)
3. Jarak & Ruang (*Spacing & Whitespace*)
4. Tipografi & Skala Teks
5. Kontras & Keterbacaan
6. Kelengkapan Status Komponen
7. Warna Fungsional
8. Batas & Elevasi
9. Gerakan Berarti (*Purposeful Motion*)
10. Sentuhan Akhir (*Polishing*)

### 2.4 Tata Letak & Larangan "Card Soup"
1. **Kartu adalah Pengelompok Semantik:** Gunakan kartu HANYA jika konten memiliki relasi mandiri, dapat ditindaklanjuti terpisah, atau membutuhkan pemisahan spasial nyata.
2. **Utamakan Struktur Langsung:** Gunakan whitespace, heading, pembatas garis (*dividers*), tabel, dan daftar terstruktur alih-alih membungkus setiap kalimat ke dalam kotak kartu melayang.
3. **[TERLARANG]** Struktur kartu bertumpuk tanpa arti (`page -> card -> sub-card -> mini-card`) dengan bayangan tebal.

### 2.5 Larangan Fabrikasi Konten & Metrik Palsu
1. **[TERLARANG]** Mengarang angka statistik, volume transaksi, testimoni palsu, ulasan tiruan, grafik fiktif, notifikasi dummy, atau logo mitra palsu.
2. **[WAJIB]** Jika data aktual belum tersedia, gunakan tampilan *Empty State* yang jujur dan instruktif atau *placeholder* eksplisit yang ditandai dengan jelas.

### 2.6 Sistem Token Desain, Warna, & Border Radius
1. **Token Terikat:** Gunakan skala terukur untuk spacing (kelipatan 4px atau 8px), skala tipe, warna, radius, dan elevasi. Dilarang menciptakan nilai acak baru di setiap komponen.
2. **Warna untuk Makna:** Gunakan warna untuk menandai aksi utama, status fungsional (sukses, peringatan, galat), hierarki, dan identitas brand.
3. **Kontras Wajib WCAG AA:**
   - Teks reguler: Rasio kontras minimal **4.5:1** terhadap latar belakang.
   - Teks besar ($\ge$ 18pt atau $\ge$ 14pt tebal): Rasio kontras minimal **3:1**.
   - Dilarang meredupkan teks menjadi abu-abu pudar berekontras rendah demi tren visual.
4. **Konsistensi Geometri Radius:** Elemen kontrol (input, tombol) dan kontainer wajib memiliki bahasa bentuk yang seragam; hindari mencampur bentuk kapsul runcing dan melengkung acak di satu layar.

### 2.7 Ikon, Ilustrasi, & Tipografi Fungsional
1. **Ikon Informatif:** Gunakan ikon untuk navigasi dan penjelas aksi; singkirkan ikon dekoratif tanpa nilai informasi.
2. **Hindari Kotak Ikon Pastel Berulang:** Dilarang membungkus setiap ikon ke dalam kotak membulat warna-warni secara seragam tanpa tujuan semantik.
3. **Satu Keluarga Ikon Konsisten:** Gunakan satu pustaka ikon seragam (Lucide, Material Icons, dll.). Dilarang mencampur pustaka ikon yang berbeda karakter.
4. **[TERLARANG]** Menggunakan emoji sebagai pengganti ikon profesional pada antarmuka sistem bisnis.
5. **Tipografi Fungsional:** Gunakan skala tipe ringkas dan koheren. Terapkan `font-variant-numeric: tabular-nums` untuk perataan angka pada tabel data.

### 2.8 Microcopy Aksi & Penanganan Galat
1. **Label Aksi Spesifik:** Gunakan label tombol yang menjelaskan aksi: `"Simpan Perubahan"`, `"Hapus Akun"`, `"Tambah Produk"`, `"Unduh Laporan"`. Hindari label ambigu seperti `"Lanjutkan"`, `"Klik Di Sini"`, `"Mulai"`.
2. **Bebas Slogan AI:** Gunakan bahasa domain bisnis yang relevan, bebas dari jargon pemasaran hampa.
3. **Pesan Galat Solutif:** Pesan galat wajib menjelaskan: (1) apa yang terjadi, dan (2) tindakan konkret yang harus diambil pengguna untuk memperbaikinya.

### 2.9 Kelengkapan 11 Status Komponen Interaktif
Setiap layar dan komponen interaktif WAJIB mengimplementasikan seluruh variasi status berikut:
1. `Initial` (Kondisi awal)
2. `Loading` (Pemuatan data)
3. `Skeleton` (Kerangka pemuatan proporsional layout akhir guna mencegah *Cumulative Layout Shift*)
4. `Empty State` (Penjelasan apa yang kosong, mengapa kosong, dan aksi selanjutnya)
5. `Partial Data` (Penyajian data sebagian)
6. `Success` (Indikasi aksi berhasil)
7. `Error & Retry` (Pesan galat informatif beserta tombol coba lagi)
8. `Submitting / Saving` (Proses pemrosesan aksi berlangsung)
9. `Permission Denied` (Akses ditolak atau otorisasi tidak cukup)
10. `Offline / Network Failure` (Penanganan putus koneksi jaringan)
11. Status interaktif elemen: `Disabled`, `Focused`, `Hovered`, `Pressed`, `Selected`

### 2.10 Formulir & Standar Interaktivitas
1. **Label Persisten:** Setiap input formulir wajib memiliki elemen `<label>` yang jelas dan persisten (bukan sekadar placeholder yang hilang saat diketik).
2. **Izin Paste:** Dilarang menonaktifkan fitur tempel teks (*paste*) pada bidang input secara sembarangan.
3. **Ukuran Target Sentuh Minimum:**
   - **Web (WCAG 2.2):** Minimal **24 x 24 CSS px**.
   - **Mobile / Touch Interface (Apple HIG & Material):** Minimal **44 x 44 pt / 48 x 48 dp**.
4. **Indikator Fokus Keyboard:** Sediakan indikator fokus yang kontras (*focus visible ring*) pada setiap kontrol interaktif. Dilarang menetapkan `outline: none` tanpa menyediakan pengganti fokus visual.

### 2.11 Desain Responsif Nyata & Reflow
1. **Ambang Batas Reflow 320px:** Seluruh antarmuka wajib teruji rapi pada lebar minimal **320px CSS width** tanpa memicu scroll horizontal yang tidak diinginkan.
2. **Tata Letak Fluida:** Gunakan CSS Grid, Flexbox, dan *Container Queries* alih-alih menumpuk media query kaku.
3. **Adaptasi Tabel Mobile:** Sediakan strategi penyajian tabel pada layar sempit (tampilan kartu ringkas bertingkat atau scroll horizontal mandiri dengan indikator visual yang jelas).

### 2.12 Animasi, Gerakan Terarah, & Reduced Motion
1. **Animasi Fungsional:** Gunakan animasi hanya untuk mengomunikasikan perubahan status, hierarki spasial, atau progres kerja.
2. **[TERLARANG]** Animasi memantul (*bounce*), pergerakan looping tanpa henti, atau efek parallax berlebih.
3. **[WAJIB] Dukungan `prefers-reduced-motion`:** Seluruh animasi dan transisi wajib dinonaktifkan atau disederhanakan seketika jika pengguna mengaktifkan pengaturan pengurangan gerakan di sistem operasi.

### 2.13 Kinerja Frontend & Core Web Vitals
1. **Indikator Web Vitals:** Pastikan antarmuka memenuhi standar performa (LCP $\le$ 2.5s, INP responsif, CLS rendah).
2. **Optimasi Media:** Gunakan format modern (WebP/AVIF) dengan atribut `width` dan `height` eksplisit untuk mencegah pergeseran tata letak.
3. **Efisiensi Rendering:** Hindari efek visual berat (seperti blur bertumpuk atau multi-layer shadow) yang memicu *frame drop* saat scrolling pada perangkat berspesifikasi rendah.

---

## 3. Kepatuhan Aksesibilitas & Evaluasi Heuristik

### 3.1 Kepatuhan Aksesibilitas Multi-Platform
1. **Adaptasi Berdasarkan Platform:** Terapkan standar aksesibilitas sesuai stack; manfaatkan API aksesibilitas bawaan platform native secara maksimal (Accessibility Labels, Semantic Nodes).
2. **Konfirmasi Pengguna:** Paparkan daftar opsi kapabilitas aksesibilitas yang relevan kepada pengguna dan mintalah konfirmasi sebelum memulai implementasi.

### 3.2 Evaluasi 10 Prinsip Heuristik Jakob Nielsen
Setiap antarmuka wajib dievaluasi berdasarkan 10 prinsip berikut:
1. **Visibilitas Status Sistem:** Berikan umpan balik instan (loading spinner, skeleton, progress bar, atau label status pemrosesan).
2. **Kecocokan Sistem dengan Dunia Nyata:** Gunakan simbol universal dan bahasa alami pengguna alih-alih istilah teknis mentah (misal: "Koneksi gagal, silakan coba lagi" alih-alih "Error 500").
3. **Kendali dan Kebebasan Pengguna:** Sediakan tombol "Batal", "Kembali", atau opsi "Undo" pada setiap alur transaksi dan aksi.
4. **Konsistensi dan Standar:** Terapkan tipografi, warna, dan pola komponen yang seragam di seluruh halaman sesuai design system proyek.
5. **Pencegahan Kesalahan:** Terapkan validasi formulir real-time, tombol disabled saat input invalid, dan modal konfirmasi untuk aksi destruktif permanen.
6. **Pengenalan Daripada Mengingat:** Sediakan dropdown autocomplete, label deskriptif, dan riwayat pencarian agar pengguna tidak perlu mengingat data di luar kepala.
7. **Fleksibilitas dan Efisiensi Penggunaan:** Sediakan jalan pintas keyboard (*hotkeys*), aksi massal (*bulk actions*), atau gestur sapuan pada mobile untuk pengguna tingkat lanjut.
8. **Desain Estetis dan Minimalis:** Hindari kepadatan visual berlebih; jaga whitespace yang cukup dan hapus elemen dekoratif non-fungsional.
9. **Membantu Mengenali, Mendiagnosis, & Memulihkan Kesalahan:** Berikan sorotan visual merah pada kolom bermasalah disertai petunjuk perbaikan yang jelas (misal: "Kata sandi minimal 8 karakter dan mengandung huruf besar").
10. **Bantuan dan Dokumentasi:** Sediakan tooltip kontekstual pada bidang input yang rumit atau alur panduan (*onboarding*) terarah.

---

## 4. Gerbang Tinjauan Visual Anti-Slop (Visual Review Gate)

### 4.1 Daftar Periksa Tinjauan Visual Aktual
Sebelum menandai tugas antarmuka selesai, AI Agent **WAJIB MENINJAU TAMPILAN VISUAL AKTUAL** (melalui prototipe atau browser preview) dan memvalidasi checklist berikut:
- [ ] **Kekhususan Produk:** Arah visual merefleksikan domain produk spesifik, bukan template SaaS generik.
- [ ] **Bebas Konten Tiruan:** Nol angka transaksi, statistik, testimoni, atau grafik fiktif buatan AI.
- [ ] **Hierarki & Titik Fokus:** Setiap layar memiliki titik fokus utama yang tegas dengan bobot visual proporsional.
- [ ] **Bebas Elemen Slop:** Tidak ada mesh gradien klise, glassmorphism buram berlebih, border neon, kartu berjejer identik, atau banner hero di dashboard operasional.
- [ ] **Kelengkapan Status Komponen:** Seluruh status fungsional (*loading*, *skeleton*, *empty*, *error*, *retry*, *disabled*, *focused*, *success*) terimplementasi.
- [ ] **Aksesibilitas & Kontras:** Kontras teks lolos WCAG AA, fokus keyboard jelas, kontrol sentuh $\ge$ 24x24px (web) / 44x44pt (mobile), dan `prefers-reduced-motion` aktif.
- [ ] **Responsif & Reflow:** Teruji rapi pada lebar 320px tanpa scroll horizontal yang tidak diinginkan.
- [ ] **Microcopy Konkret:** Seluruh tombol dan pesan memandu tindakan solutif pengguna.

### 4.2 Gerbang Keputusan Akhir (The Final Gate)
Sebelum mengajukan izin commit dan penyerahan tugas antarmuka, jawab dua pertanyaan penentu berikut:
1. *"Jika saya menghapus 20% dekorasi dan efek visual ini, apakah produk menjadi lebih jelas dan mudah digunakan?"*
   - Jika **YA** $\rightarrow$ **HAPUS DEKORASI TERSEBUT SEGERA.**
2. *"Jika saya menghapus 20% elemen UI ini, apakah tugas pengguna menjadi lebih sulit diselesaikan?"*
   - Jika **YA** $\rightarrow$ **PERTAHANKAN ELEMEN TERSEBUT.**
