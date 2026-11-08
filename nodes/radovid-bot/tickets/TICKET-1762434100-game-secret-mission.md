---
id: TICKET-1762434100
plan_id: PLAN-1762434100
title: "Game 10: Secret Mission"
status: Done
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
- [x] Game Type `secret_mission` terdaftar dalam enum entitas game dan pilihan `/radovid start`
- [x] AI Game Master (Ollama Gemma 3 4b) menghasilkan topik diskusi publik dan misi rahasia unik (kata/tata bahasa target) per pemain
- [x] Misi rahasia dikirim secara privat menggunakan Discord Ephemeral interaction response agar pemain lawan tidak dapat melihat target
- [x] Pemain mengirimkan kalimat/paragraf jawaban melalui Discord Modal atau message component
- [x] AI mengevaluasi input pemain dengan output JSON terstruktur: `target_present` (bool), `grammar_score` (1-10), `stealth_naturalness_score` (1-10), `critique_notes` (string)
- [x] Pemenang atau perolehan XP dihitung dari kombinasi skor grammar dan skor keluwesan (stealth/naturalness)
- [x] Unit testing terhadap state machine sesi game, parsing response JSON dari Ollama, dan fallback evaluasi jika AI response tidak valid

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/secret_mission.go`
- `../../../../radovid-bot-api/internal/domain/game/secret_mission_test.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/secret_mission.go`
- `../../../../radovid-bot-api/internal/usecase/game/secret_mission_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/secret_mission.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_secret_mission.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_secret_mission_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Waktu Eksekusi:** 2026-11-07 22:32:00
- **Rangkuman Eksekusi:**
  1. Mendaftarkan konstanta game type `GameTypeSecretMission = "secret_mission"` pada `internal/domain/game/entity.go` dan memverifikasi enum tests pada `entity_test.go`.
  2. Membangun model domain `SecretMissionAssignment`, `SecretMissionEvaluation`, dan `SecretMissionState` pada `internal/domain/game/secret_mission.go` bersama generator katalog misi/topik, validator panjang input, dan deteksi regex pola grammar/kata target. Unit test diuji komprehensif pada `secret_mission_test.go` (100% PASS).
  3. Menyusun template prompt Ollama `secret_mission.tmpl`, parser `parseSecretMission` pada `parser.go`, fallback data `secretMissionFallbacks` pada `fallback.go`, dan unit tests terkait.
  4. Mengembangkan use case `SecretMissionUseCase` pada `internal/usecase/game/secret_mission.go` dengan orkestrasi game lifecycle (`StartGame`, `GetMissionForPlayer`, `SubmitResponse`), integrasi fallback AI evaluation, dan kalkulasi perolehan skor/XP. Unit test use case diuji di `secret_mission_test.go` (100% PASS).
  5. Membuat builder Discord embed (`BuildSecretMissionBriefingEmbed`, `BuildSecretMissionTargetEmbed`, `BuildSecretMissionResultEmbed`) pada `internal/infrastructure/discord/game_templates.go` dan unit test pada `game_templates_test.go`.
  6. Mengimplementasikan handler Discord slash command `/radovid play secret-mission`, interaksi ephemeral button (`secretmission:view:`), modal dialog (`secretmission:open_modal:`), dan modal submission handler (`secretmission:submit:`) pada `game_secret_mission.go` serta wiring di `game_handler.go`, `command_play_wyr.go`, dan `cmd/bot/main.go`.
  7. Seluruh test suite (23 package) lulus 100%, `go vet` 0 error, `go build ./cmd/bot` berhasil, dan Zero-Comment Policy dipatuhi penuh.
