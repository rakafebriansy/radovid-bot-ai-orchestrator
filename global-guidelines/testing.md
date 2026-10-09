# Testing, Sandboxing, & Pre-flight Standards

## Isolasi Pengujian & Sandboxing
Apabila Anda perlu menguji coba simulasi perilaku algoritma, atau menampilkan pratinjau (*preview*) mandiri dari sebuah modul komponen yang terisolasi:
1.  Buatlah lingkungan *sandbox* atau direktori pengujian coba-coba di tingkat akar proyek (misalnya: membuat direktori `preview/` atau `sandbox/`).
2.  Jangan pernah mencampurkan skrip simulasi pengujian atau kode coba-coba ini secara hierarkis ke dalam kerangka produksi (*production source code*) dari aplikasi utama.
3.  **Strict Gitignore Policy:** Direktori *sandbox* atau *preview* sementara tersebut **WAJIB** dideklarasikan ke dalam berkas `.gitignore`. Artefak kode eksperimental ini dilarang keras ikut terbawa ke dalam rekam jejak repositori *Version Control* (commit) maupun bocor ke lingkup rilis (*production environment*).

## Mandatory Pre-Flight Testing & Linting
Sebelum menyatakan sebuah modul telah selesai, siap diserahkan kepada pengguna untuk ditinjau, **ataupun sebelum melakukan aksi `commit` dan `push`**:
1.  Anda **WAJIB** memastikan bahwa proses pemeriksaan prasyarat kompilasi (*build checking*), eksekusi *linter*, dan validasi ketepatan referensi atau tipe (*type checker*) telah dieksekusi.
2.  **Verifikasi Log Terminal:** Pastikan dengan mutlak tidak ditemukan adanya galat (*compile errors*), perselisihan tipe (*type mismatches*), dependensi yang putus (*missing imports*), maupun peringatan krusial (*runtime warnings/errors*) pada instrumen *console* atau *log* log sistem.
3.  **Validasi Realitas Eksekusi:** Hindari sifat berasumsi bahwa kode akan otomatis berfungsi mulus sesaat setelah ditulis. Selalu yakinkan bahwa aliran instruksi komputasi berjalan selaras, presisi, dan sesuai dengan ekspektasi atau spesifikasi batas parameter awal yang ditugaskan.
4.  **Persetujuan Eksekusi Pengujian Manual Berdurasi Panjang:** Apabila Anda (sebagai AI Agent) berniat untuk melakukan metode pengujian manual yang spesifik, berat, dan memakan waktu panjang (seperti menjalankan *emulator* perangkat keras pada proyek *mobile*, atau menginisiasi otomatisasi peramban seperti *Chromium* pada proyek *web*), Anda **WAJIB meminta izin secara eksplisit terlebih dahulu** kepada pengguna sebelum melancarkan aksi tersebut. Hal ini mutlak diberlakukan guna mencegah pemborosan durasi dan kuota konsumsi *token* pengguna.

## Alur Kerja Test-Driven Development (TDD Workflow Wajib)

> 🔴 **DISIPLIN TDD MUTLAK (TEST-FIRST):**
> AI Agent **DILARANG KERAS** langsung menulis atau memodifikasi kode logika produksi tanpa terlebih dahulu menyiapkan unit test yang memvalidasinya. Setiap pengerjaan tugas atau instruksi wajib mengikuti siklus Test-Driven Development berikut secara disiplin:

```text
[1. Analisis DoD] ──> [2. Tulis/Ubah Test (Red)] ──> [3. Tulis Kode Produksi (Green)] ──> [4. Run Test & Fix] ──> [5. Run Linter & Typecheck] ──> [6. Finish]
```

### 1. Analisis Instruksi & Definition of Done (DoD)
- Identifikasi kriteria keberhasilan (*Acceptance Criteria* / *Definition of Done*) dari tugas yang diberikan.
- Jika tugas berbasis tiket, baca kriteria checklist di seksi `Acceptance Criteria` pada file tiket `tickets/TICKET-XX.md`.
- Jika tugas diberikan langsung via prompt pengguna, petakan kriteria fungsionalitas eksplisit yang diminta oleh developer sebagai Definition of Done.

### 2. Tahap RED (Tulis / Perbarui Unit Test Terlebih Dahulu)
- Sebelum menulis sebaris pun kode logika implementasi baru atau mengubah fungsi produksi, Anda **WAJIB** membuat atau memperbarui berkas *unit test* (`*.test.ts`, `test_*.py`, dll.).
- Rancang *test cases* yang mencakup skenario normal (*happy path*), skenario batas (*edge cases*), dan penanganan galat (*error handling*).
- Jalankan test tersebut di terminal untuk memverifikasi bahwa test baru berstatus **GAGAL (RED)** karena fungsionalitas belum diimplementasikan.

### 3. Tahap GREEN (Tulis Kode Produksi Secukupnya)
- Tulis atau modifikasi kode implementasi produksi pada aplikasi utama.
- Patuhi seluruh standar koding di [coding.md](./coding.md) (Zero-Comment Policy, Fail-Fast Policy, No Magic Numbers, Modular).
- Tulis kode produksi **hanya secukupnya** untuk membuat seluruh *test case* yang tadi merah menjadi berhasil (**GREEN**).

### 4. Tahap REFACTOR & FIX (Jalankan Test & Perbaiki Galat)
- Eksekusi *test suite* menggunakan perintah testing proyek (misal: `npm run test`, `pytest`, `go test ./...`).
- Jika ada *test* yang gagal atau regresi pada tes lain:
  - Analisis *stack trace* dan pesan kegagalannya.
  - Perbaiki kode produksi hingga seluruh pengujian lulus **100% PASS** dan memenuhi syarat *100% test coverage*.

### 5. Tahap QUALITY CHECK (Jalankan Linter & Type Checker)
- Eksekusi *linter* (misal: ESLint, Prettier, Pylint) dan *type checker* (misal: `tsc --noEmit`, `mypy`).
- Pastikan tidak ada satupun *lint warning/error* atau *type mismatch*. Perbaiki jika ditemukan.

### 6. FINISH (Penyelesaian Tiket & Dokumentasi)
- Setelah kode bersih, lolos testing, dan lolos linting, tandai tiket sebagai selesai (centang seluruh checklist `[x]` pada *Acceptance Criteria*), isi *AI Execution Log*, dan catatkan pembaruan ke `CHANGELOG.md`.

## Aturan Universal Pengujian (Universal Testing Policy)

Terlepas dari spesifikasi teknis platform (Web, API, Mobile), AI Agent wajib mematuhi seluruh doktrin pengujian di bawah ini:

### 1. Strategi Piramida Testing (Testing Pyramid)
- **Unit Test:** Ini adalah fondasi utama. Anda **WAJIB** membuat *unit test* terisolasi untuk **setiap fungsi logika bisnis (*business logic*) inti** yang Anda tulis. Dilarang meninggalkan fungsi inti tanpa pengujian.
- **Integration Test:** Uji interaksi antar-modul (contoh: *Controller* berinteraksi dengan *Service* dan *Database*).
- **End-to-End (E2E) Test:** Simulasi alur penggunaan aplikasi secara menyeluruh layaknya pengguna asli.

### 2. Standar Penamaan & Organisasi File
- Anda wajib mengikuti konvensi penamaan *file test* yang umum digunakan oleh *framework* target (misal: `*.test.ts`, `*_test.dart`, `test_*.py`).
- File pengujian tidak boleh berserakan. Tempatkan secara terpusat di dalam direktori spesifik (misal: `tests/` atau `__tests__/`) atau berdampingan persis (*co-located*) dengan *file source code* aslinya jika pola kerangka kerjanya mewajibkan hal tersebut.

### 3. Pencegahan Regresi (Regression Prevention)
Setiap kali Anda ditugaskan untuk memperbaiki *bug*, selain memperbaiki *source code*, Anda **DIWAJIBKAN MUTLAK** untuk menulis minimal 1 (satu) *unit test* baru yang secara spesifik mensimulasikan kondisi terjadinya *bug* tersebut. Ini adalah bukti matematis bahwa *bug* tersebut tidak akan bisa lolos dan muncul kembali di kemudian hari.

### 4. Anti-Overfitting pada Test Suite Lama (Integritas Kode Produksi)
Ketika arsitektur aplikasi beralih dari fase prototipe/mock menuju data dinamis (seperti integrasi API live, query database, layanan eksternal/cloud, atau environment variables), unit test lama yang menguji konstanta mock mungkin akan gagal:
- **DILARANG KERAS (Anti-Pattern):** Mengambil jalan pintas dengan mempertahankan atau membungkus kembali (*wrapping/repackaging*) data *hardcoded* ke dalam fungsi helper kode produksi hanya agar *test suite* lama tetap hijau (*overfitting to legacy tests*).
- **WAJIB (Best Practice):** Rombak dan sesuaikan berkas pengujian (*test files*). Gunakan teknik *mocking* terisolasi khusus di dalam file test (misal: `jest.mock`, `vi.fn()`, *test fixtures*), atau perbarui *assertions* agar selaras dengan arsitektur dinamis yang baru. Kode produksi **WAJIB** tetap murni dinamis dan bebas dari data statis!

### 5. Isolasi Data Pengujian (Data Independence)
Skrip pengujian yang Anda rancang **DILARANG KERAS** memanggil atau bergantung pada koneksi *database* produksi (*production database*) atau *database staging* jarak jauh. Anda wajib menggunakan fitur pemalsuan data (*Mocking*), kelas *Fixtures*, atau metode peniruan entitas (*Stubs*) untuk menjamin bahwa tes berjalan terisolasi, mandiri, cepat, dan *idempotent* (tidak menimbulkan efek samping).

### 6. Integrasi Pipeline Otomatis
Seluruh pengujian yang Anda buat harus didesain sedemikian rupa agar kompatibel untuk dijalankan secara senyap (*headless*) dan otomatis di dalam *pipeline* CI/CD. Anda tidak diperkenankan menyerahkan sebuah Pull Request (PR) jika *test suite* yang Anda bangun masih membuang kode galat (*error/fail*). 

### 7. Rujukan Silang saat Tes Gagal Berulang
Sesuai dengan pedoman di `error-handling.md`, jika eksekusi tes Anda selalu gagal atau buntu (*stuck*) walau sudah dicoba berulang kali (mencapai batas *Max Retry Rule*), Anda **WAJIB** mengeksekusi dua prosedur darurat:
1. Menghentikan eksperimen paksa dan segera melapor kepada pengguna.
2. Mencatat kebingungan dan jalan buntu teknis tersebut ke dalam file `nodes/[nama-node]/retrospectives/RETROSPECTIVE.md`.

### 8. Persyaratan Cakupan Pengujian 100% (100% Coverage Requirement)
Setiap penambahan atau modifikasi *source code* **WAJIB** memenuhi standar **100% *test coverage*** (meliputi *statements*, *branches*, *functions*, dan *lines*). Tidak boleh ada satupun baris kode atau cabang logika yang terlewat dari validasi pengujian. Jika coverage kurang dari 100%, kode tidak boleh dilanjutkan ke tahap berikutnya.
**Pengecualian Mutlak:** Pengecualian berlaku untuk file konfigurasi murni, titik masuk utama (*entry point* seperti `main()`), dan kode *boilerplate* yang secara arsitektural tidak logis atau tidak dapat diuji secara terisolasi. Jika Anda menerapkan pengecualian, Anda **WAJIB** mendokumentasikan alasan logisnya di *AI Execution Log* tiket.

### 9. Perintah Eksekusi Pengujian Spesifik Teknologi (Tech-Specific Testing Commands)
Berikut adalah panduan perintah standar eksekusi pengujian beserta inspeksi *coverage* berdasarkan ekosistem teknologi yang digunakan. Saat diminta untuk melakukan tes, selalu sertakan parameter *coverage* untuk memvalidasi syarat 100% coverage:

- **Node.js (Jest / Vitest / TypeScript):**
  - Eksekusi Test: `npm run test` atau `npx jest` / `npx vitest`
  - Eksekusi Test dengan Coverage (100%): `npm run test:cov` atau `npx jest --coverage` / `npx vitest run --coverage`
- **Go (Golang):**
  - Eksekusi Test: `go test ./...`
  - Eksekusi Test dengan Coverage (100%): `go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out`
- **Python (Pytest):**
  - Eksekusi Test: `pytest`
  - Eksekusi Test dengan Coverage (100%): `pytest --cov=. --cov-report=term-missing`
- **Dart (Flutter):**
  - Eksekusi Test: `flutter test`
  - Eksekusi Test dengan Coverage (100%): `flutter test --coverage`
- **Rust:**
  - Eksekusi Test: `cargo test`
  - Eksekusi Test dengan Coverage (100%): `cargo tarpaulin --ignore-tests`
- **Java / Kotlin (Gradle / Maven dengan Jacoco):**
  - Eksekusi Test (Gradle): `./gradlew test jacocoTestReport`
  - Eksekusi Test (Maven): `mvn clean test jacoco:report`

### 10. Pembersihan Artefak Pengujian (Test Artifact Cleanup)
Anda **WAJIB** selalu memastikan bahwa setiap file hasil *build* atau file ter-generate (*generated files*) lainnya yang berasal dari sisa hasil pengujian (*testing*) segera dihapus apabila ada. Jangan biarkan file sementara dari pengujian ini mengotori repositori.

### 11. Larangan Pengujian Semu (No Vanity Testing / Anti-Test Slop)
> 🛡️ **PENGUJIAN SUBSTANSIAL, BUKAN ANGKA SEMU:**
> AI Agent **DILARANG KERAS** memproduksi *test suite* yang dangkal (*test slop*) hanya demi mengejar target *coverage* 100% tanpa menguji integritas logika yang sebenarnya.
- **Dilarang Asersi Kosong:** Dilarang menulis asersi yang pasti lolos atau tidak bermakna seperti `expect(true).toBe(true)`, `expect(result).toBeDefined()` tanpa memvalidasi isi objek, atau memanggil fungsi tanpa asersi nilai kembalian.
- **Uji Perilaku Nyata & Kasus Ekstrem:** Setiap *test case* wajib menguji:
  1. *State mutation* dan nilai kembalian (*return value*) aktual.
  2. Kondisi batas ekstrem (*edge cases & boundary limits*).
  3. Skenario kegagalan & penanganan galat (*error throwing*, *invalid payload*, *rejections*).
- **Dilarang Menguji Boilerplate Sepele:** Jangan membuat lusinan *test* yang hanya menguji properti statis bawaan bahasa jika logika bisnis intinya tidak diuji secara mendalam.

### 12. Pengujian Antarmuka Pengguna & Gerbang Uji Visual Anti-Slop (Frontend UI/Screen Testing)
Saat Anda ditugaskan untuk melakukan pengujian (*testing*) pada komponen visual atau *frontend view/screen*:
1. **Evaluasi Heuristik (Usabilitas):** Anda **WAJIB** mengecek dan menerapkan 10 prinsip *Heuristic Evaluation* Jakob Nielsen sesuai pedoman di `ui-and-assets.md`.
2. **Tinjauan Visual Nyata (Rendered Output Inspection):** AI **DILARANG KERAS** menyatakan tugas UI selesai hanya karena *linter* atau *test* kompilasi lolos. Anda **WAJIB** memeriksa tampilan yang dirender secara visual (melalui sandbox prototipe atau pratinjau browser).
3. **Anti-Slop Visual Checklist Gate:** Pastikan antarmuka telah lolos seluruh butir pemeriksaan di `ui-and-assets.md` (bebas dari estetika AI SaaS generik, bebas konten/metrik palsu, kontras teks WCAG AA terpenuhi, target sentuh minimal 24x24px / 44x44pt, serta kelengkapan status *loading/skeleton/empty/error*).

> **Enforcement (Kewajiban Bukti):** Hasil evaluasi heuristik dan kepatuhan *Anti-Slop Checklist* ini **WAJIB** didokumentasikan ke dalam seksi *AI Execution Log* pada tiket terkait. Sebutkan secara eksplisit prinsip-prinsip Heuristik dan status uji Anti-Slop yang telah diverifikasi. AI tidak diperkenankan mengklaim telah menguji antarmuka tanpa bukti dokumentasi ini.
