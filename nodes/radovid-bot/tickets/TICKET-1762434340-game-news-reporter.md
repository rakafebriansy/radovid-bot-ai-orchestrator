---
id: TICKET-1762434340
plan_id: PLAN-1762434340
title: "Game 14: News Reporter"
status: Done
priority: High
labels: [Game, Ollama, AI, Discord, Speaking, FormalEnglish]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris Tingkat Menengah ke Atas,
Saya ingin memainkan simulasi "News Reporter" di mana AI Game Master memberikan serangkaian fakta peristiwa mentah (5W1H) dan saya harus menyusun serta menyiarkan laporan berita formal dalam bahasa Inggris,
Sehingga keterampilan berbicara/menulis menggunakan register formal, struktur piramida terbalik, dan gaya bahasa jurnalistik saya meningkat secara signifikan.

News Reporter melatih ketangkasan merangkai informasi acak menjadi siaran berita yang terstruktur dan berwibawa (*journalistic broadcast format*). Alur permainan mencakup:
1. AI Game Master mengenerate memo berita mendadak (*breaking news dispatch*) yang memuat fakta mentah 5W1H: *Who* (Siapa), *What* (Apa yang terjadi), *When* (Kapan waktu peristiwa), *Where* (Di mana lokasi), *Why* (Mengapa/Pemicu), dan *How* (Bagaimana prosesnya).
2. Pemain berperan sebagai jurnalis lapangan atau pembawa berita studio, menyusun naskah siaran dalam bahasa Inggris menggunakan konvensi jurnalistik: pembuka formal (*"Good evening, this is [Name] reporting live from..."*), kalimat utama/lead sentence, penyampaian fakta secara runtut, dan kalimat penutup (*"Back to you in the studio"*).
3. AI mengevaluasi: ketepatan pencakupan fakta 5W1H, pemilihan kata formal (*journalistic register*), penggunaan pasif yang tepat (*passive voice for objectivity*), dan kohesi antarparagraf.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434340-game-news-reporter.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Game Type `news_reporter` terdaftar di enum game dan pilihan menu `/radovid play news-reporter`
- [x] AI Game Master (Ollama Gemma 3 4b) menghasilkan skenario breaking news yang bervariasi (isu lingkungan, sains & teknologi, penemuan arkeologi, kejadian unik) lengkap dengan poin fakta 5W1H
- [x] Pemain menginput naskah siaran melalui Discord Modal interaktif
- [x] AI evaluator memvalidasi kelengkapan fakta 5W1H (mendeteksi fakta kunci yang tercakup dan missing facts)
- [x] AI memberikan skor register formal (1-10) dan akurasi tata bahasa (1-10) dengan analisis pemakaian passive voice dan transisi formal
- [x] Menyajikan revisi model broadcast berita yang ideal (*exemplary news script*) sebagai bahan perbandingan belajar pemain
- [x] Unit testing terhadap pembuatan skenario berita, ekstraksi komponen 5W1H, use case orkestrasi, fallback deterministik, dan parsing evaluasi berita dari model AI

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/news_reporter.go`
- `../../../../radovid-bot-api/internal/domain/game/news_reporter_test.go`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/news_reporter.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_news_reporter_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Completed
- **Tanggal:** 2026-11-08 07:50:00
- **Ringkasan Eksekusi:**
  1. Menambahkan enum `GameTypeNewsReporter = "news_reporter"` pada `internal/domain/game/entity.go` beserta unit test di `entity_test.go`.
  2. Mengimplementasikan model domain `internal/domain/game/news_reporter.go` (`FactSheet5W1H`, `NewsDispatch`, `NewsAssessment`, `NewsReporterState`, `GetDefaultNewsDispatch`, `ValidateBroadcastScript`, `CalculateNewsReporterScore`, `CalculateNewsReporterXP`) dengan unit test komprehensif di `news_reporter_test.go` (100% coverage).
  3. Menyusun prompt template Ollama `internal/infrastructure/ollama/prompts/news_reporter.tmpl` dengan verifikasi di `embed_test.go`.
  4. Menambahkan parser respons terstruktur `parseNewsReporter` dan helper domain `ParseNewsReporterEvaluation` di `internal/infrastructure/ollama/parser.go` beserta unit test di `parser_test.go`.
  5. Menambahkan skenario fallback deterministik per tingkat kesulitan `newsReporterFallbacks` pada `internal/infrastructure/ollama/fallback.go` dan verifikasi di `fallback_test.go`.
  6. Mengimplementasikan interaksi bisnis use case `NewsReporterUseCase` pada `internal/usecase/game/news_reporter.go` dengan unit test lengkap di `news_reporter_test.go`.
  7. Membangun Discord embed visual `BuildNewsReporterDispatchEmbed` dan `BuildNewsReporterEvaluationEmbed` di `internal/infrastructure/discord/game_templates.go` serta unit test di `game_templates_test.go`.
  8. Menambahkan subcommand slash `news-reporter` (opsi `category` dan `difficulty`) di `command_def.go` dan router delegator di `command_play_wyr.go`.
  9. Mengimplementasikan delivery handlers `HandlePlayNewsReporter`, `HandleNewsReporterComponentInteraction`, dan `HandleNewsReporterModalSubmit` di `internal/delivery/discord/game_news_reporter.go` dan `game_news_reporter_test.go`.
  10. Menginjeksi `newsReporterUseCase` ke `GameHandler` dan wire-up di `cmd/bot/main.go`.
  11. Verifikasi kelulusan 100% seluruh test suite (23 packages), `go vet ./...`, dan kepatuhan mutlak zero-comment policy.
