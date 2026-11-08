---
id: TICKET-1762347640
plan_id: PLAN-1762347640
title: Game 7: Story Relay
status: Done
priority: High
labels: [Game, Storytelling, Multiplayer, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin bermain merangkai cerita kolaboratif bergantian bersama teman dan AI Game Master (Story Relay),
Sehingga kami dapat melatih kemampuan menulis naratif bahasa Inggris, memicu kreativitas dengan plot twist tak terduga dari AI, dan menerima umpan balik gramatikal tanpa memutus keseruan cerita.

Game 7: Story Relay adalah aktivitas kolaboratif berbasis giliran (turn-based) untuk 2 pemain dan 1 AI Game Master. AI mengawali cerita dengan paragraf pembuka yang dramatis. Pemain pertama menyumbangkan kelanjutan cerita (minimal 1-2 kalimat), AI merespons dengan menjembatani cerita serta menyisipkan plot twist yang menarik, lalu giliran berpindah ke pemain kedua. Siklus ini berulang hingga babak klimaks, di mana AI menyusun epilog cerita utuh disertai evaluasi tata bahasa untuk masing-masing kontribusi pemain.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347640-game-story-relay.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Manajemen antrean giliran strictly turn-based antara Player 1, AI Game Master, dan Player 2
- [x] AI Game Master mengawali sesi dengan paragraf pembuka (story hook) yang kaya narasi dan deskripsi latar
- [x] Validasi kontribusi pemain via Discord Modal: minimal 1 kalimat dan maksimal 300 karakter per giliran
- [x] AI secara cerdas menyambung cerita pemain, menjaga konsistensi logika naratif, serta menyisipkan plot twist tak terduga
- [x] Setiap giliran pemain dianalisis secara background untuk mendeteksi kesalahan tata bahasa yang signifikan
- [x] Pada akhir sesi (setelah 6 putaran kombinasi), bot merangkum seluruh cerita menjadi satu novellet utuh beserta kartu review bahasa untuk kedua pemain

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/story_relay.go`
- `../../../../radovid-bot-api/internal/domain/game/story_relay_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay.go`
- `../../../../radovid-bot-api/internal/usecase/game/story_relay_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/story_relay.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_story_relay_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Completed
- **Executed At:** 2026-11-06 08:18:00
- **Summary:**
  - Menambahkan konstanta `GameTypeStoryRelay` pada `internal/domain/game/entity.go` dan validasi pada unit test domain.
  - Mengimplementasikan domain model Story Relay `internal/domain/game/story_relay.go`: `ContributorRole` (`player_1`, `player_2`, `game_master`), `StoryGenre` (5 genre: `mystery`, `fantasy`, `sci_fi`, `adventure`, `comedy`), katalog story hook dinamis (`CatalogStoryHooks`), model `StorySegment`, `PlayerLanguageReview`, `StoryRelaySummary`, serta state machine sesi `StoryRelaySessionState` dengan validasi kepemilikan giliran (`ErrNotYourTurn`), batas panjang teks 10-300 karakter (`ErrSegmentTooShort`, `ErrSegmentTooLong`), mekanisme fallback ke single-player, dan penyusunan cerita utuh.
  - Menulis 100% table-driven unit test `internal/domain/game/story_relay_test.go`.
  - Mengimplementasikan prompt template `internal/infrastructure/ollama/prompts/story_relay.tmpl` dengan delimiter `CONTINUATION:`, `GRAMMAR_NOTES:`, `PLOT_TWIST:`, dan `IS_CLIMAX:` beserta unit test rendering `embed_test.go`.
  - Mengimplementasikan parser keluaran LLM `parseStoryRelay` di `internal/infrastructure/ollama/parser.go` dan unit test `parser_test.go`.
  - Menyediakan fallback story generation `storyRelayFallbacks` pada `internal/infrastructure/ollama/fallback.go` untuk 3 tingkat kesulitan (`Easy`, `Medium`, `Hard`) dan memverifikasi via `fallback_test.go`.
  - Mengimplementasikan Usecase layer `internal/usecase/game/story_relay.go` (`StoryRelayUseCase`) mencakup siklus `StartGame`, `SubmitSentence`, rotasi giliran multiplayer vs single-player, auto-completion di babak klimaks, dan kompilasi evaluasi bahasa untuk setiap kontributor pemain. Diverifikasi dengan table-driven unit test `story_relay_test.go`.
  - Mengimplementasikan helper visual Discord Embed `BuildStoryRelayStartEmbed`, `BuildStoryRelayTurnEmbed`, dan `BuildStoryRelaySummaryEmbed` di `internal/infrastructure/discord/game_templates.go` dan unit test `game_templates_test.go`.
  - Mengimplementasikan delivery Discord handler `internal/delivery/discord/game_story_relay.go`, mendaftarkan slash subcommand `/radovid play story-relay` dengan opsi `genre`, `partner`, dan `difficulty` di `command_def.go`, serta mendispatch perutean pada `command_play_wyr.go` dan `game_handler.go`.
  - Memperbarui `cmd/bot/main.go` untuk wiring `storyRelayUseCase` dan pendaftaran prefix rute interaction `storyrelay:`.
  - Seluruh pengujian lolos 100% (`go test -count=1 ./...`), `go vet ./...` bersih 0 isu, `go build ./...` sukses, dan Zero-Comment Policy dipatuhi secara mutlak di seluruh file Go.
