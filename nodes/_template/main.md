# AI Orchestrator: Master Entrypoint

> 🟢 **STATUS MODE SAAT INI:** [PILIH: MODE 1 / MODE 2]

> ⚠️ **PERHATIAN UNTUK AI AGENT:**
> Ini adalah dokumen pertama yang WAJIB Anda baca setiap kali memulai sesi baru atau menerima *Execution Prompt*. Jangan mengeksekusi instruksi koding pengguna sebelum Anda memahami konteks dari dokumen-dokumen di bawah ini!

Tugas Anda sebagai AI Agent bukanlah sekadar *"code generator"*, melainkan seorang Arsitek Perangkat Lunak. Untuk menjaga konsistensi proyek, Anda **WAJIB** membaca file-file fondasi berikut ke dalam konteks memori Anda.

---

## BAGIAN A — BATASAN RUANG LINGKUP

> 🛑 **ANTI-CROSS CONTAMINATION:**
> Node ini terisolasi! Anda **DILARANG KERAS** memodifikasi file di luar direktori *node* ini (seperti mengedit node lain) atau mengubah file di dalam direktori `global-*` kecuali diinstruksikan secara eksplisit oleh *Execution Prompt Multi-Node*. Fokuslah hanya pada *path* lokal di dalam node ini.

---

## BAGIAN B — PROTOKOL PEMULIHAN KONTEKS (CONTEXT RECOVERY)

> 🔄 **MEKANISME WAJIB BACA ULANG:**
> Protokol ini adalah mekanisme keselamatan untuk mencegah degradasi kualitas akibat hilangnya konteks (*context loss*) di tengah percakapan panjang.

**Anda WAJIB SECARA OTOMATIS menghentikan sementara eksekusi dan membaca ulang seluruh dokumen ini (`main.md`) beserta file-file referensi yang relevan di dalamnya** jika salah satu kondisi berikut terpenuhi:

1. **Ditegur Pengguna:** Anda ditegur atau dikoreksi oleh pengguna karena melanggar aturan, pedoman, atau konvensi yang sudah ditetapkan di ekosistem ini (seperti melanggar `coding.md`, `design-system.md`, dll).
2. **Kehilangan Arah:** Anda merasa kebingungan, tidak yakin dengan langkah selanjutnya, atau mulai mengulangi kesalahan yang sama di tengah pengerjaan.
3. **Inkonsistensi Terdeteksi:** Anda menyadari bahwa output yang Anda hasilkan tidak konsisten dengan keputusan arsitektural atau gaya desain yang sudah ditetapkan sebelumnya di sesi yang sama.
4. **Sesi Baru atau Konteks Terputus:** Anda memulai sesi percakapan baru, atau percakapan sebelumnya terputus karena batas token atau gangguan jaringan.

**Prosedur Pemulihan:**
1. Hentikan eksekusi kode yang sedang berjalan.
2. Baca ulang `main.md` ini secara utuh.
3. Baca ulang file-file yang relevan dengan tugas saat ini dari **Bagian C** di bawah (minimal: file fondasi yang bersangkutan dan pedoman mutlak terkait).
4. Laporkan kepada pengguna bahwa Anda telah memulihkan konteks dan siap melanjutkan.

---

## BAGIAN C — DAFTAR BACAAN REFERENSI

### C.1: Dokumen Fondasi (Wajib Dibaca Seluruhnya)
File-file ini adalah nyawa dari ekosistem proyek ini. Anda harus memahaminya untuk mengetahui fitur global apa yang dibangun dan bagaimana antarmukanya dirancang.
1. `../../global-docs/prd.md` (Spesifikasi fitur dan alur pengguna global)
2. `docs/system-design.md` (Arsitektur teknis spesifik node ini)
3. `../../global-docs/design-system.md` (Aturan UI/UX, warna, tipografi global)
4. `docs/development-planning.md` (Peta jalan spesifik node ini)
5. `CHANGELOG.md` (Untuk mengetahui progres terakhir di node ini)

### C.2: Pedoman Mutlak (Wajib Dibaca Seluruhnya)
Hukum besi operasional Anda. Pelanggaran terhadap pedoman ini akan merusak integritas sistem.
1. `../../global-guidelines/error-handling.md` (Aturan *Stop-and-Ask*, larangan inisiatif liar, batas percobaan)
2. `../../global-guidelines/security.md` (Larangan *hardcode API keys*)
3. `guidelines/project-context.md` (Aturan khusus & hasil pemindaian sistem dari node ini)

### C.3: Dokumen Kondisional (Baca Saat Dibutuhkan Saja)
Jangan buang token Anda untuk membaca file ini jika instruksi pengguna tidak berkaitan dengannya.
*   **Akan menulis atau memodifikasi source code (coding)?** Baca `../../global-guidelines/coding.md` (khususnya: **ZERO-COMMENT POLICY** dan **No Hacks**).
*   **Akan mengedit file secara massal, menjalankan script perubahan, atau menggunakan perintah Git yang memodifikasi file (termasuk `sed`, `git restore`, `git checkout <file>`)?** Baca `../../global-guidelines/safe-file-operations.md`. Pelanggaran terhadap pedoman ini menghilangkan kesempatan review pengguna secara permanen.
*   **Akan melakukan aktivitas Git (commit, branch, push, pengelolaan tiket)?** Baca `../../global-guidelines/version-control.md`.
*   **Akan mendeploy aplikasi, mengkonfigurasi CI/CD, atau melakukan rilis/version bump?** Baca `../../global-guidelines/deployment.md` dan `../../global-guidelines/pipeline.md`.
*   **Akan menulis unit test?** Baca `../../global-guidelines/testing.md`.
*   **Akan membuat/mengelola dependensi?** Baca `../../global-guidelines/dependencies.md`.
*   **Akan merancang skema basis data, migrasi, ERD, atau menyimpan data datetime/waktu?** Baca `../../global-guidelines/database.md`.
*   **Akan merancang UI, mengelola aset, melakukan slicing pada frontend, atau menambahkan bahasa?** Baca `../../global-guidelines/ui-and-assets.md` dan `../../global-guidelines/localization.md`.
*   **Akan membuat sketsa prototipe tampilan baru?** Baca `prototypes/README.md`.
*   **Akan mengambil, membaca, atau membuat tiket tugas?** Baca `tickets/README.md`.
*   **Terjebak error yang sama berkali-kali?** Baca `retrospectives/RETROSPECTIVE.md` untuk melihat apakah AI sebelumnya pernah memecahkan masalah ini di node ini.
*   **Ditugaskan membuat tiket Bug?** Baca `../../global-docs/templates/bug_report_template.md`.
*   **Ditugaskan membuat deskripsi PR/Commit?** Baca `../../global-docs/templates/pull_request_template.md` & `../../global-docs/templates/commit_message_template.md`.
*   **Ditugaskan menjawab pertanyaan pengguna seputar implementasi/arsitektur (Q&A)?** Baca dan patuhi protokol di `../../global-docs/LEARN.md` (HANYA diisi jika pengguna menggunakan format template resmi FASE 4 dari `README.md`; DILARANG KERAS mengisi file ini pada instruksi biasa, pertanyaan kasual, atau tugas koding!).
*   **Pengerjaan & Navigasi Kode (Graphify):** Anda **WAJIB** menggunakan `graphify query` untuk navigasi arsitektur dan pelacakan fungsi/dependensi. Jika direktori `.graphify` belum ada di *Path Codebase*, Anda **WAJIB** men-generate-nya terlebih dahulu via `graphify build` di *Path Codebase*.

---

## BAGIAN D — MODE OPERASIONAL

Ekosistem ini beroperasi dalam salah satu dari dua mode. Anda **WAJIB** mengecek **STATUS MODE SAAT INI** di bagian paling atas dokumen ini sebelum bekerja.

- **MODE 1 (Autonomous Planning):** Digerakkan oleh *roadmap*. Anda harus bertanya kepada developer apa yang harus dikerjakan secara garis besar -> Anda memperbarui `development-planning.md` dan mencetak tiket-tiket kosong yang belum dicentang -> Anda berhenti dan meminta koreksi developer -> Jika disetujui, Anda mengeksekusi semua tiket tersebut secara berurutan dan mencentangnya bila berhasil.
- **MODE 2 (Prompt-Driven):** Digerakkan oleh instruksi per-langkah dari developer. Developer memberi *prompt* instruksi -> Anda mengeksekusi kode -> Setelah selesai, Anda secara otomatis dan retrospektif membuat tiket baru untuk instruksi tersebut dan langsung mencentangnya sendiri.

---

## BAGIAN E — STANDARD OPERATING PROCEDURE (SOP) EKSEKUSI

Anda **DIWAJIBKAN SECARA MUTLAK** untuk mematuhi alur kerja berikut tanpa terkecuali setiap kali menerima *Execution Prompt* atau penugasan:

### E.1: Membaca Changelog (Wajib Awal)
Anda **WAJIB SELALU** membaca `CHANGELOG.md` terlebih dahulu untuk memahami konteks dan progres terakhir sebelum melakukan eksekusi apa pun.

### E.2: Kewajiban Penggunaan & Generasi Graphify (Wajib)
Sebelum melakukan pemindaian manual atau pengerjaan instruksi, Anda **WAJIB** mengecek apakah terdapat direktori `.graphify` di dalam *Path Codebase* (direktori proyek asli) dari node ini.

1. **JIKA DITEMUKAN (`.graphify` ada):** Anda **WAJIB** menggunakan fitur CLI `graphify` (contoh: `graphify query`) di dalam direktori *Path Codebase* tersebut untuk memahami alur kode, melacak relasi pemanggil (*callers*), menelusuri dependensi antarmodul, dan mengidentifikasi komponen terdampak alih-alih membaca puluhan file secara manual.
2. **JIKA TIDAK DITEMUKAN (`.graphify` belum ada):** Anda **WAJIB** men-generate-nya terlebih dahulu! Masuk ke direktori *Path Codebase* (`cd`), pastikan `graphify --version` terpasang (instal via `npm install -g @sentropic/graphify` jika belum ada), dan jalankan perintah `graphify build` untuk membangun Knowledge Graph node ini. Setelah selesai di-generate, gunakan hasilnya untuk menavigasi kode.
3. **Wajib Dipakai Selama Pengerjaan:** Gunakan hasil kueri Graphify ini sebagai landasan utama penyusunan *Implementation Plan* (E.5) dan saat memodifikasi kode (E.6) agar tidak memutus relasi antar-komponen yang terpetakan.
4. **Larangan Mutlak di Repositori Orchestrator:** Direktori `.graphify`, generasi graf (`graphify build`), kueri, maupun pembaruan (`graphify update`) **HANYA** boleh berada dan dieksekusi di dalam *Path Codebase* proyek/node asli. Anda **DILARANG KERAS** mengeksekusi `graphify build`, `graphify init`, atau membuat folder `.graphify` di dalam repositori `ai-orchestrator-template/`.

### E.3: Berhenti & Bertanya (Stop & Ask)
Rujuk dan patuhi secara mutlak seluruh aturan di `../../global-guidelines/error-handling.md`, khususnya seksi **"Stop-and-Ask (Anti-Looping)"** dan **"Larangan Inisiatif Liar (No Wild Initiative)"**.

### E.4: Manajemen Tiket (Tergantung Mode)
Rujuk dan patuhi secara mutlak seluruh aturan di `../../global-guidelines/version-control.md` seksi **"Ticket-Driven Development Workflow"** dan format boilerplate di `tickets/README.md`.
- Jika Anda berada di **MODE 1**, tiket dibuat di awal sebelum eksekusi berdasarkan `development-planning.md`.
- Jika Anda berada di **MODE 2**, tiket dibuat di akhir eksekusi sebagai rekam jejak (*retrospective*).
- Anda **WAJIB MUTLAK** menyalin utuh struktur `Boilerplate (Templat)` dari `tickets/README.md`. DILARANG mengarang format *markdown* sendiri.

### E.5: Pembuatan Implementation Plan (Wajib)
Anda **DIWAJIBKAN MUTLAK** untuk membuat rencana implementasi (*implementation plan*) yang detail mengenai apa yang akan dikerjakan, dan menunggu persetujuan pengguna sebelum mengeksekusi kode atau membuat perubahan file apa pun. Anda **WAJIB** memanfaatkan hasil `graphify query` (generate terlebih dahulu via `graphify build` jika belum ada sesuai SOP E.2) untuk memetakan file-file sasaran yang terdampak (*affected files*) dan ketergantungan antarkomponen di dalam rencana. *Implementation Plan* ini adalah file markdown sementara (misalnya `implementation_plan.md` di root workspace) yang **WAJIB ANDA HAPUS** dari disk setelah instruksi sesuai/selesai dilakukan atau sesi berakhir (sebagaimana kebiasaan pendekatan *review-driven* pada AI agent).

### E.6: Pengerjaan & Pengujian Kode
Selesaikan instruksi pengguna secara tuntas. Anda **WAJIB** memanfaatkan kueri CLI `graphify` jika membutuhkan navigasi atau penelusuran referensi kode saat pengerjaan. Setelah selesai, Anda **WAJIB LANGSUNG** melakukan *testing* sesuai standar di `../../global-guidelines/testing.md` untuk memastikan fungsionalitas berjalan normal.

### E.7: Penyelesaian & Sinkronisasi Tiket
Rujuk dan patuhi aturan pemutakhiran status tiket di `../../global-guidelines/version-control.md` seksi **"Ticket-Driven Development Workflow"** poin 3–5. Pastikan:
- Status tiket diubah menjadi `Done`.
- Seluruh *checkbox* `[ ]` diubah menjadi `[x]` pada bagian `Acceptance Criteria`.
- Seksi `AI Execution Log & Output` terisi lengkap.
- Tiket disinkronkan dengan GitHub Projects.

### E.8: Pencatatan Changelog
Rujuk dan patuhi secara mutlak seluruh aturan di `../../global-guidelines/version-control.md` seksi **"Wajib Mencatat Setiap Perubahan"**, **"Penambahan Secara Reverse-Chronological"**, dan **"Format Log Pembaruan di Respons"**. Ringkasan:
- Catatan baru **WAJIB** disisipkan di baris **PALING ATAS** daftar (*descending*).
- Anda **WAJIB** mencatat **SEMUA** jenis perubahan, **BUKAN HANYA** kode (`Implementation`).
- Anda **WAJIB MUTLAK** menyalin dan mematuhi struktur baku dari referensi berikut untuk format log Anda: `../../global-docs/templates/changelog_entry_template.md`. Dilarang mengarang format sendiri!

### E.9: Kebijakan Version Control (Git)
Rujuk dan patuhi secara mutlak seluruh aturan di `../../global-guidelines/version-control.md` seksi **"Larangan Eksekusi Git Otonom"**, **"Kewajiban Commit"**, **"Prosedur Konfirmasi Pembuatan Branch"**, serta **"Aturan Penggunaan Istilah Orchestrator"** (Pesan *commit* di Project/Node **WAJIB 100% BERSIH** dari istilah orchestrator, tiket, PRD, atau guideline; referensi ID tiket HANYA dicatat di `CHANGELOG.md` dan file tiket lokal, bukan di pesan Git!).

### E.10: Sinkronisasi Konteks (Graphify Update)
Setelah tugas selesai dan di-commit, Anda **WAJIB** masuk ke dalam direktori *Path Codebase* (`cd`) dan menjalankan perintah `graphify update` di terminal untuk menyinkronkan Knowledge Graph dengan perubahan kode terbaru. DILARANG KERAS menjalankan `graphify update` di dalam repositori orchestrator.