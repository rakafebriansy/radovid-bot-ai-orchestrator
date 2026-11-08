---
id: PLAN-1762433920
ticket_id: TICKET-1762433920
title: Achievements System
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Achievements System

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762433920](../tickets/TICKET-1762433920-achievements-system.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun domain katalog lencana, mesin evaluasi kriteria pencapaian (Achievement Evaluator), use case unlock otomatis berhadiah XP, dan penyaji embed perayaan pembukaan lencana di Discord.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Use case akhir game session, streak tracker, dan modul koreksi grammar.
  - Komponen Terdampak: `internal/domain/achievement`, `internal/usecase/achievement`, tabel `achievements`, dan Discord embed builder.
  - Potensi Efek Samping: Multi-unlock dalam satu sesi (misal: user menyelesaikan game pertama dan langsung mendapat First Game + Level Up) harus diproses berurutan tanpa deadlock.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/010_create_achievements.sql` | DDL pembuatan tabel achievements dan indeks unique |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/achievement/entity.go` | Struct Achievement, UserAchievement, UserMetrics |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/achievement/catalog.go` | Registry statis katalog seluruh achievement dan definisi threshold |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/achievement/evaluator.go` | Pure function mencocokkan user metric terhadap trigger criteria |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/achievement/repository.go` | Interface port repository query dan unlock achievement |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/achievement/evaluator_test.go` | Unit test evaluasi milestone pencapaian |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock.go` | Use case orkestrasi pengecekan, persistensi lencana baru, dan bonus XP |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock_test.go` | Unit test skenario unlocking dengan mock dependencies |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/achievement_repository.go` | Implementasi query SQLite untuk tabel achievements |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/achievement_repository_test.go` | Unit test in-memory SQLite untuk achievement repository |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Penambahan fungsi helper pembuatan Discord embed achievement unlocked |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test untuk embed achievement unlocked |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `internal/domain/achievement/evaluator_test.go`.
   - Tes skenario: user dengan total game = 1 dan belum punya `first_game` -> harus lolos evaluasi. User yang sudah memiliki ID `first_game` di daftar `existingAchievements` -> tidak boleh dimasukkan ulang.
   - Pastikan tes gagal (RED).
2. **Langkah 2 (Domain Modeling & Catalog):**
   - Definisikan struct `Achievement` (`ID`, `Name`, `Description`, `IconEmoji`, `XPReward`, `ConditionFunc`).
   - Buat map katalog statis di `catalog.go` yang memuat seluruh lencana fase ini.
   - Implementasikan fungsi evaluator `FindEligibleAchievements(metrics UserMetrics, earnedIDs map[string]bool) []Achievement`.
3. **Langkah 3 (Repository & Use Case Implementation):**
   - Implementasikan interface `AchievementRepository` di `sqlite/achievement_repository.go` dengan query `GetUserAchievements(ctx, userID)` dan `SaveAchievement(ctx, userAchievement)`.
   - Bangun use case `CheckAndUnlockAchievements(ctx, userID, metrics)` yang memanggil evaluator, menyimpan lencana baru dalam database, dan memicu `AwardXP` untuk setiap lencana yang berhasil dibuka.
4. **Langkah 4 (Embed Presentation & Delivery Gate):**
   - Tambahkan fungsi embed builder `BuildAchievementEmbed(achievements []Achievement) *discordgo.MessageEmbed` dengan warna emas/kuning perayaan.
   - Jalankan delivery gate: pastikan tidak ada query N+1 saat memvalidasi daftar achievement yang telah didapat pengguna.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/achievement/catalog.go
package achievement

type AchievementID string

const (
	AchievementFirstGame     AchievementID = "first_game"
	AchievementStreak7       AchievementID = "streak_7"
	AchievementStreak30      AchievementID = "streak_30"
	AchievementGrammarMaster AchievementID = "grammar_master"
	AchievementVocabChampion AchievementID = "vocab_champion"
	AchievementBossSlayer    AchievementID = "boss_slayer"
)

type Achievement struct {
	ID          AchievementID `json:"id"`
	Name        string        `json:"name"`
	Description string        `json:"description"`
	IconEmoji   string        `json:"icon_emoji"`
	XPReward    int           `json:"xp_reward"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Apabila pengguna membuka 3 achievement sekaligus dalam 1 aksi, seluruhnya harus dicatat tanpa melewatkan satupun reward XP dan dirangkum dalam satu pesan embed agar tidak membanjiri channel Discord dengan spam.
- **Strategi Rollback:** Kolom `(user_id, achievement_type)` dilindungi oleh unique constraint pada SQLite; jika ada race condition duplikasi insert, galat `sqlite3.ErrConstraintUnique` diabaikan secara anggun (*ignored gracefully*) sebagai operasi idempoten.
