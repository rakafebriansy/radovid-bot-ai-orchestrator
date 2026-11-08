---
id: PLAN-1762434280
ticket_id: TICKET-1762434280
title: "Game 13: Movie Review"
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 13: Movie Review

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434280](../tickets/TICKET-1762434280-game-movie-review.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengembangkan alur aktivitas edukatif "Movie Review" (`movie_review`) dengan state machine 3 fase interaktif, integrasi dialog terpandu bersama Ollama Gemma 3 4b, serta sistem evaluasi kecakapan ekspresi opini dan kosakata sinematik.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler `/radovid play movie-review` dan modal interaksi di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, prompt templates di `internal/infrastructure/ollama`, dan modul audit log XP.
  - Potensi Efek Samping: Panjang teks submission pada fase ulasan film memerlukan penanganan Discord Modal textarea (hingga 2000 karakter) dan penanganan prompt terstruktur agar tidak melampaui context window lokal Ollama.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/movie_review.go` | Model MovieReviewPhase, MovieReviewEvaluation, MovieReviewState, prompt builder, validator, dan skoring XP |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/movie_review_test.go` | Unit test domain model, validasi fase, prompt, dan kalkulasi XP |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeMovieReview = "movie_review"` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity_test.go` | Test registrasi game type movie review |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/movie_review.tmpl` | Template prompt evaluasi 3 fase ulasan film dengan delimiter tags |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Pengujian rendering template prompt movie_review |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser respons evaluasi movie review dan helper konversi domain |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parser respons evaluasi film |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Fallback evaluasi deterministik per tingkat kesulitan |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Pengujian fallback movie review |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/movie_review.go` | Use case orkestrasi 3 fase ulasan film, evaluasi AI, dan XP reward |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/movie_review_test.go` | Unit test use case alur ulasan film dan edge cases |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Embed kartu panduan fase dan laporan hasil evaluasi kritik film |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test builder visual embed ulasan film |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Registrasi opsi subcommand `movie-review` pada grup `play` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Routing subcommand `movie-review` ke handler |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_test.go` | Pengujian opsi command `movie-review` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Injeksi MovieReviewUseCase dan delegator component/modal |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review.go` | Delivery handlers game, modal trigger per fase, dan submit evaluasi |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_movie_review_test.go` | Unit test delivery handlers Discord |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Wiring MovieReviewUseCase dan registrasi prefix router |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Domain Modeling & Formula):**
   - Registrasi `GameTypeMovieReview = "movie_review"`.
   - Definisikan fase `MovieReviewPhasePremise`, `MovieReviewPhaseCharacters`, `MovieReviewPhaseVerdict`, `MovieReviewPhaseCompleted`.
   - Buat fungsi validator teks respon ulasan, rating validator, dan kalkulasi XP.
2. **Langkah 2 (Ollama Prompts, Parser & Fallback):**
   - Susun template prompt `movie_review.tmpl` dengan tag delimiter.
   - Buat parser `parseMovieReview` dan `ParseMovieReviewEvaluation`.
   - Tambahkan fallback respons deterministik per difficulty di `fallback.go`.
3. **Langkah 3 (Use Case & State Orchestration):**
   - Implementasikan `MovieReviewUseCase` dengan metode `StartGame`, `GetState`, dan `SubmitPhase`.
   - Orkestrasi flow: start -> Premise modal -> Characters modal -> Verdict & Rating modal -> panggil evaluasi AI -> award XP.
4. **Langkah 4 (Discord Templates & Handlers):**
   - Buat embed builder `BuildMovieReviewPhaseEmbed` dan `BuildMovieReviewResultEmbed`.
   - Implementasikan command handler `/radovid play movie-review`, component handler `open_modal`, dan modal submit handler.
5. **Langkah 5 (Verification & Zero Comments):**
   - Verifikasi 100% PASS seluruh pengujian unit di semua package (`go test -count=1 ./...`).
   - Verifikasi Zero-Comment Policy (tidak ada komentar `//` atau `/* */` di kode Go baru).

---

## 5. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain menginput judul film fiktif atau aneh. Prompt AI diarahkan untuk bersikap agnostik terhadap kebenaran fakta film riil dan memfokuskan penilaian murni pada struktur argumen dan bahasa Inggris pengguna.
- **Validasi Panjang Teks:** Teks respon ulasan dibatasi minimal 15 karakter untuk mencegah submisi kosong / asal-asalan, dan maksimal 2000 karakter agar sesuai batas Discord TextInput.
- **Graceful Fallback:** Jika inferensi Ollama mengalami timeout/gagal, fallback deterministik berbasis difficulty menyediakan evaluasi berkualitas tanpa memutus alur sesi pengguna.
