# CHANGELOG

> **[PRINSIP UTAMA]** Catatan riwayat pembaruan sistem (*Single Source of Truth*) untuk melacak seluruh modifikasi, penambahan fitur, dan perbaikan secara terstruktur.

---

## 1. Aturan & Standar Pencatatan

### 1.1 Kategori Perubahan
Setiap entri wajib diklasifikasikan ke dalam salah satu kategori:
- **`Guideline: <judul>`**: Modifikasi atau penambahan pada berkas pedoman (`guidelines/`).
- **`PRD`**: Pembaruan dokumen Product Requirements Document.
- **`Design System`**: Pembaruan spesifikasi desain visual dan token.
- **`System Design`**: Pembaruan arsitektur sistem dan diagram teknis.
- **`Development Planning: <nomor>`**: Pembaruan peta jalan atau antrean tiket.
- **`Ticket: <nomor>`**: Pengerjaan fitur atau perbaikan berbasis tiket spesifik.
- **`Prototype: <judul>`**: Pembuatan atau pembaruan purwarupa antarmuka HTML.
- **`Implementation`**: Perubahan kode sumber aplikasi. Wajib menggunakan salah satu tag awalan:
  - `[Added]`: Fitur, dokumen, atau konfigurasi baru.
  - `[Changed]`: Penyesuaian fungsionalitas, logika, atau dokumen eksisting.
  - `[Fixed]`: Perbaikan kelemahan, galat (*bug*), atau kesalahan sintaksis.
  - `[Removed]`: Eliminasi fitur, kode, atau pedoman usang (*deprecated*).

### 1.2 Ketentuan Urutan & Format Baku
1. **Urutan Kronologis Terbalik (*Reverse-Chronological*):** Entri terbaru WAJIB diletakkan di baris **PALING ATAS** daftar log (tepat di bawah judul bagian log). DILARANG menambahkan entri di bagian terbawah dokumen.
2. **Cakupan Universal:** Wajib mencatat SEMUA kategori perubahan, bukan hanya modifikasi kode (`Implementation`).
3. **Template SSoT:** Wajib mematuhi format baku pada `../../global-docs/templates/changelog_entry_template.md`.

---

## 2. Log Perubahan: [GANTI_DENGAN_NAMA_PROYEK]

> **[PERINGATAN AI AGENT]** Tambahkan entri log baru tepat di bawah garis pemisah ini. Dilarang menambahkan log di baris paling bawah!

---

### [2026-10-26 21:35:00] - Refactor: Comprehensive Hierarchy Restructure & Token Optimization
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Restrukturisasi hierarki numerik terstandarisasi dan optimasi efisiensi token di seluruh berkas orchestrator.
- **Perubahan:**
  - `[Changed]` Restrukturisasi total 15 berkas pedoman global (`coding.md`, `ui-and-assets.md`, `testing.md`, `safe-file-operations.md`, `version-control.md`, `database.md`, `security.md`, `error-handling.md`, `pipeline.md`, `deployment.md`, `dependencies.md`, `localization.md`, `delivery-gate.md`, `edge-case-atlas.md`, `README.md`) ke format hierarki numerik bertingkat bebas emoji non-ASCII (0 non-ASCII).
  - `[Changed]` Optimasi dokumen fondasi global (`prd.md`, `design-system.md`, `LEARN.md`, serta template di `global-docs/templates/`).
  - `[Changed]` Restrukturisasi direktori `nodes/_template/` (`main.md`, `system-design.md`, `development-planning.md`, `tickets/README.md`, `prototypes/README.md`, `hunt/README.md`, `retrospectives/README.md`, `project-context.md`, `CHANGELOG.md`).
  - `[Changed]` Merampingkan `README.md` root (-48% token) dengan memadatkan prompt 4 Fase tanpa kehilangan satu pun langkah instruksi.
  - `[Added]` Menambahkan `.gitignore` untuk mengecualikan berkas referensi `clone/` dan metadata OS.
- **Path File:** `README.md`, `.gitignore`, `global-guidelines/*`, `global-docs/*`, `nodes/_template/*`

### [2026-10-26 14:22:00] - Guideline: Relentless Testing Integration (Hunt Loop, Delivery Gate, & Edge Case Atlas)
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Integrasi metodologi testing ketahanan lanjutan (Hunt Loop, Delivery Gate, dan Edge Case Atlas) ke dalam orchestrator.
- **Perubahan:**
  - `[Added]` Pedoman `delivery-gate.md` (4 Pilar: Higienitas Kode/Data, Hard Gate Testing, Must Probe Loop, Tata Kelola), `edge-case-atlas.md` (9 Sumbu Kasus Ekstrem), `global-docs/templates/hunt-findings.md`, dan direktori `nodes/_template/hunt/`.
  - `[Changed]` Mengaugmentasi `global-guidelines/testing.md` dengan Hunt Loop 7 Langkah, Evidence Policy (O-03), pembuktian tes gagal di versi lama (O-07), Damage Assessment (O-09), dan Untested List (O-16).
  - `[Changed]` Memperbarui SOP E.6, E.8, dan C.3 pada `nodes/_template/main.md`, memperkaya boilerplate AI Execution Log pada `nodes/_template/tickets/README.md`, menyempurnakan `bug_report_template.md`, dan menambahkan hook Hunt Loop pada prompt eksekusi Fase 3 di `README.md`.
- **Path File:** `global-guidelines/delivery-gate.md`, `global-guidelines/edge-case-atlas.md`, `global-guidelines/testing.md`, `nodes/_template/main.md`, `nodes/_template/tickets/README.md`, `global-docs/templates/bug_report_template.md`, `global-docs/templates/hunt-findings.md`, `nodes/_template/hunt/README.md`, `README.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-21 13:10:00] - Guideline: LEARN.md Strict Trigger & Project Commit Term Cleanliness
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Penegakan aturan anti-agresivitas LEARN.md dan pembersihan kebocoran istilah orchestrator pada commit Git project/node.
- **Perubahan:**
  - `[Changed]` Menetapkan kebijakan anti-agresivitas pada `global-docs/LEARN.md` (hanya aktif via template FASE 4 README.md; dilarang keras pada instruksi biasa).
  - `[Changed]` Memperbarui `global-guidelines/version-control.md` dan `global-docs/templates/commit_message_template.md` dengan aturan pemisahan steril pesan komit: Project/Node 100% bebas istilah orchestrator vs Orchestrator scope.
  - `[Changed]` Menyelaraskan SOP `nodes/_template/main.md` dan `README.md`.
- **Path File:** `global-docs/LEARN.md`, `global-guidelines/version-control.md`, `global-docs/templates/commit_message_template.md`, `nodes/_template/main.md`, `README.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-16 21:22:00] - Guideline: Mandatory Graphify Utilization & Auto-Generation Standard
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Standarisasi penggunaan CLI Graphify untuk navigasi kode dan auto-generation graf dependensi.
- **Perubahan:**
  - `[Changed]` Menetapkan kewajiban pemakaian CLI `graphify query` untuk navigasi kode, pelacakan pemanggil, dan Implementation Plan jika `.graphify` ditemukan di Path Codebase node.
  - `[Changed]` Jika `.graphify` belum ada, AI wajib men-generate via `graphify build` di Path Codebase.
  - `[Added]` Larangan mutlak mengeksekusi build/update graphify di dalam direktori orchestrator.
- **Path File:** `global-guidelines/coding.md`, `README.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-07 17:47:00] - Guideline: Database & Datetime Storage Standard
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Penetapan dua opsi baku penyimpanan datetime pada database aplikasi.
- **Perubahan:**
  - `[Added]` Pedoman universal `database.md` (Opsi 1: Epoch Millis / `BIGINT` dan Opsi 2: Timestamp UTC / `TIMESTAMPTZ` / ISO-8601 UTC).
  - `[Changed]` Memperbarui cross-reference pada `global-guidelines/README.md`, `nodes/_template/main.md`, dan `nodes/_template/docs/system-design.md`.
- **Path File:** `global-guidelines/database.md`, `global-guidelines/README.md`, `nodes/_template/main.md`, `nodes/_template/docs/system-design.md`, `nodes/_template/CHANGELOG.md`

### [2026-09-02 08:14:00] - Guideline: Safe File Operations Policy
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Pencegahan eksekusi perintah destruktif tanpa kesempatan review pengguna.
- **Perubahan:**
  - `[Added]` Pedoman `safe-file-operations.md` melarang script mutasi massal, edit in-place (`sed -i`), `git checkout <file>`, `git restore <file>`, `git clean`, `git stash drop`, dan `write_to_file` overwrite.
  - `[Changed]` Menyelaraskan rujukan di `version-control.md`, `error-handling.md`, dan `nodes/_template/main.md`.
- **Path File:** `global-guidelines/safe-file-operations.md`, `global-guidelines/error-handling.md`, `global-guidelines/version-control.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-07-30 13:10:00] - Guideline: Comprehensive Audit Resolution
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Audit komprehensif eliminasi halusinasi dan inefisiensi orchestrator.
- **Perubahan:** `[Fixed]` Menyelesaikan 16 temuan audit (halusinasi H-01 s/d H-05, inefisiensi token I-01 s/d I-05, kontradiksi A-01 s/d A-04, kepatuhan C-01 s/d C-02 & M-01 s/d M-03).
- **Path File:** `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/templates/README.md`, `global-guidelines/coding.md`, `global-guidelines/testing.md`, `global-guidelines/ui-and-assets.md`, `global-guidelines/version-control.md`, `nodes/_template/CHANGELOG.md`, `nodes/_template/main.md`, `nodes/_template/retrospectives/README.md`, `nodes/_template/docs/diagrams/README.md`

### [2026-07-20 18:48:00] - Enhancement: Node-Level Graphify Integration
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Penataan lokasi direktori graf Graphify pada level codebase node.
- **Perubahan:** Menyesuaikan referensi `graphify build` dan `update` agar menargetkan direktori Path Codebase lokal node; menambah langkah SOP pengecekan `.graphify` di `main.md`.
- **Path File:** `README.md`, `nodes/_template/main.md`

### [2026-07-19 22:43:00] - Guideline: Master Entrypoint & SOP
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Restrukturisasi SOP Master Entrypoint template node.
- **Perubahan:** Menata hierarki `main.md`, menambah Protokol Pemulihan Konteks, memindahkan aturan Git dan inisiatif liar ke `version-control.md` dan `error-handling.md`.
- **Path File:** `nodes/_template/main.md`, `global-guidelines/error-handling.md`, `global-guidelines/version-control.md`, `global-guidelines/README.md`

### [2026-07-02 16:58:00] - Guideline: Main SOP
> **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Inisiasi SOP pembuatan tiket awal.
- **Perubahan:** `[SUPERSEDED]` SOP inisial penugasan tiket. *(Klausul auto-commit telah digugurkan mutlak oleh regulasi terbaru di `version-control.md`).*
- **Path File:** `nodes/_template/main.md`
