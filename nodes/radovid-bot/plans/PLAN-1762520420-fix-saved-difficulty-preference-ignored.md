---
id: PLAN-1762520420
ticket_id: TICKET-1762520420
title: "Honor User Saved Difficulty Preference Across All Mini-Games"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Honor User Saved Difficulty Preference

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520420](../tickets/TICKET-1762520420-fix-saved-difficulty-preference-ignored.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #8)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Testing](../../../global-guidelines/testing.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menghubungkan `SettingsUseCase` dengan seluruh 14 game handler dan component handler di `internal/delivery/discord/` melalui helper `h.ResolveDifficulty(ctx, userID, username, explicitDifficultyStr)` agar preferensi kesulitan yang dipilih pengguna pada `/radovid settings` benar-benar diterapkan saat memulai atau melanjutkan permainan.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: Seluruh `handlePlay*` dan component/modal handler di `internal/delivery/discord/`.
  - Komponen Terdampak: `internal/delivery/discord/game_handler.go` dan 16 berkas handler game.
  - Potensi Efek Samping: Jika `h.settingsUseCase` bernilai `nil` (misal pada unit test terisolasi), helper secara aman memberikan fallback ke `gamemaster.DifficultyMedium`.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Menambahkan method `ResolveDifficulty(ctx, userID, username, explicitDiff)` pada `GameHandler` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler_test.go` | Menambahkan unit test resolusi difficulty dari `SettingsUseCase` ke `gamemaster.Difficulty` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/*.go` | Mengganti fallback statis `gamemaster.DifficultyMedium` dengan `h.ResolveDifficulty(...)` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test di mana pengguna menyetel difficulty `Advanced` lewat `settingsUseCase`, lalu menjalankan `/radovid play boss-fight` tanpa opsi `difficulty`.
   - Jalankan `go test` dan buktikan tes **GAGAL (RED)** karena game dimulai dengan `DifficultyMedium` alih-alih `DifficultyHard`.
2. **Langkah 2 (Implementasi Inti):**
   - Implementasikan `ResolveDifficulty` di `game_handler.go`:
     - Jika `explicitDiff` valid (`easy`, `medium`, `hard`), gunakan nilai tersebut.
     - Jika kosong dan `h.settingsUseCase != nil`, ambil `u, err := h.settingsUseCase.GetUserSettings(ctx, userID, username)` lalu petakan `Beginner -> easy`, `Intermediate -> medium`, `Advanced -> hard`.
     - Jika `settingsUseCase == nil` atau error, gunakan `gamemaster.DifficultyMedium`.
   - Perbarui seluruh handler game untuk memanggil `h.ResolveDifficulty`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh test di `internal/delivery/discord` lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
func (h *GameHandler) ResolveDifficulty(ctx context.Context, userID, username, explicitStr string) gamemaster.Difficulty {
	cleaned := gamemaster.Difficulty(strings.ToLower(strings.TrimSpace(explicitStr)))
	if cleaned.IsValid() {
		return cleaned
	}
	if h.settingsUseCase != nil && strings.TrimSpace(userID) != "" {
		u, err := h.settingsUseCase.GetUserSettings(ctx, userID, username)
		if err == nil && u != nil {
			switch u.Difficulty {
			case domainUser.DifficultyBeginner:
				return gamemaster.DifficultyEasy
			case domainUser.DifficultyIntermediate:
				return gamemaster.DifficultyMedium
			case domainUser.DifficultyAdvanced:
				return gamemaster.DifficultyHard
			}
		}
	}
	return gamemaster.DifficultyMedium
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Argumen eksplisit pada slash command (misal `/radovid play taboo difficulty:hard`) tetap memiliki prioritas lebih tinggi daripada preferensi default pengguna.
- **Strategi Rollback:** Revert perubahan pada `game_handler.go` dan berkas handler terkait.

