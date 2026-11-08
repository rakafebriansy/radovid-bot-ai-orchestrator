---
id: PLAN-1762434160
ticket_id: TICKET-1762434160
title: "Game 11: Grammar Boss Fight"
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 11: Grammar Boss Fight

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434160](../tickets/TICKET-1762434160-game-grammar-boss-fight.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengembangkan mode game RPG interaktif "Grammar Boss Fight" (`grammar_boss_fight`) di mana pemain menyerang HP monster virtual dengan cara mengidentifikasi dan membetulkan kalimat salah grammar yang digenerate oleh Ollama, disertai visualisasi HP bar interaktif pada Discord embed.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord interaction dispatcher (`command_play_wyr.go` dan modal submission handler `game_boss_fight.go`).
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, progress XP awarder, dan achievement checker (`boss_slayer`).
  - Potensi Efek Samping: Multi-user turn synchronization: jika ada 2 pemain dalam sesi, giliran menyerang atau submit jawaban bergantian secara teratur.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/boss_fight.go` | Model Boss, AttackTier, BossFightState, ApplyBossDamage, ApplyPlayerCounterAttack |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/boss_fight_test.go` | Unit test domain model, tier damage, pengurangan HP, lives, dan sanitasi input |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeGrammarBossFight = "grammar_boss_fight"` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity_test.go` | Pengujian enum konstanta `GameTypeGrammarBossFight` |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/grammar_boss.tmpl` | Template prompt evaluasi perbaikan kalimat dan perhitungan damage |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Pengujian render template prompt `grammar_boss` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser respons evaluasi `parseGrammarBoss` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parser respons boss fight |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Fallback data challenge dan evaluasi boss fight |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback data boss fight |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/boss_fight.go` | Use case alur pertarungan boss, evaluasi perbaikan, mutasi HP & lives |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/boss_fight_test.go` | Unit test use case alur pertarungan, critical hit, miss, victory, game over, dan fallback |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Builder Discord battle embed dengan HP bar boss `[████████░░] 80/100 HP`, player hearts, victory & defeat embeds |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test visual embed boss battle, HP bar, victory, dan defeat |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Pendaftaran opsi subcommand `grammar-boss-fight` pada grup `play` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_test.go` | Pengujian pendaftaran subcommand `grammar-boss-fight` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Perutean subcommand `grammar-boss-fight` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Integrasi `BossFightUseCase` ke `GameHandler` dan penambahan delegator handler |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight.go` | Delivery handler `HandlePlayBossFight`, `HandleBossFightComponentInteraction`, `HandleBossFightModalSubmit` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight_test.go` | Unit test delivery handler boss fight |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi usecase dan registrasi router prefix `bossfight:` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/usecase/game/boss_fight_game_test.go`.
   - Simulasikan skenario pertarungan: Boss 100 HP diserang dengan koreksi Critical (30 damage) -> sisa HP 70. Koreksi salah -> damage 0, player hearts berkurang 1.
   - Jalankan `go test` dan pastikan hasil RED.
2. **Langkah 2 (Domain Modeling & Damage Calculation):**
   - Definisikan struct `BossState`: `Name`, `MaxHP`, `CurrentHP`, `PlayerLives`, `RoundNumber`, `ActivePrompt`.
   - Buat fungsi murni `ApplyDamage(currentHP int, damage int) (int, bool)` untuk menghitung sisa HP dan status kekalahan boss (`isDefeated`).
3. **Langkah 3 (Use Case & AI Integration):**
   - Rancang prompt `GenerateBossChallenge(difficulty)` di `prompts.go` yang menghasilkan kalimat dengan kesalahan spesifik dan penjelasan kunci jawaban tersembunyi.
   - Rancang prompt `EvaluateBossCorrection(original, broken, playerInput)` yang menghasilkan JSON: `{ "is_correct": true, "accuracy_tier": "critical", "damage": 30, "explanation": "..." }`.
   - Hubungkan penuntasan game: saat HP <= 0, picu pemberian 100 base XP + unlock achievement `boss_slayer`.
4. **Langkah 4 (Delivery Gate & Visual Polish):**
   - Buat embed Discord dinamis yang menampilkan gambar ilustrasi boss, nama boss (misal: "🐲 Syntax Dragon"), dan Unicode health bar `[██████░░░░] 60/100 HP`.
   - Pastikan sanitasi input pemain sebelum dikirim ke engine evaluasi AI.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/boss_fight.go
package game

type AttackTier string

const (
	AttackMiss     AttackTier = "miss"
	AttackStandard AttackTier = "standard"
	AttackCritical AttackTier = "critical"
)

type BossFightState struct {
	BossName    string     `json:"boss_name"`
	MaxHP       int        `json:"max_hp"`
	CurrentHP   int        `json:"current_hp"`
	PlayerLives int        `json:"player_lives"`
	BrokenText  string     `json:"broken_text"`
	TargetError string     `json:"target_error"`
	Status      GameStatus `json:"status"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain mengirim kembali kalimat rusak tanpa perubahan apapun. Evaluator harus langsung menetapkan tier `AttackMiss` (0 damage) tanpa menghabiskan kuota inferensi kompleks.
- **Strategi Rollback:** Jika terjadi timeout respon dari Ollama di tengah pertarungan, simpan state HP saat ini dan izinkan submission ulang tanpa mengurangi nyawa pemain.
