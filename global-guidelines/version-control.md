# Mandatory Changelog & Version Control Policy

> **[PRINSIP UTAMA]** Standar tata kelola kontrol versi, kedisiplinan pencatatan riwayat pembaruan, isolasi percabangan Git, dan protokol penyerahan kode (*Pull Request*) bagi AI Agent.

---

## 1. Tata Kelola Lingkungan & Percabangan Git (Branching Governance)

### 1.1 Decision Gate: Identifikasi Lingkungan Kerja
Sebelum mengeksekusi aksi Git apa pun (commit, branch, push, PR), AI Agent **WAJIB MUTLAK** mengidentifikasi lingkup direktori kerja:
1. **Repositori Orchestrator (`ai-orchestrator-template/`):**
   - Mencakup `global-docs/`, `global-guidelines/`, `nodes/`, dan berkas template orkestrasi.
   - Seluruh perubahan langsung di-push ke cabang `main`.
   - **[TERLARANG]** Membuat branch baru di repositori orchestrator.
2. **Repositori Project/Node (Source Code Aplikasi Asli):**
   - Mencakup kode sumber aplikasi mandiri (`frontend-app/`, `backend-api/`, dsb.) yang berada di luar repositori orchestrator.
   - Wajib mengikuti prosedur percabangan fitur terisolasi.

### 1.2 Aturan Penamaan Branch
1. **Di Direktori Project/Node:**
   - **[TERLARANG]** Menggunakan format ID tiket kapital seperti `feat(TICKET-01)`.
   - **[WAJIB]** Gunakan deskripsi fitur murni tanpa istilah ekosistem orchestrator (contoh: `feat(feature-name)` atau `feat(user-authentication)`).
2. **Di Repositori Orchestrator:**
   - Tidak perlu membuat branch baru; seluruh pembaruan dikerjakan langsung pada `main`.

### 1.3 Prosedur Konfirmasi Pembuatan Branch Baru
Prosedur ini HANYA berlaku di direktori **Project/Node**:
1. Evaluasi apakah tugas berada pada domain fitur baru atau konteks terpisah (*hotfix*, *refactor*, penulisan *test*) dari branch aktif.
2. Jangan membuat branch baru untuk setiap tiket jika tugas masih berada dalam satu domain fitur yang sama.
3. Jika domain fitur terbukti berbeda dan membutuhkan branch baru, minta persetujuan pengguna terlebih dahulu sebelum mengeksekusi `git checkout -b type/nama-branch`.

---

## 2. Tata Kelola Komit, Validasi, & Rollback

### 2.1 Standar Pesan Komit Semantik (Semantic Commits)
1. **[WAJIB]** Selaraskan pesan komit dengan template baku pada `global-docs/templates/commit_message_template.md`.
2. Gunakan konvensi tipe terstruktur (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`). Dilarang menulis pesan komit yang ambigu atau tidak deskriptif.

### 2.2 Larangan Istilah Orchestrator pada Kode Aplikasi
Terminologi ekosistem orchestrator meliputi: *ticket*, *prd*, *design system*, *guideline*, *system design*, *retrospective*, *prototype*, *node*, *orchestrator*, *template*.
1. **Di Project/Node (Aplikasi Produksi):**
   - **[TERLARANG]** Menyisipkan istilah orchestrator atau ID tiket ke dalam pesan komit, nama branch, dan judul/deskripsi PR.
   - `[SALAH]`: `feat(node-auth): implement login based on prd guideline (TICKET-01)`
   - `[SALAH]`: `fix(ticket-03): resolve auth looping issue in orchestrator`
   - `[BENAR]`: `feat(auth): add jwt authentication and refresh token handler`
   - `[BENAR]`: `fix(auth): handle expired token on api client interceptor`
2. **Di Repositori Orchestrator:**
   - **[WAJIB]** Gunakan istilah orchestrator sebagai cakupan (*scope*) komit (contoh: `docs(template): update`, `docs(orchestrator): update version control guideline`).

### 2.3 Larangan Eksekusi Git Otonom (Mandatory User Approval)
1. **[TERLARANG]** Melakukan auto-commit, `git add`, atau `git push` secara otonom tanpa izin eksplisit pengguna.
2. **[WAJIB]** Paparkan daftar berkas yang diubah beserta penjelasannya kepada pengguna sebelum meminta konfirmasi `git add` atau komit.
3. Komit wajib granular per-tugas/tiket; dilarang menumpuk banyak tugas ke dalam satu komit raksasa.

### 2.4 Gerbang Validasi Pra-Komit & Pra-Push (Pre-Flight Gate)
Sebelum meminta izin komit atau push kepada pengguna, tuntaskan validasi sesuai [testing.md](./testing.md):
1. **Backend / API:** Seluruh *unit test* wajib 100% PASS tanpa galat.
2. **Frontend / Mobile:** *Linter*, *type checker*, dan *test suite* wajib lolos 100%.
3. **[TERLARANG]** Mengajukan izin komit jika validasi prasyarat ini belum dieksekusi atau masih menyisakan peringatan/galat kompilasi.

### 2.5 Mekanisme Rollback & Pembatalan Perubahan
1. Jika terjadi galat fatal dan pengguna menginstruksikan pembatalan tugas terakhir, terapkan rollback ke komit stabil sebelumnya via `git reset --hard HEAD~1` (atau target commit hash) setelah mendapat konfirmasi persetujuan pengguna.
2. **[TERLARANG]** Menjalankan pembatalan per-berkas (`git checkout -- <file>` atau `git restore <file>`) tanpa review eksplisit pengguna. Regulasi ini mengacu secara mutlak pada [safe-file-operations.md](./safe-file-operations.md).

---

## 3. Disiplin Dokumentasi Changelog

### 3.1 Kewajiban Pencatatan Universal
1. **[WAJIB]** Setiap penambahan kode, perbaikan bug, atau konfigurasi wajib dicatat secara rinci ke dalam `CHANGELOG.md` milik node terkait.
2. Kewajiban ini berlaku mutlak untuk seluruh jenis perubahan: kode implementasi, `README.md`, pedoman (`guidelines/`), desain sistem, dan pembaruan tiket.

### 3.2 Urutan Kronologis Terbalik (Reverse-Chronological)
1. **[WAJIB]** Tempatkan entri pembaruan terbaru di bagian PALING ATAS daftar (urutan *descending*, tepat di bawah header utama dokumen).
2. **[TERLARANG]** Menambahkan entri baru di bagian terbawah dokumen.

### 3.3 Metadata Wajib Entri
1. Setiap entri wajib mencantumkan nama branch Git yang aktif saat pengerjaan beserta tautan (*link*) repositori VCS terkait.

### 3.4 Standar Format Log Pembaruan
1. Gunakan format daftar terstruktur (*structured list*); hindari penyajian log dalam bentuk tabel.
2. **[WAJIB]** Patuhi struktur baku yang tercantum pada `global-docs/templates/changelog_entry_template.md`.

---

## 4. Alur Kerja Tiket & Kebijakan Pull Request (PR)

### 4.1 Siklus Pengerjaan Berbasis Tiket (Ticket-Driven Development)
1. **Rujuk Tiket Terlebih Dahulu:** Baca panduan struktural tiket pada `nodes/[nama-node]/tickets/README.md`. Buat tiket baru menggunakan boilerplate resmi, atau gunakan `global-docs/templates/bug_report_template.md` untuk tiket pelaporan bug.
2. **Pengujian Mandatori:** Uji kode fungsional segera setelah perombakan logika selesai.
3. **Siklus Status Tiket:** Ubah frontmatter `status: Todo` menjadi `status: In Progress` saat pengerjaan dimulai. Ubah menjadi `status: Done` dan centang seluruh checkbox `[x]` pada *Acceptance Criteria* hanya setelah pengujian lulus tuntas.
4. **Dokumentasi AI Execution Log:** Catat bukti eksekusi teknis, pertimbangan arsitektur, dan daftar hal yang belum diuji pada seksi `AI Execution Log & Output` di bagian bawah tiket.
5. **Referensi Tiket:** Cantumkan ID tiket (misal: "Referensi: TICKET-01") pada laporan kemajuan di chat dan pada `CHANGELOG.md` node.

### 4.2 Kebijakan Pengajuan Pull Request (PR Policy)
Jika pengembangan melibatkan penggabungan kode lintas cabang:
1. **Wajib Menggunakan Template PR:** Isi deskripsi PR menggunakan boilerplate baku pada `global-docs/templates/pull_request_template.md`.
2. **Sintaksis Penutup Tiket Otomatis:** Cantumkan sintaksis `Resolves [TICKET-ID]` (contoh: `Resolves TICKET-01`) di dalam deskripsi PR untuk memicu sinkronisasi status papan proyek GitHub.
3. **[TERLARANG] Auto-Merge:** AI Agent HANYA bertugas mengajukan PR (`gh pr create`). DILARANG mengeksekusi merge (`gh pr merge`, `git merge`) secara otonom.
4. **Tindakan Pasca Pembuatan PR:** Laporkan URL PR kepada pengguna dan BERHENTI. Tunggu instruksi pengguna apakah mereka akan meninjau mandiri atau memberikan izin eksplisit merge kepada AI.
