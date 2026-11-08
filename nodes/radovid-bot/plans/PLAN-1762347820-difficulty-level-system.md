---
id: PLAN-1762347820
ticket_id: TICKET-1762347820
title: Difficulty Level System
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Difficulty Level System

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762347820](../tickets/TICKET-1762347820-difficulty-level-system.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun sistem klasifikasi tingkat kemahiran bahasa Inggris (`Difficulty Level System`) yang mengintegrasikan layer domain user, persistensi database SQLite, command handler Discord `/radovid settings`, serta orkestrasi prompt inferensi Ollama Gemma 3 4b. Level kemahiran diklasifikasikan menjadi tiga tingkatan terdefinisi (`Beginner`, `Intermediate`, `Advanced`) yang memodulasi kompleksitas leksikal, panjang kalimat AI, dan derajat toleransi evaluasi tata bahasa pada seluruh 15 game Radovid Bot.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Command `/radovid settings`, inisialisasi sesi game di `internal/usecase/game`, dan evaluator di `internal/usecase/feedback`.
  - Komponen Terdampak: `internal/domain/user`, `internal/infrastructure/sqlite`, `internal/infrastructure/ollama`, `internal/delivery/discord`.
  - Potensi Efek Samping: Pengguna lama yang belum memiliki preferensi kesulitan memerlukan nilai default (`Beginner`) yang aman tanpa memerlukan migrasi ulang data yang merusak integritas database (backward compatible).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/007_add_difficulty_to_users.sql` | DDL penambahan kolom `difficulty` pada tabel `users` dengan default `'Beginner'` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/user/difficulty.go` | Definisi enum `Difficulty`, validasi level, metode `CEFRLevel()`, dan fungsi `ParseDifficulty()` |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/user/difficulty_test.go` | Unit test validasi enum, pemetaan CEFR, dan fungsi parsing difficulty |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/user/entity.go` | Penambahan field `Difficulty Difficulty` pada struct `User` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/user/entity_test.go` | Pembaruan unit test struct `User` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/user/repository.go` | Penambahan metode `UpdateDifficulty` dan `GetOrCreateByDiscordID` pada port repository |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/user_repository.go` | Implementasi SQLite repository untuk entitas `User` dan operasi mutasi difficulty |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/user_repository_test.go` | Unit test in-memory SQLite untuk `UserRepository` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/user/settings.go` | Logika usecase pembacaan profil pengguna dan pembaruan preferensi kesulitan |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/user/settings_test.go` | Unit test table-driven usecase pengaturan preferensi pengguna |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Builder Discord Embed tampilan antarmuka pengaturan profil pengguna |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test visual embed builder untuk pengaturan profil |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Pendaftaran slash command `/radovid settings` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/handler.go` | Dispatching router untuk perintah `/radovid settings` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings.go` | Handler slash command `/radovid settings` dan komponen tombol/select menu pembaruan level |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings_test.go` | Unit test handler interaksi Discord untuk pengaturan difficulty |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi `userRepo`, `settingsUseCase`, dan pendaftaran prefix komponen `settings:` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat migrasi SQL `009_add_difficulty_to_users.sql`.
   - Buat unit test pada `settings_test.go` untuk memverifikasi validasi parsing: `ParseDifficulty("beginner")` -> `DifficultyBeginner`, string sembarang -> return `ErrInvalidDifficulty`.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan tipe enum `Difficulty` dan constant: `DifficultyBeginner`, `DifficultyIntermediate`, `DifficultyAdvanced` pada `internal/domain/user/difficulty.go`.
   - Perbarui struct `User` di `internal/domain/user/entity.go` untuk menyertakan field `Difficulty Difficulty`.
   - Implementasikan usecase `UpdateUserDifficulty` dan perbarui query SQLite di `user_repository.go`.
3. **Langkah 3 (Refactor & Integrasi UI / AI):**
   - Buat slash command handler `/radovid settings` yang merespons secara ephemeral dengan tombol atau select menu 3 pilihan level.
   - Hubungkan parameter `Difficulty` ke seluruh pemanggilan prompt template Ollama melalui field konteks `{{.Difficulty}}`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji penanganan akun baru (default tersimpan ke `Beginner`), pengubahan berulang (Beginner -> Advanced -> Intermediate), dan uji konsistensi prompt injection.
   - Verifikasi build sukses tanpa CGO via `CGO_ENABLED=0 go build ./cmd/bot`.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/user/difficulty.go
package user

import "fmt"

type Difficulty string

const (
	DifficultyBeginner     Difficulty = "Beginner"
	DifficultyIntermediate Difficulty = "Intermediate"
	DifficultyAdvanced     Difficulty = "Advanced"
)

func ParseDifficulty(val string) (Difficulty, error) {
	switch val {
	case "Beginner", "beginner":
		return DifficultyBeginner, nil
	case "Intermediate", "intermediate":
		return DifficultyIntermediate, nil
	case "Advanced", "advanced":
		return DifficultyAdvanced, nil
	default:
		return DifficultyBeginner, fmt.Errorf("unknown difficulty level: %s", val)
	}
}

func (d Difficulty) Description() string {
	switch d {
	case DifficultyBeginner:
		return "A1-A2: Focus on basic sentences, everyday vocabulary, and supportive coaching."
	case DifficultyIntermediate:
		return "B1-B2: Natural conversational pace, idiomatic expressions, and balanced feedback."
	case DifficultyAdvanced:
		return "C1-C2: Complex professional topics, sophisticated vocabulary, and meticulous syntax coaching."
	default:
		return "Standard difficulty"
	}
}
```

```go
// migrations/009_add_difficulty_to_users.sql
ALTER TABLE users ADD COLUMN difficulty TEXT NOT NULL DEFAULT 'Beginner';
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Nilai input yang tidak dikenali dari interaksi Discord di-fallback secara aman ke `DifficultyBeginner` dengan log peringatan.
- **Strategi Rollback:** Kolom SQLite `difficulty` memiliki nilai default statis sehingga migration bersifat non-breaking; jika dibatalkan, fallback runtime mengabaikan kolom tanpa memicu panic.

