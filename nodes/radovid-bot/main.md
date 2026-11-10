> **STATUS MODE SAAT INI:** MODE 1 — Autonomous Planning
>
> **[PERINGATAN MASTER]** Ini adalah dokumen pertama yang WAJIB dibaca setiap kali memulai sesi baru atau menerima *Execution Prompt*. Dilarang mengeksekusi instruksi koding sebelum memuat seluruh konteks dari dokumen-dokumen rujukan di bawah ini.

AI Agent bertindak sebagai Arsitek Perangkat Lunak. Untuk menjaga konsistensi ekosistem, Anda **WAJIB** membaca dokumen fondasi dan mematuhi seluruh batasan berikut ke dalam memori kerja.

---

## 1. Batasan Ruang Lingkup & Isolasi Node

> **[ATURAN ISOLASI NODE - ANTI-CROSS CONTAMINATION]**
> Node ini terisolasi! AI Agent **DILARANG KERAS** memodifikasi berkas di luar direktori node ini (mengedit node lain) atau mengubah berkas di dalam direktori `global-*` kecuali diinstruksikan secara eksplisit oleh *Execution Prompt Multi-Node*. Pusatkan eksekusi hanya pada path lokal di dalam node ini.

---

## 2. Protokol Pemulihan Konteks (Context Recovery)

AI Agent **WAJIB MENGHENTIKAN SEMENTARA EKSEKUSI DAN MEMBACA ULANG** dokumen `main.md` ini beserta seluruh berkas referensi terkait apabila menemui salah satu dari 4 kondisi pemicu:
1. **Ditegur Pengguna:** Dikoreksi karena melanggar aturan, pedoman, atau konvensi ekosistem (seperti `coding.md`, `design-system.md`, dll.).
2. **Kehilangan Arah:** Mengalami kebingungan teknis, ragu menentukan langkah, atau mengulang galat yang sama.
3. **Inkonsistensi Terdeteksi:** Menyadari bahwa keluaran kode tidak selaras dengan keputusan arsitektur atau gaya desain sesi aktif.
4. **Sesi Baru / Konteks Terputus:** Memulai sesi baru, atau percakapan terputus akibat limit token maupun kendala koneksi.

### Prosedur Pemulihan Bertahap:
1. Hentikan eksekusi kode yang sedang berjalan seketika.
2. Baca ulang `main.md` ini secara menyeluruh.
3. Muat ulang berkas relevan dari Bagian 3 (minimal dokumen fondasi dan pedoman domain terkait).
4. Laporkan pemulihan konteks kepada pengguna sebelum melanjutkan pengerjaan.

---

## 3. Matriks Bacaan Referensi & Pustaka Pedoman

### 3.1 Dokumen Fondasi (Wajib Dipahami Utuh)
1. `../../global-docs/prd.md` (Spesifikasi fitur dan alur pengguna global)
2. `docs/system-design.md` (Arsitektur teknis spesifik node ini)
3. `../../global-docs/design-system.md` (Standar UI/UX, warna, dan tipografi global)
4. `docs/development-planning.md` (Peta jalan dan backlog pasangan tiket & rencana spesifik node ini)
5. `CHANGELOG.md` (Riwayat progres pengerjaan terakhir di node ini)

### 3.2 Pedoman Mutlak Operasional (Wajib Kepatuhan Penuh)
1. `../../global-guidelines/error-handling.md` (Aturan *Stop-and-Ask*, larangan inisiatif liar, batas percobaan, *Anti-Slop Debugging*)
2. `../../global-guidelines/security.md` (Larangan *hardcode secrets*, fail-fast env, sanitasi input)
3. `../../global-guidelines/safe-file-operations.md` (Larangan perintah destruktif & otorisasi mutasi state vs read-only)
4. `../../global-guidelines/coding.md` (Standar rekayasa kode, Zero-Comment, Fail-Fast, No Magic Numbers)
5. `guidelines/project-context.md` (Aturan khusus & hasil pemindaian sistem node ini)

### 3.3 Pedoman Kondisional Sesuai Domain Tugas
- **Menulis/mengubah kode logika:** Wajib baca `../../global-guidelines/coding.md` (termasuk kebijakan wajib relative path).
- **Modifikasi berkas massal, skrip bulk, atau Git per-file (`sed`, `git restore`, `git checkout <file>`):** Wajib baca `../../global-guidelines/safe-file-operations.md`.
- **Aktivitas Git (commit, branch, push, PR):** Wajib baca `../../global-guidelines/version-control.md`.
- **Deployment, konfigurasi CI/CD, rilis:** Wajib baca `../../global-guidelines/deployment.md` dan `../../global-guidelines/pipeline.md`.
- **Penulisan pengujian (unit/integration test):** Wajib baca `../../global-guidelines/testing.md` (*TDD Workflow* & *No Vanity Testing*).
- **Pengelolaan dependensi & lockfile:** Wajib baca `../../global-guidelines/dependencies.md`.
- **Skema basis data, migrasi, ERD, datetime:** Wajib baca `../../global-guidelines/database.md`.
- **Perancangan UI, slicing frontend, aset, i18n:** Wajib baca `../../global-guidelines/ui-and-assets.md` dan `../../global-guidelines/localization.md`.
- **Audit keamanan, review kerentanan & isolasi data:** Wajib baca `../../global-guidelines/security.md`.
- **Manajemen Tiket & Rencana Implementasi (Ticket & Plan):** Wajib baca `tickets/README.md` dan `plans/README.md` secara beriringan (menggunakan pasangan berkas `TICKET-[EPOCH_TIMESTAMP]` dan `PLAN-[EPOCH_TIMESTAMP]`). Untuk pelaporan bug, gunakan format `../../global-docs/templates/bug_report_template.md`.
- **Pembuatan prototipe antarmuka baru:** Wajib baca `prototypes/README.md`.
- **Terjebak galat berulang:** Wajib baca `retrospectives/RETROSPECTIVE.md`.
- **Validasi serah terima tugas (Done):** Wajib verifikasi `../../global-guidelines/delivery-gate.md`.
- **Eksplorasi kasus batas ekstrem (Hunt Loop):** Rujuk 9 sumbu di `../../global-guidelines/edge-case-atlas.md`.
- **Audit QA / exploratory testing:** Dokumentasikan ke `hunt/` dengan format `../../global-docs/templates/hunt-findings.md`.
- **Pengajuan PR & Pesan Komit:** Gunakan format `../../global-docs/templates/pull_request_template.md` & `../../global-docs/templates/commit_message_template.md`.
- **Tanya-jawab implementasi/arsitektur (Q&A):** Rujuk `../../global-docs/LEARN.md` (HANYA diisi jika pemicu resmi FASE 4 README.md; dilarang keras saat tugas koding biasa).
- **Navigasi & Analisis Dependensi Kode:** Wajib gunakan `graphify query` / `graphify build` di *Path Codebase*.

---

## 4. Klasifikasi Mode Operasional

AI Agent WAJIB memeriksa **STATUS MODE SAAT INI** di awal dokumen sebelum bekerja:
- **MODE 1 (Autonomous Planning):** Digerakkan roadmap. Tanyakan lingkup besar ke developer -> Perbarui `development-planning.md` -> Terbitkan pasangan tiket & rencana implementasi kosong (`TICKET-[EPOCH]` & `PLAN-[EPOCH]`) -> Minta persetujuan developer -> Eksekusi berurutan dan centang bila selesai.
- **MODE 2 (Prompt-Driven):** Digerakkan instruksi per langkah developer. Terima prompt -> Paparkan rencana implementasi teknis -> Eksekusi kode -> Catat pasangan tiket & rencana secara retrospektif di akhir tugas dan centang sendiri.

---

## 5. Prosedur Operasi Standar (SOP Eksekusi Sekuensial)

AI Agent WAJIB mematuhi alur kronologis berikut tanpa terkecuali setiap kali menerima penugasan:

### 5.1 Pemuatan Konteks Dasar (Context Ingestion Awal)
1. **[WAJIB BACA]** Buka berkas: `CHANGELOG.md`, `guidelines/project-context.md`, dan `docs/system-design.md`.
2. **Inspeksi Read-Only:** Diperbolehkan langsung mengeksekusi perintah pasif (`ls`, `view_file`, `cat`, `grep`, `git status`, `git log`, `SELECT` non-locking) untuk mempercepat pengumpulan konteks.

### 5.2 Penentuan & Pembacaan Pedoman Domain
1. Identifikasi domain tugas dan wajib baca dokumen terkait dari Bagian 3.3 sebelum merumuskan rencana (coding, database, UI, testing, safe-file-operations, deployment).

### 5.3 Pemetaan Relasi & Ketergantungan Kode (Graphify Navigation)
1. Periksa direktori `.graphify` di *Path Codebase* proyek asli:
   - **Jika ada:** Gunakan `graphify query` untuk memetakan fungsi pemanggil, komponen terdampak, dan dependensi.
   - **Jika belum ada:** Masuk ke *Path Codebase*, pastikan instalasi CLI tersedia, dan jalankan `graphify build`.
2. **[TERLARANG]** Menjalankan build, query, atau update graphify di dalam repositori orchestrator `ai-orchestrator-template/`.

### 5.4 Manajemen Beriringan Tiket & Rencana Implementasi (Ticket & Plan Pre-Mutation Gate)
1. **Prinsip Pasangan Beriringan (Inseparable Pair):** Setiap penugasan WAJIB memiliki pasangan berkas yang dibuat beriringan menggunakan timestamp UNIX epoch yang sama:
   - **Tiket (`tickets/TICKET-[EPOCH_TIMESTAMP]-[judul].md`):** Menyimpan *User Stories*, *Acceptance Criteria* (DoD), daftar berkas terdampak, dan *AI Execution Log*.
   - **Rencana Teknis (`plans/PLAN-[EPOCH_TIMESTAMP]-[judul].md`):** Menyimpan analisis arsitektur mendalam, pemetaan dependensi (Graphify), *step-by-step procedure*, dan *diff preview*.
2. **Alur Berdasarkan Mode:**
   - **MODE 1:** Buat pasangan tiket & rencana di awal berbasis `development-planning.md`, paparkan rencana, dan minta persetujuan sebelum mutasi state.
   - **MODE 2:** Paparkan rencana teknis sebelum mutasi state, eksekusi kode, lalu catat pasangan tiket & rencana secara retrospektif.
3. **[TERLARANG] Mutasi State Tanpa Otorisasi Rencana:** Dilarang meminta izin terminal untuk mutasi state (edit file, migrasi DDL, install paket, ubah `.env`) sebelum memaparkan dokumen rencana teknis (`plans/PLAN-[EPOCH]`) dan pratinjau diff kepada pengguna (patuhi `safe-file-operations.md`).
4. **Penyimpanan Permanen & Relative Path:** Berkas rencana disimpan permanen di `plans/` (bukan file sementara) dan ditautkan ke tiket pasangannya menggunakan relative path. Terapkan prinsip *Stop-and-Ask* pada `error-handling.md` jika menemui ambiguitas.
5. **[TERLARANG] Mutasi Berkas via Terminal Stdin/Skrip:** Dilarang membuat atau memodifikasi berkas menggunakan terminal heredoc/stdin (`cat << 'EOF'`, `echo >`, dsb.) maupun skrip otomasi (Python/Node/Bash). Seluruh pembuatan berkas WAJIB menggunakan tool native pembuatan berkas (contoh: `write_file`) dan modifikasi kode WAJIB menggunakan tool native file editor platform AI (contoh: `edit_file` pada Antigravity IDE) secara granular agar perubahan terekam dan memunculkan diff preview di IDE (patuhi `safe-file-operations.md`).

### 5.5 Pengerjaan Kode Berbasis TDD & Standar Anti-Slop
Eksekusi implementasi menggunakan siklus Test-Driven Development sesuai `testing.md`:
1. **Analisis DoD:** Petakan kriteria sukses dari *Acceptance Criteria* tiket tugas serta target spesifikasi pada rencana implementasi pasangannya (atau prompt eksplisit pengguna).
2. **Tahap RED:** Buat/perbarui unit test sebelum kode produksi. Buktikan tes gagal di terminal. Khusus perbaikan bug, wajib membuktikan tes baru **GAGAL pada kode lama** (Disiplin O-07).
3. **Tahap GREEN:** Tulis kode implementasi minimal sesuai standar `coding.md` hingga seluruh tes berstatus lulus.
4. **Tahap REFACTOR & FIX:** Jalankan test suite, perbaiki regresi hingga 100% PASS dan memenuhi target coverage.
5. **Tahap QUALITY CHECK:** Jalankan linter dan type checker; pastikan nol peringatan dan nol galat.
6. **Tahap HUNT LOOP:** Eksekusi minimal 1 putaran Hunt Loop (minimal 10 kasus ekstrem berbasis `edge-case-atlas.md`, reproduksi, jadikan tes permanen, hingga nol temuan baru).

### 5.6 Verifikasi Uji Visual Antarmuka (UI Review Gate)
Khusus tugas yang memodifikasi visual frontend atau mobile:
1. Periksa tampilan render visual aktual (melalui prototipe atau browser preview).
2. Validasi Anti-Slop Visual Checklist pada `ui-and-assets.md` (kontras WCAG AA, target sentuh 24x24px / 44x44pt, reflow 320px, kelengkapan status loading/empty/error).
3. Evaluasi Gerbang Akhir: pangkas 20% dekorasi jika tidak menambah kejelasan fungsi.

### 5.7 Verifikasi Delivery Gate & Penyelesaian Tiket & Rencana
Sebelum menandai pasangan tiket & rencana menjadi `Done` / `Completed` atau menyerahkan pekerjaan:
1. Verifikasi seluruh butir pemeriksaan pada [delivery-gate.md](../../global-guidelines/delivery-gate.md) (4 Pilar: Higienitas Kode, Hard Gate Testing, Must Probe Loop, dan Tata Kelola).
2. Ubah status tiket menjadi `status: Done` dan status rencana menjadi `status: Completed`.
3. Tandai seluruh checklist `[x]` pada *Acceptance Criteria* tiket dan verifikasi kelengkapan ringkasan perubahan pada rencana implementasi.
4. Isi seksi `AI Execution Log & Output` lengkap pada tiket serta perbarui status rencana pasangannya (ringkasan delivery gate, bukti eksekusi terminal, evaluasi UI, dan *Untested List*).
5. Sinkronkan pasangan tiket & rencana dengan GitHub Projects (jika fitur GitHub Project diaktifkan).

### 5.8 Pencatatan Riwayat Changelog
1. Sisipkan catatan baru di baris **PALING ATAS** daftar (*descending*) pada `CHANGELOG.md` node terkait.
2. Catat seluruh jenis perubahan (kode, pedoman, dokumentasi, pasangan tiket & rencana).
3. Wajib menyalin struktur baku dari `../../global-docs/templates/changelog_entry_template.md`.

### 5.9 Disiplin Kontrol Versi (Git)
1. **[TERLARANG]** Melakukan auto-commit atau auto-push tanpa izin eksplisit pengguna.
2. **Pesan Komit Bersih:** Pesan komit di direktori Project/Node WAJIB 100% BERSIH dari istilah orchestrator, tiket, rencana, PRD, atau guideline sesuai `version-control.md`.

### 5.10 Sinkronisasi Konteks Akhir (Graphify Update)
1. Setelah komit selesai, masuk ke *Path Codebase* (`cd`) dan jalankan `graphify update` untuk menyinkronkan Knowledge Graph dengan kode terbaru.
2. **[TERLARANG]** Menjalankan `graphify update` di dalam repositori orchestrator.
