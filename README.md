# AI Orchestrator Template

> **[PRINSIP UTAMA]** Kerangka kerja rekayasa perangkat lunak terstruktur bagi AI Agent untuk mencegah halusinasi, mengeliminasi galat berulang (*looping error*), menegakkan standar *Anti-AI-Slop*, dan mematuhi dokumentasi tunggal (*Single Source of Truth*).

---

## 1. Alur Interaksi & Empat Fase Prompt

Pengembangan dengan AI Orchestrator terbagi menjadi 4 fase prompt:
1. **FASE 1: Startup Prompt** (Inisialisasi lingkungan proyek tunggal atau multi-proyek).
2. **FASE 2: Scaling Prompt** (Penambahan node/aplikasi baru ke ekosistem eksisting).
3. **FASE 3: Execution Prompt** (Eksekusi tugas koding pada single-node atau multi-node).
4. **FASE 4: Implementation Q&A Prompt** (Tanya-jawab implementasi dan dokumentasi *Knowledge Base*).

---

## 2. Katalog Prompt Baku

### FASE 1: Startup Prompt (Inisialisasi Proyek)
Gunakan salah satu opsi berikut satu kali di awal inisiasi proyek. Salin, isi bagian `[ DALAM KURUNG SIKU ]`, dan kirimkan ke AI Agent:

#### Opsi A: Startup Prompt (Single-Project Environment)
```text
Saya ingin menginisialisasi Single-Project Environment menggunakan kerangka kerja AI Orchestrator ini.

DEFINISI PROYEK:
1. Nama Sistem: [Nama aplikasi, misal: Personal Portfolio Website]
2. Ide/Konsep Utama: [Fitur utama sistem]
3. Vibe/Estetika UI: [Misal: Modern, minimalis, palet netral]
4. Batasan Sistem: [Misal: Responsif, SEO-friendly, performa tinggi]
5. Path Codebase: [Path absolut ke direktori proyek lokal, misal: `/Users/.../my-app`]
6. Tech Stack: [Misal: Next.js, TypeScript, TailwindCSS]

INSTRUKSI STARTUP (JANGAN TULIS KODE APLIKASI):
1. Pahami struktur `ai-orchestrator-template`. Alokasikan satu Node utama untuk proyek ini.
2. Wawancara Mandatori: Jika deskripsi di atas belum cukup untuk menyusun Dokumen Mandatory (`prd.md`, `design-system.md`, `system-design.md`, `development-planning.md`), WAJIB BERHENTI dan tanyakan secara terurut:
   a. Visi, target pengguna, dan batasan fungsional teknis.
   b. "Mode Operasional mana yang dipilih default? (1) Mode 1: Autonomous Planning (digerakkan roadmap), atau (2) Mode 2: Prompt-Driven (digerakkan instruksi mikro)".
   c. "Apakah ingin menggunakan fitur CLI Graphify untuk pemetaan arsitektur otomatis? (Ya/Tidak)".
   d. Preferensi testing framework dan linter sesuai teknologi proyek.
   e. Opsi fitur aksesibilitas yang diinginkan jika proyek berupa web/mobile app.
2.5. Pengecekan Environment (Kondisional): Jika pengguna memilih YA untuk Graphify, jalankan `graphify --version`. Jika belum terpasang, instal via `npm install -g @sentropic/graphify`. Jika TIDAK, lewati.
3. Susun draf komprehensif `global-docs/prd.md` dan `global-docs/design-system.md` berakar pada tugas nyata bebas estetika klise AI (*Anti-AI-Slop*).
4. Inisialisasi GitHub Project di bawah kepemilikan User dan tautkan ke repositori ini.
5. Inisialisasi Node Proyek:
   a. Gandakan `nodes/_template/` menjadi `nodes/[nama-proyek]/`. Tuliskan Mode Operasional terpilih di baris pertama `nodes/[nama-proyek]/main.md`. (DILARANG menghapus folder `nodes/_template/`!).
   b. Pindai source code asli pada Path Codebase untuk memetakan arsitektur dan dependensi.
   c. Catat hasil pindai dan batasan lokal ke `nodes/[nama-proyek]/guidelines/project-context.md`.
   d. Susun `nodes/[nama-proyek]/docs/system-design.md`.
   e. Rancang diagram arsitektur PlantUML (ERD, Flowchart, State Diagram, User Journey) sebagai file terpisah `.puml` di `docs/diagrams/` atau `global-docs/diagrams/`, lalu tautkan ke dokumen terkait.
   f. Susun `nodes/[nama-proyek]/docs/development-planning.md` berisi backlog pasangan tiket & rencana implementasi fase pertama.
   g. Catat log inisialisasi perdana ke `nodes/[nama-proyek]/CHANGELOG.md` menggunakan format baku.
   h. Sinkronisasi Graf (Kondisional): Jika Graphify aktif, masuk ke Path Codebase (`cd`) dan jalankan `graphify build`. (DILARANG menjalankan `graphify build` di repositori orchestrator).
Setelah selesai, berikan ringkasan struktur yang terbentuk dan tunggu persetujuan sebelum mulai eksekusi pasangan tiket & rencana!
```

#### Opsi B: Startup Prompt (Multi-Project Environment)
```text
Saya ingin menginisialisasi Multi-Project Environment menggunakan kerangka kerja AI Orchestrator ini.

A. ENVIRONMENT (Global Scope)
1. Nama Sistem: [Nama sistem keseluruhan, misal: Smart E-Commerce Platform]
2. Ide/Konsep Utama: [Fitur utama ekosistem]
3. Vibe/Estetika UI: [Misal: Modern, basis netral, satu aksen fungsional]
4. Batasan Sistem: [Misal: Secure, GDPR compliant, arsitektur modular]

B. NODES (Sub-Project Scope)
- Node 1: Frontend App
  - Path Codebase: [Path absolut ke folder frontend, misal: `/Users/.../my-frontend`]
  - Tech Stack: [Misal: Next.js, TailwindCSS]
- Node 2: Backend API
  - Path Codebase: [Path absolut ke folder backend, misal: `/Users/.../my-backend`]
  - Tech Stack: [Misal: NestJS, PostgreSQL]
(Tambahkan node lain bila ada)

INSTRUKSI STARTUP (JANGAN TULIS KODE APLIKASI):
1. Pahami struktur `ai-orchestrator-template` berbasis multi-node.
2. Wawancara Mandatori: Tanyakan klarifikasi visi global, Mode Operasional default (Mode 1 / Mode 2), persetujuan CLI Graphify, preferensi testing/linter tiap node, dan fitur aksesibilitas jika web/mobile.
2.5. Pengecekan Environment: Jika Graphify disetujui, verifikasi dan instal `graphify` jika belum ada.
3. Susun draf `global-docs/prd.md` dan `global-docs/design-system.md` berstandar Anti-Slop (pisahkan Global Scope vs Node-Specific Scope).
4. Inisialisasi GitHub Project dan tautkan ke repositori ini.
5. Untuk SETIAP Node yang terdaftar:
   a. Gandakan `nodes/_template/` menjadi `nodes/[nama-node]/`. Cantumkan Mode terpilih pada baris pertama `main.md`. (DILARANG menghapus `nodes/_template/`!).
   b. Pindai source code di Path Codebase lokal node tersebut.
   c. Tulis hasil analisis ke `nodes/[nama-node]/guidelines/project-context.md`.
   d. Susun `nodes/[nama-node]/docs/system-design.md`.
   e. Rancang diagram PlantUML terpisah di `docs/diagrams/` spesifik milik node dan tautkan ke dokumen.
   f. Susun `nodes/[nama-node]/docs/development-planning.md` untuk backlog pasangan tiket & rencana node.
   g. Catat log inisialisasi ke `nodes/[nama-node]/CHANGELOG.md`.
   h. Sinkronisasi Graf (Kondisional): Jika Graphify aktif, jalankan `graphify build` di Path Codebase node terkait. (DILARANG di repositori orchestrator).
Setelah seluruh node terkonfigurasi, berikan rangkuman arsitektur dan tunggu persetujuan sebelum eksekusi pasangan tiket & rencana!
```

---

### FASE 2: Scaling Prompt (Penambahan Node)
Gunakan prompt ini saat menambahkan aplikasi/node baru ke dalam ekosistem yang sedang berjalan:

```text
Saya ingin menambahkan Node baru ke dalam ekosistem AI Orchestrator ini.

DEFINISI NODE BARU:
1. Nama Node: [Nama node baru, misal: Mobile App]
2. Ide/Konsep Utama: [Fungsionalitas utama node ini]
3. Path Codebase: [Path absolut ke folder node baru, misal: `/Users/.../my-mobile-app`]
4. Tech Stack: [Misal: Flutter, Firebase]

INSTRUKSI PENAMBAHAN NODE (JANGAN TULIS KODE APLIKASI):
0. Deteksi Graphify: Cek apakah folder `.graphify` ada di Path Codebase node eksisting. Jika ada, pastikan CLI `graphify` terpasang di terminal.
1. Pahami struktur orchestrator eksisting dan periksa dokumen `global-docs/prd.md`.
2. Pembaruan Dokumen Global:
   - Jika ekosistem sebelumnya Single-Project: Rombak `global-docs/prd.md`, `global-docs/design-system.md`, dan diagram global menjadi hierarki Multi-Project (Global Scope vs Node-Specific Scope).
   - Daftarkan fitur, kebutuhan fungsional, dan komponen Node baru ke dokumen global tersebut.
3. Wawancara Pengguna: Ajukan pertanyaan kritis terkait integrasi antarnode, preferensi testing/linter, dan aksesibilitas untuk Node baru.
4. Inisialisasi Node Baru:
   a. Gandakan `nodes/_template/` menjadi `nodes/[nama-node-baru]/`. Tuliskan Mode Operasional ekosistem di baris pertama `main.md`.
   b. Pindai source code pada Path Codebase untuk analisis arsitektur.
   c. Catat hasil analisis ke `nodes/[nama-node-baru]/guidelines/project-context.md`.
   d. Susun `nodes/[nama-node-baru]/docs/system-design.md`.
   e. Buat diagram PlantUML di `nodes/[nama-node-baru]/docs/diagrams/` dan tautkan ke dokumen.
   f. Buat `nodes/[nama-node-baru]/docs/development-planning.md` untuk backlog pasangan tiket & rencana baru.
   g. Catat log inisialisasi ke `nodes/[nama-node-baru]/CHANGELOG.md`.
   h. Sinkronisasi Graf: Jika Graphify aktif, jalankan `graphify build` di Path Codebase node baru. (DILARANG di direktori orchestrator).
Berikan rangkuman arsitektur ekosistem terbaru dan tunggu konfirmasi sebelum melanjutkan eksekusi!
```

---

### FASE 3: Execution Prompt (Pengerjaan Tugas)
Gunakan salah satu opsi berikut sesuai cakupan tugas yang ingin dikerjakan:

#### Opsi A: Execution Prompt (Single-Node)
```text
Kamu WAJIB membaca `nodes/[NAMA_NODE_ANDA]/main.md` sebagai Master Entrypoint. Patuhi seluruh pedoman arsitektur, larangan mutlak, serta standar Anti-AI-Slop di dalamnya.

NAVIGASI & PENGGUNAAN GRAPHIFY:
Cek direktori `.graphify` di Path Codebase proyek ini:
- JIKA ADA: Wajib gunakan CLI `graphify query` di Path Codebase untuk menelusuri pemanggil, dependensi, dan file terdampak saat menyusun Implementation Plan dan selama modifikasi kode.
- JIKA BELUM ADA: Masuk ke Path Codebase (`cd`), pastikan CLI terpasang, dan jalankan `graphify build`. Wajib gunakan hasilnya untuk navigasi tugas ini.
(Dilarang membuat folder `.graphify` di dalam direktori orchestrator!).

DISIPLIN EKSEKUSI:
1. Evaluasi kebutuhan branch baru jika konteks berbeda (fitur baru/hotfix) dan minta persetujuan pengguna.
2. Siapkan dan verifikasi pasangan tiket & rencana implementasi (`TICKET-[EPOCH_TIMESTAMP]` di `tickets/` dan `PLAN-[EPOCH_TIMESTAMP]` di `plans/`) serta tunggu persetujuan pengguna sebelum mutasi state.
3. Seluruh referensi berkas di dalam tiket dan rencana implementasi WAJIB menggunakan relative path (DILARANG absolute path).
4. Terapkan TDD Workflow (wajib buktikan tes baru gagal di versi lama sebelum fix jika perbaikan bug - Disiplin O-07).
5. Jalankan minimal 1 putaran Hunt Loop (sebutkan >=10 kasus ekstrem berbasis `edge-case-atlas.md`, reproduksi, dan pastikan nol temuan di putaran akhir).
6. Loloskan evaluasi Delivery Gate (`delivery-gate.md`) sebelum menandai tiket & rencana selesai (`Done` / `Completed`).
7. Masuk ke Path Codebase (`cd`) dan jalankan `graphify update` setelah selesai.
8. Pesan komit Git dan deskripsi PR WAJIB 100% bersih dari istilah orchestrator atau ID internal (tiket, rencana, dsb.)!

Instruksi Tugas: [TULIS_INSTRUKSI_ATAU_PASANGAN_TIKET_DAN_RENCANA_DI_SINI]
```

#### Opsi B: Execution Prompt (Multi-Node / Lintas Proyek)
```text
Tugas ini bersifat lintas-proyek (Multi-Node).
Periksa ketersediaan `.graphify` di Path Codebase masing-masing node terkait. Gunakan `graphify query` untuk memetakan alur integrasi dan dependensi antarmodul, atau jalankan `graphify build` jika belum ada. Baca `main.md` dari setiap node terkait.

DISIPLIN INTEGRASI:
1. Evaluasi branch baru dan minta konfirmasi persetujuan pengguna sebelum koding.
2. Susun pasangan tiket & rencana implementasi lintas-node (di tickets/ dan plans/ masing-masing node) dan tunggu persetujuan sebelum mutasi state.
3. Seluruh referensi dokumen WAJIB menggunakan relative path (DILARANG absolute path).
4. Jalankan TDD Workflow, minimal 1 putaran Hunt Loop, dan evaluasi kelulusan Delivery Gate (`delivery-gate.md`) sebelum menyerahkan hasil.
5. Jalankan `graphify update` di Path Codebase masing-masing node setelah selesai.
6. Pesan komit Git dan deskripsi PR WAJIB 100% bersih dari istilah orchestrator atau ID internal (tiket, rencana, dsb.)!

Instruksi Tugas: [TULIS_INSTRUKSI_LINTAS_NODE_DI_SINI]
```

---

### FASE 4: Implementation Q&A Prompt (Knowledge Base)
Gunakan prompt ini khusus saat mengajukan pertanyaan teknis seputar arsitektur atau implementasi kode tanpa mengubah kode aplikasi:

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
3. Jawab HANYA pertanyaan yang secara eksplisit tertulis di atas.
4. Dasarkan seluruh jawaban pada analisis nyata basis kode dan dokumen orchestrator (sertakan path file dan nomor baris).
5. Berikan jawaban komprehensif langsung pada chat sidebar.
6. Simpan rekaman Q&A ini ke `global-docs/LEARN.md` sesuai format baku, tentukan Kategori Utama & Tags, serta perbarui Indeks Kategori di berkas tersebut.
```

---

## 3. Penghargaan & Kredit (Credits & Acknowledgments)

- https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md
- https://github.com/altamisatmaja/security-audit-skill
- https://github.com/altamisatmaja/outofthebox
