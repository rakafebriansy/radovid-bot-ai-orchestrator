# Mandatory Changelog & Version Control Policy

## Wajib Mencatat Setiap Perubahan (Semua Jenis File)
Setiap kali Anda memproduksi baris kode baru, merevisi *bug*, atau melakukan pengaturan konfigurasi apa pun atas instruksi pengguna di suatu *node*, Anda **WAJIB** mendokumentasikan catatan perubahan tersebut secara rinci ke dalam berkas `CHANGELOG.md` milik *node* yang bersangkutan. **Pencatatan ini TIDAK HANYA berlaku untuk perubahan kode (Implementation), melainkan MUTLAK untuk segala bentuk perubahan: pembuatan/pembaruan file `README.md`, dokumen pedoman (`guidelines/`), dokumen desain sistem, pembuatan/pembaruan tiket, dan lain-lain. Semua harus tercatat!** Tidak boleh ada satu pun perubahan yang terlewat dari pencatatan.

## Penambahan Secara Reverse-Chronological (Wajib Descending)
Setiap penambahan log riwayat versi terbaru **WAJIB** diletakkan di bagian **PALING ATAS** dari daftar pembaruan (urutan *descending*, tepat di bawah *header* utama dokumen *changelog*). AI **DILARANG KERAS** menambahkan log di baris terbawah. Dengan demikian, versi terbaru selalu menduduki urutan pertama dan riwayat versi yang lebih lama perlahan terdorong ke bawah.

## Kewajiban Informasi Branch & VCS
Setiap entri pembaruan yang dicatatkan wajib memuat informasi mengenai nama *branch Git* yang sedang digarap, beserta tautan (*link*) menuju repositori *Version Control System* (VCS) yang bersangkutan.

## Aturan Penamaan Branch

### Decision Gate: Identifikasi Lingkungan Kerja
Sebelum mengeksekusi aksi Git apa pun (commit, branch, push, PR), Anda **WAJIB MUTLAK** mengidentifikasi terlebih dahulu di lingkungan mana Anda sedang bekerja:
- **Repositori Orchestrator** (`ai-orchestrator-template/` — mencakup `global-docs/`, `global-guidelines/`, `nodes/`, `README.md`, dan seluruh file di dalamnya): Anda bekerja di lingkup **Orchestrator**. **Semua** perubahan langsung di-push ke `main`. **DILARANG** membuat branch baru.
- **Repositori Project/Node** (source code aplikasi asli seperti `frontend-app/`, `backend-api/`, dll. yang berada di **luar** `ai-orchestrator-template/`): Anda bekerja di lingkup **Project/Node**. Ikuti prosedur branch di bawah ini.

**Gate ini WAJIB dievaluasi setiap kali** menerima tugas baru. Jangan pernah melewatkannya.

1. **Pada Direktori Project/Node:** Anda **DILARANG KERAS** menggunakan format penamaan *branch* dengan ID tiket kapital seperti `feat(TICKET-XX)`. Penamaan *branch* untuk pengembangan kode aplikasi di dalam *node* diharuskan menggunakan deskripsi fitur secara langsung tanpa mengandung unsur istilah dari ai orchestrator, contoh: `feat(feature-name)`.
2. **Pada Repositori Orchestrator:** Khusus untuk repositori ekosistem Orchestrator (template yang sudah di-*startup*), Anda **TIDAK PERLU** membuat *branch* baru. Semua perubahan pada tingkat Orchestrator (seperti dokumentasi global, template, atau konfigurasi) cukup langsung di-*push* ke `main`.

## Aturan Penggunaan Istilah Orchestrator (seperti ticket, prd, design system, guideline, dll)
Istilah-istilah yang dimaksud meliputi namun tidak terbatas pada: *ticket*, *prd*, *design system*, *guideline*, *system design*, *development planning*, *changelog* (dalam konteks orchestrator), *retrospective*, *prototype*, *node*, *orchestrator*, *template*, dan seluruh terminologi yang merujuk pada artefak ekosistem AI Orchestrator ini.
Untuk seluruh aksi Git (*commit*, penamaan *branch*, *Pull Request*, dsb):
- **Di Project/Node:** Anda **DILARANG KERAS** menggunakan istilah dari AI orchestrator di dalam *commit message*, *branch*, *PR*, dan semua aksi Git. Pesan commit di Project/Node harus murni mencerminkan domain teknis aplikasi.
  - ❌ **SALAH di Project/Node:** `feat(node-auth): implement login based on prd guideline (TICKET-01)`
  - ❌ **SALAH di Project/Node:** `fix(ticket-03): resolve auth looping issue in orchestrator`
  - ✅ **BENAR di Project/Node:** `feat(auth): add jwt authentication and refresh token handler`
  - ✅ **BENAR di Project/Node:** `fix(auth): handle expired token on api client interceptor`
- **Di Repositori Orchestrator:** Anda **WAJIB** menggunakan istilah dari AI orchestrator sebagai scope dalam *commit message* (misal: `docs(template): update`), *branch*, *PR*, dan aksi Git terkait lainnya.

## Prosedur Konfirmasi Pembuatan Branch (Branch Switching)

> **PRASYARAT WAJIB:** Evaluasi **Decision Gate** di atas terlebih dahulu. Jika Anda sedang berada di **Repositori Orchestrator**, **HENTIKAN** prosedur ini sepenuhnya — Anda **TIDAK** memerlukan branch baru dan **WAJIB** langsung bekerja di `main`. Prosedur di bawah ini **HANYA** berlaku untuk direktori **Project/Node**.

Setiap kali Anda menerima tugas/instruksi eksekusi baru di dalam **direktori Project/Node**, Anda **WAJIB** mengevaluasi apakah tugas tersebut berada pada domain fitur yang berbeda atau memiliki konteks yang berbeda (misalnya: *hotfix*, penulisan *testing*, *refactor*, dsb.) dari *branch* yang saat ini sedang aktif.
Meskipun pembuatan *branch* baru disarankan untuk domain fitur yang baru, **pastikan tidak setiap instruksi atau setiap tiket dibuatkan *branch* baru**. Jika tugas masih berkaitan erat dengan fitur yang sedang dikembangkan, gunakan *branch* yang sama.
Jika konteks atau domain fiturnya terbukti berbeda dan benar-benar membutuhkan *branch* baru, Anda **WAJIB** secara otomatis menanyakan dan meminta persetujuan pengguna untuk membuat serta berpindah ke *branch* baru (contoh: `git checkout -b type/nama-branch`) SEBELUM Anda mulai mengeksekusi modifikasi kode apa pun.
**CATATAN PENTING:** Pembuatan *branch* baru **hanya boleh terjadi pada direktori Project/Node**. Orchestrator tidak perlu membuat *branch* baru dan langsung *push* semua perubahan ke `main`.

## Kewajiban Commit & Mekanisme Rollback
1. **Kewajiban Meminta Persetujuan Commit:** Setiap kali Anda selesai mengeksekusi sebuah tiket/tugas dan telah mencatat riwayat pembaruan ke dalam file `CHANGELOG.md`, Anda **DILARANG KERAS** melakukan *auto-commit*. Anda **WAJIB MUTLAK** meminta izin dan persetujuan pengguna terlebih dahulu sebelum mengeksekusi *git commit*. Jangan pernah menumpuk banyak perubahan dari berbagai tugas ke dalam satu *commit* besar. Hal ini bertujuan agar setiap tiket atau tugas terisolasi secara aman dan rapi dalam satu *commit history* yang bersih.
2. **Mekanisme Rollback (Pembatalan Perubahan):** Berkat kewajiban *commit* per-tugas di atas, jika di kemudian hari terjadi *error* fatal, fitur merusak sistem, atau pengguna meminta Anda membatalkan *task* terakhir, Anda **WAJIB** menerapkan mekanisme *rollback* Git (kembali ke versi stabil sebelumnya). Gunakan perintah seperti `git reset --hard HEAD~1` (atau nama *commit hash* terkait). Anda **WAJIB meminta konfirmasi persetujuan pengguna** terlebih dahulu sebelum mengeksekusi *rollback* destruktif semacam ini.

   > ⚠️ **Perintah yang DILARANG saat Rollback:** `git restore <file>` dan `git checkout -- <file>` (varian yang menarget path/file tertentu) **DILARANG KERAS** dieksekusi tanpa review eksplisit dari user, karena keduanya menghapus perubahan yang belum di-commit secara permanen. Lihat **[safe-file-operations.md](./safe-file-operations.md)** untuk detail larangan dan alternatifnya.

## Kewajiban Pre-Commit & Pre-Push (Testing & Linting)
Sebelum Anda (AI Agent) meminta persetujuan pengguna untuk melakukan aksi `git commit` maupun `git push`, Anda **WAJIB MUTLAK** menjalankan dan memastikan hal-hal berikut:
1. **Untuk Backend/Fullstack:** Mengeksekusi seluruh *unit test* dan memastikannya berjalan sukses tanpa galat (*error/fail*).
2. **Untuk Frontend/Mobile/Fullstack:** Mengeksekusi *linter*, *type checker*, dan *unit test* (jika ada) serta memastikannya lolos 100%.

Anda **DILARANG KERAS** mengajukan permohonan `commit` atau `push` jika tahapan validasi prasyarat ini belum dijalankan atau masih membuang kode galat. Untuk detail standar pengujian dan perintah spesifik, Anda **WAJIB** merujuk pada pedoman di **[testing.md](./testing.md)**.

## Larangan Eksekusi Git Otonom (Mandatory User Approval)
1. **Dilarang Auto-Commit:** Anda **DILARANG KERAS** melakukan `git add`, `git commit`, maupun `git push` secara otomatis tanpa sepengetahuan dan izin eksplisit dari pengguna.
2. **Wajib Menjelaskan Terlebih Dahulu:** Anda juga **DILARANG** meminta, mengeluarkan *prompt*, atau mengeksekusi perintah `git add` sebelum memberikan penjelasan secara terperinci kepada pengguna terlebih dahulu mengenai apa saja yang akan diproses.
3. **Pemisahan Referensi Tiket & Pesan Commit:**
   - **Di Project/Node:** Pesan *commit* **DILARANG KERAS** memuat referensi ID tiket (seperti `TICKET-01`), kata kunci penutup tiket (`Resolves TICKET-XX`), maupun istilah orchestrator lainnya. Pelacakan ID tiket **HANYA** dicatat di repositori orchestrator melalui `CHANGELOG.md` dan file tiket lokal (`tickets/TICKET-XX.md`).
   - **Di Repositori Orchestrator:** Pesan *commit* dapat merujuk cakupan pembaruan orchestrator (misal: `docs(orchestrator): update version control guideline`).

## Format Log Pembaruan di Respons
Setelah Anda menuntaskan sebuah tugas atau instruksi, Anda wajib menyertakan log pembaruan di bagian akhir respons Anda. Hindari penggunaan format tabel; gunakan format daftar berstruktur (*structured list*) yang rapi guna memaksimalkan keterbacaan (*readability*) dan kemudahan pengarsipan tiket internal. 

> ⚠️ **Penting: Single Source of Truth Format**
> AI Agent dilarang mengarang format pencatatan log. Anda **WAJIB MUTLAK** menyalin, merujuk, dan mematuhi struktur baku yang terdapat pada template berikut:
> `global-docs/templates/changelog_entry_template.md`

## Ticket-Driven Development Workflow
Infrastruktur proyek AI Orchestrator ini menganut sistem manajemen tugas *offline* terpusat berbasis tiket di direktori `tickets/` yang berada di dalam masing-masing *node*. Selaku AI Agent, Anda dituntut mematuhi protokol berikut selama fase pengerjaan kode:
1.  **Rujuk pada Tiket:** Jangan mengeksekusi logika secara membabi buta tanpa arah. Pertama-tama, Anda **WAJIB membaca panduan struktural tiket** pada file referensi **`nodes/[nama-node]/tickets/README.md`**. Jika diperintahkan **membuat tiket baru**, Anda **wajib menyalin mentah-mentah format Boilerplate** dari file tersebut. Setelah memahaminya, barulah buka file tiket spesifik yang relevan (contoh: `nodes/[nama-node]/tickets/TICKET-01-login.md`). Apabila tiket tersebut berisi laporan masalah, tiket **WAJIB** dikonstruksikan menggunakan standar `global-docs/templates/bug_report_template.md`.
2.  **Pengujian Kode (Testing):** Setelah Anda menyelesaikan perombakan logika/kode pada tiket, Anda **WAJIB LANGSUNG** melakukan uji coba fungsional (testing) untuk mendeteksi *error* kompilasi atau galat logika.
3.  **Pemutakhiran Status & Checklist:** Saat Anda mulai menggarap sebuah tugas, Anda berhak mengubah properti *frontmatter* `status: Todo` menjadi `status: In Progress` pada file tiket lokal. Jika seluruh kriteria pengerjaan dan pengujian telah sukses secara tuntas, Anda **WAJIB LANGSUNG** mengubahnya menjadi `status: Done` dan menandai secara mutlak (*mencentang*) seluruh *checkbox* (`- [x]`) di bagian *Acceptance Criteria* tiket tersebut.
4.  **Kewajiban Pengisian Log AI:** Anda diwajibkan menjabarkan secara rinci jejak teknis, modifikasi, dan pertimbangan arsitektural di bawah seksi `AI Execution Log & Output` pada dasar file tiket terkait agar transparansi keputusan terjamin.
5.  **Tautan Changelog:** Pastikan Anda menyertakan ID referensi tiket (misal: "Referensi: TICKET-01") pada detail penulisan log saat melaporkan kemajuan pembaruan di layar obrolan maupun di dalam arsip `CHANGELOG.md` lokal.

## Pull Request (PR) Submission Policy
Apabila siklus pengembangan Anda melibatkan pembaruan kode lintas cabang (misalnya transisi dari cabang `feature/` ke `main`), Anda (AI Agent) **DILARANG KERAS** membiarkan deskripsi *Pull Request* kosong atau diisi secara sembarangan. Anda dituntut untuk senantiasa mematuhi pedoman pengajuan PR berikut:
1.  **Wajib Menggunakan Templat Referensi:** Segera setelah Anda siap mengajukan permintaan penggabungan kode (*merge request*), Anda **WAJIB** menyalin secara utuh struktur *boilerplate* yang tersimpan di **`global-docs/templates/pull_request_template.md`** sebagai format dasar pengisian deskripsi PR.
2.  **Kewajiban Auto-Closing Tiket:** Di dalam tubuh deskripsi PR tersebut, Anda **DIWAJIBKAN SECARA MUTLAK** untuk menautkan kata kunci penutup tiket dengan sintaksis `Resolves [TICKET-ID]` (contoh: `Resolves TICKET-01`). Sintaksis sakral ini berfungsi ganda sebagai pemicu pemindahan kartu (*card movement trigger*) di papan visual GitHub Projects sehingga tetap tersinkronisasi 100% dengan status penyelesaian tiket di dalam kerangka kerja lokal kita.
3.  **Larangan Melakukan Merge Otonom (No Auto-Merge):** Mengajukan PR **BUKAN** berarti Anda telah melakukan *merge* (penggabungan kode) ke branch tujuan. Sebagai AI Agent, Anda **HANYA** bertugas **mengajukan** PR. Anda **DILARANG KERAS** mengeksekusi perintah penggabungan (seperti `gh pr merge`, `git merge`, dsb.) secara otonom.
4.  **Tindakan Pasca Pembuatan PR:** Setelah PR berhasil diajukan, Anda **WAJIB** melaporkan tautan (URL) PR tersebut kepada pengguna dan **BERHENTI SEJENAK**. Tunggu instruksi dari pengguna apakah mereka akan meninjau dan melakukan *merge* secara mandiri, atau mereka memberikan persetujuan eksplisit kepada Anda untuk mengeksekusi *merge*.

## Standar Pesan Commit (Semantic Commits)
Setiap kali Anda menjejakkan perubahan ke dalam riwayat repositori (*git commit*), Anda **WAJIB** menyelaraskan penulisan pesannya (*commit message*) dengan standar baku yang tersimpan di dalam **`global-docs/templates/commit_message_template.md`**. Jangan pernah menulis pesan *commit* yang sembarangan, tidak deskriptif, atau mengabaikan format struktur label *Semantic Commit* (seperti `feat:`, `fix:`, `chore:`, dsb.).
