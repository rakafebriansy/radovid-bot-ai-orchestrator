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
- **`Development Planning: <nomor>`**: Pembaruan peta jalan atau antrean pasangan tiket & rencana implementasi.
- **`Ticket & Plan: <epoch>`**: Pengerjaan fitur atau perbaikan berbasis pasangan tiket dan rencana implementasi spesifik.
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

### [2026-11-06 08:14:00] - Feat: Integration of Hallmark Design & Craft Standards into Global UI Guidelines
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Mengintegrasikan standar kualitas desain dan ketahanan rekayasa frontend dari repositori `hallmark` ke dalam `ai-orchestrator-template` dengan hanya mengambil insight penting yang belum ada (tanpa duplikasi).
- **Perubahan:**
  - `[Changed]` Memperkaya [global-guidelines/ui-and-assets.md](../../global-guidelines/ui-and-assets.md) dengan standar baru dari `hallmark`: *Surface Ladder* 4 tingkat & invarian halaman (maksimal 1 gaya tombol primary berisi, larangan menggabungkan border + wide shadow), geometri kanvas & *screen word budget* (~80 kata per 900px, rasio spasi atas judul $\ge 2\times$ spasi bawah, alternatif pembuka seksi tanpa *eyebrow* mekanis, anatomi berbasis tugas), hirarki bukti visual & larangan bingkai browser palsu / ilustrasi SVG dekoratif / *fake form send*, sistem warna OKLCH (*tinted neutrals* tanpa `#000`/`#fff` murni, anggaran aksen $\le 5\%–8\%$), re-derivasi Dark Mode via tingkat kecerahan (*lightness*) tanpa *drop shadow*, standar visualisasi data jujur, stabilitas dimensi input & cadangan ruang `1lh`, pola *Silent Success* & *Undo over Confirm*, ketahanan CSS (`overflow-x: clip`, `dvh`/`svh`, `minmax(0, 1fr)`), transisi khusus `transform`/`opacity`, anggaran tekstur/koreografi scroll, serta disiplin mempertahankan *wiring* saat redesign.
  - `[Changed]` Menyelaraskan 8 komponen spesifikasi wajib pada [global-docs/design-system.md](../../global-docs/design-system.md) dengan token OKLCH, re-derivasi Dark Mode, *Surface Ladder*, radius konsentris, dan aturan visualisasi data.
  - `[Changed]` Menambahkan penghargaan kredit `https://github.com/nutlope/hallmark` pada [README.md](../../README.md).
- **Path File:** `global-guidelines/ui-and-assets.md`, `global-docs/design-system.md`, `README.md`, `nodes/_template/CHANGELOG.md`

### [2026-11-06 07:48:00] - Refactor: Conditional GitHub Project Initialization & Consolidation of Skills into Global Guidelines
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Mengubah inisialisasi GitHub Project pada Fase 1 Startup Prompt menjadi opsi kondisional (ditanyakan kepada pengguna seperti Graphify) serta mengonsolidasikan seluruh insight unik dari direktori `skills/` (`frontend-design` dan `security-audit`) langsung ke dalam `global-guidelines/` tanpa duplikasi.
- **Perubahan:**
  - `[Changed]` Memperbarui Fase 1 Startup Prompt (Opsi A Single-Project & Opsi B Multi-Project) pada [README.md](../../README.md), [nodes/_template/main.md](./main.md), dan [nodes/_template/tickets/README.md](./tickets/README.md) agar AI Agent menanyakan persetujuan penggunaan GitHub Project terlebih dahulu sebelum menginisialisasinya secara kondisional.
  - `[Changed]` Menggabungkan insight baru non-duplikat dari `skills/frontend-design/` ke dalam [global-guidelines/ui-and-assets.md](../../global-guidelines/ui-and-assets.md) (identitas berakar domain, proses perancangan dua tahap, prinsip *restraint*, kalibrasi anti-klise AI terbaru, batas panjang baris `< 80ch`, larangan *typographic AI tells*, spesifisitas selektor CSS, dan kohesi kosakata alur).
  - `[Changed]` Menggabungkan insight baru non-duplikat dari `skills/security-audit/` ke dalam [global-guidelines/security.md](../../global-guidelines/security.md) (standar kriptografi CSPRNG & *constant-time comparison*, *second-order injection*, SSRF redirect & TOCTOU, keamanan sesi/JWT/OAuth/SAML/MFA/Passkey/mTLS, isolasi data multi-tenant & *derived copies*, keamanan *client-side*/RPC/AI-LLM/native IPC/cloud/CI supply chain, serta protokol audit keamanan defensif 6 elemen).
  - `[Changed]` Menghapus ketergantungan tautan eksternal ke `skills/` pada [nodes/_template/main.md](./main.md) dan [global-docs/design-system.md](../../global-docs/design-system.md) sehingga `global-guidelines/` menjadi sumber kebenaran tunggal (*Single Source of Truth*).
- **Path File:** `README.md`, `global-guidelines/ui-and-assets.md`, `global-guidelines/security.md`, `global-docs/design-system.md`, `nodes/_template/main.md`, `nodes/_template/tickets/README.md`, `nodes/_template/CHANGELOG.md`

### [2026-11-01 11:28:00] - Policy: Prohibition of Terminal Heredoc (CAT EOF) & Mutation Scripts
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Penegakan hukum besi pelarangan pembuatan/pengeditan berkas melalui terminal stdin/heredoc (`cat << 'EOF'`, `echo >`) dan skrip massal (Python/Node/Bash) guna menjamin seluruh perubahan ter-record di preview visual diff platform IDE (contoh: Antigravity IDE).
- **Perubahan:**
  - `[Added]` Menambahkan Subseksi 2.3 pada [global-guidelines/safe-file-operations.md](../../global-guidelines/safe-file-operations.md) berisi katalog larangan heredoc/stdin (`cat << 'EOF' > ...`), rasional teknis, contoh kode terlarang, dan kewajiban menggunakan tool native editor AI (contoh: `write_file` / `edit_file` pada Antigravity).
  - `[Changed]` Memperbarui [global-guidelines/testing.md](../../global-guidelines/testing.md) Seksi 1.1 melarang pembuatan skrip uji siluman lewat heredoc lalu dihapus via `rm`.
  - `[Changed]` Menambahkan butir larangan mutasi terminal pada SOP 5.4 di [nodes/_template/main.md](../main.md).
- **Path File:** `global-guidelines/safe-file-operations.md`, `global-guidelines/testing.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-11-01 10:15:00] - Refactor: Complete Purge of Orchestrator Terms from Git Policy & Tandem Enforcement
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Menghapus sepenuhnya subseksi alur tiket & rencana dari pedoman kontrol versi Git (`version-control.md`) agar tidak menimbulkan kerancuan dengan aturan sterilitas Git message/PR, serta menuntaskan penyebutan berpasangan tandem di seluruh pedoman pendukung.
- **Perubahan:**
  - `[Removed]` Mengeliminasi Subseksi 4.1 (*Siklus Pengerjaan Berbasis Tiket & Rencana*) dari [global-guidelines/version-control.md](../../global-guidelines/version-control.md), sehingga Seksi 4 murni menjadi Kebijakan Pengajuan Pull Request (PR Policy) tanpa ada kebocoran instruksi pencatatan ID internal.
  - `[Changed]` Membersihkan penyebutan istilah 'tiket' pada klausul kontrol versi di [global-guidelines/version-control.md](../../global-guidelines/version-control.md) (aturan branch, granularitas komit, dan pencatatan changelog).
  - `[Changed]` Menyelaraskan seluruh penyebutan tugas pada [global-guidelines/testing.md](../../global-guidelines/testing.md), [global-guidelines/delivery-gate.md](../../global-guidelines/delivery-gate.md), [global-guidelines/safe-file-operations.md](../../global-guidelines/safe-file-operations.md), dan [nodes/_template/main.md](../main.md) agar selalu menyebut pasangan tiket dan rencana secara beriringan (*tandem*).
- **Path File:** `global-guidelines/version-control.md`, `global-guidelines/testing.md`, `global-guidelines/delivery-gate.md`, `global-guidelines/safe-file-operations.md`, `nodes/_template/main.md`, `nodes/_template/CHANGELOG.md`

### [2026-11-01 10:05:00] - Refactor: Git Message Sterility & Inseparable Ticket-Plan Tandem Pairing
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Pembersihan mutlak pesan Git (Pull Request & Commit) dari istilah internal orchestrator dan penegakan prinsip penyebutan beriringan tanpa redundansi untuk Tiket & Rencana Implementasi (*Ticket & Plan*).
- **Perubahan:**
  - `[Changed]` Menulis ulang [global-docs/templates/pull_request_template.md](../../global-docs/templates/pull_request_template.md) menjadi template standar produksi yang 100% steril dari istilah internal orchestrator (`ticket`, `plan`, `prd`, `guideline`, dsb.).
  - `[Changed]` Memperbarui [global-guidelines/version-control.md](../../global-guidelines/version-control.md) untuk menghapus instruksi pengaitan tiket internal pada deskripsi PR produksi serta menegaskan kebersihan riwayat Git.
  - `[Changed]` Memastikan seluruh penyebutan tiket dan rencana implementasi (*Ticket & Plan*) berpasangan secara beriringan (*inseparable pair*) di `README.md`, `main.md`, `development-planning.md`, `tickets/README.md`, `plans/README.md`, `testing.md`, `delivery-gate.md`, dan `coding.md`.
  - `[Changed]` Mengeliminasi pemanggilan redundan pada `nodes/_template/main.md` dengan menyatukan tata kelola tiket dan rencana ke dalam satu subseksi terpadu (SOP 5.4 Pre-Mutation Gate).
- **Path File:** `global-docs/templates/pull_request_template.md`, `global-docs/templates/README.md`, `global-guidelines/version-control.md`, `global-guidelines/delivery-gate.md`, `global-guidelines/testing.md`, `global-guidelines/coding.md`, `nodes/_template/main.md`, `nodes/_template/docs/development-planning.md`, `nodes/_template/tickets/README.md`, `nodes/_template/plans/README.md`, `README.md`, `nodes/_template/CHANGELOG.md`

### [2026-11-01 09:45:00] - Refactor: Dedicated Plans Directory, Epoch-Based Tickets, Relative Path Policy, & Skill Integrations
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `ai-orchestrator-template`
- **Konteks:** Pemisahan direktori rencana teknis (`plans/`) dari tiket, konversi penamaan tiket berbasis Epoch timestamp, penegakan Relative Path Policy, serta integrasi keahlian frontend-design dan security-audit.
- **Perubahan:**
  - `[Added]` Membuat direktori `nodes/_template/plans/` dan panduan `plans/README.md` untuk menyimpan rencana implementasi teknis mendalam secara terpisah dari tiket tugas.
  - `[Added]` Menambahkan template `global-docs/templates/plan_template.md` dan memperbarui `global-docs/templates/README.md`.
  - `[Changed]` Mengonversi format penamaan tiket dari `TICKET-01-` menjadi `TICKET-[EPOCH_TIMESTAMP]-[judul].md` untuk mencegah tabrakan ID pada pengerjaan multi-agent, serta memperbarui seluruh berkas template dan pedoman terkait.
  - `[Changed]` Memperketat batasan konten tiket di `tickets/README.md` agar tetap ringkas (User Stories dan Acceptance Criteria) dan menautkan rincian teknis ke berkas di `plans/`.
  - `[Changed]` Menegakkan klausul wajib Relative Path (`Relative Path Policy`) pada `global-guidelines/coding.md`, `safe-file-operations.md`, `README.md`, `main.md`, dan seluruh template.
  - `[Added]` Mengintegrasikan keahlian resmi Anthropic `skills/frontend-design/` ke dalam ekosistem dan menautkannya pada `ui-and-assets.md`, `design-system.md`, dan `main.md`.
  - `[Added]` Mengintegrasikan paket keahlian Altamis Atmaja `skills/security-audit/` dan menautkannya pada `security.md` dan `main.md`.
  - `[Added]` Menambahkan seksi apresiasi dan kredit di bagian bawah `README.md` untuk Anthropic `frontend-design`, Altamis Atmaja `security-audit-skill`, serta Altamis Atmaja `outofthebox`.
- **Path File:** `README.md`, `nodes/_template/plans/*`, `nodes/_template/tickets/*`, `nodes/_template/main.md`, `global-guidelines/*`, `global-docs/*`, `skills/*`

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
