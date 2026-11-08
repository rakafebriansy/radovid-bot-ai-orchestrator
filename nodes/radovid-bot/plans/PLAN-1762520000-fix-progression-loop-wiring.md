---
id: PLAN-1762520000
ticket_id: TICKET-1762520000
title: "Fix Progression Loop Wiring (XP, Streak & Achievement Integration)"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix Progression Loop Wiring

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520000](../tickets/TICKET-1762520000-fix-progression-loop-wiring.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #1)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Testing](../../../global-guidelines/testing.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menyambungkan use case gamifikasi (`AwardXPUseCase`, `RecordDailyActivityUseCase`, `CheckAndUnlockUseCase`) ke dalam `GameHandler` di layer delivery Discord dan `cmd/bot/main.go` sehingga penyelesaian mini-game maupun jurnal harian secara otomatis memperbarui XP, level, streak harian, dan lencana pencapaian pemain di SQLite.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `cmd/bot/main.go`, `internal/delivery/discord/game_handler.go`, dan seluruh handler penyelesaian game/diary.
  - Komponen Terdampak: `internal/delivery/discord`, `internal/usecase/progress`, `internal/usecase/achievement`.
  - Potensi Efek Samping: Foreign key pada `user_progress`, `user_xp_log`, dan `achievements` merujuk ke `users(id)`. Oleh karena itu, sebelum mencatat XP/streak, helper wajib memanggil `userRepo.GetOrCreateByDiscordID` untuk mendapatkan `u.ID` yang sah.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi `AwardXPUseCase`, `RecordDailyActivityUseCase`, `CheckAndUnlockUseCase` dan registrasi ke `GameHandler` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Menambahkan dependency progression use cases dan method `RecordGameCompletion` pada `GameHandler` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler_test.go` | Menambahkan unit test verifikasi pencatatan XP, streak, dan achievement saat game selesai |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/*.go` | Memanggil `h.RecordGameCompletion` saat sesi mini-game atau diary selesai |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test pada `internal/delivery/discord/game_handler_test.go` yang menyelesaikan sesi game lalu memeriksa apakah XP dan streak pengguna bertambah melalui `GetProgressUseCase`.
   - Jalankan `go test` dan buktikan tes **GAGAL (RED)** karena XP tetap 0 pada kode lama.
2. **Langkah 2 (Implementasi Inti):**
   - Tambahkan setter atau konstruktor ekstensi `SetProgressionUseCases(userRepo, awardXP, recordDaily, checkUnlock)` pada `GameHandler` agar kompatibel dengan signature yang ada serta mudah diuji.
   - Implementasikan method `RecordGameCompletion(ctx, discordUserID, username, baseXP, difficulty, sourceActivity, bossDefeated)` pada `GameHandler`.
   - Hubungkan seluruh handler penyelesaian mini-game (15 game + diary) ke `RecordGameCompletion`.
3. **Langkah 3 (Integrasi `main.go`):**
   - Di `cmd/bot/main.go`, inisialisasi `awardXPUC`, `recordDailyUC`, dan `checkUnlockUC`, lalu pasang ke `gameHandler`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go test -race ./...` dan pastikan 100% PASS.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
func (h *GameHandler) RecordGameCompletion(
	ctx context.Context,
	discordUserID string,
	username string,
	baseXP int,
	diff gamemaster.Difficulty,
	sourceActivity string,
	bossDefeated bool,
) []*domainAchievement.Achievement {
	if h.userRepo == nil || h.awardXPUseCase == nil {
		return nil
	}
	u, err := h.userRepo.GetOrCreateByDiscordID(ctx, discordUserID, username)
	if err != nil || u == nil {
		return nil
	}
	if h.recordDailyUseCase != nil {
		_, _ = h.recordDailyUseCase.RecordDailyActivity(ctx, u.ID, time.Now().UnixMilli())
	}
	if baseXP > 0 {
		actDiff := mapGameDifficultyToProgress(diff)
		_, _ = h.awardXPUseCase.AwardXP(ctx, u.ID, baseXP, actDiff, sourceActivity)
	}
	return h.evaluateAndUnlockAchievements(ctx, u.ID, bossDefeated)
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Jika `baseXP <= 0` (misal pemain kalah di ronde awal tanpa skor), aktivitas harian tetap tercatat namun `AwardXP` tidak dipanggil dengan nilai 0 agar tidak gagal validasi.
- **Strategi Rollback:** Gunakan `git checkout` atau revert commit jika terjadi regresi pada handler pengujian yang sudah ada.

