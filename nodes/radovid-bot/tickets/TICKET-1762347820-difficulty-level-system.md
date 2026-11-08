---
id: TICKET-1762347820
plan_id: PLAN-1762347820
title: Difficulty Level System
status: Done
priority: High
labels: [Settings, User, Core, Architecture, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin dapat mengatur tingkat kemahiran bahasa Inggris saya (Beginner, Intermediate, Advanced) melalui perintah `/radovid settings`,
Sehingga AI Game Master menyesuaikan tingkat kerumitan pertanyaan, kekayaan kosakata, serta gaya umpan balik bahasa agar sesuai dengan kapasitas pemahaman saya.

Sistem Radovid Bot membutuhkan mekanisme penyesuaian tingkat kemahiran (Difficulty Level System) yang persisten di tingkat profil pengguna. Level kesulitan ini mempengaruhi seluruh game dan modul evaluasi:
1. **Beginner (A1-A2):** Kosakata dasar, kalimat pendek, nada feedback sangat suportif, toleran terhadap kesalahan minor.
2. **Intermediate (B1-B2):** Percakapan natural, ungkapan umum & idiomatis, feedback berimbang antara apresiasi dan koreksi gramatikal.
3. **Advanced (C1-C2):** Kosakata kompleks, topik abstrak/profesional, evaluasi ketat mencakup kolokasi, kesesuaian register, dan kealamian gaya bahasa.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347820-difficulty-level-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Definisi tipe enum `Difficulty` (`Beginner`, `Intermediate`, `Advanced`) dengan metode validasi dan pemetaan CEFR di domain user
- [x] Skema database `users` diperbarui via migration SQL untuk menyimpan kolom `difficulty` dengan nilai default `Beginner`
- [x] Slash command `/radovid settings` menampilkan antarmuka konfigurasi tingkat kesulitan dengan interaksi Discord Select Menu / Buttons
- [x] Pembaruan profil difficulty tersimpan persisten ke SQLite dan aktif seketika untuk semua sesi game berikutnya
- [x] Seluruh prompt template Ollama (`{{.Difficulty}}`) menyertakan instruksi penyesuaian gaya respons AI berdasarkan level profil pengguna
- [x] Unit test mencakup validasi parsing nilai difficulty, transisi level, dan fallback default jika input tidak dikenali

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/007_add_difficulty_to_users.sql`
- `../../../../radovid-bot-api/internal/domain/user/difficulty.go`
- `../../../../radovid-bot-api/internal/domain/user/difficulty_test.go`
- `../../../../radovid-bot-api/internal/domain/user/entity.go`
- `../../../../radovid-bot-api/internal/domain/user/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/user/repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/user_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/user_repository_test.go`
- `../../../../radovid-bot-api/internal/usecase/user/settings.go`
- `../../../../radovid-bot-api/internal/usecase/user/settings_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Branch:** `main`
- **Execution Summary:**
  1. Dibuat migrasi DDL `migrations/007_add_difficulty_to_users.sql` menambahkan kolom `difficulty TEXT NOT NULL DEFAULT 'Beginner'` pada tabel `users`.
  2. Dibuat domain enum `Difficulty` pada `internal/domain/user/difficulty.go` beserta metode `CEFRLevel()`, `Description()`, `PromptGuideline()`, `ParseDifficulty()`, dan unit test table-driven `difficulty_test.go`.
  3. Diperbarui entitas `User` pada `internal/domain/user/entity.go` menyertakan `Difficulty Difficulty`, dan port repository `internal/domain/user/repository.go` menyertakan `GetOrCreateByDiscordID` dan `UpdateDifficulty`.
  4. Diimplementasikan `UserRepository` SQLite murni pada `internal/infrastructure/database/user_repository.go` dan unit test in-memory `user_repository_test.go`.
  5. Dibuat `SettingsUseCase` pada `internal/usecase/user/settings.go` (`GetUserSettings`, `UpdateUserDifficulty`) dan unit test `settings_test.go`.
  6. Ditambahkan builder visual Discord Embed `BuildSettingsEmbed` pada `internal/infrastructure/discord/game_templates.go` dan unit test `game_templates_test.go`.
  7. Ditambahkan subcommand `/radovid settings` dengan opsi opsional `level` pada `internal/delivery/discord/command_def.go`, handler routing pada `internal/delivery/discord/handler.go`, dan implementasi interaksi buttons (`🟢 Beginner`, `🟡 Intermediate`, `🔴 Advanced`) pada `internal/delivery/discord/radovid_settings.go` beserta unit test `radovid_settings_test.go`.
  8. Diintegrasikan wiring dependensi `userRepo` dan `settingsUseCase` serta pendaftaran router prefix `settings:` pada `cmd/bot/main.go`.
  9. Seluruh unit test lulus 100% (`go test -count=1 ./...`), `go vet ./...` lulus bersih (0 error/warning), dan `go build ./cmd/bot` berhasil.

