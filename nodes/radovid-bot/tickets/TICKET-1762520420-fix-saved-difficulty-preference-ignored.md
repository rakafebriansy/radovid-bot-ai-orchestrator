---
id: TICKET-1762520420
plan_id: PLAN-1762520420
title: "[BUG] Preferensi Difficulty Pengguna (/radovid settings) Diabaikan di Seluruh Mini-Game"
status: Done
priority: Medium
severity: Broken Flow
labels: [Bug, Discord, Game, Settings, User]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Fitur `/radovid settings` (Tiket 3.8) mengizinkan pemain menyimpan preferensi level kesulitan (`Beginner`, `Intermediate`, `Advanced`) ke tabel `users` agar persisten lintas sesi. Akan tetapi, seluruh 14 game handler di `internal/delivery/discord/` tidak pernah membaca preferensi kesulitan pengguna dari `h.settingsUseCase.GetUserDifficulty` ketika argumen opsional `difficulty` tidak diisi pada slash command, melainkan selalu melakukan hardcode fallback ke `gamemaster.DifficultyMedium`. Selain itu, pemilihan dropdown pada `game_job_interview.go:141`, `game_roleplay.go:143`, serta pergantian ronde di `interaction_wyr_button.go:100` dan `interaction_taboo_modal.go:91` juga meng-hardcode `gamemaster.DifficultyMedium`.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520420-fix-saved-difficulty-preference-ignored.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Pengguna mengubah pengaturan level kesulitan menjadi `Advanced` melalui `/radovid settings`.
2. Pengguna memulai permainan apapun (misal `/radovid play grammar-boss-fight`) tanpa mengisi opsi `difficulty` di command.
3. Amati bahwa permainan dimulai pada tingkat kesulitan `medium` alih-alih `hard` (`Advanced`).

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Jika pengguna tidak menyertakan parameter `difficulty` secara eksplisit saat menjalankan `/radovid play <game>`, bot mengambil preferensi kesulitan pengguna dari `settingsUseCase` (`Beginner` -> `easy`, `Intermediate` -> `medium`, `Advanced` -> `hard`).
- **Aktual (Actual):** Seluruh game mengabaikan preferensi `/radovid settings` dan memaksa `DifficultyMedium`.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] `GameHandler` menyediakan helper `ResolveDifficulty(ctx, userID, username, explicitDiffStr)` yang memetakan preferensi tersimpan (`Beginner` -> `easy`, `Intermediate` -> `medium`, `Advanced` -> `hard`) apabila parameter `explicitDiffStr` kosong.
- [x] Seluruh handler game dan komponen lanjutan menggunakan `ResolveDifficulty` alih-alih hardcode `DifficultyMedium`.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_job_interview.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_roleplay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_one_minute.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_five_word.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_secret_mission.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_transformation.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  Handler game (`HandlePlay*`) dan handler interaksi komponen/modal (`interaction_wyr_button.go`, `interaction_taboo_modal.go`, `game_job_interview.go`, `game_roleplay.go`) mengonversi parameter slash command `difficulty` secara langsung ke `gamemaster.Difficulty` tanpa membaca pengaturan yang tersimpan di `users.difficulty` melalui `SettingsUseCase`. Jika opsi tidak diisi pemain, handler langsung jatuh ke fallback statis `gamemaster.DifficultyMedium`.
- **Langkah Perbaikan:**
  1. Menambahkan method `ResolveDifficulty(ctx context.Context, userID, username, explicitStr string) gamemaster.Difficulty` pada `GameHandler` di `internal/delivery/discord/game_handler.go`. Helper ini memprioritaskan argumen eksplisit (`easy`, `medium`, `hard`), dan jika kosong atau tidak valid, memanggil `h.settingsUseCase.GetUserSettings(ctx, userID, username)` untuk memetakan preferensi (`Beginner` -> `DifficultyEasy`, `Intermediate` -> `DifficultyMedium`, `Advanced` -> `DifficultyHard`). Jika `settingsUseCase == nil` atau terjadi error, fallback aman ke `gamemaster.DifficultyMedium`.
  2. Menambahkan unit test `TestGameHandler_ResolveDifficulty` di `game_handler_test.go` dan membuktikan kegagalan (RED) pada implementasi stub sebelum method dihubungkan ke `SettingsUseCase`.
  3. Memperbarui seluruh 14 game handler dan component/round advancement handler (`game_one_minute.go`, `game_story_relay.go`, `game_five_word.go`, `game_secret_mission.go`, `game_boss_fight.go`, `game_transformation.go`, `game_movie_review.go`, `game_news_reporter.go`, `game_dream_house.go`, `game_job_interview.go`, `game_roleplay.go`, `command_play_wyr.go`, `command_play_taboo.go`, `command_play_20q.go`, `interaction_wyr_button.go`, `interaction_taboo_modal.go`) untuk memanfaatkan `h.ResolveDifficulty(...)`.
  4. Menjalankan seluruh pengujian unit test dan race detector (`go test -race ./...`) memastikan 100% lulus tanpa regresi.
- **Ringkasan Berkas Terdampak:**
  - `internal/delivery/discord/game_handler.go`
  - `internal/delivery/discord/game_handler_test.go`
  - `internal/delivery/discord/command_play_wyr.go`
  - `internal/delivery/discord/command_play_taboo.go`
  - `internal/delivery/discord/command_play_20q.go`
  - `internal/delivery/discord/game_job_interview.go`
  - `internal/delivery/discord/game_roleplay.go`
  - `internal/delivery/discord/game_one_minute.go`
  - `internal/delivery/discord/game_story_relay.go`
  - `internal/delivery/discord/game_five_word.go`
  - `internal/delivery/discord/game_secret_mission.go`
  - `internal/delivery/discord/game_boss_fight.go`
  - `internal/delivery/discord/game_transformation.go`
  - `internal/delivery/discord/game_movie_review.go`
  - `internal/delivery/discord/game_news_reporter.go`
  - `internal/delivery/discord/game_dream_house.go`
  - `internal/delivery/discord/interaction_wyr_button.go`
  - `internal/delivery/discord/interaction_taboo_modal.go`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v -count=1 -race ./internal/delivery/discord/...`
- **Output Terminal Aktual:**
  ```text
=== RUN   TestGameHandler_ResolveDifficulty
=== RUN   TestGameHandler_ResolveDifficulty/resolves_user_saved_advanced_setting_when_explicit_is_empty
=== RUN   TestGameHandler_ResolveDifficulty/resolves_user_saved_beginner_setting_when_explicit_is_empty
=== RUN   TestGameHandler_ResolveDifficulty/explicit_argument_overrides_user_saved_setting
=== RUN   TestGameHandler_ResolveDifficulty/fallback_to_medium_when_settings_usecase_is_nil
--- PASS: TestGameHandler_ResolveDifficulty (0.00s)
    --- PASS: TestGameHandler_ResolveDifficulty/resolves_user_saved_advanced_setting_when_explicit_is_empty (0.00s)
    --- PASS: TestGameHandler_ResolveDifficulty/resolves_user_saved_beginner_setting_when_explicit_is_empty (0.00s)
    --- PASS: TestGameHandler_ResolveDifficulty/explicit_argument_overrides_user_saved_setting (0.00s)
    --- PASS: TestGameHandler_ResolveDifficulty/fallback_to_medium_when_settings_usecase_is_nil (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.384s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `go test -v -run TestGameHandler_ResolveDifficulty ./internal/delivery/discord`
  - Pesan Kegagalan:
    ```text
    === RUN   TestGameHandler_ResolveDifficulty/resolves_user_saved_advanced_setting_when_explicit_is_empty
        game_handler_test.go:165: expected hard difficulty for advanced user preference, got medium
    === RUN   TestGameHandler_ResolveDifficulty/resolves_user_saved_beginner_setting_when_explicit_is_empty
        game_handler_test.go:172: expected easy difficulty for beginner user preference, got medium
    === RUN   TestGameHandler_ResolveDifficulty/explicit_argument_overrides_user_saved_setting
        game_handler_test.go:179: expected easy difficulty from explicit override, got medium
    --- FAIL: TestGameHandler_ResolveDifficulty (0.01s)
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** Tabel `users` telah menyimpan nilai kolom `difficulty` (`Beginner`, `Intermediate`, `Advanced`) dengan benar sejak implementasi `/radovid settings`. Dengan diterapkannya `ResolveDifficulty`, seluruh sesi permainan yang baru akan langsung mengonsumsi nilai preferensi yang telah disimpan oleh pemain tanpa perlu migrasi atau pembersihan basis data.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):**
  - HG-1 (Zero Comment Policy): LULUS. Seluruh berkas Go bersih tanpa komentar `//` atau `/* */`.
  - HG-2 (Clean Build & Vet): LULUS. `go vet ./...` dan `go build ./cmd/bot` berhasil tanpa error.
  - HG-3 (Race Free Tests): LULUS. `go test -race ./...` lulus 100%.
  - HG-4 (Working on Main Approved): LULUS. Telah dikonfirmasi dan disetujui langsung pada branch `main`.
- **Must Probe Status (MP-1 s/d MP-7):**
  - MP-1 (Regression check): LULUS. Seluruh 26 package lulus unit test.
  - MP-2 (Code Hygiene): LULUS. Zero comments di codebase Go.
  - MP-3 (TDD Red-Green cycle): LULUS. Dibuktikan dengan log kegagalan sebelum fix dan log kelulusan sesudah fix.
  - MP-4 (Error handling): LULUS. Fallback ke `DifficultyMedium` jika data user tidak ditemukan atau terjadi query error.
  - MP-5 (Explicit override): LULUS. Argumen explicit selalu didahulukan dibanding preferensi tersimpan.
  - MP-6 (Context propagation): LULUS. Context diteruskan secara lengkap ke `settingsUseCase.GetUserSettings(ctx, ...)`.
  - MP-7 (Documentation sync): LULUS. Tiket, Plan, Development Planning, dan CHANGELOG disinkronkan.
- **Daftar Belum Diuji (Untested List - O-16):** Nihil (semua jalur cabang: explicit valid, explicit override, user saved beginner, user saved advanced, dan nil settings usecase telah diuji secara komprehensif pada unit test).
