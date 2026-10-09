# User Interface (UI) & Asset Integration Standards

*(Pedoman teknis ini berlaku krusial untuk fase pengerjaan atas entitas proyek apa pun yang mengelola subsistem antarmuka visual)*

## Kewajiban Penggunaan Prototipe (Pre-Coding)
Sebelum Anda (AI Agent) secara langsung menulis *source code* antarmuka (*UI*) menggunakan *framework* utama aplikasi (seperti Flutter, SwiftUI, Next.js, dsb.), Anda **DIWAJIBKAN** merancang sketsa draf visualnya di dalam direktori spesifik *node* yaitu `nodes/[nama-node]/prototypes/`. 
Anda **WAJIB MUTLAK** membaca dan mematuhi seluruh arsitektur *Rapid HTML-based Previewing* dan *Single-File Policy* yang tertulis di dalam file referensi **`nodes/[nama-node]/prototypes/README.md`** sebelum menyusun kode UI apa pun.

---

## 🚫 STANDAR ANTI-AI-SLOP UI/UX & FRONTEND

> 🛡️ **PRINSIP DASAR ANTI-SLOP:**
> Jangan pernah mengoptimasi antarmuka agar sekadar *"terlihat keren di screenshot"*. Optimasi UI agar **"membuat tugas pengguna jelas, cepat, dan mudah diselesaikan"**.
> Setiap elemen visual harus memiliki fungsi nyata: mengomunikasikan informasi, membangun hierarki, memfasilitasi aksi, memberi feedback, navigasi, identitas brand, pemahaman, atau aksesibilitas. Jika suatu elemen tidak memiliki tujuan fungsional, **HAPUS SEGERA**.

### 1. Larangan Keras Estetika AI SaaS Generik (Anti-Slop Hard Rules)
AI Agent **DILARANG KERAS** menggunakan resep visual default generik berikut sebagai *template* otomatis:
- ❌ **Gradien Ungu/Biru Klise:** Latar belakang mesh gradien ungu/biru gelap yang selalu sama di setiap proyek.
- ❌ **Dark Navy + Neon Accent:** Palet default navy gelap dengan aksen neon biru/ungu tanpa alasan branding.
- ❌ **Glassmorphism Berlebih:** Panel kaca buram (*frosted-glass*), blur tebal, dan kartu transparan bertumpuk yang mengaburkan teks.
- ❌ **Glowing Borders & Neon Blobs:** Efek garis tepi menyala (*glow*), partikel mengambang, atau blob dekoratif tanpa arti produk.
- ❌ **Teks Gradien Dekoratif:** Memakai *gradient text* di sembarang tempat hanya demi kesan "futuristik".
- ❌ **Pola Kartu Seragam Berulang:** Mengubah semua konten menjadi deretan 6 kartu identik berjejer dengan pola membosankan: [ikon dalam kotak berwarna] + [heading tebal] + [2 baris teks filler].
- ❌ **Pills & Sudut Membulat Berlebih:** Menggunakan kapsul/pill raksasa dan sudut melengkung ekstrem (*nested extreme border-radius*) pada setiap kontainer.
- ❌ **Fake Dashboard & Metrik Hiasan:** Menyajikan kartu statistik palsu, grafik dekoratif tanpa data, atau metrik pura-pura di layar operasional.
- ❌ **Hero Section Raksasa di Layar Operasional:** Menyisipkan banner hero besar di dalam aplikasi admin, dashboard, POS, atau alat internal tempat pengguna seharusnya langsung bekerja.
- ❌ **Teks Filler Generik AI:** Menggunakan copy hampa makna seperti: *"Unlock your potential"*, *"Supercharge your workflow"*, *"AI-powered next-generation experience"*, *"Transform your business"*, *"Seamless and effortless"*.
- ❌ **Animasi Asal Ramai:** Menambahkan efek *bounce-on-hover*, *spring-on-everything*, atau transisi lambat yang menghambat kecepatan kerja pengguna.

### 2. Definisi Niat Desain Sebelum Menulis Kode (Design Intent First)
Sebelum menulis kode antarmuka apa pun, Anda **DILARANG** memulai dengan instruksi samar seperti *"Mari buat tampilan modern, clean, sleek, dan premium"*. Anda **WAJIB** mendefinisikan batasan konkret:

```text
Produk / Domain:
Target Pengguna:
Tugas Utama (Primary Task):
Tugas Pendukung (Secondary Tasks):
Densitas Informasi: [Rendah / Sedang / Tinggi]
Arah Visual & Tone: [Misal: Utilitarian padat, basis netral hangat, 1 warna aksen fungsional]
Aksi Utama (Primary CTA):
Batasan Aksesibilitas: [WCAG AA, Touch Targets, Screen Reader]
Perilaku Responsif: [320px fluid, breakpoints, container queries]
```

### 3. Hierarki Visual Sebelum Dekorasi (Grayscale First)
Halaman yang gagal dalam tampilan hitam-putih (*grayscale*) **TIDAK AKAN BISA** diselamatkan dengan menambah gradien, warna menyala, bayangan tebal, atau animasi.
Selesaikan antarmuka dengan urutan prioritas mutlak berikut:
1. Hierarki Informasi (*Information Hierarchy*)
2. Tata Letak (*Layout & Flow*)
3. Jarak & Ruang (*Spacing & Whitespace*)
4. Tipografi & Skala Teks (*Typography Scale*)
5. Kontras & Keterbacaan (*Contrast*)
6. Kelengkapan Status Komponen (*Component States*)
7. Warna Fungsional (*Functional Color*)
8. Batas & Kedalaman (*Borders & Elevation*)
9. Gerakan Berarti (*Purposeful Motion*)
10. Sentuhan Akhir (*Polishing*)

### 4. Tata Letak & Larangan "Semua Jadi Kartu" (No Card Soup)
- **Kartu adalah Pengelompok Semantik:** Gunakan *cards* HANYA jika konten memiliki relasi mandiri, dapat ditindaklanjuti secara terpisah, atau membutuhkan pemisahan spasial nyata.
- **Utamakan Layout Langsung:** Gunakan *whitespace*, *headings*, *dividers*, tabel, dan daftar terstruktur alih-alih membungkus setiap kalimat ke dalam kotak kartu putih melayang.
- **Hindari Nested Floating Soup:** Dilarang membuat halaman berlapis kartu di dalam kartu dengan bayangan raksasa (`page -> card -> sub-card -> mini-card`).

### 5. Larangan Fabrikasi Konten & Metrik Palsu (No Fake UI Data)
- **DILARANG KERAS** mengarang angka statistik, jumlah pengguna, volume transaksi, testimoni palsu, review tiruan, grafik fiktif, notifikasi dummy, atau logo mitra palsu untuk mengisi area kosong di UI.
- Jika data nyata belum tersedia, gunakan tampilan *Empty State* yang jujur dan instruktif atau *explicit placeholder* yang ditandai dengan jelas.

### 6. Sistem Token Desain, Warna, & Border Radius
- **Gunakan Token Terikat:** Tetapkan skala terukur untuk *spacing* (kelipatan 4px/8px), ukuran huruf, warna, *radius*, dan elevasi. Jangan menciptakan nilai acak baru di setiap komponen.
- **Warna untuk Makna:** Warna digunakan untuk menunjukkan aksi utama, status (sukses, peringatan, galat), hierarki, dan identitas brand — bukan mewarnai setiap kartu dengan warna pelangi yang berbeda-beda.
- **Kontras Aksesibilitas Mandatori (WCAG AA):**
  - Teks reguler: Rasio kontras minimal **4.5:1** terhadap latar belakang.
  - Teks besar (≥18pt atau ≥14pt bold): Rasio kontras minimal **3:1**.
  - Dilarang sengaja menggunakan teks abu-abu pudar berekontras rendah demi tren visual.
- **Konsistensi Geometri Radius:** Elemen kontrol (input, tombol) dan kontainer harus berbagi bahasa geometri yang konsisten. Hindari mencampur tombol berbentuk kapsul bulat (*pill*), kotak runcing (*sharp*), dan sudut melengkung acak di satu layar.

### 7. Ikon, Ilustrasi, & Tipografi
- **Ikon Harus Informatif:** Gunakan ikon untuk mempercepat pemindaian navigasi atau menjelaskan aksi. Hapus ikon dekoratif yang tidak memberi nilai informasi.
- **Hindari Ikon dalam Kotak Warna Warni:** Jangan otomatis membungkus setiap ikon ke dalam kotak membulat berwarna pastel/neon tanpa tujuan semantik.
- **Satu Keluarga Ikon Konsisten:** Dilarang mencampur pustaka ikon (misal: Lucide, FontAwesome, Material Icons, emoji) secara acak.
- **Dilarang Menjadikan Emoji Sebagai Desain UI:** Jangan gunakan emoji sebagai pengganti ikon profesional pada antarmuka sistem bisnis.
- **Tipografi Terstruktur:** Gunakan skala tipe kecil yang koheren. Hindari menggunakan terlalu banyak bobot (*weights*) atau ukuran font raksasa di area kerja fungsional. Gunakan *tabular numbers* (`font-variant-numeric: tabular-nums`) untuk data numerik tabel.

### 8. Microcopy & Tombol Aksi (Action-Driven UI)
- **Label Tombol Spesifik:** Gunakan label aksi yang menjelaskan tugas: `"Simpan Perubahan"`, `"Hapus Akun"`, `"Tambah Produk"`, `"Unduh Laporan"`. Hindari tombol ambigu seperti `"Lanjutkan"`, `"Klik Di Sini"`, `"Mulai"`.
- **Bebas Buzzword AI:** Dilarang menyisipkan slogan hampa marketing pada layar kerja. Gunakan bahasa teknis dan domain bisnis yang relevan dengan pengguna.
- **Pesan Galat yang Membantu Pemulihan:** Pesan galat harus menjelaskan: (1) apa yang terjadi, dan (2) apa langkah konkret yang harus dilakukan pengguna untuk memperbaikinya.

### 9. Kelengkapan Status Komponen (Never Happy-Path Only)
Setiap layar dan komponen interaktif **WAJIB** menangani seluruh variasi status berikut:
```text
- Initial (Kondisi awal)
- Loading (Pemuatan data)
- Skeleton (Kerangka pemuatan proporsional)
- Empty State (Kondisi data kosong)
- Partial Data (Data sebagian)
- Success (Aksi berhasil)
- Error & Retry (Galat dan tombol coba lagi)
- Submitting / Saving (Sedang memproses aksi)
- Permission Denied (Akses ditolak)
- Offline / Network Failure (Gagal koneksi)
- Disabled, Focused, Hovered, Pressed, Selected
```
- **Empty State Informatif:** Layar kosong harus menjawab: apa yang kosong, mengapa kosong, dan apa tombol/aksi yang bisa dilakukan pengguna selanjutnya (bukan gambar kartun raksasa dengan teks puitis).
- **Loading Skeleton Proporsional:** Skeleton harus mencerminkan bentuk tata letak akhir secara presisi untuk mencegah pergeseran tata letak (*Cumulative Layout Shift*).

### 10. Formulir & Interaksi Tombol
- Setiap input form **WAJIB** memiliki elemen label yang jelas dan persisten (bukan sekadar teks *placeholder* yang hilang saat diketik).
- Jangan mematikan fitur *paste* secara sembarangan.
- Berikan ukuran target sentuh (*touch target size*) yang memadai:
  - **Web (WCAG 2.2):** Minimal **24 × 24 CSS px**.
  - **Mobile / Touch Interfaces (Apple HIG & Material):** Minimal **44 × 44 pt / 48 × 48 dp**.
- Sediakan indikator fokus keyboard (*focus visible ring*) yang jelas pada seluruh kontrol interaktif. Dilarang menghapus `outline: none` tanpa menyediakan pengganti fokus visual!

### 11. Desain Responsif Nyata (Bukan Desktop yang Menyusut)
- Seluruh antarmuka harus lulus uji pada lebar layar minimal **320px CSS width** tanpa memicu *horizontal scrollbar* yang tidak diinginkan (*reflow compliance*).
- Gunakan tata letak fluida (*fluid layout*), CSS Grid/Flexbox, dan *Container Queries* alih-alih menumpuk puluhan *media query* kaku untuk menambal layout yang cacat.
- Tabel data pada tampilan *mobile* harus memiliki strategi adaptasi yang jelas (misal: kartu ringkas bertingkat atau scroll horizontal tabel dengan indikator jelas).

### 12. Animasi, Gerakan, & Aksesibilitas (Reduced Motion)
- Gunakan animasi hanya untuk menjelaskan perubahan status, hierarki spasial, atau progres.
- Dilarang membuat animasi memantul (*bounce*), pergerakan konstan tanpa henti, atau efek parallax berlebihan.
- **Wajib Mendukung `prefers-reduced-motion`:** Seluruh transisi dan animasi wajib dinonaktifkan atau disederhanakan seketika jika pengguna mengaktifkan preferensi *reduced motion* di sistem operasinya.

### 13. Optimasi Performa UI
- Pastikan antarmuka memenuhi indikator *Core Web Vitals* (LCP ≤ 2.5s, INP responsif, CLS rendah).
- Kompres dan optimasi aset gambar, gunakan format modern (WebP/AVIF), dan tentukan dimensi gambar eksplisit (`width` & `height`) untuk mencegah *layout shift*.
- Hindari efek visual berat (seperti multi-layer blur tebal dan bayangan berlapis) yang menyebabkan *frame drop* saat scrolling di perangkat berspesifikasi rendah.

---

## Standar Konversi Aset Visual ke Kode Implementatif
Setiap kali mengeksekusi konversi dari objek aset visual mentah (contoh: templat *raw* SVG, berkas HTML orisinal, XML tata letak struktural) menjadi *codebase* berbasis *framework* fungsional:
1.  Patuhi secara saksama kaidah penamaan konvensi internal dari target *framework* tersebut. Misalnya, atribut kode yang semula berbasis sintaks *kebab-case* **wajib dikonversi** ke dalam format sintaksis *camelCase* apabila tata kelola bahasa memandatkannya.
2.  Gugurkan segala *value default* dari sumber asli visual yang sudah tidak relevan di lingkungan eksekusi, format ulang penulisan sintaksis pelapisan desain (*styling parameters*), serta tertibkan penutup *tag* agar sepenuhnya memenuhi prasyarat kompiler atau penata dokumen terkait.

## Prosedur Autorisasi Sumber Aset Lintas Domain
1.  Di saat lapisan visual dituntut untuk merender media statis pendukung (*placeholder image/stream resource*) langsung dari sumber infrastruktur (*domain*) pihak luar secara jarak jauh (*remote fetching*):
2.  Aturan tata batas mutlak mewajibkan Anda untuk meregistrasikan alamat domain eksternal secara sadar ke dalam konfigurasi daftar aman (*whitelist* / *safe origins setting*) kerangka kerja proyek, guna mencegah terblokirnya materi media akibat *policy restriction* seperti batasan CORS atau *invalid internal runtime exceptions*.

## Metrik Skalabilitas Satuan Antarmuka (Aksesibilitas Mutlak)
1.  **Tinggalkan Absolutisme Unit Piksel (Khusus Web/CSS):** Sangat dihindari dalam spesifikasi pengembangan web menggunakan pendefinisian jarak absolut dan kaku melalui piksel (`px`) untuk menetapkan hierarki ukuran area pandang (*margins/spacing*), parameter kerangka blok, maupun topografi *font* tulisan secara *hardcoded*.
2.  **Inisiatif Satuan Dimensi Responsif:** Sistem harus dibangun dengan berorientasi pada nilai porsi rasio proporsional—seperti `rem`, `em`, atau persentase basis (`%`). Regulasi ini esensial bukan sebatas aspek *layouting responsif*, melainkan agar aplikasi secara mendasar mampu bersanding mulus terhadap adaptasi pengaturan aksesibilitas skala *font default* secara sistemik di OS (*Operating System*) atau peramban dari sisi preferensi klien akhir.
3.  **Pengecualian Native Framework:** Aturan larangan `px` ini **TIDAK BERLAKU** untuk *framework native* (seperti Flutter, SwiftUI, React Native) yang mana secara baku sudah menggunakan satuan responsif abstrak (seperti *logical pixels*, *points*, atau *density-independent pixels*). Pada *framework* tersebut, silakan gunakan satuan baku bawaannya.

## Kepatuhan Aksesibilitas (Accessibility Compliance)
Jika *project/node* yang sedang dikerjakan merupakan aplikasi antarmuka pengguna seperti **Web (Frontend)** atau **Mobile App**, Anda **WAJIB** memastikan bahwa hasil pekerjaan memenuhi standar aksesibilitas (*accessibility* / a11y).
1. **Adaptasi Berdasarkan Teknologi:** Pendekatan aksesibilitas harus disesuaikan dengan jenis teknologi yang digunakan. Jika menggunakan teknologi *native*, manfaatkan kapabilitas dan API aksesibilitas bawaan dari *platform native* tersebut secara maksimal.
2. **Prosedur Implementasi & Konfirmasi:** Anda harus mendaftar dan memberikan seluruh opsi fitur aksesibilitas yang relevan dan dapat diimplementasikan sesuai dengan *stack* teknologi yang dipakai. Setelah memberikan daftar opsi tersebut, **wajib** tanyakan kepada *developer* (pengguna) mengenai fitur mana saja yang ingin diimplementasikan.

## Evaluasi Heuristik (Heuristic Evaluation)
Anda **WAJIB** menggunakan metode *Heuristic Evaluation* untuk meninjau dan mengevaluasi kualitas usabilitas antarmuka pengguna (UI). Pengecekan ini mutlak dilakukan untuk memastikan seluruh desain dan interaksi telah sejalan dengan prinsip dasar kemudahan penggunaan.

Berikut adalah 10 prinsip Heuristik Jakob Nielsen yang wajib dicek beserta contoh *approach* (pendekatan) penerapannya pada tahap pengembangan antarmuka:

1. **Visibilitas Status Sistem (Visibility of system status)**
   - *Approach:* Selalu berikan *feedback* instan kepada pengguna. Gunakan *loading spinner* atau *skeleton screen* saat mengambil/mengirim data via API. Jika ada proses latar belakang, sediakan *progress bar* atau teks status (seperti "Menyimpan...").
2. **Kecocokan Sistem dengan Dunia Nyata (Match between system and the real world)**
   - *Approach:* Gunakan ikon universal (misal: ikon tong sampah untuk "Hapus", ikon disket untuk "Simpan"). Jangan tampilkan pesan *error* teknis (contoh: "Error 500" atau "Null Reference"), ubahlah menjadi bahasa natural ("Maaf, koneksi gagal. Silakan coba lagi").
3. **Kendali dan Kebebasan Pengguna (User control and freedom)**
   - *Approach:* Pastikan setiap alur memiliki jalan keluar. Selalu sediakan tombol "Batal" (*Cancel*), "Kembali" (*Back*), atau opsi "Urungkan" (*Undo*) jika pengguna tidak sengaja mengeklik atau menghapus suatu *item*.
4. **Konsistensi dan Standar (Consistency and standards)**
   - *Approach:* Terapkan penamaan, warna, tipografi, dan gaya komponen (seperti tombol *Primary/Secondary*) yang konsisten di seluruh halaman aplikasi. Pastikan implementasinya mematuhi standar *Design System* dari *framework* atau desain awal.
5. **Pencegahan Kesalahan (Error prevention)**
   - *Approach:* Terapkan validasi form secara *real-time* saat pengetikan (contoh: memvalidasi format email sebelum ditekan *Submit*). Gunakan *state disabled* pada tombol kirim jika input masih kosong/salah. Tampilkan modal konfirmasi untuk tindakan yang merusak/destruktif ("Apakah Anda yakin ingin menghapus ini?").
6. **Pengenalan Daripada Mengingat (Recognition rather than recall)**
   - *Approach:* Jangan memaksa pengguna mengingat informasi di luar kepala. Gunakan *dropdown autocomplete*, *placeholder* yang deskriptif, dan fitur "Pencarian Terakhir" (*Recent Searches*) agar pengguna dapat sekadar memilih objek yang pernah ditangani.
7. **Fleksibilitas dan Efisiensi Penggunaan (Flexibility and efficiency of use)**
   - *Approach:* Rancang UI agar bisa digunakan secara cepat oleh pemakai tingkat lanjut (*expert user*). Sediakan jalan pintas *keyboard* (*shortcuts/hotkeys*), fitur aksi jamak (*bulk actions*), atau gestur sapuan (*swipe actions* pada versi *mobile*).
8. **Desain Estetis dan Minimalis (Aesthetic and minimalist design)**
   - *Approach:* Hindari *clutter* atau tata letak elemen yang terlalu padat. Jaga *white space* atau jarak margin/padding yang cukup untuk menonjolkan fokus utama layar. Hapus semua elemen dekoratif yang tidak menunjang fungsionalitas.
9. **Membantu Pengguna Mengenali, Mendiagnosis, dan Memulihkan Kesalahan (Help users recognize, diagnose, and recover from errors)**
   - *Approach:* Jika validasi *form* gagal, *highlight* warna merah khusus pada kolom/bidang yang bermasalah. Jangan hanya menulis "Input Invalid", melainkan beri tahu cara memperbaikinya ("Password harus memiliki minimal 8 karakter dan huruf kapital").
10. **Bantuan dan Dokumentasi (Help and documentation)**
    - *Approach:* Berikan *tooltip* bantuan (?) pada label yang mungkin membingungkan. Apabila UI melibatkan proses langkah-langkah rumit, berikan layar pengantar interaktif (*onboarding walkthrough*) atau tautan "Pelajari Lebih Lanjut".

---

## 📋 DAFTAR PERIKSA TINJAUAN VISUAL ANTI-SLOP (VISUAL REVIEW CHECKLIST)

Sebelum menandai tugas antarmuka pengguna selesai, AI Agent **WAJIB MENINJAU TAMPILAN VISUAL AKTUAL** (bukan sekadar memastikan kode lolos kompilasi) dan memvalidasi checklist berikut:

- [ ] **Kekhususan Produk:** Arah visual mencerminkan domain produk nyata, bukan *template* SaaS AI generik.
- [ ] **Bebas Konten Tiruan:** Tidak ada metrik, angka transaksi, testimoni, atau grafik fiktif buatan AI.
- [ ] **Hierarki & Fokus Jelas:** Setiap layar memiliki titik fokus utama yang nyata, dan elemen pendukung memiliki bobot visual lebih rendah.
- [ ] **Bebas Elemen Slop:** Tidak ada mesh gradien ungu/biru tanpa konteks, tanpa *glassmorphism* berlebihan, tanpa *glow* neon, tanpa kartu berulang identik, dan tanpa banner hero di dashboard operasional.
- [ ] **Kelengkapan State:** Seluruh status (*loading*, *skeleton*, *empty*, *error*, *retry*, *disabled*, *focused*, *success*) terimplementasikan secara fungsional.
- [ ] **Aksesibilitas & Kontras:** Kontras teks memenuhi WCAG AA, fokus keyboard terlihat jelas, kontrol sentuh berukuran minimal 24x24px (web) / 44x44pt (mobile), dan `prefers-reduced-motion` didukung.
- [ ] **Responsif & Reflow:** Antarmuka teruji rapi pada lebar 320px tanpa *horizontal scroll* yang tidak disengaja.
- [ ] **Microcopy Konkret:** Seluruh tombol dan pesan galat memandu tindakan nyata pengguna, bebas dari slogan hampa AI.

### 🚪 GERBANG AKHIR ANTI-SLOP (THE FINAL GATE)
Sebelum meminta persetujuan commit, jawab dua pertanyaan penentu ini:
1. **"Jika saya menghapus 20% dekorasi dan efek visual ini, apakah produk menjadi lebih jelas dan mudah digunakan?"** $\rightarrow$ Jika **YA**, **HAPUS DEKORASI TERSEBUT!**
2. **"Jika saya menghapus 20% elemen UI ini, apakah tugas pengguna menjadi lebih sulit?"** $\rightarrow$ Jika **YA**, **PERTAHANKAN ELEMEN TERSEBUT!**
