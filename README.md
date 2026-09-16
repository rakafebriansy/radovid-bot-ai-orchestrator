# AI Orchestrator Template

Selamat datang di ekosistem **AI Orchestrator**. Berbeda dengan sekadar *prompting* AI biasa untuk menulis kode, repositori ini adalah sebuah kerangka kerja (*framework*) rekayasa perangkat lunak yang dirancang agar AI Agent (seperti Cursor, GitHub Copilot Workspace, atau agen otonom lainnya) bekerja layaknya seorang *Software Engineer* terstruktur yang tidak mudah berhalusinasi, terhindar dari *looping error*, dan selalu merujuk pada dokumentasi mutlak (*Single Source of Truth*).

---

## 🚀 Cara Penggunaan

Proses pengembangan dan interaksi dengan Orchestrator ini dibagi menjadi 4 fase Prompt utama: Inisialisasi (*Startup*), Penambahan Node (*Scaling*), Pengerjaan Tugas (*Execution*), dan Tanya-Jawab Implementasi (*Knowledge Q&A*).

### FASE 1: Startup Prompt (Inisialisasi Proyek)
Gunakan salah satu *prompt* di bawah ini hanya **satu kali** di awal proyek (tergantung apakah proyek Anda hanya satu aplikasi tunggal atau gabungan dari beberapa aplikasi/node). Tujuannya adalah menginisialisasi seluruh dokumen kosong di `docs/` dan menyiapkan pedoman proyek Anda.

#### Opsi A: Startup Prompt (Single-Project Environment)
Gunakan *prompt* ini jika Anda hanya ingin mengatur satu aplikasi/repositori (misal: hanya Frontend atau Fullstack Monorepo). Salin, isi bagian `[ DALAM KURUNG SIKU ]`, dan kirimkan utuh ke AI Agent:

```text
Saya ingin menginisialisasi Single-Project Environment menggunakan kerangka kerja AI Orchestrator ini.

Berikut adalah definisi Proyek Utama yang akan diatur:

A. INFORMASI PROYEK
1. Nama Sistem: [Nama aplikasi, misal: Personal Portfolio Website]
2. Ide/Konsep Utama: [Jelaskan fitur utama dari sistem ini]
3. Vibe/Estetika UI: [Misal: Modern, minimalis, dominasi warna gelap]
4. Batasan Sistem Utama: [Misal: Harus responsif, cepat, SEO-friendly]
5. Path Codebase: [Path absolut ke folder proyek, misal: `/Users/.../my-portfolio`]
6. Tech Stack: [Misal: Next.js, TailwindCSS]

INSTRUKSI STARTUP ANDA:
Berbekal informasi di atas, JANGAN MENULIS KODE APLIKASI SAMA SEKALI. Lakukan langkah-langkah otonom berikut secara berurutan:
1. Pahami struktur `ai-orchestrator-template` ini. Karena ini adalah lingkungan proyek tunggal, kita hanya akan menggunakan satu Node utama.
2. Evaluasi & Wawancara Pengguna: Jika deskripsi yang saya berikan di atas masih terlalu dangkal atau belum cukup untuk mengisi Dokumen Mandatory secara detail dan maksimal, Anda WAJIB BERHENTI mengeksekusi langkah selanjutnya. (Catatan: Yang tergolong Dokumen Mandatory adalah: `prd.md`, `design-system.md`, `system-design.md`, dan `development-planning.md`). Ajukan daftar pertanyaan kritis kepada saya terkait visi, batasan teknis, target pengguna, dan spesifikasi fungsionalitas. Selain itu, SEBELUM Anda bertanya tentang Graphify, Anda WAJIB menanyakan: *"Mode Operasional mana yang ingin Anda gunakan secara default? (1) Mode 1: Autonomous Planning (digerakkan oleh roadmap), atau (2) Mode 2: Prompt-Driven (digerakkan instruksi mikro)"*. SETELAH ITU, tanyakan: *"Apakah Anda ingin menggunakan fitur CLI Graphify untuk pemetaan arsitektur otomatis? (Ya/Tidak)"*. SELANJUTNYA, tanyakan preferensi alat pengujian (*testing*) dan *linter* yang ingin digunakan sesuai teknologi proyek. JIKA proyek berupa *web* atau *mobile app*, tanyakan juga opsi fitur aksesibilitas (*accessibility*) apa saja yang ingin diimplementasikan (opsional). Ulangi proses tanya-jawab ini hingga Anda memiliki konteks yang solid.
2.5. Pengecekan Environment (KONDISIONAL): JIKA pengguna menjawab YA untuk pemakaian Graphify pada tahap wawancara, eksekusi `graphify --version` di terminal. Jika gagal (not found), instal segera dengan `npm install -g @sentropic/graphify`. Jika pengguna menjawab TIDAK, lewati langkah ini sepenuhnya.
3. Setelah informasi dirasa memadai, buatkan draf komprehensif untuk `global-docs/prd.md` dan `global-docs/design-system.md` berdasarkan spesifikasi proyek di atas.
4. Buatkan GitHub Project di awal untuk repositori ini. Pastikan GitHub Project tersebut dibuat di bawah kepemilikan (*belongs to*) *User* dan ditautkan (disambungkan) ke repositori ini.
5. Untuk inisialisasi Node proyek utama:
   a. Gandakan (copy) folder `nodes/_template/` menjadi `nodes/[nama-proyek]/`. Setelah itu, WAJIB tuliskan Mode Operasional pilihan pengguna (MODE 1 atau MODE 2) ke bagian paling atas file `nodes/[nama-proyek]/main.md` untuk menggantikan placeholder `[PILIH: MODE 1 / MODE 2]`.
   b. Pindai (scan) source code asli dari Path Codebase yang diberikan untuk menganalisis pola arsitektur, legacy code, dan pustaka eksisting.
   c. Tuliskan hasil pindai dan batasan spesifik proyek ke dalam `nodes/[nama-proyek]/guidelines/project-context.md`.
   d. Buat `nodes/[nama-proyek]/docs/system-design.md`.
   e. Berdasarkan tipe aplikasi (Backend, Web, Mobile, Game), rancang seluruh arsitektur menggunakan PlantUML (ERD, Flowchart, State Diagram, User Journey, Use Case). Simpan kode arsitektur tersebut sebagai file-file `.puml` terpisah secara eksplisit di dalam folder `docs/diagrams/` (untuk Node) atau `global-docs/diagrams/` (untuk Global), lalu tautkan (link) file tersebut ke dalam `prd.md` dan `system-design.md` sesuai pedoman.
   f. Buat `nodes/[nama-proyek]/docs/development-planning.md` yang merancang daftar backlog tiket (TICKET-XX.md) yang harus dikerjakan di fase pertama.
   g. Tuliskan entri log inisialisasi awal ke dalam file `nodes/[nama-proyek]/CHANGELOG.md` menggunakan templat dari `global-docs/templates/changelog_entry_template.md` yang mencatat tanggal, status pembuatan node, dan ringkasan arsitektur dasar yang baru saja ditetapkan.
   h. Sinkronisasi Graf (KONDISIONAL): JIKA pengguna menyetujui penggunaan Graphify di tahap awal, masuk ke dalam direktori *Path Codebase* dan jalankan perintah `graphify build` di terminal untuk membangun Knowledge Graph perdana. DILARANG KERAS menjalankan `graphify build` atau membuat direktori `.graphify` di dalam repositori orchestrator (`ai-orchestrator-template/`).

Setelah seluruh dokumen mandatory terlengkapi dan fase di atas selesai sempurna, berikan saya rangkuman singkat terkait struktur baru yang terbentuk dan tanyakan persetujuan saya sebelum kita masuk ke mode eksekusi tiket harian!

CATATAN PENTING UNTUK AI: JANGAN menghapus folder `nodes/_template/` setelah Anda menggandakannya. Folder tersebut harus tetap utuh dan tidak boleh disentuh sebagai cetak biru jika di masa depan kita beralih ke multi-proyek!
```

#### Opsi B: Startup Prompt (Multi-Project Environment)
Gunakan prompt ini jika ekosistem Anda terdiri dari beberapa aplikasi/node yang terpisah (misal: Frontend terpisah dari Backend). Salin, isi bagian [ DALAM KURUNG SIKU ], dan kirimkan utuh ke AI Agent:

```text
Saya ingin menginisialisasi Multi-Project Environment menggunakan kerangka kerja AI Orchestrator ini.

Berikut adalah definisi Ekosistem (Environment) dan Sub-Proyek (Nodes) yang akan diatur:

A. ENVIRONMENT (Global Scope)
1. Nama Sistem: [Nama sistem keseluruhan, misal: Smart E-Commerce Platform]
2. Ide/Konsep Utama: [Jelaskan fitur utama dari sistem ini]
3. Vibe/Estetika UI: [Misal: Modern, dominasi biru tua, minimalis]
4. Batasan Sistem Utama: [Misal: Harus secure, GDPR compliant]

B. NODES (Sub-Project Scope)
- Node 1: Frontend App
  - Path Codebase: [Path absolut ke folder frontend, misal: `/Users/.../my-frontend`]
  - Tech Stack: [Misal: Next.js, TailwindCSS]
- Node 2: Backend API
  - Path Codebase: [Path absolut ke folder backend, misal: `/Users/.../my-backend`]
  - Tech Stack: [Misal: NestJS, PostgreSQL]
(Tambahkan node lain jika ada)

INSTRUKSI STARTUP ANDA:
Berbekal informasi di atas, JANGAN MENULIS KODE APLIKASI SAMA SEKALI. Lakukan langkah-langkah otonom berikut secara berurutan:
1. Pahami struktur `ai-orchestrator-template` yang berbasis nodes ini.
2. Evaluasi & Wawancara Pengguna: Jika deskripsi yang saya berikan di atas masih terlalu dangkal atau belum cukup untuk mengisi Dokumen Mandatory secara detail dan maksimal, Anda WAJIB BERHENTI mengeksekusi langkah selanjutnya. (Catatan: Yang tergolong Dokumen Mandatory adalah: `prd.md`, `design-system.md`, `system-design.md`, dan `development-planning.md`). Ajukan daftar pertanyaan kritis kepada saya terkait visi, batasan teknis, target pengguna, dan spesifikasi fungsionalitas. Selain itu, SEBELUM Anda bertanya tentang Graphify, Anda WAJIB menanyakan: *"Mode Operasional mana yang ingin Anda gunakan secara default? (1) Mode 1: Autonomous Planning (digerakkan oleh roadmap), atau (2) Mode 2: Prompt-Driven (digerakkan instruksi mikro)"*. SETELAH ITU, tanyakan: *"Apakah Anda ingin menggunakan fitur CLI Graphify untuk pemetaan arsitektur otomatis? (Ya/Tidak)"*. SELANJUTNYA, tanyakan preferensi alat pengujian (*testing*) dan *linter* yang ingin digunakan untuk masing-masing Node. JIKA Node berupa *web* atau *mobile app*, tanyakan juga opsi fitur aksesibilitas (*accessibility*) apa saja yang ingin diimplementasikan (opsional). Ulangi proses tanya-jawab ini hingga Anda memiliki konteks yang solid.
2.5. Pengecekan Environment (KONDISIONAL): JIKA pengguna menjawab YA untuk pemakaian Graphify pada tahap wawancara, eksekusi `graphify --version` di terminal. Jika gagal (not found), instal segera dengan `npm install -g @sentropic/graphify`. Jika pengguna menjawab TIDAK, lewati langkah ini sepenuhnya.
3. Setelah informasi dirasa memadai, buatkan draf komprehensif untuk `global-docs/prd.md` dan `global-docs/design-system.md` berdasarkan spesifikasi lingkungan (Environment) di atas.
4. Buatkan GitHub Project di awal untuk repositori ini. Pastikan GitHub Project tersebut dibuat di bawah kepemilikan (*belongs to*) *User* dan ditautkan (disambungkan) ke repositori ini.
5. Untuk SETIAP Node yang terdaftar di atas:
   a. Gandakan (copy) folder `nodes/_template/` menjadi `nodes/[nama-node]/`. Setelah itu, WAJIB tuliskan Mode Operasional pilihan pengguna (MODE 1 atau MODE 2) ke bagian paling atas file `nodes/[nama-node]/main.md` untuk menggantikan placeholder `[PILIH: MODE 1 / MODE 2]`.
   b. Pindai (scan) source code asli dari Node tersebut di Path Codebase yang diberikan untuk menganalisis pola arsitektur, legacy code, dan pustaka eksisting.
   c. Tuliskan hasil pindai dan batasan spesifik node tersebut ke dalam `nodes/[nama-node]/guidelines/project-context.md`.
   d. Buat `nodes/[nama-node]/docs/system-design.md` khusus untuk node tersebut.
   e. Berdasarkan tipe aplikasi (Backend, Web, Mobile, Game), rancang seluruh arsitektur menggunakan PlantUML (ERD, Flowchart, State Diagram, User Journey, Use Case). Simpan kode arsitektur tersebut sebagai file-file `.puml` terpisah secara eksplisit di dalam folder `docs/diagrams/` spesifik milik node tersebut, lalu tautkan (link) file tersebut ke dalam prd.md and system-design.md sesuai pedoman.
   f. Buat `nodes/[nama-node]/docs/development-planning.md` yang merancang daftar backlog tiket (TICKET-XX.md) yang harus dikerjakan di fase pertama node ini.
   g. Tuliskan entri log inisialisasi awal ke dalam file `nodes/[nama-node]/CHANGELOG.md` menggunakan templat dari `global-docs/templates/changelog_entry_template.md` yang mencatat tanggal, status pembuatan node, dan ringkasan arsitektur dasar yang baru saja ditetapkan.
   h. Sinkronisasi Graf (KONDISIONAL): JIKA pengguna menyetujui penggunaan Graphify di tahap awal, setelah selesai memproses sebuah Node, masuk ke dalam direktori *Path Codebase* dari node tersebut dan jalankan perintah `graphify build` di terminal untuk membangun Knowledge Graph lokal node tersebut. DILARANG KERAS menjalankan `graphify build` atau membuat direktori `.graphify` di dalam repositori orchestrator (`ai-orchestrator-template/`).

Setelah seluruh dokumen mandatory terlengkapi dan fase di atas selesai sempurna, berikan saya rangkuman singkat terkait struktur baru yang terbentuk dan tanyakan persetujuan saya sebelum kita masuk ke mode eksekusi tiket harian!

CATATAN PENTING UNTUK AI: JANGAN menghapus folder `nodes/_template/` setelah Anda menggandakannya. Folder tersebut harus tetap utuh dan tidak boleh disentuh sebagai cetak biru untuk penambahan node baru di masa depan!
```

### FASE 2: Scaling Prompt (Penambahan Node)
Gunakan *prompt* ini jika Anda ingin menambahkan aplikasi/node baru ke dalam ekosistem proyek yang sudah berjalan. *Prompt* ini akan secara otomatis menangani transisi dokumen jika proyek Anda sebelumnya didefinisikan sebagai *Single-Project*.

Salin, isi bagian `[ DALAM KURUNG SIKU ]`, dan kirimkan utuh ke AI Agent:

```text
Saya ingin menambahkan Node baru ke dalam ekosistem AI Orchestrator ini.

Berikut adalah definisi Node baru yang akan ditambahkan:
1. Nama Node: [Nama node baru, misal: Mobile App]
2. Ide/Konsep Utama: [Jelaskan fungsionalitas utama node ini]
3. Path Codebase: [Path absolut ke folder node baru, misal: `/Users/.../my-mobile-app`]
4. Tech Stack: [Misal: Flutter, Firebase]

INSTRUKSI PENAMBAHAN NODE ANDA:
Berbekal informasi di atas, JANGAN MENULIS KODE APLIKASI SAMA SEKALI. Lakukan langkah-langkah otonom berikut secara berurutan:
0. Pengecekan Detektif Lingkungan: Cek apakah terdapat folder tersembunyi `.graphify` di dalam *Path Codebase* node eksisting. JIKA ADA, itu artinya ekosistem ini menggunakan fitur Graphify. Eksekusi `graphify --version` di terminal. Jika gagal, instal dengan `npm install -g @sentropic/graphify`. Jika folder `.graphify` TIDAK ADA, abaikan langkah ini sepenuhnya.
1. Pahami struktur `ai-orchestrator-template` eksisting. Periksa dokumen `global-docs/prd.md` dan diagram arsitektur global.
2. Deteksi Lingkungan & Pembaruan Dokumen Global (SANGAT KRUSIAL):
   - JIKA ekosistem sebelumnya adalah Single-Project: Anda WAJIB SECARA MUTLAK merestrukturisasi dan merombak seluruh dokumen global (`global-docs/prd.md`, `global-docs/design-system.md`, dan diagram arsitektur global) dari format aplikasi tunggal menjadi hierarki Multi-Project/Ecosystem. Pisahkan antara *Global Scope* dan *Node-Specific Scope*.
   - WAJIB INTEGRASI DOKUMEN: Terlepas dari status sistem sebelumnya, Anda WAJIB memperbarui `global-docs/prd.md` dan `global-docs/design-system.md` untuk secara eksplisit mendaftarkan fitur, kebutuhan fungsional, dan komponen antarmuka dari Node baru ini ke dalam dokumen global yang sudah ada. Jangan sampai Node baru ini tidak terdokumentasi di tingkat ekosistem!
3. Wawancara Pengguna: Jika informasi Node baru di atas masih kurang jelas atau akan berdampak besar pada arsitektur global, ajukan pertanyaan kritis mengenai batasan teknis dan interaksinya dengan Node eksisting. Selain itu, WAJIB tanyakan preferensi alat pengujian (*testing*) dan *linter* yang ingin digunakan untuk Node baru ini. JIKA Node berupa *web* atau *mobile app*, tanyakan juga opsi fitur aksesibilitas (*accessibility*) apa saja yang ingin diimplementasikan (opsional). Anda harus menyelesaikan wawancara ini sebelum mengeksekusi pembuatan dokumen.
4. Inisialisasi Node Baru:
   a. Gandakan (copy) folder `nodes/_template/` menjadi `nodes/[nama-node-baru]/`. Pastikan menyalin Mode Operasional yang sedang digunakan ekosistem ini dan menuliskannya di baris pertama file `nodes/[nama-node-baru]/main.md`.
   b. Pindai (scan) source code asli dari Path Codebase yang diberikan untuk menganalisis pola arsitektur, legacy code, dan pustaka eksisting.
   c. Tuliskan hasil pindai dan batasan spesifik node tersebut ke dalam `nodes/[nama-node-baru]/guidelines/project-context.md`.
   d. Buat `nodes/[nama-node-baru]/docs/system-design.md`.
   e. Rancang seluruh arsitektur node baru menggunakan PlantUML secara eksplisit di dalam folder `nodes/[nama-node-baru]/docs/diagrams/`, lalu tautkan file tersebut ke dalam dokumen yang relevan.
   f. Buat `nodes/[nama-node-baru]/docs/development-planning.md` untuk backlog tiket node baru ini.
   g. Tuliskan entri log inisialisasi awal ke dalam file `nodes/[nama-node-baru]/CHANGELOG.md` menggunakan templat `global-docs/templates/changelog_entry_template.md`.
   h. Sinkronisasi Graf (KONDISIONAL): JIKA ekosistem ini terdeteksi menggunakan Graphify (dari langkah 0), masuk ke dalam direktori *Path Codebase* node baru tersebut dan jalankan perintah `graphify build` di terminal untuk membangun Knowledge Graph. DILARANG KERAS menjalankan perintah ini atau membuat direktori `.graphify` di dalam direktori orchestrator (`ai-orchestrator-template/`).

Setelah penambahan Node selesai, berikan saya rangkuman arsitektur ekosistem terbaru dan tanyakan persetujuan saya sebelum kita masuk ke mode eksekusi tiket harian!
```

### FASE 3: Execution Prompt (Pengerjaan Tugas)
Gunakan salah satu dari dua Execution Prompt di bawah ini sesuai dengan ruang lingkup tugas yang ingin Anda kerjakan.

#### Opsi A: Execution Prompt (Single-Node)
Gunakan prompt ini jika Anda hanya ingin fokus mengerjakan fitur di SATU proyek spesifik (misalnya hanya mengubah UI Frontend).

```text
Kamu WAJIB membaca `nodes/[NAMA_NODE_ANDA]/main.md` sebagai Master Entrypoint. Patuhi seluruh pedoman arsitektur dan larangan mutlak yang tertulis di dalamnya.

KEWAJIBAN PENGGUNAAN & GENERATE GRAPHIFY:
Cek apakah terdapat direktori `.graphify` di dalam *Path Codebase* proyek ini:
- JIKA DITEMUKAN: Kamu WAJIB menggunakan fitur CLI `graphify` (seperti `graphify query`) di dalam *Path Codebase* untuk memahami arsitektur, melacak fungsi pemanggil (*callers*), menelusuri dependensi, dan memetakan file terdampak alih-alih membaca puluhan file secara manual. Wajib gunakan data graf ini saat menyusun Implementation Plan dan selama memodifikasi kode.
- JIKA TIDAK DITEMUKAN: Kamu WAJIB men-generate-nya terlebih dahulu! Masuk ke direktori *Path Codebase* (`cd`), pastikan CLI `graphify` terpasang (instal via `npm install -g @sentropic/graphify` jika belum ada), dan jalankan `graphify build` untuk membangun Knowledge Graph. Setelah selesai di-generate, kamu WAJIB memakainya untuk menavigasi kode pada tugas ini.
Ingat: direktori `.graphify` HANYA boleh berada di dalam *Path Codebase* dan DILARANG KERAS dibuat di dalam repositori orchestrator!

SEBELUM menulis kode, evaluasi apakah tugas ini menuntut konteks atau domain fitur yang berbeda (contoh: fitur baru, *hotfix*, *testing*). Jika berbeda, WAJIB tanyakan kepada saya untuk membuat *branch* baru.

Setelah kodemu berhasil dan tugas ini rampung, kamu WAJIB masuk ke *Path Codebase* (`cd`) dan eksekusi perintah `graphify update` untuk menyinkronkan konteks kode barumu.

Instruksi Tugas: [TULIS_INSTRUKSI_ATAU_ID_TIKET_DI_SINI]
```

#### Opsi B: Execution Prompt (Multi-Node / Lintas Proyek)
Gunakan *prompt* ini jika Anda memiliki tugas integrasi besar yang melibatkan banyak proyek sekaligus (misalnya menyambungkan API Backend ke Frontend).

```text
Tugas ini bersifat lintas-proyek (Multi-Node).
Pengecekan & Kewajiban Graphify: Cek apakah ada direktori `.graphify` di *Path Codebase* dari node yang dituju:
- JIKA DITEMUKAN: Eksekusi `graphify --version` (instal via npm jika belum ada), lalu WAJIB masuk ke direktori tersebut (`cd`) dan gunakan CLI `graphify` (seperti `graphify query`) untuk memetakan alur pemanggil, dependensi lintas-modul, serta mengidentifikasi komponen terdampak, dan baca `main.md` spesifik dari node yang teridentifikasi. Wajib gunakan kueri Graphify ini saat menyusun rencana dan menulis kode integrasi.
- JIKA TIDAK DITEMUKAN: Masuk ke *Path Codebase* node tersebut (`cd`) dan WAJIB men-generate-nya terlebih dahulu dengan menjalankan `graphify build`, lalu wajib gunakan hasilnya untuk analisis tugas ini.
Ingat: direktori `.graphify` HANYA boleh berada di dalam *Path Codebase* node terkait dan DILARANG KERAS dibuat di dalam repositori orchestrator!
SEBELUM menulis kode, evaluasi apakah tugas ini butuh *branch* baru (beda konteks/fitur) dan WAJIB tanyakan kepada saya persetujuannya. Pastikan integrasi antarsistem mematuhi pedoman global. Setelah tugas selesai, masuk ke *Path Codebase* masing-masing node dan eksekusi `graphify update`.

Instruksi Tugas: [TULIS_INSTRUKSI_LINTAS_NODE_DI_SINI]
```

---

### FASE 4: Implementation Q&A Prompt (Tanya Jawab Implementasi & Knowledge Base)
Berbeda dengan *Execution Prompt* yang bertujuan untuk memodifikasi kode atau menyelesaikan tiket tugas, **Implementation Q&A Prompt** dirancang khusus saat Anda ingin mengajukan pertanyaan eksplisit mengenai implementasi teknis, keputusan arsitektur, atau alur logika sistem yang telah dibangun.

**Karakteristik & Mekanisme Kerja:**
1. **Hanya Menjawab Pertanyaan Eksplisit:** AI Agent **TIDAK AKAN** membuat penjelasan otomatis yang tidak diminta. Agen hanya akan menganalisis dan menjawab daftar pertanyaan yang secara eksplisit Anda berikan (contoh: *1. Pertanyaan A, 2. Pertanyaan B*).
2. **Grounding Nyata Berbasis Codebase:** Setiap jawaban didasarkan langsung pada analisis kode sumber (*source code*) aktual dan dokumen referensi orchestrator, lengkap dengan rujukan file dan penjelasannya (bebas dari halusinasi).
3. **Dual Output & Auto-Archive:** AI Agent akan mengetikkan jawaban terstruktur langsung di **chat sidebar** percakapan **DAN** secara otomatis mencatat, mengkategorisasikan, serta menyimpannya ke dalam file `global-docs/LEARN.md` sebagai *Knowledge Base* permanen agar mudah dibaca dan dicari di masa depan.

Salin, isi bagian `[ DALAM KURUNG SIKU ]`, dan kirimkan ke AI Agent:

```text
Saya memiliki beberapa pertanyaan spesifik terkait implementasi pada proyek ini.

Cakupan / Node: [Global / Nama Node spesifik, misal: Frontend / Backend / Path Codebase]

Daftar Pertanyaan Eksplisit:
1. [Tulis pertanyaan 1 di sini, misal: Bagaimana alur autentikasi JWT diimplementasikan antara frontend dan backend?]
2. [Tulis pertanyaan 2 di sini, misal: Mengapa memilih Zustand dibandingkan Redux untuk state management di node ini?]
(Tambahkan pertanyaan lain jika ada)

INSTRUKSI AI AGENT:
1. JANGAN memodifikasi kode aplikasi atau mengambil inisiatif tugas koding baru dalam sesi ini.
2. BACA `global-docs/LEARN.md` untuk memahami protokol Q&A, format boilerplate, dan taksonomi kategori.
3. Jawab HANYA pertanyaan-pertanyaan yang saya ajukan di atas secara eksplisit.
4. Dasarkan seluruh jawaban Anda pada analisis nyata terhadap codebase dan dokumen orchestrator terkait (sertakan path file dan baris kode sebagai referensi konkret).
5. Berikan jawaban komprehensif, terstruktur, dan mudah dipahami secara langsung di chat sidebar.
6. Simpan rekaman Tanya-Jawab ini secara utuh ke dalam `global-docs/LEARN.md` sesuai format entri baku, kategorisasikan dengan tepat (Kategori Utama & Tags), dan perbarui Indeks Kategori di file tersebut.
```

---

Dengan *prompt* terstruktur di atas, ekosistem pengembangan Anda tidak hanya menghasilkan kode yang disiplin, tetapi juga membangun *Knowledge Base* yang terdokumentasi rapi seiring berjalannya proyek.
