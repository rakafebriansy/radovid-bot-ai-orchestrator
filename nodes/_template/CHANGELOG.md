# CHANGELOG

Changelog berfungsi sebagai catatan riwayat perubahan untuk proyek ini. Tujuan dari changelog ini adalah untuk melacak semua modifikasi, penambahan, dan perbaikan yang dilakukan secara terstruktur agar mudah dipahami oleh seluruh tim.

## Kategori Perubahan

Setiap entri log perubahan harus dikategorikan berdasarkan jenis file atau area yang diubah. Kategori-kategori tersebut terbagi menjadi:

*   **Guideline: <judul>**: Perubahan atau penambahan pada dokumen panduan (guideline).
*   **PRD**: Perubahan pada dokumen Product Requirements Document (PRD).
*   **Design System**: Pembaruan atau modifikasi yang berkaitan dengan Design System.
*   **System Design**: Perubahan pada arsitektur atau dokumen System Design.
*   **Development Planning: <nomor>**: Pembaruan yang terkait dengan perencanaan pengembangan (Development Planning).
*   **Ticket: <nomor>**: Perbaikan atau penambahan fitur yang merujuk pada tiket tertentu.
*   **Prototype: <judul>**: Pembaruan atau pembuatan prototipe desain/aplikasi.
*   **Implementation**: Implementasi coding pada folder yang dituju (harus menyertakan penjelasan detail mengenai apa saja yang diubah di dalam folder project).

Khusus untuk kategori **Implementation**, penjelasan pada kolom "Perubahan" **wajib** mencantumkan salah satu tag status spesifik berikut di awal kalimatnya:
*   `[Added]`: Penambahan fitur, dokumen, atau konfigurasi baru.
*   `[Changed]`: Modifikasi atau penyesuaian pada fungsionalitas, logika, atau dokumen yang sudah ada.
*   `[Fixed]`: Perbaikan atas suatu kelemahan, galat (*bug*), atau kesalahan *syntax*.
*   `[Removed]`: Penghapusan fitur, pedoman, atau kode usang (*deprecated*).

## Format Changelog

Setiap penambahan log versi terbaru **WAJIB MUTLAK** diletakkan di baris **PALING ATAS** daftar, tepat di bawah judul "Log Perubahan" (urutan *descending* / *reverse-chronological*). AI **DILARANG KERAS** menambahkan log baru di baris terbawah. Anda **WAJIB** mencatat SEMUA kategori perubahan secara disiplin, bukan hanya modifikasi kode (`Implementation`).

> ⚠️ **Satu-satunya Format yang Sah (Single Source of Truth):**
> AI Agent dilarang mengarang format changelog sendiri. Anda **WAJIB MUTLAK** menyalin dan mematuhi struktur baku yang terdapat pada berkas referensi berikut:
> `../../global-docs/templates/changelog_entry_template.md`

## Log Perubahan (_Judul Proyek_)

*(⚠️ PERHATIAN AI AGENT: TAMBAHKAN ENTRI LOG BARU ANDA TEPAT DI BAWAH BARIS INI. JANGAN DI PALING BAWAH DOKUMEN!)*

### [2026-09-21 13:10:00] - Guideline: LEARN.md Strict Trigger & Project Commit Term Cleanliness
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** "tambahkan rules baru untuk LEARN.md agar hanya diisi saat menggunakan template README.md, dan perbaiki kebocoran istilah orchestrator pada commit git milik node/project"
- **Perubahan:** `[Changed]` Menetapkan kebijakan anti-agresivitas pada `global-docs/LEARN.md` di mana file hanya boleh diisi saat menggunakan template baku FASE 4 dari `README.md`, dan dilarang keras diisi pada instruksi biasa. `[Changed]` Menghapus kontradiksi kewajiban referensi tiket pada pesan commit di `global-guidelines/version-control.md`, memperbarui `global-docs/templates/commit_message_template.md` dengan contoh terpisah yang bersih untuk Project/Node vs Orchestrator, serta menegaskan larangan istilah orchestrator/tiket pada commit node di `nodes/_template/main.md` dan `README.md`.
- **Path File:** `global-docs/LEARN.md`, `global-guidelines/version-control.md`, `global-docs/templates/commit_message_template.md`, `nodes/_template/main.md`, `README.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-16 21:22:00] - Guideline: Mandatory Graphify Utilization & Auto-Generation Standard
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** "ubah kalimatnya, jika ada graphify ditemukan di directory node, harus pakai. jika tidak ditemukan, maka generate lah"
- **Perubahan:** `[Changed]` Menetapkan kebijakan mutlak untuk Graphify: AI **WAJIB** memakai CLI `graphify query` untuk navigasi kode, pelacakan pemanggil/dependensi, dan penyusunan Implementation Plan jika direktori `.graphify` ditemukan di direktori project/node (*Path Codebase*). Jika direktori `.graphify` TIDAK ditemukan, AI **WAJIB** men-generate-nya terlebih dahulu dengan menjalankan `graphify build` di dalam *Path Codebase* node tersebut. `[Added]` Menegaskan aturan larangan mutlak pembatas direktori bahwa folder `.graphify`, pembuatan (`graphify build`), maupun pembaruan (`graphify update`) DILARANG KERAS dieksekusi di dalam repositori orchestrator.
- **Path File:** `global-guidelines/coding.md`, `README.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-07 17:47:00] - Guideline: Database & Datetime Storage Standard
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** "tambahkan panduan di orchestrator, untuk menyimpan datetime di aplikasi database buatlah dua opsi, yakni menggunakan epoch time millis (konversi jam saat ini ke epoch time) dan menyimpan timestamp + timezone utc di database (konversi jam saat ini di timezone user ke utc, baru di insert)"
- **Perubahan:** `[Added]` Menambahkan pedoman universal `database.md` yang menetapkan standar penyimpanan datetime di basis data dengan dua opsi baku (Opsi 1: Epoch Time Milliseconds / `BIGINT` dan Opsi 2: Timestamp with Timezone UTC / `TIMESTAMPTZ` / ISO-8601 UTC) beserta aturan konversi, tipe data, skenario penggunaan, dan matriks perbandingan. Memperbarui `global-guidelines/README.md`, `nodes/_template/main.md`, dan `nodes/_template/docs/system-design.md` dengan cross-reference ke pedoman baru tersebut.
- **Path File:** `global-guidelines/database.md`, `global-guidelines/README.md`, `nodes/_template/main.md`, `nodes/_template/docs/system-design.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-02 08:14:00] - Guideline: Safe File Operations Policy
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** "saya seringkali kehilangan kesempatan me-review ketika mengiyakan ai agent untuk menggunakan: 1. scripts (perubahan massal) dengan js, python, dll 2. sed -i ... 3. git checkout <file> 4. git restore <file> berikan batasan untuk penggunaan hal tersebut"
- **Perubahan:** `[Added]` Menambahkan pedoman `safe-file-operations.md` yang melarang operasi file destruktif (mass change scripts, in-place edit seperti `sed -i`, `git checkout <file>`, `git restore <file>`, `git clean`, `git stash drop`, `write_to_file` overwrite) serta mewajibkan alternatif reversible. Memperbarui `version-control.md`, `error-handling.md`, dan `nodes/_template/main.md` dengan cross-reference ke pedoman baru tersebut.
- **Path File:** `global-guidelines/safe-file-operations.md`, `global-guidelines/error-handling.md`, `global-guidelines/version-control.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-07-30 13:10] - Guideline: Comprehensive Audit Resolution
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Instruksi User:** "periksalah seluruh ai-orchestrator-template untuk potensi halu atau inefisiensi oleh AI"
- **Perubahan:** `[Fixed]` Menyelesaikan 16 temuan audit mencakup perbaikan halusinasi (H-01 s/d H-05), optimasi token (I-01 s/d I-05), penjernihan kontradiksi (A-01 s/d A-04), dan penutupan celah kepatuhan (C-01 s/d C-02, M-01 s/d M-03).
- **Path File:** `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/templates/README.md`, `global-guidelines/coding.md`, `global-guidelines/testing.md`, `global-guidelines/ui-and-assets.md`, `global-guidelines/version-control.md`, `nodes/_template/CHANGELOG.md`, `nodes/_template/main.md`, `nodes/_template/retrospectives/README.md`, `nodes/_template/docs/diagrams/README.md`

### [2026-07-20 18:48] - Enhancement: Node-Level Graphify Integration
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Instruksi User:** "ubahlah pendekatan graphify, jangan dibuat di directory orchestrator melainkan dibuat di directory milik project/node... mintalah cek terlebih dahulu apakah ada graphify di dalam project/node... sebelum melakukan eksekusi/scanning manual"
- **Perubahan:** Mengubah referensi `graphify build` dan `graphify update` pada root `README.md` dan `nodes/_template/main.md` agar menargetkan direktori spesifik *Path Codebase*. Menambahkan langkah wajib baru di SOP `main.md` (Poin E.2) untuk mengecek ketersediaan `.graphify` demi menghemat *token* sebelum pemindaian manual.
- **Path File:** `README.md`, `nodes/_template/main.md`

### [2026-07-19 22:43] - Guideline: Master Entrypoint & SOP
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Instruksi User:** "masukkan aturan ke `nodes/_template/main.md` dan refine lah main md buatlah lebih terstruktur, dan refine lagi terkait hierarki supaya tidak tumpang tindih. berlakukan aturan ke seluruh file di orchestrator template"
- **Perubahan:** Restrukturisasi total `main.md` menjadi hierarki A-E yang lebih jelas, menambahkan Protokol Pemulihan Konteks (Bagian B), memindahkan aturan Git & Inisiatif Liar ke file pedoman global (`version-control.md` & `error-handling.md`), serta menambahkan keterangan SSoT pada `global-guidelines/README.md`.
- **Path File:** `nodes/_template/main.md`, `global-guidelines/error-handling.md`, `global-guidelines/version-control.md`, `global-guidelines/README.md`

### [2026-07-02 16:58] - Guideline: Main SOP
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Instruksi User:** "harus tetap auto commit dan setiap auto commit harus berdasarkan ticket, jika instruksi tak memiliki ticket maka harus dibuatkan ticket! tambahkan di @[nodes/_template/main.md]"
- **Perubahan:** `[SUPERSEDED]` Menambahkan poin ke-7 pada SOP di `main.md` yang mewajibkan auto-commit berbasis tiket. *(Catatan: Aturan ini telah digugurkan. Berdasarkan revisi terbaru di `version-control.md`, AI dilarang keras melakukan auto-commit).*
- **Path File:** `nodes/_template/main.md`

