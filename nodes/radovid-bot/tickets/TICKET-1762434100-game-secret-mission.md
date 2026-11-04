---
id: TICKET-1762434100
plan_id: PLAN-1762434100
title: "Game 10: Secret Mission"
status: Todo
priority: High
labels: [Game, Ollama, AI, Discord, SecretMission]
---

# Deskripsi
Sebagai Pemain / Peserta Sesi Belajar,
Saya ingin memainkan mini-game "Secret Mission" di mana AI Game Master memberikan misi kosakata atau pola tata bahasa rahasia yang harus saya selipkan ke dalam percakapan tanpa terkesan janggal,
Sehingga kemampuan menulis kreatif dan keluwesan penggunaan kosakata kontekstual saya terlatih secara menyenangkan dan menantang.

Secret Mission adalah game sosial berbasis teks di mana setiap pemain secara rahasia (via Discord Ephemeral message atau DM) menerima "Target Word" atau "Grammar Target" (contoh: kata *flabbergasted*, atau struktur *Third Conditional*). Pemain harus membuat paragraf atau kalimat diskusi sesuai topik yang ditentukan oleh AI. Setelah pemain merespons, AI Game Master mengevaluasi:
1. Apakah target rahasia berhasil digunakan secara gramatikal dan akurat?
2. Seberapa alami (*naturalness score*) integrasi kalimat tersebut dalam alur wacana (apakah terkesan dipaksakan)?
3. AI memberikan penilaian deteksi dan feedback ekspresi bahasa Inggris yang lebih elegan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434100-game-secret-mission.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Game Type `secret_mission` terdaftar dalam enum entitas game dan pilihan `/radovid start`
- [ ] AI Game Master (Ollama Gemma 3 4b) menghasilkan topik diskusi publik dan misi rahasia unik (kata/tata bahasa target) per pemain
- [ ] Misi rahasia dikirim secara privat menggunakan Discord Ephemeral interaction response agar pemain lawan tidak dapat melihat target
- [ ] Pemain mengirimkan kalimat/paragraf jawaban melalui Discord Modal atau message component
- [ ] AI mengevaluasi input pemain dengan output JSON terstruktur: `target_present` (bool), `grammar_score` (1-10), `stealth_naturalness_score` (1-10), `critique_notes` (string)
- [ ] Pemenang atau perolehan XP dihitung dari kombinasi skor grammar dan skor keluwesan (stealth/naturalness)
- [ ] Unit testing terhadap state machine sesi game, parsing response JSON dari Ollama, dan fallback evaluasi jika AI response tidak valid

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/secret_mission.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/secret_mission_game.go`
- `../../../../radovid-bot-api/internal/usecase/game/secret_mission_game_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
