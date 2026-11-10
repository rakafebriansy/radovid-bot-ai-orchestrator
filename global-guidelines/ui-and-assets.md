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

### 2.0 Prinsip Identitas Visual Berakar Domain & Proses Desain Dua Tahap
Dalam menyusun antarmuka visual baru, komponen web, atau mendesain ulang UI eksisting, AI Agent WAJIB bertindak sebagai *design lead* yang memberikan identitas visual unik (*distinctive point of view*), menolak estetika klise AI slop, dan mematuhi prinsip perancangan berikut:
1. **Identitas Berakar Subjek & Vernacular Domain (*Subject-Matter Grounding*):** Gali arah estetika langsung dari industri, materi subjek, dan bahasa visual (*vernacular*) domain produk nyata (contoh: desain antarmuka mainan anak usia 8–11 tahun harus sangat berbeda dari dashboard analis finansial). Untuk halaman web yang membutuhkan *hero section* (seperti landing page), buka dengan elemen paling khas dari dunia subjek tersebut (demo interaktif langsung, visual produk, atau olahan tipografi aktif)—jangan gunakan formula *default* (angka statistik besar + label kecil + aksen gradien) kecuali benar-benar paling tepat.
2. **Alur Perancangan Dua Tahap (*Two-Pass Design Process*):**
   - **Pass 1 (Sistem Token Ringkas & Wireframe ASCII):** Susun rencana desain singkat berbasis brief mencakup: *Color* (4–6 nilai hex bernama sebagai palet dasar), *Type* (pemilihan typeface dan perannya), *Layout* (konsep tata letak dalam 1 kalimat beserta *ASCII wireframe* dan panduan perataan kiri/tengah/rata kanan-kiri), serta *Principles* (panduan pembeda halaman).
   - **Pass 2 (Uji Keunikan & Kritik Mandiri):** Tinjau rencana terhadap brief sebelum koding. Jika bagian mana pun masih terasa seperti *default* generik yang akan dihasilkan untuk halaman serupa lainnya, revisi bagian tersebut dan nyatakan alasannya sebelum mulai menulis kode.
3. **Prinsip Restraint (*Spend Boldness in One Place*):** Fokuskan keberanian visual pada satu elemen paling berkesan (*the memorable thing*) dan jaga elemen di sekelilingnya tetap tenang serta disiplin. Kurangi satu aksesori atau dekorasi yang tidak melayani brief sebelum menyelesaikan desain.

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
11. **[TERLARANG] Animasi Asal Ramai:** Efek *bounce-on-hover*, animasi memantul tak berkesudahan, entrances *fade-and-slide-up* mekanis di setiap seksi, atau transisi lambat yang menghambat kecepatan kerja pengguna.
12. **[TERLARANG] Palet & Layout Default Kalibrasi AI Tanpa Dasar Brief:** Latar krem hangat (`#F4F1EA`) dengan serif kontras tinggi dan aksen terracotta/clay (`#D97757` — aksen interaksi Claude yang langsung terbaca sebagai template AI), latar hampir hitam (`#0B0B0B` / `#111`) dengan satu aksen *acid-green* atau vermilion, maupun tata letak koran (*broadsheet*) dengan garis *hairline* tipis dan radius `0` untuk produk yang tidak relevan.
13. **[TERLARANG] Ornamen Chrome Template Mekanis (*Template Chrome Tells*):** Label *eyebrow* `ALL-CAPS` berjarak lebar (*tracked-out*) di atas setiap heading, string metadata yang disambung titik tengah (`A · B · C`), label berformat `'KATA — fragmen'` dengan *em-dash* berspasi, font *monospace* untuk seluruh label data kecil, penomoran urut (`01 / 02 / 03`) pada konten yang bukan urutan sekuensial (*non-sequential*), serta tanda panah `'→'` yang ditempelkan di setiap teks tombol atau tautan.

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

### 2.4 Tata Letak, Tangga Permukaan (*Surface Ladder*), & Geometri Kanvas
1. **Kartu adalah Pengelompok Semantik:** Gunakan kartu HANYA jika konten memiliki relasi mandiri, dapat ditindaklanjuti terpisah, atau membutuhkan pemisahan spasial nyata.
2. **Utamakan Struktur Langsung:** Gunakan whitespace, heading, pembatas garis (*dividers*), tabel, dan daftar terstruktur alih-alih membungkus setiap kalimat ke dalam kotak kartu melayang.
3. **[TERLARANG]** Struktur kartu bertumpuk tanpa arti (`page -> card -> sub-card -> mini-card`) dengan bayangan tebal.
4. **Tangga Permukaan (*The Surface Ladder*) & Invarian Halaman:**
   - Pisahkan kelompok konten secara bervariasi menaiki 4 anak tangga: (1) *Whitespace* murni $\rightarrow$ (2) *Hairline rule* (`1px solid var(--color-rule)`) $\rightarrow$ (3) *Tinted well* (`--color-paper-2` tanpa border) $\rightarrow$ (4) *Card* (hanya untuk elemen mandiri yang dapat ditindaklanjuti).
   - **[TERLARANG] Menggabungkan Border & Wide Shadow:** Sebuah kartu memilih *hairline border* ATAU *shadow*, dilarang memasang keduanya sekaligus pada satu permukaan.
   - **Invarian Halaman:** Maksimal **satu gaya tombol primary berisi (*filled primary button*) per layar/halaman** (aksi sekunder menggunakan gaya *outlined* atau tipografis), satu lebar kontainer utama, satu ukuran celah kolom di bawah hero, serta jumlah grup judul rata-tengah (*centered*) tidak boleh melebihi yang rata-kiri (*left-aligned*).
5. **Geometri Kanvas, Kepadatan Layar, & Anatomi Berbasis Tugas:**
   - **Batas Kata per Layar (*Screen Word Budget*) & Keterlihatan Fold:** Untuk halaman *landing/marketing*, batasi sekitar **80 kata per jendela tinggi 900px** (maksimal ~450 kata terlihat per halaman). Pada resolusi `1280x800`, judul hero ($\le 7$ kata), satu kalimat *lede* ($\le 12$ kata), CTA utama, dan titik fokus visual wajib terlihat utuh tanpa scroll. Pada `1920x1080`, kontainer tetap terkunci di tengah (`max-width: 1360px–1520px`) sehingga hanya *gutter* tepi yang melebar, bukan jumlah kolomnya.
   - **Rasio Spasi Heading (*Crowded Head Rule*):** Jarak vertikal di atas judul seksi (`h2`–`h4`) wajib **minimal 2x jarak di bawahnya** agar judul terikat secara visual ke konten yang dipimpinnya, bukan ke seksi sebelumnya. Ukuran `h2` seksi tidak boleh kerdil ($\ge 2.5\text{rem}$ pada desktop).
   - **Alternatif Pembuka Seksi Tanpa "Eyebrow" Mekanis:** Hindari menaruh label *eyebrow* kecil di atas setiap judul seksi; gunakan judul yang langsung menyebut subjek di 2 kata pertama, *lead-in* tebal di awal paragraf pertama, *hairline rule* bervariasi, `<ol>` asli untuk tahapan sekuensial, atau perubahan warna kertas (*paper band*).
   - **Anatomi & Penutup Halaman Berbasis Tugas (*Job-Based Anatomy*):** Susun struktur halaman dari tugas nyata pengunjung (*subscribe, book, visit, attend, buy, try, sign up, hire*), bukan urutan klise `hero -> 3 fitur -> angka -> form email`. Dilarang menutup halaman dengan form *newsletter/email sign-up* kecuali tugas utama halaman memang berlangganan; tutup halaman dengan aksi utama yang relevan.

### 2.5 Kejujuran Material, Gambar, & Larangan Fabrikasi Konten
1. **[TERLARANG]** Mengarang angka statistik, volume transaksi, testimoni palsu, ulasan tiruan, grafik fiktif, notifikasi dummy, atau logo mitra palsu.
2. **[WAJIB]** Jika data aktual belum tersedia, gunakan tampilan *Empty State* yang jujur dan instruktif atau *placeholder* eksplisit yang ditandai dengan jelas.
3. **Hirarki Bukti Visual (*Show, Do Not Describe*):** Utamakan (1) aset/foto asli proyek, (2) tangkapan layar (*screenshot*) produk nyata yang sedang berjalan, (3) antarmuka produk fungsional yang dibangun langsung di dalam slot, atau (4) fotografi berlisensi terbuka dengan kredit jelas (hindari hotlink stok generik atau foto wajah stok sebagai testimoni). Di bawah hero, foto maksimal menempati **1/3 tinggi halaman** dan dilarang meletakkan dua seksi foto penuh secara berurutan tanpa diselingi seksi tipografi/tabel/produk.
4. **[TERLARANG] Bingkai Perangkat Palsu, Ilustrasi Vektor Dekoratif, & "Fake Send":**
   - Dilarang menggambar bilah peramban palsu (*traffic lights* merah-kuning-hijau, *URL pill*) atau *title bar* OS tiruan di sekitar screenshot.
   - Dilarang membuat ilustrasi SVG/CSS dekoratif berupa gambar objek, tanaman, hewan, maskot, atau pemandangan di atas ukuran ikon (SVG hanya untuk ikon, logo asli, atau diagram/grafik data nyata), maupun deretan tanda hitung dekoratif (*counting marks* $\ge 24$ titik/garis identik).
   - **Larangan "Fake Send" pada Formulir:** Formulir yang belum terhubung ke endpoint backend dilarang menampilkan pesan sukses palsu (*"Terkirim"*, *"Terima kasih"*); wajib menyatakan secara jujur bahwa form belum terhubung hingga endpoint diikat.

### 2.6 Sistem Token Desain, Warna OKLCH, Dark Mode, & Border Radius
1. **Token Terikat:** Gunakan skala terukur untuk spacing (kelipatan 4px atau 8px), skala tipe, warna, radius, dan elevasi. Dilarang menciptakan nilai acak baru di setiap komponen.
2. **Warna untuk Makna:** Gunakan warna untuk menandai aksi utama, status fungsional (sukses, peringatan, galat), hierarki, dan identitas brand.
3. **Kontras Wajib WCAG AA:**
   - Teks reguler: Rasio kontras minimal **4.5:1** terhadap latar belakang.
   - Teks besar ($\ge$ 18pt atau $\ge$ 14pt tebal): Rasio kontras minimal **3:1**.
   - Dilarang meredupkan teks menjadi abu-abu pudar berekontras rendah demi tren visual.
4. **Konsistensi Geometri & Radius Konsentris:** Elemen kontrol (input, tombol) dan kontainer wajib memiliki bahasa bentuk yang seragam; hindari mencampur bentuk kapsul runcing dan melengkung acak di satu layar. Gunakan rumus radius bersarang (*concentric radii*): `inner_radius = max(0, outer_radius - padding)`.
5. **Disiplin Spesifisitas Selektor CSS (*CSS Selector Specificity*):** Jaga hierarki selektor CSS agar tidak saling membatalkan (*cancel out*), khususnya benturan aturan `margin` atau `padding` antara selektor kontainer bagian (`.section`) dan selektor elemen aksi (`.cta`).
6. **Ruang Warna OKLCH, Tinted Neutrals, & Anggaran Aksen ($\le 5\%$):**
   - **[TERLARANG] Hitam/Putih Absolut Murni (`#000000` / `#ffffff`):** Hindari hitam murni atau putih murni pada permukaan (*paper*) maupun tinta (*ink*). Gunakan ruang warna **OKLCH** dan beri sedikit *tint* menuju *anchor hue* (`chroma 0.005–0.018` pada netral; kertas terang `L 92–98%`, kertas gelap `L 12–18%`).
   - **Batas Porsi Aksen (*Accent Footprint $\le 5\%–8\%$*):** Pada postur *Restrained* default, warna aksen berfungsi sebagai sinyal fokus (item navigasi aktif, focus ring, kontrol utama, satu seri data utama) maksimal **5%–8% dari viewport**, bukan pewarna latar masif, kecuali postur warna penuh dideklarasikan secara eksplisit dalam brief.
7. **Dark Mode adalah Re-derivasi, Bukan Inversi:**
   - **[TERLARANG]** Menggunakan `filter: invert()` atau sekadar membalik tangga abu-abu.
   - **Elevasi via Kecerahan (*Lightness*), Bukan Drop Shadow:** Pada Dark Mode, nyatakan elevasi permukaan dengan tingkat kecerahan (`+3% L` per level di atas kanvas dasar) atau *inset hairline* (`box-shadow: inset 0 0 0 1px var(--color-rule)`), **BUKAN drop shadow** (karena *drop shadow* di latar gelap terbaca sebagai *glow* keruh; permukaan kartu harus lebih terang dari kanvas dasarnya, bukan lebih gelap).
   - Turunkan saturasi/chroma aksen pada mode gelap (`chroma -0.02 s/d -0.04`, `L +5–10%`) untuk mencegah silau (*glare*), dan kurangi bobot font body sekitar 50 unit (misal `400` ke `350`) karena teks terang di atas latar gelap tampak lebih tebal secara optik.

### 2.7 Ikon, Visualisasi Data, & Tipografi Fungsional
1. **Ikon Informatif:** Gunakan ikon untuk navigasi dan penjelas aksi; singkirkan ikon dekoratif tanpa nilai informasi.
2. **Hindari Kotak Ikon Pastel Berulang:** Dilarang membungkus setiap ikon ke dalam kotak membulat warna-warni secara seragam tanpa tujuan semantik.
3. **Satu Keluarga Ikon Konsisten:** Gunakan satu pustaka ikon seragam (Lucide, Material Icons, dll.). Dilarang mencampur pustaka ikon yang berbeda karakter.
4. **[TERLARANG]** Menggunakan emoji sebagai pengganti ikon profesional pada antarmuka sistem bisnis.
5. **Tipografi Fungsional, Pemilihan Typeface Anti-Saturasi, & Keterbacaan Baris:**
   - Gunakan maksimal 1 atau 2 keluarga *typeface* (jika menggunakan 2, pastikan perbedaannya tegas antara *display/headline* dan *body*) dengan maksimal **3 bobot font (*weights*)** di seluruh halaman, serta pasang `font-synthesis: none` dan `font-display: swap`.
   - Hindari penggunaan font default yang terlalu jenuh pada halaman pemasaran/editorial baru (seperti Inter, Roboto, Open Sans, Poppins, Lato, Montserrat) sebagai *display face* kecuali sudah menjadi standar eksisting proyek.
   - Batasi panjang baris (*line length*) teks body **di bawah 80 karakter** (`< 80ch`), berikan *line-height* sedikit lebih longgar apabila menggunakan font *serif*, serta terapkan `text-wrap: balance` pada judul dan `text-wrap: pretty` pada paragraf body.
   - Terapkan `font-variant-numeric: tabular-nums` untuk perataan angka pada tabel data.
6. **[TERLARANG] Perlakuan Tipografi Klise AI (*Typographic AI Tells*):**
   - Dilarang memberi aksen (miring, tebal, atau warna berbeda) hanya pada satu kata atau frasa di tengah kalimat judul (*single-word headline accenting*).
   - Dilarang menggunakan huruf kapital semua (*ALL-CAPS*) secara seragam untuk label umum atau menyisipkan label tipografi tak perlu di atas konten.
7. **Standar Visualisasi Data Jujur (*Honest Data Visualization*):**
   - Pilih bentuk visual dari struktur data: 1 angka $\rightarrow$ *stat tile* (bukan grafik 1 batang); 2–5 angka $\rightarrow$ *stat row* atau batang horizontal; deret waktu $\rightarrow$ garis/*sparkline*; $>7$ kategori $\rightarrow$ tabel; bagian dari keseluruhan (*parts of a whole*) $\rightarrow$ *stacked horizontal bar* (**hindari pie chart, donut chart, atau gauge wall**).
   - Batang grafik wajib dimulai dari basis **nol (`0`)**, gunakan **satu sumbu Y** (dilarang *dual y-axis*), garis grid *hairline* tipis (`3–5` garis, tanpa garis putus-putus), satu seri beraksen (seri lain menggunakan *lightness ramp* pada hue yang sama atau netral `--color-muted`), dan teks label tidak boleh diwarnai mengikuti warna seri data.

### 2.8 Microcopy Aksi & Penanganan Galat
1. **Label Aksi Spesifik & Kohesi Alur (*Verb Cohesion*):** Gunakan kalimat aktif (*active voice*), *sentence case*, dan label tombol yang menjelaskan hasil aksi: `"Simpan Perubahan"`, `"Hapus Akun"`, `"Tambah Produk"`, `"Unduh Laporan"` (hindari label kabur seperti `"Submit"`, `"Lanjutkan"`, `"Klik Di Sini"`). Pertahankan kata kerja yang sama di sepanjang alur interaksi (contoh: tombol `"Terbitkan"` menghasilkan umpan balik toast `"Diterbitkan"`).
2. **Kosakata Perspektif Pengguna Akhir (*End-User Vocabulary*):** Namai fitur menggunakan istilah yang dipahami pengguna dalam bahasa sederhana, bukan cara sistem dibangun di belakang layar (contoh: gunakan `"Kelola Notifikasi"`, bukan `"Konfigurasi Webhook"`). Bebaskan teks dari jargon pemasaran hampa.
3. **Pesan Galat Solutif & Tanpa Basa-Basi:** Pesan galat wajib menjelaskan secara lugas: (1) apa yang terjadi, dan (2) tindakan konkret untuk memperbaikinya menggunakan suara antarmuka yang objektif (tanpa permintaan maaf berlebih), serta perlakukan layar kosong (*empty state*) sebagai arahan bertindak.

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

### 2.10 Formulir, Stabilitas Input, & Umpan Balik Interaksi
1. **Label Persisten:** Setiap input formulir wajib memiliki elemen `<label>` yang jelas dan persisten (bukan sekadar placeholder yang hilang saat diketik).
2. **Izin Paste:** Dilarang menonaktifkan fitur tempel teks (*paste*) pada bidang input secara sembarangan.
3. **Ukuran Target Sentuh Minimum:**
   - **Web (WCAG 2.2):** Minimal **24 x 24 CSS px**.
   - **Mobile / Touch Interface (Apple HIG & Material):** Minimal **44 x 44 pt / 48 x 48 dp**.
4. **Indikator Fokus Keyboard:** Sediakan indikator fokus yang kontras (*focus visible ring*) pada setiap kontrol interaktif. Dilarang menetapkan `outline: none` tanpa menyediakan pengganti fokus visual.
5. **Stabilitas Dimensi Input & Cadangan Ruang Helper (`1lh`):** Lebar border input **dilarang berubah** antar-state (gunakan `outline` transparan yang berubah warna saat `:focus-visible`, bukan mengubah `border-width`); tinggi input wajib sama persis dengan tombol di sebelahnya; wajib mencadangkan `min-height: 1lh` untuk baris teks bantuan/error agar halaman tidak bergeser (*layout shift*) saat validasi muncul. Jalankan validasi pertama saat `blur` (setelah disentuh), lalu per-input (jangan menghukum di ketikan karakter pertama).
6. **Silent Success & Undo over Confirm:**
   - **Silent Success:** Jika hasil aksi sudah terlihat langsung di layar (misal: item berpindah, baris terhapus, label tombol berubah menjadi *"Disalin"*), **jangan munculkan toast**. Gunakan toast hanya untuk kegagalan (*error* dengan tombol *Coba Lagi*) atau efek latar belakang tak terlihat.
   - **Undo over Confirm:** Untuk aksi yang dapat dibatalkan, jalankan langsung secara optimis (*optimistic update*) dengan opsi **Undo (5–10 detik)**; gunakan dialog konfirmasi hanya untuk aksi destruktif permanen yang tidak dapat dipulihkan.

### 2.11 Desain Responsif Nyata, Reflow, & Ketahanan CSS Layout
1. **Ambang Batas Reflow 320px:** Seluruh antarmuka wajib teruji rapi pada lebar minimal **320px CSS width** tanpa memicu scroll horizontal yang tidak diinginkan.
2. **Tata Letak Fluida:** Gunakan CSS Grid, Flexbox, dan *Container Queries* alih-alih menumpuk media query kaku.
3. **Adaptasi Tabel Mobile:** Sediakan strategi penyajian tabel pada layar sempit (tampilan kartu ringkas bertingkat atau scroll horizontal mandiri dengan indikator visual yang jelas).
4. **Ketahanan Overflow & Unit Viewport Aman:** Gunakan `overflow-x: clip` pada kontainer halaman (bukan `overflow-x: hidden` yang merusak `position: sticky`, dan dilarang menggunakan `100vw` yang memicu horizontal scroll). Gunakan unit `dvh` atau `svh` untuk tinggi hero (dilarang `vh` mentah). Track CSS Grid yang memuat gambar atau konten lebar wajib menggunakan `minmax(0, 1fr)` (bukan `1fr` telanjang), dan judul besar wajib memasang `overflow-wrap: break-word; min-width: 0`.

### 2.12 Animasi, Transisi Spesifik, Tekstur, & Reduced Motion
1. **Animasi Fungsional:** Gunakan animasi hanya untuk mengomunikasikan perubahan status, hierarki spasial, atau progres kerja.
2. **[TERLARANG]** Animasi memantul (*bounce*), pergerakan looping tanpa henti, atau efek parallax berlebih.
3. **[WAJIB] Dukungan `prefers-reduced-motion`:** Seluruh animasi dan transisi wajib dinonaktifkan atau disederhanakan seketika jika pengguna mengaktifkan pengaturan pengurangan gerakan di sistem operasi.
4. **Hanya Animasikan `transform` & `opacity` (Larangan `transition: all`):**
   - **[TERLARANG]** Menggunakan `transition: all` atau menganimasikan properti tata letak (`width`, `height`, `top`, `left`, `margin`, `padding`). Hanya animasikan `transform` dan `opacity` dengan durasi terukur (micro `120ms`, short `220ms`, long `420ms`; animasi keluar pada `60–75%` durasi masuk).
   - Focus ring wajib muncul **instan (`0ms`)** tanpa efek *fade-in*; tooltip memiliki *hover delay* `800–1000ms` tetapi **focus delay `0ms`**; konten *scroll-reveal* wajib tetap terlihat jika JavaScript mati (gunakan scoping `.js [data-reveal]` atau `@starting-style`); setiap animasi/video berulang $>5$ detik wajib berhenti saat *off-screen* dan memiliki kontrol **Pause** yang terlihat.
5. **Anggaran Tekstur & Koreografi Scroll:**
   - Jika tema menggunakan tekstur cetak (grain, halftone, scanline), batasi maksimal **1 keluarga artefak per halaman**, tidak pernah dianimasikan, `pointer-events: none`, `aria-hidden="true"`, dengan opasitas bertoken ($\le 0.08$ seluruh halaman, $\le 0.05$ di bawah teks body).
   - Koreografi berbasis scroll (`animation-timeline: view()`) maksimal **1 pola per halaman**, hanya pada elemen struktural (dilarang pada teks paragraf body), dan wajib dibungkus di dalam `@supports` serta `@media (prefers-reduced-motion: no-preference)`.

### 2.13 Kinerja Frontend & Core Web Vitals
1. **Indikator Web Vitals:** Pastikan antarmuka memenuhi standar performa (LCP $\le$ 2.5s, INP responsif, CLS rendah).
2. **Optimasi Media:** Gunakan format modern (WebP/AVIF) dengan atribut `width` dan `height` eksplisit untuk mencegah pergeseran tata letak.
3. **Efisiensi Rendering:** Hindari efek visual berat (seperti blur bertumpuk atau multi-layer shadow) yang memicu *frame drop* saat scrolling pada perangkat berspesifikasi rendah.

### 2.14 Disiplin Redesign & Pemolesan Antarmuka Eksisting
Saat melakukan *redesign*, *restyle*, atau *polish* pada kode antarmuka yang sudah berjalan:
1. **[WAJIB] Pertahankan Konten Faktual & Wiring Fungsional:** Setiap tautan `href`, atribut formulir (`name`, `id`, `type`, `required`), `data-testid`, event handler, pemanggilan API, dan rute navigasi wajib tetap utuh dan berfungsi.
2. **[TERLARANG]** Menghapus rute, komponen, fitur tersembunyi, atau melonggarkan pengujian otomatis hanya demi merapikan tampilan tanpa persetujuan eksplisit pada rencana implementasi.

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
- [ ] **Bebas Konten & Bingkai Tiruan:** Nol angka transaksi, statistik, testimoni, grafik fiktif, bilah browser palsu (*traffic lights*), ilustrasi vektor dekoratif di atas ukuran ikon, atau *fake form send*.
- [ ] **Hierarki, Tangga Permukaan, & Invarian Halaman:** Titik fokus tegas, pemisahan konten bervariasi menaiki *Surface Ladder* (tanpa *border + wide shadow* bersamaan), dan hanya 1 gaya tombol primary berisi per halaman.
- [ ] **Disiplin Warna OKLCH & Dark Mode:** Bebas hitam/putih absolut (`#000`/`#fff`), porsi aksen $\le 5\%–8\%$ (pada postur *Restrained*), serta elevasi Dark Mode menggunakan tingkat kecerahan (*lightness*), bukan *drop shadow*.
- [ ] **Bebas Elemen Slop:** Tidak ada mesh gradien klise, glassmorphism buram berlebih, border neon, kartu berjejer identik, atau banner hero di dashboard operasional.
- [ ] **Kelengkapan Status & Stabilitas Input:** Seluruh status fungsional terimplementasi, lebar border input stabil antar-state, ruang helper `1lh` dicadangkan, dan pola *Silent Success* / *Undo over Confirm* diterapkan.
- [ ] **Aksesibilitas, Transisi, & Kontras:** Kontras teks lolos WCAG AA, fokus keyboard instan (`0ms`), kontrol sentuh $\ge$ 24x24px (web) / 44x44pt (mobile), hanya menganimasikan `transform`/`opacity`, dan `prefers-reduced-motion` aktif.
- [ ] **Responsif, Reflow, & Ketahanan CSS:** Teruji rapi pada lebar 320px, `1280x800`, dan `1920x1080` dengan `overflow-x: clip`, unit `dvh`/`svh`, serta `minmax(0, 1fr)` tanpa scroll horizontal yang tidak diinginkan.
- [ ] **Microcopy Konkret & Wiring Utuh:** Seluruh tombol dan pesan memandu tindakan solutif pengguna, serta seluruh *wiring* fungsional tetap utuh saat redesign.

### 4.2 Gerbang Keputusan Akhir (The Final Gate)
Sebelum mengajukan izin commit dan penyerahan tugas antarmuka, jawab dua pertanyaan penentu berikut:
1. *"Jika saya menghapus 20% dekorasi dan efek visual ini, apakah produk menjadi lebih jelas dan mudah digunakan?"*
   - Jika **YA** $\rightarrow$ **HAPUS DEKORASI TERSEBUT SEGERA.**
2. *"Jika saya menghapus 20% elemen UI ini, apakah tugas pengguna menjadi lebih sulit diselesaikan?"*
   - Jika **YA** $\rightarrow$ **PERTAHANKAN ELEMEN TERSEBUT.**
