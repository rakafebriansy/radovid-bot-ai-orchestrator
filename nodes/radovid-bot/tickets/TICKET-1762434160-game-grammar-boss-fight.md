---
id: TICKET-1762434160
plan_id: PLAN-1762434160
title: "Game 11: Grammar Boss Fight"
status: Done
priority: High
labels: [Game, Ollama, AI, Discord, BossFight, Gamification]
---

# Deskripsi
Sebagai Pemain / Peserta Sesi Kooperatif,
Saya ingin bertarung melawan "Boss Grammar" (seperti The Syntax Dragon atau Lexicon Fiend) di mana saya dan rekan sesi harus memperbaiki kalimat-kalimat yang rusak secara tata bahasa untuk memberikan serangan (damage) pada HP Boss,
Sehingga proses belajar perbaikan kesalahan grammar (error analysis & correction) menjadi seru, kompetitif, dan terasa seperti permainan RPG.

Grammar Boss Fight mengintegrasikan mekanisme pertarungan RPG ke dalam pembelajaran grammar bahasa Inggris. AI Game Master menciptakan Boss dengan nama tematik dan total HP tertentu (misal: 100 HP). Setiap ronde:
1. Boss melancarkan serangan "Corrupted Sentence" (kalimat berbahasa Inggris dengan kesalahan spesifik: tense mismatch, subject-verb agreement, misplaced modifier, atau dangling participle).
2. Pemain mengirimkan versi kalimat yang sudah diperbaiki via Discord Modal.
3. AI mengevaluasi perbaikan:
   - Koreksi sempurna (*flawless*) = Serangan Kritis / Critical Hit (25-35 damage).
   - Koreksi cukup baik dengan sedikit kekurangan = Serangan Normal / Standard Hit (10-15 damage).
   - Koreksi salah / masih rusak = Serangan Meleset (0 damage) dan Boss melancarkan serangan balasan (*player HP / hearts* berkurang).
4. Pertarungan berlanjut hingga HP Boss habis (kemenangan pemain) atau batas ronde terlampaui.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434160-game-grammar-boss-fight.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Game Type `grammar_boss_fight` terdaftar di enum game dan pilihan menu `/radovid play grammar-boss-fight`
- [x] Entitas Boss Game menyimpan status state: `boss_name`, `max_hp`, `current_hp`, `player_lives`, `current_round`, dan daftar kesalahan terdeteksi
- [x] Ollama Gemma 3 4b menghasilkan kalimat dengan kesalahan tata bahasa tersembunyi sesuai tingkat kesulitan (Easy: subject-verb agreement, Medium: irregular tense/passive, Hard: subjunctive/inversion)
- [x] AI evaluator membandingkan koreksi pemain dengan kalimat rusak asli dan mengembalikan kalkulasi damage (0, 10-15, 25-35) serta penjelasan edukatif
- [x] Tampilan visual HP Boss disajikan menggunakan Unicode Bar grafis (`[████████░░] 80/100 HP`) pada Discord Embed interaktif
- [x] Kondisi akhir terkelola dengan baik: Kemenangan (Boss HP <= 0) memberikan bonus XP besar + achievement `boss_slayer`, atau Kekalahan (pemain kehabisan nyawa/ronde)
- [x] Unit testing terhadap kalkulasi damage, state machine HP, dan parsing format evaluasi pertarungan

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/game/boss_fight.go`
- `../../../../radovid-bot-api/internal/domain/game/boss_fight_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/grammar_boss.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_boss_fight_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Branch:** `main`
- **Delivery Gate:**
  - Higienitas Kode: 100% mematuhi Zero-Comment Policy, English identifier, sanitasi input modal, relative path references.
  - Hard Gate Testing: Seluruh paket (23 packages) lulus 100% via `go test -count=1 ./...`, `go vet ./...` 0 warnings, `go build ./cmd/bot` sukses.
  - Battle Mechanics:
    - Attack tiers: `miss` (0 dmg), `standard` (10-15 dmg), `critical` (25-35 dmg).
    - Unicode Health Bar: `RenderBossHPBar` menghasilkan representasi grafis persis `[████████░░] 80/100 HP`.
    - Player hearts: visualisasi emoji `❤️` dan `💔`.
    - Win condition (Boss HP <= 0): Kemenangan epik, bonus XP besar, memicu achievement `boss_slayer`.
    - Loss condition (Lives <= 0 atau rounds habis): Kekalahan, pesan penyemangat dan XP hiburan.
