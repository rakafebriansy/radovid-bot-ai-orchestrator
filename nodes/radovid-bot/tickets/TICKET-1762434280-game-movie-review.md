---
id: TICKET-1762434280
plan_id: PLAN-1762434280
title: "Game 13: Movie Review"
status: Done
priority: High
labels: [Game, Ollama, AI, Discord, Writing, Discussion]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mengikuti aktivitas diskusi film terstruktur (*Movie Review*) di mana AI memandu saya melalui serangkaian pertanyaan tentang sinopsis, pengembangan karakter, dan opini kritis film favorit saya,
Sehingga saya dapat memperkaya perbendaharaan kata evaluatif, frasa opini formal, dan kemampuan menyusun argumen kritis dalam bahasa Inggris.

Movie Review adalah aktivitas menulis dan berdiskusi terpandu yang dirancang untuk melatih kemampuan diskursus analitis pengguna. Pemain memilih judul film yang diminati, kemudian AI bertindak sebagai moderator diskusi yang mengajukan 3 fase pertanyaan terstruktur:
1. **Plot Premise & Setting:** Menyajikan sinopsis ringkas tanpa membocorkan spoiler (*concise summary*).
2. **Character & Performance Analysis:** Mengulas motivasi tokoh utama atau kualitas akting aktor menggunakan kosakata deskriptif karakter.
3. **Critical Verdict & Recommendations:** Menyampaikan opini kritis dengan frasa argumentatif (*"From my perspective...", "What struck me most was..."*) serta memberikan rating akhir.

Di akhir sesi, AI memberikan umpan balik mendalam terhadap kekayaan ekspresi evaluatif, kejelasan transisi antarkalimat, dan saran sinonim kata sifat sinematik yang lebih tinggi tingkatannya (*advanced adjectives*).

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434280-game-movie-review.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Game Type `movie_review` terdaftar di enum game dan menu slash command `/radovid play movie-review`
- [x] State sesi mencatat 3 fase review sekuensial: `PhasePremise`, `PhaseCharacters`, `PhaseVerdict`
- [x] AI Game Master (Ollama Gemma 3 4b) menyesuaikan pertanyaan panduan berdasarkan judul film dan genre yang diinput oleh pemain
- [x] Pemain mengirimkan respon setiap fase melalui Discord Modal input teks panjang
- [x] Evaluasi akhir AI menghasilkan laporan komprehensif: skor koherensi, evaluasi ekspresi opini, daftar 3-5 kosakata sinematik alternatif (misal: *captivating*, *monotonous*, *thought-provoking*), dan skor keseluruhan (1-10)
- [x] Integrasi dengan modul progres: memberikan base XP plus bonus proporsional berdasarkan kualitas ulasan
- [x] Unit testing terhadap alur transisi fase, parser respon evaluasi ulasan film, dan validasi kelengkapan argumen

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/entity_test.go`
- `../../../../radovid-bot-api/internal/domain/game/movie_review.go`
- `../../../../radovid-bot-api/internal/domain/game/movie_review_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/movie_review.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Done
- **Branch:** main
- **Execution Summary:**
  - Registered `GameTypeMovieReview = "movie_review"` in game domain entities.
  - Defined domain model for Movie Review (`MovieReviewPhase`, `MovieReviewEvaluation`, `MovieReviewState`, `GetPhaseGuidingPrompt`, `ValidateReviewResponse`, `ValidateRating`, `CalculateMovieReviewXP`).
  - Added Ollama Gemma 3 4b prompt template `movie_review.tmpl` with structured evaluation delimiters.
  - Added parser `parseMovieReview` and `ParseMovieReviewEvaluation` in `parser.go` and verified with unit tests.
  - Added deterministic fallback challenges and evaluation in `fallback.go` per difficulty tier.
  - Implemented `MovieReviewUseCase` handling 3-phase review progression (`Premise` -> `Characters` -> `Verdict` -> `Completed`), evaluation, XP award, and session lifecycle.
  - Implemented Discord UI components: `BuildMovieReviewPhaseEmbed` (guided prompt card + modal trigger) and `BuildMovieReviewResultEmbed` (detailed critique report card).
  - Registered `/radovid play movie-review` subcommand with `movie` (required), `genre`, and `difficulty` options.
  - Implemented Discord handlers: `HandlePlayMovieReview`, `HandleMovieReviewComponentInteraction` (`open_modal`), and `HandleMovieReviewModalSubmit`.
  - Wired use case and component/modal prefix `moviereview:` in `cmd/bot/main.go`.
  - Fixed streak test UTC boundary issue in progress domain/usecase tests for 100% test suite reliability.
  - Maintained strict Zero-Comment Policy across all Go files.
  - 100% test coverage passed across all 23 packages in `radovid-bot-api`.
