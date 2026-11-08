---
id: TICKET-1762433920
plan_id: PLAN-1762433920
title: Achievements System
status: Done
priority: High
labels: [Backend, Go, Gamification, Domain, Discord]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mendapatkan lencana penghargaan (achievements/badges) ketika mencapai pencapaian tertentu (seperti menyelesaikan game pertama, mempertahankan streak 7 hari berturut-turut, atau menguasai tata bahasa),
Sehingga saya merasa diapresiasi dan memiliki tujuan jangka panjang dalam ekosistem bot.

Sistem achievement memberikan penghargaan atas pencapaian milestone penting pengguna. Setiap achievement memiliki atribut unik (ID, nama, deskripsi, emoji icon, XP reward, dan trigger condition). Ketika aktivitas pengguna memicu kondisi pembukaan lencana baru yang belum pernah diraih, bot akan:
1. Menyimpan data lencana ke tabel `achievements`.
2. Menghadiahkan bonus XP langsung ke profil pengguna via use case `AwardXP`.
3. Membangun dan mengirimkan notifikasi perayaan achievement ke pengguna via pesan Discord.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433920-achievements-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Tersedia katalog achievement statis di domain layer mencakup: `first_game` (First Steps - 50 XP), `streak_7` (Week Warrior - 150 XP), `streak_30` (Monthly Devotion - 500 XP), `grammar_master` (Syntax Sorcerer - 200 XP), `vocab_champion` (Lexicon Master - 200 XP), `boss_slayer` (Dragon Tamer - 250 XP)
- [x] Engine evaluator (`EvaluateAchievements`) mengevaluasi statistik pengguna pasca-aktivitas dan hanya mengembalikan daftar lencana baru yang belum pernah diraih
- [x] Idempotensi terjamin: pengguna tidak dapat memperoleh achievement yang sama lebih dari satu kali (dicegah pada level logika aplikasi dan SQLite unique constraint)
- [x] Pemberian bonus XP otomatis dipicu untuk setiap lencana baru yang terbuka
- [x] Format embed Discord notifikasi achievement (`BuildAchievementUnlockedEmbed`) menyajikan emoji, nama lencana, deskripsi, serta bonus XP yang didapat
- [x] Unit testing menyeluruh untuk evaluator kondisi milestone (kondisi tercapai vs belum tercapai, filter duplicate achievements)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/010_create_achievements.sql`
- `../../../../radovid-bot-api/internal/domain/achievement/entity.go`
- `../../../../radovid-bot-api/internal/domain/achievement/catalog.go`
- `../../../../radovid-bot-api/internal/domain/achievement/evaluator.go`
- `../../../../radovid-bot-api/internal/domain/achievement/evaluator_test.go`
- `../../../../radovid-bot-api/internal/domain/achievement/repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/achievement_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/achievement_repository_test.go`
- `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock.go`
- `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Branch:** `main`
- **Execution Summary:**
  1. Dibuat migrasi DDL `migrations/010_create_achievements.sql` membuat tabel `achievements` (id PK, user_id FK, achievement_type, earned_at) dengan indeks dan unique constraint `UNIQUE(user_id, achievement_type)`.
  2. Dibuat entitas domain `internal/domain/achievement/entity.go` (`Achievement`, `UserAchievement`, `UserMetrics`).
  3. Dibuat katalog statis `internal/domain/achievement/catalog.go` mencakup 6 lencana wajib (`first_game`, `streak_7`, `streak_30`, `grammar_master`, `vocab_champion`, `boss_slayer`) beserta fungsi `GetAchievement` dan `AllAchievements`.
  4. Dibuat engine evaluator murni `internal/domain/achievement/evaluator.go` (`EvaluateAchievements`) yang memfilter lencana yang sudah diraih (`earnedIDs`) dan hanya mengembalikan lencana baru yang memenuhi syarat milestone.
  5. Dibuat unit test table-driven `internal/domain/achievement/evaluator_test.go` mencakup zero metrics, milestone thresholds, filter duplikasi, dan integritas katalog.
  6. Dibuat port interface `internal/domain/achievement/repository.go` (`AchievementRepository`).
  7. Dibuat repositori SQLite `internal/infrastructure/database/achievement_repository.go` (`GetUserAchievements`, `SaveAchievement` dengan `INSERT OR IGNORE` idempoten) dan unit test `achievement_repository_test.go`.
  8. Dibuat use case `internal/usecase/achievement/check_and_unlock.go` (`CheckAndUnlockUseCase`) mengorkestrasi pengecekan lencana, penyimpanan riwayat perolehan, dan pemberian bonus XP via `AwardXPUseCase` (source: `achievement:<id>`).
  9. Dibuat unit test use case `internal/usecase/achievement/check_and_unlock_test.go` dengan mock repository.
  10. Ditambahkan pembangun visual Discord Embed `BuildAchievementUnlockedEmbed` dan `BuildAchievementsUnlockedEmbed` pada `internal/infrastructure/discord/game_templates.go` dan unit test pada `game_templates_test.go`.
  11. Seluruh test suite (20 package) 100% PASS (`go test -count=1 ./...`), `go vet ./...` 0 error/warning, `go build ./cmd/bot` berhasil bersih. Zero-comment policy dipatuhi penuh.
