# Code Quality & Formatting Standard

## ZERO-COMMENT POLICY
Anda **DILARANG KERAS** menambahkan komentar apa pun di dalam *source code* yang Anda hasilkan. Kode harus sangat bersih, jelas, dan bisa menjelaskan dirinya sendiri (*self-documenting code*). Jika Anda perlu menjelaskan logika atau alur kode, berikan penjelasan tersebut secara naratif di dalam teks Markdown (di luar blok kode).

### Cakupan Larangan
Larangan ini mencakup **seluruh bentuk** anotasi teks non-fungsional di dalam kode dari berbagai kerangka kerja (*framework*) dan bahasa pemrograman, tanpa terkecuali:
- **Komentar JSX/TSX (React/Next.js/React Native):** `{/* ini komponen header */}`, `{/* TODO: refactor state */}`
- **Komentar HTML/Vue/Svelte/Angular:** `<!-- bagian sidebar -->`, `<!-- // HACK: z-index fix -->`
- **Inline comment (JS/TS/Go/Dart/Java/C/PHP dll):** `// ini fungsi login`, `// validasi regex`
- **Inline comment (Python/Ruby/Bash/YAML):** `# ambil data user`, `# setup environment`
- **Block comment (CSS/SCSS/JS/TS/SQL dll):** `/* background khusus dark mode */`, `/* index tabel users */`
- **Multi-line Docstring (Python):** `""" kelas ini menangani autentikasi pengguna """`
- **Trailing comment:** `const x = 10; // jumlah maksimal`, `margin-top: 10px; /* spacing */`
- **Penanda sementara (Universal):** `// TODO:`, `// FIXME:`, `// HACK:`, `// NOTE:`, `<!-- TODO: -->`, `{# FIXME: #}`
- **Docstring deskriptif naratif:** Penjelasan fungsi yang bersifat *narasi* atau *tutorial* (seperti `/// Fungsi ini digunakan untuk...`). **Pengecualian:** *Type annotations*, *parameter hints*, atau *contract-based annotations* yang **secara fungsional** dibutuhkan oleh compiler/linter (seperti `@override`, `@param {string}` tanpa deskripsi naratif, `@throws`, tipe return) **DIIZINKAN** karena bersifat fungsional, bukan komentar.

### Kewajiban Self-Check Sebelum Menyerahkan Kode
Sebelum Anda menyerahkan atau menampilkan blok kode apa pun kepada pengguna, Anda **WAJIB** melakukan pemeriksaan mandiri berikut:
1. Pindai setiap baris kode yang Anda tulis/modifikasi. Jika ditemukan komentar → **hapus segera**.
2. Jika kode yang Anda edit sudah memiliki komentar bawaan dari penulisnya, **jangan sentuh** komentar eksisting tersebut. Aturan ini berlaku hanya untuk kode **baru** yang Anda hasilkan.
3. Pelanggaran terhadap kebijakan ini dianggap sebagai **kegagalan eksekusi** yang setara dengan *compile error*.

## Modularitas & Linter
1.  **Linter & Formatter:** Pastikan konfigurasi linter (misalnya: ESLint, Pylint, SwiftLint) dan *formatter* (misalnya: Prettier, Black, Gofmt) yang Anda berikan tidak saling bentrok. Selalu gunakan konfigurasi standar yang direkomendasikan secara global.
2.  **Modular & DRY (Don't Repeat Yourself):** Jangan menulis kode yang berulang. Pisahkan logika ke dalam fungsi utilitas, layanan (*services*), modul, atau *class* yang dapat digunakan kembali (*reusable*).
3.  **Kewajiban Membaca Konteks (Context Reading Policy):** Sebelum Anda menyisipkan sebaris kode baru atau mengedit fungsi spesifik di dalam berkas yang sudah ada, Anda **WAJIB SECARA MUTLAK** untuk memindai/membaca keseluruhan isi dokumen tersebut terlebih dahulu. Pahami pola eksistingnya, konvensi penamaan lokalnya, dan logika di sekitarnya. Jangan langsung menyuntikkan kode buta yang merusak harmoni *file*!

## Penjelasan Skrip CLI
1.  Jika Anda memberikan perintah terminal/CLI (seperti eksekusi skrip, instalasi dependensi, atau *build*), jelaskan secara ringkas fungsi dari setiap *flag* atau argumen yang digunakan di luar blok kode agar mudah dipahami.

## Standar Preferensi Bahasa (Language Preference)
1. **Bahasa Inggris sebagai Standar Utama (English by Default):**
   - Seluruh elemen kode sumber yang Anda hasilkan **WAJIB MENGGUNAKAN BAHASA INGGRIS SECARA DEFAULT**, termasuk:
     - String antarmuka pengguna (*UI strings*, labels, placeholders, tooltips).
     - Pesan galat dan eksepsi (*error & exception messages*).
     - Log sistem (*logging / console output*).
     - Respon API (*API response messages & error payloads*).
     - Penamaan variabel, fungsi, modul, kelas, dan tipe.
     - Komentar fungsional / *type annotations* / *docstrings* (pada kasus yang diizinkan compiler/linter).
2. **Pengecualian & Kewajiban Pencatatan:**
   - Jika pengguna secara eksplisit meminta bahasa lain (misalnya: *"Buat semua teks UI dan pesan error dalam bahasa Indonesia"*):
     - Anda diperbolehkan menggunakan bahasa yang diminta tersebut.
     - Anda **WAJIB SECARA MUTLAK MENCATATKAN PERMINTAAN EKSPLISIT INI KE DALAM FILE `nodes/[nama-node]/guidelines/project-context.md`** di bawah seksi *"Preferensi Bahasa"* agar konsistensi bahasa terjaga pada seluruh sesi pengembangan berikutnya.

## Larangan Mutlak Fallback & Hardcoded Data (Fail-fast Policy)

> 🔴 **KEBIJAKAN FAIL-FAST MUTLAK:**
> AI Agent **DILARANG KERAS** membuat nilai fallback diam-diam (*silent fallback*), nilai default tiruan (*dummy defaults*), atau data tiruan (*mock data/arrays*) di dalam kode produksi hanya demi membuat aplikasi "terlihat tidak error" atau menjaga agar pengujian tetap hijau. Jika sebuah variabel lingkungan, konfigurasi bisnis, atau data dinamis (API, Database, Storage, Layanan Eksternal, SDK) tidak ditemukan atau kosong, sistem **WAJIB MELEMPAR RUNTIME ERROR EKSPLISIT (*FAIL-FAST*)**.

### 1. Larangan Silent Fallback & Dummy Defaults
1. **Dilarang Menggunakan Fallback Semu pada Environment Variables:**
   - **TERLARANG:** `process.env.NEXT_PUBLIC_API_KEY || "your_api_key"`
   - **TERLARANG:** `process.env.API_BASE_URL || "https://api.example.com"`
   - **TERLARANG:** `const dbUrl = process.env.DATABASE_URL || "postgres://user:pass@localhost:5432/mydb"`
   - **WAJIB (Fail-Fast):** Periksa keberadaan variabel lingkungan. Jika `undefined` atau kosong (`""`), lemparkan `Error` waktu proses (*runtime error*) secara eksplisit dengan pesan yang jelas dan informatif.
2. **Dilarang Menyediakan Fallback Defensif pada Data Dinamis:**
   - Dilarang memberikan array statis tiruan, entitas bisnis dummy (misal: daftar produk tiruan, user tiruan, transaksi dummy), atau data placeholder sebagai *fallback* ketika API, query database, atau koneksi layanan eksternal gagal atau belum terhubung.
   - Kegagalan pengambilan data nyata harus ditangani melalui pola penanganan galat (*error handling pattern* / *error boundaries* / *try-catch* yang melempar error atau menampilkan UI status error yang jujur), BUKAN menyamarkannya dengan data palsu.

### 2. Larangan Data Hardcoded / Mock di Kode Produksi
1. Seluruh data operasional, daftar entitas bisnis, URL endpoint, konfigurasi jaringan, dan kunci integrasi wajib dibaca langsung dari sumber aslinya: *Environment Variables*, API backend/third-party, Database, atau Storage service.
2. **DILARANG KERAS** membuat daftar konstanta array/objek statis di dalam file konfigurasi atau modul bisnis yang berpura-pura menjadi representasi data nyata.

### 3. Syarat Pengecualian Mutlak (Explicit Request Only)
Pembuatan data *hardcoded*, data tiruan (*mock data*), atau nilai bawaan buatan **HANYA DIPERBOLEHKAN JIKA DAN HANYA JIKA PENGGUNA MEMINTANYA SECARA EKSPLISIT MELALUI INSTRUKSI KATA-PER-KATA**, contohnya:
- *"Buat data ini secara hardcoded untuk sementara"*
- *"Gunakan mock data untuk prototipe layar ini"*
- *"Sediakan fallback offline dummy untuk pengujian lokal tanpa internet"*

**Tanpa adanya instruksi eksplisit seperti di atas, asumsi default Anda adalah: 100% DINAMIS, NYATA, DAN FAIL-FAST.**

> **Pengecualian Konvensi Framework & UX (Non-Bisnis):**
> Nilai default teknis murni yang berasal dari konvensi kerangka kerja atau kepraktisan UX tetap diizinkan, seperti:
> - Paginasi default (contoh: `limit = 10` atau `page = 1`)
> - Batas waktu jaringan default (contoh: `timeoutMs = 5000`)
> - Nilai opsional konfigurasi UI (`variant = 'primary'`, `isOpen = false`)

### 4. Larangan Kepatuhan Dangkal (Anti-Shallow Compliance / No Repackaging)
1. **Dilarang Melakukan Kamuflase Kode:** Ketika pengguna menegur Anda karena keberadaan data *hardcoded* atau *fallback*, Anda **DILARANG KERAS** sekadar membungkus ulang (*repackaging*) atau memindahkan data statis tersebut ke bentuk sintaksis lain, seperti:
   - Memindahkan array statis dari variabel konstanta ke dalam nilai kembalian (*return value*) sebuah fungsi helper (misal: `function getCuratedItems() { return [...] }` atau `function getProductList() { return [...] }`).
   - Memindahkan data statis ke dalam objek *registry* atau file kamus lain.
   - Mengubah nama variabel tanpa membuang substansi data palsunya.
2. **Hilangkan Substansi Datanya:** Yang ditolak oleh pengguna adalah **keberadaan data statis itu sendiri**, bukan nama variabel atau bungkus kodenya. Hapus data statis tersebut secara tuntas dan ganti dengan pembacaan dinamis / validasi error!

### 5. Pola Implementasi Wajib (Pola Error Handling & Helper Validasi)
Gunakan pola helper validasi yang melempar galat secara tegas untuk memastikan seluruh konfigurasi wajib terdefinisi sebelum aplikasi mengeksekusi logika lanjutan:

```typescript
function getRequiredEnv(key: string): string {
  const value = process.env[key];
  if (!value || value.trim() === "") {
    throw new Error(`[Configuration Error] Missing required environment variable: ${key}. Please check your .env file.`);
  }
  return value;
}
```

## Larangan Magic Number & Standarisasi Konstanta (No Magic Number Policy)

> 🚫 **ZERO MAGIC NUMBERS:**
> AI Agent **DILARANG KERAS** menyisipkan angka mentah (*raw numeric literals*) atau string kode arbitrer langsung ke dalam blok logika bisnis, rumus kalkulasi, validasi kondisi, perulangan, atau penentu batas (*threshold*). Setiap nilai numerik yang memiliki arti fungsional atau bisnis **WAJIB DIABSTRAKSIKAN** menjadi konstanta bernama (*named constants*) atau tipe `Enum` di dalam berkas terpisah yang terdedikasi.

### 1. Cakupan Larangan (Anti-Patterns)
- **Status & Kode Arbitrer:**
  - ❌ `if (user.role === 2)` $\rightarrow$ ✅ `if (user.role === UserRole.ADMIN)`
  - ❌ `if (order.status === 4)` $\rightarrow$ ✅ `if (order.status === OrderStatus.COMPLETED)`
- **Durasi & Waktu (Timeouts / Delays / Cache TTL):**
  - ❌ `setTimeout(fetchData, 86400000)` $\rightarrow$ ✅ `setTimeout(fetchData, ONE_DAY_IN_MS)`
  - ❌ `jwt.sign(payload, secret, { expiresIn: 3600 })` $\rightarrow$ ✅ `expiresIn: JWT_EXPIRATION_SECONDS`
- **Ukuran Data & Batas Kapasitas (*File Size & Limits*):**
  - ❌ `if (file.size > 10485760)` $\rightarrow$ ✅ `if (file.size > MAX_FILE_SIZE_BYTES)`
- **Faktor Perhitungan & Rasio Bisnis:**
  - ❌ `const tax = subtotal * 0.11` $\rightarrow$ ✅ `const tax = subtotal * VAT_RATE_PERCENTAGE`
  - ❌ `const discount = total * 0.05` $\rightarrow$ ✅ `const discount = total * EARLY_BIRD_DISCOUNT_RATE`

### 2. Standar Pengelolaan Berkas Konstanta (Best Practice Architecture)
Konstanta tidak boleh berserakan di sembarang tempat atau disisipkan secara *ad-hoc* di dalam komponen/controller lokal jika bernilai reusable atau merepresentasikan aturan sistem. Pisahkan konstanta ke dalam modul/berkas terpusat:

1. **Struktur Direktori Standar:**
   - Tempatkan di direktori khusus seperti `constants/`, `config/`, atau `types/` (misal: `src/constants/limits.ts`, `src/constants/time.ts`, `src/config/pricing.ts`).
2. **Konvensi Penamaan:**
   - Gunakan format **`SCREAMING_SNAKE_CASE`** untuk konstanta tunggal (contoh: `MAX_RETRY_ATTEMPTS`, `DEFAULT_PAGE_SIZE`, `ONE_HOUR_IN_MS`).
   - Gunakan objek `as const` bertingkat atau `Enum` untuk konstanta bertema (contoh: `const HTTP_STATUS = { OK: 200, NOT_FOUND: 404 } as const`).
3. **Deskripsi Arti Angka:**
   - Nama konstanta harus mendeskripsikan **maksud bisnis (*intent*) dan satuan ukurannya** (misal: `_MS`, `_SECONDS`, `_BYTES`, `_PERCENTAGE`, `_DAYS`).

### 3. Pengecualian yang Diizinkan (Idiomatic Numbers)
Angka-angka fundamental yang maknanya sudah sangat jelas secara sintaksis dan universal diizinkan tanpa konstanta terpisah:
- Inisialisasi awal indeks array atau pencacah perulangan: `let i = 0` atau `array[0]`.
- Penambahan/pengurangan inkremental dasar: `count += 1` atau `index - 1`.
- Pengecekan paritas biner: `value % 2 === 0`.
- Representasi nilai kosong/not-found standar bahasa: `indexOf(...) === -1`.

### 4. Contoh Format Berkas Konstanta yang Dianjurkan

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

## Standar Rekayasa Kode Anti-AI-Slop (Anti-AI-Slop Code Quality)

> 🛡️ **KODE DENGAN TUJUAN NYATA (PURPOSE-DRIVEN CODE):**
> AI Agent dilarang menghasilkan kode yang bertele-tele, terlalu banyak lapisan pembungkus (*over-engineered*), menyembunyikan logika di balik abstraksi semu, atau menambal tata letak dengan angka ajaib (*magic numbers*). Kode yang baik adalah kode yang lugas, efisien, bermakna, dan mudah dipelihara.

### 1. Larangan Abstraksi Semu & Over-Engineering (No Superficial Abstraction / YAGNI)
- **Ekstraksi Komponen Hanya Berdasarkan Kebutuhan Nyata:** Komponen, modul, atau kelas baru hanya boleh dibuat jika memiliki:
  1. Logika perilaku berulang (*reusable behavior*).
  2. Struktur semantik yang jelas dan berulang.
  3. Kontrak API atau batas *state* independen yang terisolasi.
- **Dilarang Over-Abstracting Div Soup:** Jangan memecah setiap elemen `div` atau blok 3 baris kode menjadi berkas komponen mikro terpisah jika komponen tersebut hanya digunakan satu kali dan tidak memiliki logika mandiri.
- **Terapkan Prinsip YAGNI (You Aren't Gonna Need It):** Jangan membangun arsitektur generik raksasa, *factory pattern* bertingkat, atau *design system* super-kompleks untuk prototipe fitur yang sederhana.

### 2. Semantik Native & Platform-First (No Reinventing the Wheel)
- **Utamakan Elemen Semantik Bawaan:** Selalu gunakan elemen native platform sebelum mencoba merekayasa ulang perilaku menggunakan elemen umum:
  - Gunakan `<button>` untuk aksi pemicu, `<a href>` untuk navigasi rute, `<label>` untuk formulir, dan `<table>` untuk komparasi data dua dimensi.
  - ❌ **TERLARANG:** Membuat `<div onClick={...}>` untuk tombol atau tautan navigasi tanpa penanganan keyboard (`Enter`/`Space`), *focus management*, dan *screen reader semantics*.
- **Gunakan Kontrol Standar Platform:** Jangan membuat komponen kustom (*custom dropdown*, *custom scrollbar*, *custom modal*) yang rapuh jika kontrol bawaan platform atau pustaka komponen teruji sudah menyediakannya dengan aksesibilitas lengkap.

### 3. Kualitas CSS & Layout Primitives (No Magic Number CSS Hacks)
- **DILARANG Menggunakan Angka Ajaib untuk Patching Tata Letak:**
  - ❌ **TERLARANG:** Menambal posisi elemen yang bergeser menggunakan koordinat absolut sembarangan, margin negatif acak, atau transformasi serampangan:
    ```css
    /* CONTOH TERLARANG */
    position: absolute;
    top: 13px;
    left: 37px;
    width: 417px;
    transform: translateX(7px);
    ```
  - ✅ **WAJIB:** Perbaiki arsitektur model tata letak menggunakan *Layout Primitives* baku:
    ```css
    /* CONTOH BENAR */
    display: grid;
    grid-template-columns: minmax(0, 1fr) auto;
    gap: var(--space-4);
    align-items: center;
    ```
- **Utamakan Intrinsic Sizing & Modern CSS:** Gunakan Flexbox, CSS Grid, *logical properties* (`margin-inline`, `padding-block`), unit fluida (`minmax()`, `clamp()`), serta *Container Queries* untuk komponen yang responsif terhadap kontainernya.

### 4. Integritas State & Sinkronisasi URL (URL-Driven State)
- **Navigasi & Filter Harus Bertahan di URL:**
  - Parameter antarmuka yang seharusnya dapat dibagikan (*shareable*), disimpan dalam bookmark, atau bertahan saat halaman dimuat ulang (*refresh*) maupun saat navigasi tombol *Back/Forward* **WAJIB** disimpan ke dalam *URL Search Params / Query String*.
  - Ini mencakup: kata kunci pencarian (*search query*), filter aktif, urutan pengurutan (*sorting*), nomor halaman paginasi, dan tab aktif utama.
  - ❌ **TERLARANG:** Mengunci seluruh parameter filter penting hanya di dalam *ephemeral local state* (seperti `useState`) sehingga reset saat di-refresh dan tidak bisa dibagikan tautannya kepada pengguna lain.

### 5. Integritas Logika Bisnis (Zero Business Logic Fabrication)
- AI **DILARANG KERAS** mengarang sendiri aturan bisnis, batasan otorisasi, rumus kalkulasi keuangan, atau skema validasi yang bertentangan atau tidak tercantum di dalam `prd.md` dan `system-design.md`.
- Setiap logika percabangan kritis harus didasarkan pada spesifikasi kebutuhan nyata, bukan karangan intuitif sepihak dari AI.

---

## Dilarang Mem-Bypass Arsitektur (No Hacks)
1.  **DILARANG KERAS menggunakan *inline styles* atau jalan pintas (*shortcuts*):** Anda dilarang menggunakan pendekatan pintas (seperti *inline styles* pada UI atau *hardcode* modifikasi lokal) sekadar untuk mengakali *bug* atau kegagalan konfigurasi spesifik.
2.  **Perbaiki Akar Masalah (*Root Cause*):** Jika ada konfigurasi atau sistem penataan yang gagal teraplikasikan, Anda wajib menelusuri dan memperbaiki akar masalahnya hingga ke file pengaturan utama atau arsitektur dasarnya. Jangan gunakan *hack* lokal sebagai solusi.
3.  **DILARANG Mengubah Skema/DDL Database Secara Langsung:** Dilarang melakukan eksekusi perintah DDL (`ALTER TABLE`, `CREATE TABLE`, `DROP COLUMN`, dll.) secara langsung pada *live database* sebagai jalan pintas. Seluruh perubahan struktur data wajib dikelola melalui berkas skrip migrasi terstruktur sesuai [database.md](./database.md).
4.  **Kepatuhan Protokol Mutasi State:** Dilarang langsung meminta *permission* atau mengeksekusi aksi yang memodifikasi *state* (menambah/mengubah/menghapus berkas, mutasi konfigurasi, eksekusi DDL/DML massal) tanpa memaparkan rencana implementasi (*Implementation Plan*) dan mendapatkan persetujuan pengguna sesuai [safe-file-operations.md](./safe-file-operations.md).

## Visualisasi Dokumentasi Berbasis Teks (PlantUML)
Sistem dokumentasi arsitektur di ekosistem ini **DILARANG KERAS** menggunakan lampiran gambar statis eksternal (`.png`, `.jpg`) untuk menggambarkan alur, struktur basis data, atau bagan interaksi.
1.  **Pemisahan File PlantUML:** Segala bentuk visualisasi (seperti Flowchart, ERD, Use Case, State Diagram, atau User Journey) **WAJIB MUTLAK** ditulis menggunakan tata bahasa [PlantUML](https://plantuml.com/) dan disimpan sebagai file berekstensi `.puml` terpisah secara eksplisit di dalam direktori `docs/diagrams/` (untuk lingkup spesifik node) atau `global-docs/diagrams/` (untuk lingkup global ekosistem). Jangan meletakkannya di root direktori node.
2.  **Rujukan (*Linking*):** Di dalam dokumen Markdown (seperti `prd.md` atau `system-design.md`), Anda **DILARANG** menulis blok kode ````plantuml````. Anda hanya diizinkan untuk membuat rujukan atau tautan Markdown menuju file `.puml` tersebut (contoh: `[Lihat Flowchart Game Loop](./diagrams/flowchart.puml)`).
3.  **Kemudahan Modifikasi (Text-Searchable):** Ini bertujuan agar AI Agent dapat melakukan pencarian teks, dan pengguna manusia dapat melihat diagram dengan mudah menggunakan ekstensi PlantUML di *code editor* (VS Code) tanpa merusak atau memperberat pembacaan file Markdown.

## Standarisasi Templat Pengembangan
Saat membangun fungsi-fungsi fundamental tertentu, Anda diwajibkan menyusun dokumentasinya menggunakan struktur *boilerplate* yang telah disediakan:
1.  **Dokumentasi API (`global-docs/templates/api_documentation_template.md`):** Khusus apabila Anda sedang merancang aplikasi yang bersifat API (seperti *backend server* atau integrasi *endpoint* murni), semua struktur URL dan *payload* wajib didokumentasikan menggunakan templat tersebut. *(Peringatan: Gunakan templat ini HANYA pada proyek berbasis API)*.
2.  **Peta Perutean (`global-docs/templates/routing_template.md`):** Segala bentuk tata letak lalu lintas halaman antarmuka (untuk Web/Frontend) atau rute lalu lintas API (untuk Backend) wajib dipetakan kelebarannya secara terpusat menggunakan standar templat *routing* ini guna mencegah rute yatim-piatu (*orphan routes*).

---

## Strict Directory & Documentation Boundaries

### Pemisahan Kode dan Dokumentasi
1.  Anda **DILARANG KERAS** meletakkan file dokumentasi (seperti berkas PRD, System Design, atau Markdown penjelas) berserakan di dalam direktori *source code* inti aplikasi (seperti di dalam `src/`, `app/`, `lib/`, atau `core/`).
2.  Sistem **AI Orchestrator** secara eksplisit dirancang agar **tidak tertanam (*embedded*) di dalam folder aplikasi utama**, melainkan berdiri sendiri di luarnya. Hal ini bertujuan agar sistem orkestrasi ini dapat dengan mudah dipasang, dipindahkan, atau digunakan pada proyek baru maupun proyek lama (*legacy*) tanpa menimbulkan konflik hierarki.

### Standar Struktur Tingkat Akar (*Root-Level*)
Gunakan struktur direktori terpisah berikut sebagai acuan logika pemisahan ruang kerja. Selalu tuliskan *path* berkas secara akurat di awal setiap blok kode yang Anda instruksikan:

```text
📁 [Root Environment]
├── 📁 frontend-app/               <-- Source code aplikasi asli Frontend
├── 📁 backend-api/                <-- Source code aplikasi asli Backend
│
└── 📁 ai-orchestrator-template/   <-- Lingkungan mandiri pusat kendali AI Multi-Project
    ├── 📄 README.md               <-- Global Startup Prompt
    ├── 📁 global-docs/            <-- Pusat dokumen fondasi (PRD Utama, Design System).
    │   └── 📁 diagrams/           <-- Pusat file arsitektur global (.puml).
    ├── 📁 global-guidelines/      <-- Aturan mutlak lintas-node (security, testing, version-control).
    │
    └── 📁 nodes/                  <-- Pusat komando sub-proyek
        └── 📁 _template/          <-- Draf kosong yang akan digandakan untuk setiap proyek
            ├── 📄 main.md         <-- Entrypoint harian KHUSUS untuk node ini
            ├── 📄 CHANGELOG.md    <-- Arsip historis pembaruan node ini
            ├── 📁 docs/           <-- System Design spesifik & Development Planning.
            │   └── 📁 diagrams/   <-- Diagram spesifik arsitektur node (.puml).
            ├── 📁 guidelines/     <-- Aturan lokal & hasil scan legacy codebase.
            ├── 📁 prototypes/     <-- Area sketsa HTML statis sandbox UI.
            ├── 📁 retrospectives/ <-- Pusat pembelajaran AI untuk kegagalan node ini.
            └── 📁 tickets/        <-- Manajemen task/isu offline node ini.
```

> **Catatan Penting Terkait Direktori Proyek:**
> Isi dan struktur aplikasi asli (seperti `frontend-app/` atau `backend-api/`) di luar template ini tidak diatur secara ketat. Aturan mutlak pada pedoman ini hanyalah menegakkan **pemisahan letak lingkungan secara fisik** antara lingkup *source code* aplikasi Anda dengan direktori `ai-orchestrator-template/`.

### Isolasi Tooling Pemetaan Kode (Graphify & Node-Level Artifacts)
1. **Lokasi Eksklusif & Kewajiban Penggunaan:** Seluruh artefak Knowledge Graph (`.graphify`), proses inisialisasi/generasi (`graphify build`), kueri arsitektur, serta pemutakhiran graf (`graphify update`) **WAJIB MUTLAK** hanya berada dan dieksekusi di dalam direktori *source code* proyek/node asli (*Path Codebase*). Jika direktori `.graphify` ditemukan di direktori node, AI **WAJIB** memakainya untuk navigasi kode. Jika tidak ditemukan, AI **WAJIB** men-generate-nya terlebih dahulu (`graphify build`) di direktori node tersebut.
2. **Larangan Mutlak di Repositori Orchestrator:** AI Agent **DILARANG KERAS** mengeksekusi `graphify build`, `graphify init`, atau membuat folder `.graphify` di dalam root direktori repositori `ai-orchestrator-template/`. Repositori orchestrator adalah lingkungan pusat kendali dokumentasi dan pedoman, bukan target pemetaan arsitektur kode aplikasi.

