---
id: TICKET-1794373590
plan_id: PLAN-1794373590
title: Configurable Round Count and Early-End Support for Would You Rather Game
status: Done
priority: High
labels: [Game, Would-You-Rather, Backend, Discord, UX, Feature]
---

# Deskripsi
Sebagai Pemain Radovid Bot pada game Would You Rather (`/radovid play would-you-rather`),
Saya ingin dapat memilih jumlah soal/ronde yang ingin dimainkan sejak awal serta memiliki fleksibilitas untuk mengakhiri sesi permainan kapan saja sebelum seluruh target soal selesai,
Sehingga durasi permainan dapat disesuaikan dengan ketersediaan waktu pemain tanpa terkunci pada jumlah ronde kaku atau terjebak dalam sesi permainan yang panjang.

### Konteks & Latar Belakang Masalah
Saat ini, game Would You Rather memiliki konfigurasi minimal 3 ronde (`MinRoundsBeforeFinish = 3`). Namun, permainannya bersifat tanpa batas akhir otomatis (*open-ended*):
1. Tombol `Finish & View Summary` baru muncul setelah ronde 3 selesai, namun tombol `Next: Round X` tetap dimunculkan tanpa batas maksimal (pemain bisa lanjut tanpa henti).
2. Pemain tidak memiliki opsi untuk menentukan jumlah putaran di awal perintah slash (misal ingin sesi kilat 1 ronde atau marathon 5 ronde).
3. Pemain tidak dapat mengakhiri sesi lebih awal secara sah (dengan mendapatkan rangkuman dan XP) jika waktu bermain terbatas sebelum ronde 3 selesai.

Tiket ini menambahkan opsi integer `rounds` pada perintah slash `/radovid play would-you-rather` dan membuka opsi penyelesaian awal (*early finish*) di setiap akhir ronde yang telah selesai dimainkan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794373590-wyr-configurable-rounds-and-early-end.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Subcommand `/radovid play would-you-rather` pada `command_def.go` memiliki opsi integer opsional `rounds` (contoh: pilihan 1, 3, 5, 10 ronde atau rentang 1 s/d 10) dengan nilai baku bawaan (*default*) 3 ronde.
- [x] Handler `HandlePlayCommand` pada `command_play_wyr.go` membaca opsi `rounds`, melakukan validasi (rentang 1-10), dan menyimpannya ke dalam konfigurasi sesi pada usecase.
- [x] Embed lobi pendaftaran dan embed soal menampilkan informasi progres putaran yang informatif (contoh: `Round 1/5: Would You Rather?` alih-alih hanya `Round 1`).
- [x] Tombol `Finish & View Summary` tersedia setelah ronde 1 selesai dimainkan, sehingga pemain dapat menyudahi permainan lebih awal (*early end*) kapan saja antar-ronde tanpa harus menunggu seluruh soal terjawab.
- [x] Ketika permainan mencapai jumlah target ronde (`RoundNumber == targetRounds`), sistem HANYA menyajikan tombol `Finish & View Summary` (tombol `Next: Round X` dihilangkan).
- [x] Logika `GetSessionSummary` pada `would_you_rather.go` mengizinkan penyelesaian dan pembuatan rangkuman jika telah menyelesaikan minimal 1 ronde (`canFinish := totalRounds >= 1`).
- [x] Seluruh unit test Go baru dan pengujian regresi lulus 100% (`go test -race ./...`) dengan memenuhi prinsip TDD (Disiplin O-07).
- [x] Mematuhi kebijakan Zero-Comment (`coding.md` Seksi 1.1) pada seluruh berkas Go yang dimodifikasi.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/wyr_entity.go`
- `../../../../radovid-bot-api/internal/domain/game/wyr_entity_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Melakukan TDD RED phase dengan menambahkan unit test gagal untuk konstanta & struct konfigurasi sesi pada `wyr_entity_test.go`, verifikasi opsi command `rounds` pada `command_test.go`, target rounds setting & summary 1 ronde pada `would_you_rather_test.go`, tombol finish only & embed `Round 1/5` pada `command_play_wyr_test.go`, serta tombol early end & target finish pada `interaction_wyr_button_test.go`.
  2. Mengimplementasikan konstanta `DefaultWYRRounds = 3`, `MinAllowedRounds = 1`, `MaxAllowedRounds = 10`, dan struct `WYRSessionConfig` pada `wyr_entity.go`.
  3. Mengimplementasikan konkurensi aman `sync.RWMutex` untuk manajemen `targetRounds` di `WouldYouRatherUseCase` (`SetTargetRounds`, `GetTargetRounds`), serta merelaksasi `canFinish := totalRounds >= 1` di `GetSessionSummary`.
  4. Mendaftarkan opsi integer `rounds` pada subcommand `would-you-rather` di `command_def.go` dengan pilihan 1, 3, 5, 10 ronde.
  5. Menambahkan fungsi helper `BuildWYRFinishOnlyButton` dan memperbarui `BuildWYRQuestionEmbed` agar menampilkan `Round X/N: Would You Rather?` pada `command_play_wyr.go`.
  6. Mengaitkan ekstraksi opsi `rounds` dan penyimpanan target rounds ke usecase serta memperbarui embed lobi dan ronde pertama pada `command_play_wyr.go`.
  7. Menyesuaikan handler voting `interaction_wyr_button.go`: menampilkan tombol `BuildWYRFinishOrNextButtons` pada setiap ronde sebelum target rounds, dan `BuildWYRFinishOnlyButton` saat mencapai target rounds.
  8. Memperbarui teks panduan pada `radovid_help.go`.
- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/internal/domain/game/wyr_entity.go`
  - `radovid-bot-api/internal/domain/game/wyr_entity_test.go`
  - `radovid-bot-api/internal/usecase/game/would_you_rather.go`
  - `radovid-bot-api/internal/usecase/game/would_you_rather_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_def.go`
  - `radovid-bot-api/internal/delivery/discord/command_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
  - `radovid-bot-api/internal/delivery/discord/radovid_help.go`
- **Catatan & Keputusan Arsitektural:**
  - Konfigurasi target ronde disimpan di layer usecase dengan konkurensi aman (`sync.RWMutex`) tanpa memerlukan migrasi skema basis data SQLite, menjaga integritas isolasi state game.
  - Nilai di luar rentang 1-10 secara fail-fast dinormalisasi ke `DefaultWYRRounds` (3).
  - Fleksibilitas early end dijamin dengan selalu menyediakan tombol `Finish & View Summary` sejak ronde 1 selesai dijawab, dan relaksasi `totalRounds >= 1` pada summary builder.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -race ./...` dan `go test -count=1 ./...`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	0.929s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.797s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	0.956s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/achievement	2.262s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/diary	1.343s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/feedback	2.672s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	3.205s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	4.507s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/notification	3.079s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/progress	3.583s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	4.564s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	4.582s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	4.715s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	4.637s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	4.820s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	4.302s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	4.381s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	4.656s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	4.424s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	4.576s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	3.540s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	3.405s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	3.349s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	3.542s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASS (Zero-comment policy dipatuhi 100%, fail-fast validation, tidak ada magic numbers, file ops native IDE)
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, bukti konkret eksekusi terminal tercantum, tanpa pelonggaran tes, repositori bersih)
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS - Terbukti gagal pada RED phase sebelum kode implementasi diterapkan
  - O-08 (Edge Case Atlas): PASS - Nilai rounds < 1, > 10, empty session, early end di ronde 1, target reached
  - O-09 (Damage Assessment): PASS - Kompatibel penuh dengan sesi yang berjalan tanpa opsi rounds (otomatis fallback ke default 3 ronde)
  - O-10 (Full Suite Metrics): PASS - 100% test suite lulus dengan `-race` dan `go vet` nol peringatan
- **Pilar 4: Tata Kelola Dokumen & Git (GG-1 s/d GG-5):** PASS (Tiket berpasangan dengan rencana, relative path konsisten, status tersinkronisasi)

### Daftar Belum Diuji (Untested List - O-16)
- Interaksi riil gateway Discord menggunakan bot token aktif di server produksi (diuji melalui unit test mock framework DiscordGo).
- Inferensi LLM riil pada Ollama live instance port 11434 (diuji via mock LLM service dan fallback content generation).

### Daftar Belum Diuji (Untested List - O-16)
- Interaksi riil gateway Discord menggunakan bot token aktif di server produksi (diuji melalui unit test mock framework DiscordGo).
- Inferensi LLM riil pada Ollama live instance port 11434 (diuji via mock LLM service dan fallback content generation).

