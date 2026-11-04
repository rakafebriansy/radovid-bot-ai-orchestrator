---
id: TICKET-1762434160
plan_id: PLAN-1762434160
title: "Game 11: Grammar Boss Fight"
status: Todo
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
- [ ] Game Type `grammar_boss_fight` terdaftar di enum game dan pilihan menu `/radovid start`
- [ ] Entitas Boss Game menyimpan status state: `boss_name`, `max_hp`, `current_hp`, `player_lives`, `current_round`, dan daftar kesalahan terdeteksi
- [ ] Ollama Gemma 3 4b menghasilkan kalimat dengan kesalahan tata bahasa tersembunyi sesuai tingkat kesulitan (Easy: subject-verb agreement, Medium: irregular tense/passive, Hard: subjunctive/inversion)
- [ ] AI evaluator membandingkan koreksi pemain dengan kalimat rusak asli dan mengembalikan kalkulasi damage (0, 10-15, 25-35) serta penjelasan edukatif
- [ ] Tampilan visual HP Boss disajikan menggunakan Unicode Bar grafis (`[████████░░] 80/100 HP`) pada Discord Embed interaktif
- [ ] Kondisi akhir terkelola dengan baik: Kemenangan (Boss HP <= 0) memberikan bonus XP besar + achievement `boss_slayer`, atau Kekalahan (pemain kehabisan nyawa/ronde)
- [ ] Unit testing terhadap kalkulasi damage, state machine HP, dan parsing format evaluasi pertarungan

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/boss_fight.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
