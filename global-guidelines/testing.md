# Testing, Sandboxing, & Pre-flight Standards

> **[PRINSIP UTAMA]** Standar pengujian mutlak untuk menjamin keandalan, integritas logika bisnis, pencegahan regresi, dan validasi fungsionalitas sebelum kode diserahkan kepada pengguna.

---

## 1. Fondasi Lingkungan Uji & Sandboxing

### 1.1 Isolasi Pengujian & Direktori Sandbox
1. **[WAJIB]** Pisahkan skrip eksperimen atau pengujian coba-coba ke direktori terisolasi di tingkat akar proyek (contoh: `preview/` atau `sandbox/`).
2. **[TERLARANG]** Mencampurkan kode simulasi atau berkas eksperimen ke dalam hierarki kode produksi (`src/`, `app/`, `lib/`).
3. **[WAJIB]** Daftarkan direktori *sandbox* atau *preview* ke dalam `.gitignore`. Artefak uji eksperimental dilarang masuk ke riwayat commit Git maupun lingkungan rilis.

### 1.2 Pemeriksaan Pra-Penerbangan (Mandatory Pre-Flight Checks)
Sebelum menyerahkan modul, menandai pasangan tiket dan rencana selesai (`Done` / `Completed`), atau mengajukan izin commit dan push:
1. **[WAJIB]** Eksekusi kompilasi (*build checking*), *linter*, dan validasi tipe (*type checker*).
2. **[WAJIB]** Verifikasi log terminal untuk memastikan nol galat kompilasi (*compile errors*), perselisihan tipe (*type mismatches*), impor hilang (*missing imports*), atau peringatan krusial runtime.
3. **[ATURAN]** Dilarang berasumsi kode berfungsi tanpa eksekusi langsung. Pastikan aliran instruksi berjalan sesuai parameter dan spesifikasi logika.

### 1.3 Otorisasi Pengujian Berat (Heavy Testing Approval)
1. **[WAJIB]** Minta izin eksplisit kepada pengguna sebelum menjalankan pengujian berdurasi panjang atau berkonsumsi sumber daya tinggi (seperti emulator hardware perangkat bergerak atau otomasi browser Chromium penuh). Hal ini wajib guna mencegah pemborosan durasi dan kuota token.

---

## 2. Alur Kerja Wajib Test-Driven Development (TDD Workflow)

AI Agent DILARANG menulis atau mengubah kode implementasi produksi sebelum menyiapkan unit test yang memvalidasinya.

```text
[2.1 Analisis DoD] --> [2.2 Tulis Test (RED)] --> [2.3 Tulis Kode (GREEN)] --> [2.4 Refactor & Fix] --> [2.5 Quality Check] --> [2.6 Hunt Loop] --> [2.7 Finish & Delivery]
```

### 2.1 Analisis Instruksi & Definition of Done (DoD)
1. Identifikasi kriteria keberhasilan (*Acceptance Criteria* / *Definition of Done*) dari tugas yang diberikan.
2. Pada tugas berbasis tiket & rencana (*ticket & plan*), verifikasi checklist `Acceptance Criteria` pada berkas tiket (`tickets/TICKET-[EPOCH_TIMESTAMP]-[judul].md`) dan target lingkup perubahan pada berkas rencana implementasi (`plans/PLAN-[EPOCH_TIMESTAMP]-[judul].md`). Pada instruksi langsung pengguna, petakan kriteria fungsionalitas secara eksplisit.

### 2.2 Tahap RED: Tulis Tes Gagal Terlebih Dahulu
1. Buat atau perbarui berkas unit test (`*.test.ts`, `test_*.py`, dll.) sebelum menyentuh kode produksi.
2. Rancang test case yang mencakup skenario normal (*happy path*), skenario batas (*edge cases*), dan penanganan galat (*error handling*).
3. Buktikan kegagalan pengujian di terminal:
   - **Fitur Baru:** Tes wajib gagal karena implementasi belum ada.
   - **Perbaikan Bug (Disiplin O-07):** Wajib membuktikan bahwa tes baru **GAGAL** pada kode lama yang belum diperbaiki, memeriksa pesan kegagalannya, dan memverifikasi kegagalan terjadi persis akibat anomali bug tersebut (bukan galat impor atau penulisan tes).

### 2.3 Tahap GREEN: Tulis Kode Produksi Minimal
1. Tulis atau modifikasi kode implementasi pada aplikasi utama.
2. Patuhi seluruh standar arsitektur pada [coding.md](./coding.md) (Zero-Comment Policy, Fail-Fast, No Magic Numbers, Modular).
3. Tulis kode produksi **hanya secukupnya** untuk membuat seluruh test case berstatus berhasil (**GREEN**).

### 2.4 Tahap REFACTOR & FIX: Perbaiki Regresi
1. Eksekusi seluruh *test suite* proyek (`npm test`, `pytest`, `go test ./...`).
2. Jika ditemukan kegagalan atau regresi logika:
   - Analisis *stack trace* dan pesan kegagalannya.
   - Perbaiki kode produksi hingga seluruh pengujian lulus **100% PASS** dan memenuhi cakupan kode 100%.

### 2.5 Tahap QUALITY CHECK: Linter & Type Checker
1. Eksekusi linter (ESLint, Prettier, Pylint) dan type checker (`tsc --noEmit`, `mypy`).
2. Pastikan nol *lint warning/error* dan nol perselisihan tipe. Perbaiki seluruh temuan sebelum melanjutkan.

### 2.6 Tahap HUNT LOOP: Eksplorasi Kasus Ekstrem
1. Jalankan minimal 1 putaran **Hunt Loop** (lihat Bagian 4): petakan risiko, sebutkan secara verbal minimal 10 kasus ekstrem berbasis [edge-case-atlas.md](./edge-case-atlas.md), reproduksi, jadikan tes regresi permanen, dan pastikan seluruh test suite tetap hijau.

### 2.7 Tahap FINISH: Delivery Gate & Dokumentasi
1. Jalankan checklist [delivery-gate.md](./delivery-gate.md).
2. Tandai checklist `[x]` pada *Acceptance Criteria*, isi *AI Execution Log* (bukti eksekusi dan daftar hal yang belum diuji), dan catat perubahan ke `CHANGELOG.md`.

---

## 3. Kebijakan Universal Pengujian (Universal Testing Policies)

### 3.1 Strategi Piramida Testing
1. **Unit Test (Fondasi Utama):** Wajib membuat unit test terisolasi untuk **setiap fungsi logika bisnis inti** yang ditulis. Dilarang meninggalkan fungsi inti tanpa pengujian.
2. **Integration Test:** Uji interaksi antar-modul (contoh: *Controller* berinteraksi dengan *Service* dan *Database*).
3. **End-to-End (E2E) Test:** Simulasi alur penggunaan aplikasi secara menyeluruh merefleksikan alur interaksi pengguna nyata.

### 3.2 Standar Penamaan & Organisasi Berkas
1. Gunakan konvensi penamaan baku framework target (`*.test.ts`, `*_test.dart`, `test_*.py`).
2. Kelola berkas pengujian secara terpusat dalam direktori khusus (`tests/`, `__tests__/`) atau berdampingan langsung (*co-located*) dengan modul terkait jika pola framework mensyaratkannya.

### 3.3 Pencegahan Regresi (Regression Prevention)
1. **[WAJIB]** Setiap perbaikan bug wajib disertai minimal 1 unit test baru yang secara presisi mereproduksi kondisi kegagalan bug tersebut.

### 3.4 Integritas Kode Produksi & Anti-Overfitting
Saat arsitektur beralih dari fase mock menuju integrasi dinamis (API live, database, cloud, environment variable):
1. **[TERLARANG]** Membungkus atau mempertahankan data mock ke dalam helper kode produksi hanya agar test suite lama tetap hijau (*overfitting to legacy tests*).
2. **[WAJIB]** Perbarui berkas pengujian menggunakan teknik mocking terisolasi (`jest.mock`, `vi.fn()`, fixtures) atau sesuaikan asersi dengan arsitektur dinamis yang baru. Kode produksi wajib tetap murni dinamis.

### 3.5 Isolasi Data Pengujian
1. **[TERLARANG]** Bergantung pada database produksi atau staging jarak jauh untuk pengujian otomatis.
2. **[WAJIB]** Gunakan Mocking, Fixtures, atau In-Memory Stubs agar tes berjalan mandiri, cepat, dan *idempotent* tanpa efek samping data.

### 3.6 Otomasi CI/CD & Headless Execution
1. Seluruh pengujian wajib kompatibel untuk dijalankan secara senyap (*headless*) pada pipeline CI/CD.
2. Dilarang menandai pasangan tiket dan rencana selesai (`Done` / `Completed`) atau mengajukan Pull Request jika test suite menghasilkan status gagal (*error/fail*).

### 3.7 Batas Percobaan & Prosedur Eskalasi
Jika eksekusi tes mengalami kegagalan berulang hingga mencapai batas *Max Retry Rule* sesuai [error-handling.md](./error-handling.md):
1. Hentikan percobaan perbaikan sepihak dan laporkan kendala kepada pengguna.
2. Dokumentasikan akar masalah ke dalam `nodes/[nama-node]/retrospectives/RETROSPECTIVE.md`.

### 3.8 Standar Cakupan Kode 100% (100% Coverage Requirement)
1. **[WAJIB]** Setiap penambahan atau modifikasi kode wajib memenuhi **100% test coverage** (*statements*, *branches*, *functions*, dan *lines*).
2. **Pengecualian Mutlak:** Pengecualian hanya berlaku untuk berkas konfigurasi statis, titik masuk utama (*entry point* seperti `main()`), dan boilerplate yang secara arsitektural tidak logis diuji terisolasi. Seluruh pengecualian wajib dicatat alasannya pada *AI Execution Log* tiket serta rencana implementasinya.

### 3.9 Perintah Eksekusi Pengujian Spesifik Bahasa
Gunakan parameter evaluasi cakupan (*coverage*) berikut untuk memvalidasi ambang batas 100%:

| Ekosistem | Perintah Eksekusi Standar | Perintah Eksekusi dengan Coverage (100%) |
|---|---|---|
| **Node.js (Jest / Vitest)** | `npm run test` / `npx vitest` | `npm run test:cov` / `npx jest --coverage` / `npx vitest run --coverage` |
| **Go (Golang)** | `go test ./...` | `go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out` |
| **Python (Pytest)** | `pytest` | `pytest --cov=. --cov-report=term-missing` |
| **Dart (Flutter)** | `flutter test` | `flutter test --coverage` |
| **Rust** | `cargo test` | `cargo tarpaulin --ignore-tests` |
| **Java / Kotlin** | `./gradlew test` / `mvn clean test` | `./gradlew test jacocoTestReport` / `mvn clean test jacoco:report` |

### 3.10 Pembersihan Artefak Pengujian
1. **[WAJIB]** Hapus berkas kompilasi sementara, snapshot uji coba, atau output log sisa hasil pengujian agar repositori tetap bersih.

### 3.11 Larangan Pengujian Semu (Anti-Test Slop)
AI Agent DILARANG memproduksi test suite dangkal hanya demi mengejar metrik coverage 100%:
1. **[TERLARANG]** Asersi kosong atau asersi tautologis (`expect(true).toBe(true)`, `expect(result).toBeDefined()` tanpa validasi nilai spesifik, atau pemanggilan fungsi tanpa memeriksa return value).
2. **[WAJIB]** Uji perilaku mutasi state, kondisi batas ekstrem (*boundary limits*), dan skenario kegagalan (*error throwing*, *invalid payload*).
3. **[TERLARANG]** Menguji boilerplate sepele bawaan bahasa sementara logika bisnis intinya diabaikan.

### 3.12 Gerbang Uji Visual Antarmuka (Frontend UI/Screen Testing)
Saat melakukan pengujian komponen visual atau layar antarmuka:
1. **Evaluasi Heuristik:** Terapkan 10 prinsip *Heuristic Evaluation* Jakob Nielsen sesuai [ui-and-assets.md](./ui-and-assets.md).
2. **Inspeksi Render Nyata:** Wajib memeriksa visual hasil render (melalui sandbox prototipe atau browser preview). Dilarang menganggap tugas UI selesai hanya karena lulus kompilasi/linter.
3. **Gerbang Checklist Anti-Slop:** Verifikasi kepatuhan terhadap [ui-and-assets.md](./ui-and-assets.md) (bebas estetika generik, bebas konten palsu, kontras WCAG AA terpenuhi, target sentuh minimal 24x24px / 44x44pt, serta kelengkapan status *loading/skeleton/empty/error*).
4. **Kewajiban Bukti:** Dokumentasikan prinsip heuristik dan status verifikasi visual ke dalam *AI Execution Log* tiket serta rencana implementasinya.

---

## 4. Protokol Pengujian Lanjutan & Pelacakan Bug (Advanced Hunt Protocol)

### 4.1 Siklus Hunt Loop 7 Langkah
Terapkan siklus pencarian celah secara iteratif hingga satu putaran penuh tidak lagi menemukan kegagalan baru:
1. **Map (Pemetaan Dampak):** Petakan perubahan kode, modul interaksi, transisi state, penulisan storage, panggilan eksternal, dan asumsi implisit sistem.
2. **Chatter (Deklarasi Risiko Terbuka):** Sebutkan minimal **10 kasus ekstrem konkret** di awal sesi berbasis 9 sumbu [edge-case-atlas.md](./edge-case-atlas.md), diurutkan berdasarkan dampak: Kehilangan Uang/Data $\rightarrow$ Keamanan/Privasi $\rightarrow$ Alur Rusak $\rightarrow$ Tampilan/Kosmetik. Dilarang membuat pernyataan umum tanpa menyebut kasus spesifik.
3. **Reproduce (Reproduksi Nyata):** Buktikan kegagalan secara nyata via skrip tes, request API, eksekusi CLI, atau form input. Kasus yang tidak dapat direproduksi dicatat sebagai *"tidak terbukti"*.
4. **Pin (Pengujian Permanen):** Ubah kasus terbukti menjadi unit test permanen yang gagal sebelum diperbaiki dan lulus setelah perbaikan (patuhi O-07).
5. **Fix / Report:** Perbaiki kode implementasi langsung pada tugas aktif, atau dokumentasikan ke `hunt-findings.md` pada sesi audit QA.
6. **Run All (Eksekusi Seluruh Suite):** Jalankan seluruh test suite proyek dan laporkan angka pastinya (contoh: *"412/412 tests passed"*).
7. **Next Round (Evaluasi Dampak Lanjutan):** Evaluasi celah turunan yang mungkin terbuka akibat perbaikan, analisis kasus tetangga, dan periksa integritas data historis (O-09).

### 4.2 Kebijakan Pengujian Berbasis Bukti (Evidence-Based Testing - O-03)
AI Agent DILARANG menyatakan status lulus berdasarkan asumsi (*"seharusnya aman"*, *"tampaknya berfungsi"*). Setiap klaim **PASS** wajib menyertakan bukti konkret:
1. **Perintah Eksekusi:** Perintah terminal persis yang dijalankan.
2. **Output Terminal Aktual:** Cuplikan log yang menunjukkan jumlah tes lulus, durasi, dan persentase coverage.
3. **Kondisi Data (State/DB):** Hasil kueri atau respons JSON yang membuktikan perubahan state valid.
4. **Tinjauan Visual (UI):** Pratinjau render aktual atau screenshot verifikasi.

### 4.3 Pembuktian Gagal pada Versi Lama (Prove Failure on Old Version - O-07)
1. Saat memperbaiki bug atau menutup celah logika, eksekusi tes baru tersebut terhadap kode lama sebelum perbaikan diterapkan.
2. Pastikan tes gagal secara spesifik karena akar masalah bug tersebut, bukan karena kesalahan impor atau penulisan pengujian.
3. Terapkan perbaikan, lalu pastikan tes baru tersebut berhasil (**GREEN**).

### 4.4 Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
Jika bug berkaitan dengan manipulasi state atau persistensi data:
1. Analisis apakah anomali bug pada versi lama telah merusak atau membuat data eksisting menjadi tidak konsisten.
2. Laporkan lokasi dan volume data terdampak.
3. Jalankan skrip pembersihan/migrasi jika memiliki otorisasi, atau dokumentasikan secara rinci pada *AI Execution Log* tiket serta rencana implementasinya.

### 4.5 Deklarasi Pengujian yang Belum Dilakukan (Untested List Declaration - O-16)
1. **[WAJIB]** Deklarasikan secara eksplisit daftar fungsionalitas atau skenario yang **TIDAK DITEST** beserta alasan teknisnya (contoh: tidak tersedianya sandbox gateway pembayaran live, butuh perangkat keras fisik, atau keterbatasan hak akses).
2. Cantumkan deklarasi ini pada seksi `AI Execution Log` tiket serta rencana teknis pasangannya, atau laporan audit QA.
