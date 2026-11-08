---
id: TICKET-1762433800
plan_id: PLAN-1762433800
title: XP Calculation & Level System
status: Done
priority: High
labels: [Backend, Go, Gamification, Database]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mendapatkan Experience Points (XP) dan naik level setiap kali menyelesaikan aktivitas belajar atau game di Radovid Bot,
Sehingga motivasi belajar saya meningkat melalui perkembangan level yang terukur dan kompetitif.

Fitur gamifikasi membutuhkan sistem kalkulasi XP yang adil dan transparan. Perolehan XP dipengaruhi oleh jenis aktivitas, tingkat kesulitan game (*difficulty multiplier*), serta performa jawaban. Seluruh riwayat perolehan XP harus dicatat dalam tabel audit log (`user_xp_log`) untuk integritas data dan mencegah eksploitasi, serta rumus kenaikan level menggunakan formula kuadratik konstan $\text{XP}_{\text{required}} = \text{level}^2 \times 100$.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433800-xp-calculation-level-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Tersedia skema migrasi database `009_create_user_xp_log.sql` untuk mencatat riwayat perolehan XP (id, user_id, amount, source_activity, multiplier, created_at)
- [x] Formula kalkulasi kenaikan level diimplementasikan: $\text{Level}(XP) = \lfloor\sqrt{\frac{XP}{100}}\rfloor$ dan ambang batas level $N$: $N^2 \times 100$ XP
- [x] Pengganda tingkat kesulitan (*difficulty multiplier*) diterapkan secara akurat: Easy (1.0x), Medium (1.5x), Hard (2.0x)
- [x] Use case `AwardXP` secara atomik memperbarui `user_progress` (total_xp, current_level, updated_at) dan memasukkan baris baru pada `user_xp_log` dalam satu transaksi database SQLite
- [x] Event / status level-up terdeteksi ketika akumulasi XP melampaui batas level saat ini dan mengembalikan payload informasi kenaikan level
- [x] Unit test mencakup perhitungan batas level nol, kelipatan batas, multiplier desimal (pembulatan integer ke bawah / `math.Floor`), dan validasi input negatif

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/008_create_user_progress.sql`
- `../../../../radovid-bot-api/migrations/009_create_user_xp_log.sql`
- `../../../../radovid-bot-api/internal/domain/progress/entity.go`
- `../../../../radovid-bot-api/internal/domain/progress/calculator.go`
- `../../../../radovid-bot-api/internal/domain/progress/repository.go`
- `../../../../radovid-bot-api/internal/domain/progress/calculator_test.go`
- `../../../../radovid-bot-api/internal/usecase/progress/award_xp.go`
- `../../../../radovid-bot-api/internal/usecase/progress/award_xp_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/progress_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/progress_repository_test.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Branch:** `main`
- **Execution Summary:**
  1. Dibuat migrasi `migrations/008_create_user_progress.sql` membuat tabel `user_progress` (user_id PK, total_xp, current_level, streak_days, last_activity_at, updated_at).
  2. Dibuat migrasi `migrations/009_create_user_xp_log.sql` membuat tabel `user_xp_log` (id, user_id FK, amount, source_activity, multiplier REAL, created_at) dengan indeks pada user_id dan created_at.
  3. Dibuat `internal/domain/progress/entity.go` mendefinisikan `ActivityDifficulty`, `UserProgress`, `XPLog`, dan `LevelUpResult`.
  4. Dibuat `internal/domain/progress/calculator.go` mengimplementasikan fungsi murni `CalculateLevel`, `XPRequiredForLevel`, dan `ApplyMultiplier` menggunakan konstanta `LevelScaleFactor = 100` tanpa magic number; formula level menggunakan `math.Floor(math.Sqrt(xp / 100))`.
  5. Dibuat `internal/domain/progress/repository.go` mendefinisikan port interface `ProgressRepository` dengan `GetOrCreateProgress` dan `AddXPAndLog`.
  6. Dibuat `internal/domain/progress/calculator_test.go` dengan 20+ test case table-driven mencakup batas nol/negatif, threshold tepat, multi-level jump, dan pembulatan desimal multiplier.
  7. Dibuat `internal/usecase/progress/award_xp.go` (`AwardXPUseCase`) orkestrasi validasi input, `ApplyMultiplier`, `GetOrCreateProgress`, `CalculateLevel`, deteksi `LeveledUp`, dan `AddXPAndLog` atomik.
  8. Dibuat `internal/usecase/progress/award_xp_test.go` dengan mock repository, test case positif/negatif mencakup seluruh acceptance criteria.
  9. Dibuat `internal/infrastructure/database/progress_repository.go` implementasi SQLite dengan transaksi `BEGIN/COMMIT/ROLLBACK` atomik pada `AddXPAndLog`, `GetOrCreateProgress` dengan auto-create.
  10. Dibuat `internal/infrastructure/database/progress_repository_test.go` dengan in-memory SQLite end-to-end.
  11. Semua unit test 100% PASS (`go test -count=1 ./...`), `go vet ./...` 0 error, `go build ./cmd/bot` berhasil.
