---
id: TICKET-1762347580
plan_id: PLAN-1762347580
title: Game 6: One Minute Challenge
status: Done
priority: High
labels: [Game, Fluency, Writing, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin mengikuti tantangan kelancaran berekspresi (One Minute Challenge) dengan topik tak terduga dan batas waktu ketat 60 detik,
Sehingga saya dapat melatih kecepatan berpikir dan kelancaran (fluency) menyusun kalimat bahasa Inggris secara spontan tanpa rasa takut salah.

Game 6: One Minute Challenge menitikberatkan pada aspek kelancaran (fluency) dan produksi bahasa Inggris spontan, bukan pada kesempurnaan gramatikal semata. Bot menyajikan topik acak yang menarik, mengaktifkan timer 60 detik di Discord, dan menerima paragraf ekspresif pengguna. AI kemudian mengevaluasi kuantitas kata (word count), kekayaan kosakata (lexical variety), koherensi gagasan, dan memberikan skor kelancaran secara objektif.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347580-game-one-minute-challenge.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] AI Game Master menghasilkan topik impromptu yang kreatif dan memancing imajinasi secara dinamis
- [x] Timer 60 detik ditampilkan di Discord Embed menggunakan format Discord relative timestamp (`<t:TIMESTAMP:R>`)
- [x] Tombol submit memicu modal penulisan teks dengan validasi batas waktu (submission setelah deadline ditolak atau diberi penalti)
- [x] Sistem menghitung metrik kuantitatif: total jumlah kata (word count) dan rasio variasi leksikal (unique words ratio)
- [x] AI mengevaluasi aspek kualitatif: kelancaran alur pemikiran, koherensi paragraf, serta saran ekspresi lanjutan
- [x] Hasil evaluasi disajikan dalam embed terstruktur dengan badge skor kelancaran (Speed & Fluency Score)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/one_minute.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_one_minute.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Completed
- **Timestamp:** 2026-11-06 00:25:00
- **Hasil Eksekusi:**
  1. Menambahkan entitas `internal/domain/game/one_minute.go` (`FluencyRank`, `OneMinuteTopic`, `CatalogOneMinuteTopics`, `FluencyMetrics`, `OneMinuteChallengeState`), helper komputasi analitik (`TokenizeWords`, `CalculateLexicalDiversity`, `CalculateSpeedFluencyScore`, `DetermineFluencyRank`), konstanta `GameTypeOneMinuteChallenge` pada `entity.go`, serta unit test `one_minute_test.go`.
  2. Menyusun template prompt Ollama `internal/infrastructure/ollama/prompts/one_minute.tmpl` dan unit test rendering cache `embed_test.go`.
  3. Mengimplementasikan parser keluaran LLM `parseOneMinute` pada `internal/infrastructure/ollama/parser.go`, unit test `parser_test.go`, bank fallback `oneMinuteFallbacks` di `internal/infrastructure/ollama/fallback.go`, dan unit test `fallback_test.go`.
  4. Mengimplementasikan use case layer `internal/usecase/game/one_minute.go` (`OneMinuteUseCase`), validasi waktu penyerahan (grace period 10s), sanitasi masukan teks, transisi status sesi ke `COMPLETED`, dan unit test table-driven `one_minute_test.go`.
  5. Membangun visual Discord Embed `BuildOneMinuteChallengeEmbed` (dengan countdown `<t:EPOCH:R>`) dan `BuildOneMinuteEvaluationEmbed` (dengan visual card skor kelancaran, analitik kuantitatif, dan saran kosakata) pada `internal/infrastructure/discord/game_templates.go` serta pengujian di `game_templates_test.go`.
  6. Mengimplementasikan Discord delivery handler `internal/delivery/discord/game_one_minute.go` dan unit test `game_one_minute_test.go`.
  7. Menambahkan subcommand `one-minute-challenge` pada `command_def.go`, perutean pada `command_play_wyr.go` dan `game_handler.go`, serta wiring komponen di `cmd/bot/main.go`.
  8. Memverifikasi seluruh pengujian (`go test -count=1 ./...`), `go vet ./...`, dan `go build ./...` lulus 100% dan mematuhi Zero-Comment Policy.

