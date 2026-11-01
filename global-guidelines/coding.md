# Code Quality & Formatting Standard

## 1. Standar Sintaksis, Bahasa & Format Kode (Syntax & Source Hygiene)

### 1.1 ZERO-COMMENT POLICY
Dilarang menambahkan komentar apapun di dalam source code baru. Kode wajib *self-documenting*. Penjelasan logika atau alur wajib ditulis secara naratif di luar blok kode (pada teks Markdown).

#### Cakupan Larangan
Larangan mencakup seluruh bentuk anotasi teks non-fungsional di semua bahasa dan framework:
- **Komentar JSX/TSX:** `{/* ini komponen header */}`, `{/* TODO: refactor state */}`
- **Komentar HTML/Vue/Svelte:** `<!-- bagian sidebar -->`, `<!-- HACK: z-index fix -->`
- **Inline comment (JS/TS/Go/Dart/Java/C/PHP dll):** `// ini fungsi login`, `// validasi regex`
- **Inline comment (Python/Ruby/Bash/YAML):** `# ambil data user`, `# setup environment`
- **Block comment (CSS/SCSS/JS/TS/SQL dll):** `/* background khusus dark mode */`, `/* index tabel users */`
- **Multi-line Docstring (Python):** `""" kelas ini menangani autentikasi pengguna """`
- **Trailing comment:** `const x = 10; // jumlah maksimal`, `margin-top: 10px; /* spacing */`
- **Penanda sementara (Universal):** `TODO`, `FIXME`, `HACK`, `NOTE` dalam format apapun.
- **Docstring naratif:** Penjelasan fungsi yang bersifat narasi atau tutorial (misal: `/// Fungsi ini digunakan untuk...`).

#### Pengecualian yang Diizinkan
Anotasi fungsional yang dibutuhkan compiler atau linter (`@override`, `@throws`, tipe return, `@param {string}` tanpa narasi tutorial) diizinkan karena bersifat fungsional.

#### Kewajiban Self-Check
Sebelum menyerahkan kode:
1. Pindai setiap baris kode baru/modifikasi. Hapus seluruh komentar yang ditemukan.
2. Jangan ubah atau hapus komentar bawaan pada kode lama (aturan hanya berlaku untuk kode baru).
3. Kode baru yang mengandung komentar dianggap gagal eksekusi (setara compile error).

### 1.2 Konfigurasi Linter & Formatter
Gunakan konfigurasi standar rekomendasi resmi (ESLint, Pylint, SwiftLint, Prettier, Black, Gofmt). Dilarang membuat konfigurasi yang saling bentrok.

### 1.3 Modularitas, DRY & Context Reading Policy
1. **Modular & DRY:** Pisahkan logika berulang ke dalam fungsi utilitas, service, modul, atau class reusable. Dilarang duplikasi kode.
2. **Context Reading Policy:** Sebelum menyisipkan atau mengedit kode pada berkas eksisting, wajib membaca keseluruhan isi berkas terlebih dahulu untuk memahami pola, konvensi penamaan, dan logika sekitarnya. Dilarang menyuntikkan kode tanpa memahami konteks berkas.

### 1.4 Standar Preferensi Bahasa (English by Default)
1. **Default Bahasa Inggris:** Seluruh elemen source code wajib menggunakan Bahasa Inggris:
   - UI strings (label, placeholder, tooltip).
   - Pesan galat dan eksepsi (error & exception messages).
   - Log sistem (console & logging output).
   - Respon API (response message & error payload).
   - Penamaan variabel, fungsi, modul, class, dan tipe.
   - Anotasi tipe dan docstring fungsional.
2. **Pengecualian:** Jika pengguna meminta bahasa lain secara eksplisit (misal: *"Buat semua teks UI dan pesan error dalam Bahasa Indonesia"*), bahasa tersebut diizinkan dan wajib dicatat pada `nodes/[nama-node]/guidelines/project-context.md` di seksi *"Preferensi Bahasa"* demi konsistensi sesi berikutnya.

### 1.5 Penjelasan Skrip CLI
Setiap perintah terminal/CLI (eksekusi skrip, dependensi, build) wajib disertai penjelasan ringkas fungsi flag dan argumen di luar blok kode.

### 1.6 Kebijakan Referensi Path Wajib Relatif (Relative Path Policy)
1. **[ATURAN MUTLAK]** Setiap kali merujuk, mencantumkan tautan, atau menyebutkan lokasi berkas di dalam dokumen apa pun (berkas Markdown, pedoman, PRD, tiket, dokumen rencana, diagram, changelog, dan catatan teknis), AI Agent **WAJIB** menggunakan **relative path** yang dihitung dari posisi berkas aktif saat ini.
2. **[TERLARANG KERAS]** Menggunakan **absolute path** (seperti `/Users/...`, `/home/...`, atau `C:\...`).
3. **Rasional Portabilitas:** Absolute path mengikat referensi ke satu mesin lokal tertentu, merusak keterbacaan pada mesin developer lain, menggagalkan lingkungan CI/CD, dan memicu disorientasi navigasi pada agen AI lain.
   - `[SALAH]`: `Lihat spesifikasi pada /Users/developer/project/global-docs/prd.md`
   - `[BENAR]`: `Lihat spesifikasi pada ../../global-docs/prd.md` (dari node) atau `../docs/system-design.md` (dari tiket)

---

## 2. Integritas Data & Logika Bisnis (Data Integrity & Fail-Fast Principles)

### 2.1 Fail-Fast Policy (Larangan Fallback & Hardcoded Data)

> **KEBIJAKAN FAIL-FAST MUTLAK:**
> Dilarang membuat silent fallback, dummy defaults, atau mock data pada kode produksi. Jika environment variable, konfigurasi bisnis, atau data dinamis (API, Database, Storage, SDK) tidak ditemukan atau kosong, sistem WAJIB melempar runtime error eksplisit (Fail-Fast).

#### 2.1.1 Larangan Silent Fallback & Dummy Defaults
1. **Environment Variables:**
   - [TERLARANG] `process.env.NEXT_PUBLIC_API_KEY || "your_api_key"`
   - [TERLARANG] `process.env.API_BASE_URL || "https://api.example.com"`
   - [TERLARANG] `const dbUrl = process.env.DATABASE_URL || "postgres://user:pass@localhost:5432/mydb"`
   - [WAJIB] Validasi variabel lingkungan. Jika `undefined` atau string kosong (`""`), lempar runtime error eksplisit dengan pesan informatif.
2. **Data Dinamis:**
   - Dilarang menyediakan array statis tiruan, entitas dummy (produk, user, transaksi dummy), atau placeholder saat API, DB, atau layanan eksternal gagal/belum terhubung.
   - Tangani kegagalan dengan pola penanganan error (error boundaries, try-catch dengan lemparan error, atau UI status error yang jujur), bukan menyamarkannya dengan data palsu.

#### 2.1.2 Larangan Data Hardcoded / Mock di Kode Produksi
1. Seluruh data operasional, entitas bisnis, URL endpoint, dan kunci integrasi wajib dibaca langsung dari sumber aslinya: Environment Variables, API, Database, atau Storage.
2. Dilarang membuat daftar konstanta array/objek statis di file konfigurasi atau modul bisnis yang berpura-pura merepresentasikan data nyata.

#### 2.1.3 Syarat Pengecualian Mutlak
- Mock data atau fallback dummy HANYA diizinkan jika pengguna memintanya secara eksplisit kata-per-kata (contoh: *"Buat data ini secara hardcoded sementara"*, *"Gunakan mock data untuk prototipe layar ini"*, *"Sediakan fallback offline dummy untuk test lokal"*).
- Tanpa instruksi eksplisit tersebut, asumsi mutlak adalah 100% dinamis, nyata, dan fail-fast.
- **Pengecualian Default Teknis / UX:** Default teknis non-bisnis tetap diizinkan: paginasi default (`limit = 10`, `page = 1`), batas waktu jaringan (`timeoutMs = 5000`), konfigurasi varian UI (`variant = 'primary'`, `isOpen = false`).

#### 2.1.4 Larangan Kamuflase Kode (Anti-Shallow Compliance)
1. Dilarang membungkus ulang (repackaging) data statis ke bentuk lain, seperti memindahkannya ke return value fungsi helper (`function getProductList() { return [...] }`), objek registry, atau sekadar mengganti nama variabel.
2. Hapus substansi data statis secara tuntas dan ganti dengan pembacaan dinamis serta validasi error.

#### 2.1.5 Pola Implementasi Wajib (Helper Validasi)
Gunakan helper validasi fail-fast untuk memastikan konfigurasi wajib terdefinisi sebelum eksekusi lanjutan:

```typescript
function getRequiredEnv(key: string): string {
  const value = process.env[key];
  if (!value || value.trim() === "") {
    throw new Error(`[Configuration Error] Missing required environment variable: ${key}. Please check your .env file.`);
  }
  return value;
}
```

### 2.2 Larangan Magic Number & Standarisasi Konstanta (No Magic Number Policy)

> **ZERO MAGIC NUMBERS:**
> Dilarang menyisipkan angka mentah atau string arbitrer langsung pada logika bisnis, rumus kalkulasi, validasi kondisi, perulangan, atau batas threshold. Setiap nilai dengan arti fungsional/bisnis WAJIB diabstraksikan menjadi named constants atau Enum di file terpisah.

#### 2.2.1 Cakupan Larangan (Anti-Patterns)
- **Status & Kode Arbitrer:**
  - [SALAH] `if (user.role === 2)` -> [BENAR] `if (user.role === UserRole.ADMIN)`
  - [SALAH] `if (order.status === 4)` -> [BENAR] `if (order.status === OrderStatus.COMPLETED)`
- **Durasi & Waktu (Timeouts / Delays / Cache TTL):**
  - [SALAH] `setTimeout(fetchData, 86400000)` -> [BENAR] `setTimeout(fetchData, ONE_DAY_IN_MS)`
  - [SALAH] `jwt.sign(payload, secret, { expiresIn: 3600 })` -> [BENAR] `expiresIn: JWT_EXPIRATION_SECONDS`
- **Ukuran Data & Batas Kapasitas:**
  - [SALAH] `if (file.size > 10485760)` -> [BENAR] `if (file.size > MAX_FILE_SIZE_BYTES)`
- **Faktor Perhitungan & Rasio Bisnis:**
  - [SALAH] `const tax = subtotal * 0.11` -> [BENAR] `const tax = subtotal * VAT_RATE_PERCENTAGE`
  - [SALAH] `const discount = total * 0.05` -> [BENAR] `const discount = total * EARLY_BIRD_DISCOUNT_RATE`

#### 2.2.2 Standar Pengelolaan Berkas Konstanta
Pisahkan konstanta reusable ke modul terpusat:
1. **Struktur Direktori:** Tempatkan di `constants/`, `config/`, atau `types/` (misal: `src/constants/limits.ts`, `src/constants/time.ts`, `src/config/pricing.ts`).
2. **Konvensi Penamaan:** Gunakan `SCREAMING_SNAKE_CASE` untuk konstanta tunggal (`MAX_RETRY_ATTEMPTS`, `DEFAULT_PAGE_SIZE`). Gunakan objek bertingkat `as const` atau `Enum` untuk kelompok konstanta.
3. **Satuan Ukuran:** Nama konstanta wajib mendeskripsikan maksud bisnis dan satuan ukurannya (`_MS`, `_SECONDS`, `_BYTES`, `_PERCENTAGE`, `_DAYS`).

#### 2.2.3 Pengecualian yang Diizinkan (Idiomatic Numbers)
Angka fundamental dengan makna sintaksis universal diizinkan tanpa konstanta terpisah:
- Inisialisasi indeks array atau loop counter: `let i = 0` atau `array[0]`.
- Penambahan/pengurangan inkremental dasar: `count += 1` atau `index - 1`.
- Pengecekan paritas biner: `value % 2 === 0`.
- Representasi nilai not-found standar bahasa: `indexOf(...) === -1`.

#### 2.2.4 Contoh Berkas Konstanta yang Dianjurkan

```typescript
export const TIME_CONSTANTS = {
  ONE_SECOND_IN_MS: 1000,
  ONE_MINUTE_IN_MS: 60 * 1000,
  ONE_HOUR_IN_MS: 60 * 60 * 1000,
  ONE_DAY_IN_MS: 24 * 60 * 60 * 1000,
} as const;

export const UPLOAD_LIMITS = {
  MAX_FILE_SIZE_BYTES: 10 * 1024 * 1024,
  MAX_ATTACHMENTS_COUNT: 5,
} as const;

export const PAGINATION = {
  DEFAULT_PAGE_SIZE: 20,
  MAX_PAGE_SIZE: 100,
} as const;
```

### 2.3 Integritas Logika Bisnis (Zero Business Logic Fabrication)
1. Dilarang mengarang aturan bisnis, batas otorisasi, rumus kalkulasi, atau skema validasi yang tidak tercantum pada `prd.md` dan `system-design.md`.
2. Setiap logika percabangan kritis wajib berdasar spesifikasi kebutuhan nyata, bukan asumsi intuitif sepihak AI.

---

## 3. Kualitas Arsitektur & Rekayasa Antarmuka (Architecture & Engineering Standards)

### 3.1 Anti-AI-Slop Code Quality (Purpose-Driven Code)

> **KODE DENGAN TUJUAN NYATA (PURPOSE-DRIVEN CODE):**
> Dilarang menghasilkan kode over-engineered, abstraksi semu, atau tata letak berbasis magic numbers. Kode wajib lugas, efisien, bermakna, dan mudah dipelihara.

#### 3.1.1 Larangan Abstraksi Semu & Over-Engineering (YAGNI)
- Komponen, modul, atau class baru hanya boleh dibuat jika memiliki:
  1. Logika perilaku berulang (reusable behavior).
  2. Struktur semantik yang jelas dan berulang.
  3. Kontrak API atau batas state independen terisolasi.
- **Dilarang Over-Abstracting Div Soup:** Jangan memecah elemen `div` sederhana atau blok 3 baris kode menjadi komponen terpisah jika hanya digunakan satu kali tanpa logika mandiri.
- **Terapkan Prinsip YAGNI:** Dilarang membangun factory pattern bertingkat atau design system raksasa untuk fitur sederhana.

#### 3.1.2 Semantik Native & Platform-First
- Utamakan elemen native platform: `<button>` untuk aksi, `<a href>` untuk navigasi rute, `<label>` untuk formulir, `<table>` untuk komparasi data dua dimensi.
- **[TERLARANG]:** Membuat `<div onClick={...}>` untuk tombol atau navigasi tanpa keyboard handling (`Enter`/`Space`), focus management, dan screen reader semantics.
- Gunakan kontrol standar platform atau pustaka komponen teruji daripada merekayasa kontrol kustom yang rapuh (custom dropdown, custom scrollbar, custom modal).

### 3.2 Rekayasa Tata Letak & Styling (No Magic Number CSS Hacks)
[TERLARANG] Menambal posisi dengan koordinat sembarangan:
```css
position: absolute; top: 13px; left: 37px; transform: translateX(7px);
```
[WAJIB] Gunakan layout primitives baku:
```css
display: grid;
grid-template-columns: minmax(0, 1fr) auto;
gap: var(--space-4);
align-items: center;
```
Utamakan Flexbox, CSS Grid, logical properties (`margin-inline`), unit fluida (`clamp()`, `minmax()`), dan Container Queries.

### 3.3 Integritas State & Sinkronisasi URL (URL-Driven State)
- Parameter antarmuka yang shareable, bookmarkable, atau bertahan saat refresh / navigasi Back-Forward WAJIB disimpan di URL Search Params (Query String):
  - Kata kunci pencarian (search query), filter aktif, pengurutan (sorting), nomor halaman paginasi, dan tab aktif utama.
- **[TERLARANG]:** Mengunci parameter filter penting di local state sementara (`useState`) sehingga hilang saat halaman dimuat ulang.

### 3.4 Larangan Bypass Arsitektur (Architectural Integrity & No Hacks)
1. **Dilarang Inline Styles & Shortcuts:** Dilarang menggunakan inline styles atau modifikasi lokal untuk mengakali bug. Wajib telusuri dan perbaiki akar masalah (root cause) pada konfigurasi atau arsitektur dasarnya.
2. **Dilarang Eksekusi DDL Langsung:** Dilarang mengeksekusi DDL (`ALTER TABLE`, `CREATE TABLE`, `DROP COLUMN`) langsung pada live database. Seluruh perubahan struktur data wajib dikelola via skrip migrasi terstruktur sesuai [database.md](./database.md).
3. **Kepatuhan Protokol Mutasi State:** Dilarang meminta permission atau mengeksekusi mutasi state (tambah/ubah/hapus berkas, konfigurasi, DDL/DML massal) tanpa memaparkan Implementation Plan dan mendapat persetujuan pengguna sesuai [safe-file-operations.md](./safe-file-operations.md).

---

## 4. Standar Dokumentasi, Templat & Diagram (Documentation & Visual Artifacts)

### 4.1 Visualisasi Arsitektur Berbasis Teks (PlantUML Standard)
Dilarang menggunakan gambar statis (`.png`, `.jpg`) untuk diagram arsitektur.
1. **Pemisahan Berkas:** Seluruh visualisasi (Flowchart, ERD, Use Case, State Diagram, User Journey) WAJIB ditulis dengan PlantUML dan disimpan sebagai berkas `.puml` di `docs/diagrams/` (node) atau `global-docs/diagrams/` (global). Jangan simpan di root direktori node.
2. **Rujukan (Linking):** Di dokumen Markdown, dilarang menulis blok kode ````plantuml````. Gunakan tautan Markdown ke file `.puml` terkait (contoh: `[Lihat Flowchart Game Loop](./diagrams/flowchart.puml)`).
3. **Kemudahan Modifikasi (Text-Searchable):** Berkas `.puml` dapat dicari via teks dan divisualisasikan langsung menggunakan ekstensi PlantUML di VS Code tanpa merusak rendering Markdown.

### 4.2 Standarisasi Templat Pengembangan (API Docs & Routing Map)
1. **Dokumentasi API (`global-docs/templates/api_documentation_template.md`):** Wajib digunakan untuk mendokumentasikan URL dan payload pada proyek berbasis API.
2. **Peta Perutean (`global-docs/templates/routing_template.md`):** Wajib digunakan untuk memetakan rute frontend dan endpoint backend guna mencegah orphan routes.

---

## 5. Batasan Direktori & Isolasi Lingkungan (Repository & Boundary Isolation)

### 5.1 Pemisahan Fisik Kode Produksi & AI Orchestrator
1. Dilarang meletakkan file dokumentasi (PRD, System Design, Markdown penjelas) di dalam direktori source code aplikasi (`src/`, `app/`, `lib/`, `core/`).
2. Repositori `ai-orchestrator-template/` berdiri mandiri di luar folder aplikasi utama (tidak tertanam) agar portabel dan bebas konflik hierarki.

### 5.2 Standar Struktur Tingkat Akar (Root-Level Reference)
Tuliskan path berkas secara akurat di awal setiap blok instruksi kode:

```text
[Root Environment]
├── frontend-app/               <-- Source code aplikasi asli Frontend
├── backend-api/                <-- Source code aplikasi asli Backend
│
└── ai-orchestrator-template/   <-- Lingkungan mandiri pusat kendali AI Multi-Project
    ├── README.md               <-- Global Startup Prompt
    ├── global-docs/            <-- Pusat dokumen fondasi (PRD Utama, Design System)
    │   └── diagrams/           <-- Pusat file arsitektur global (.puml)
    ├── global-guidelines/      <-- Aturan mutlak lintas-node (security, testing, version-control)
    └── nodes/                  <-- Pusat komando sub-proyek
        └── _template/          <-- Draf kosong yang akan digandakan untuk setiap proyek
            ├── main.md         <-- Entrypoint harian KHUSUS untuk node ini
            ├── CHANGELOG.md    <-- Arsip historis pembaruan node ini
            ├── docs/diagrams/  <-- Diagram spesifik arsitektur node (.puml)
            ├── guidelines/     <-- Aturan lokal & hasil scan legacy codebase
            ├── prototypes/     <-- Area sketsa HTML statis sandbox UI
            ├── retrospectives/ <-- Pusat pembelajaran AI untuk kegagalan node ini
            ├── hunt/           <-- Log audit exploratory testing
            ├── plans/          <-- Dokumen rencana teknis terperinci (berpasangan dengan tickets/)
            └── tickets/        <-- Manajemen tiket tugas (user stories & acceptance criteria)
```

> **Catatan:** Struktur aplikasi asli di luar template ini tidak diatur ketat. Aturan mutlak hanyalah menegakkan **pemisahan fisik** antara source code aplikasi dengan direktori `ai-orchestrator-template/`.

### 5.3 Isolasi Tooling Pemetaan Kode (Graphify)
1. **Lokasi Eksklusif:** Seluruh artefak Knowledge Graph (`.graphify`), proses inisialisasi (`graphify build`), kueri, dan update WAJIB hanya berada dan dieksekusi di direktori source code proyek asli (Path Codebase). Jika `.graphify` ditemukan -> wajib dipakai untuk navigasi kode; jika belum ada -> jalankan `graphify build` di direktori node tersebut terlebih dahulu.
2. **Larangan Mutlak di Repositori Orchestrator:** Dilarang keras mengeksekusi `graphify build`, `graphify init`, atau membuat folder `.graphify` di dalam root repositori `ai-orchestrator-template/`. Repositori orchestrator adalah lingkungan pusat kendali dokumentasi dan pedoman, bukan target pemetaan kode.
