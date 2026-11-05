# CHANGELOG — Radovid Bot Node

> **Format:** Descending (terbaru di atas). Setiap entry WAJIB menggunakan struktur baku dari `../../global-docs/templates/changelog_entry_template.md`.

---

### [2026-11-05 08:08:00] - Database Query Builder Utility (TICKET-1762261090)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762261090` dan rencana `PLAN-1762261090` (Fase 2, Item 2.4: Core Game Engine / Infrastructure) untuk pustaka utilitas query builder SQL berparameter yang dapat digunakan kembali (*reusable*) mencakup bulk insert, batch update, bulk case update, batch delete, dan batch select-in berbasis standard library Go dengan sanitasi identifier anti-SQL injection.
- **Perubahan:**
  - [Added] Utilitas query builder `internal/infrastructure/database/query_builder.go` (`ValidateIdentifier`, `BuildBulkInsertQuery`, `BuildBatchUpdateQuery`, `BuildCaseUpdateQuery`, `BuildBatchDeleteQuery`, `BuildSelectInQuery`, `buildPlaceholders`) dengan alokasi buffer efisien dan validasi regex/rune identifier.
  - [Added] Unit test table-driven `internal/infrastructure/database/query_builder_test.go` dengan 35+ test case memvalidasi seluruh variasi fungsi builder, kasus batas (0 baris, slice kosong), dan deteksi upaya injeksi SQL ilegal pada identifier.
  - [Changed] Me-refactor `CreateSession` pada `internal/infrastructure/database/session_repository.go` untuk mengonsumsi `BuildBulkInsertQuery` secara terpadu dan efisien.
  - [Changed] Memperbarui status `TICKET-1762261090` menjadi `Done` dan `PLAN-1762261090` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 2.4 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/internal/infrastructure/database/query_builder.go`, `../../../../radovid-bot-api/internal/infrastructure/database/query_builder_test.go`, `../../../../radovid-bot-api/internal/infrastructure/database/session_repository.go`, `../../../../radovid-bot-api/internal/infrastructure/database/session_repository_test.go`, `tickets/TICKET-1762261090-database-query-builder-utility.md`, `plans/PLAN-1762261090-database-query-builder-utility.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 19:58:00] - Game Session State Machine (TICKET-1762261060)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762261060` dan rencana `PLAN-1762261060` (Fase 2, Item 2.3: Core Game Engine) untuk finite state machine (FSM) siklus hidup sesi game Discord (`pending` -> `active` -> `completed` / `cancelled`), persistensi tabel SQLite (`game_sessions`, `session_participants`), pencegahan sesi aktif ganda, serta background goroutine ticker pembersih sesi kedaluwarsa.
- **Perubahan:**
  - [Added] Migrasi database `migrations/002_create_game_sessions.sql` (tabel sesi dengan epoch ms dan indeks komposit `(channel_id, status)`) dan `migrations/003_create_session_participants.sql` (tabel peserta relasi FK ke sesi dan pengguna).
  - [Added] Domain entity & FSM `internal/domain/session/entity.go` (`GameSession`, `SessionParticipant`, `Start`, `Complete`, `Cancel`, `AddParticipant`, validasi terminal state dan kapasitas peserta).
  - [Added] Typed sentinel errors `internal/domain/session/errors.go` (`ErrInvalidStateTransition`, `ErrSessionNotFound`, `ErrSessionAlreadyActive`, `ErrMaxParticipantsReached`, `ErrParticipantAlreadyExists`, dll.).
  - [Added] Repository interface `internal/domain/session/repository.go` (`Repository`).
  - [Added] Unit test domain FSM `internal/domain/session/entity_test.go` memverifikasi seluruh siklus hidup transisi dan kasus batas partisipan.
  - [Added] Concrete SQLite repository `internal/infrastructure/database/session_repository.go` dengan query terparameterisasi dan atomic state transition.
  - [Added] Integration test SQLite repository `internal/infrastructure/database/session_repository_test.go` memvalidasi operasi CRUD dan bulk cancellation sesi stale.
  - [Added] Use case session manager `internal/usecase/session/session_manager.go` (`SessionManager`, `CreateSession`, `JoinSession`, `StartSession`, `CompleteSession`, `CancelSession`, `GetActiveSession`) dengan proteksi sesi aktif duplikat per-channel.
  - [Added] Background timeout ticker `internal/usecase/session/timeout_ticker.go` (`TimeoutTicker`, `Start`, `Stop`, `CleanupExpiredSessions`) untuk mendeteksi dan membatalkan sesi kedaluwarsa secara otomatis.
  - [Added] Unit test usecase `internal/usecase/session/session_manager_test.go` memvalidasi alur use case, concurrency ticker, dan mock repository.
  - [Changed] Memperbarui status `TICKET-1762261060` menjadi `Done` dan `PLAN-1762261060` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 2.3 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/migrations/002_create_game_sessions.sql`, `../../../../radovid-bot-api/migrations/003_create_session_participants.sql`, `../../../../radovid-bot-api/internal/domain/session/entity.go`, `../../../../radovid-bot-api/internal/domain/session/errors.go`, `../../../../radovid-bot-api/internal/domain/session/repository.go`, `../../../../radovid-bot-api/internal/domain/session/entity_test.go`, `../../../../radovid-bot-api/internal/infrastructure/database/session_repository.go`, `../../../../radovid-bot-api/internal/infrastructure/database/session_repository_test.go`, `../../../../radovid-bot-api/internal/usecase/session/session_manager.go`, `../../../../radovid-bot-api/internal/usecase/session/timeout_ticker.go`, `../../../../radovid-bot-api/internal/usecase/session/session_manager_test.go`, `tickets/TICKET-1762261060-game-session-state-machine.md`, `plans/PLAN-1762261060-game-session-state-machine.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 18:24:00] - Docker Containerization & Ollama Orchestration (TICKET-1762261030)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762261030` dan rencana `PLAN-1762261030` (Fase 2, Item 2.2) untuk kontainerisasi aplikasi Radovid Bot dan orkestrasi layanan Ollama offline dengan otomasi penarikan model Gemma 4b (`gemma3:4b`).
- **Perubahan:**
  - [Added] Konfigurasi `.dockerignore` untuk mengabaikan direktori git, cache build, database lokal, dan file environment sensitif.
  - [Added] Multi-stage `Dockerfile` (build stage `golang:alpine` memproduksi biner CGO-free, runner stage `alpine:3.20` dengan bundle certificates, tzdata, dan volume SQLite `/app/data`).
  - [Added] Orkestrasi `docker-compose.yml` terpadu dengan healthcheck pada service `ollama`, auto-pull model `gemma3:4b` pada service `ollama-pull`, serta service `radovid-bot` dengan dependensi sekuensial terjamin (`service_healthy` dan `service_completed_successfully`).
  - [Changed] Memperbarui status `TICKET-1762261030` menjadi `Done` dan `PLAN-1762261030` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 2.2 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/.dockerignore`, `../../../../radovid-bot-api/Dockerfile`, `../../../../radovid-bot-api/docker-compose.yml`, `tickets/TICKET-1762261030-docker-containerization-ollama.md`, `plans/PLAN-1762261030-docker-containerization-ollama.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 16:30:00] - Offline Ollama HTTP Client Wrapper (TICKET-1762261000)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762261000` dan rencana `PLAN-1762261000` (Fase 2, Item 2.1) untuk implementasi HTTP client wrapper inferensi lokal offline menggunakan Ollama API (`http://localhost:11434`) model Gemma 4b (`gemma3:4b`).
- **Perubahan:**
  - [Added] Typed sentinel errors `internal/infrastructure/ollama/errors.go` (`ErrTimeout`, `ErrConnectionRefused`, `ErrOllamaAPI`, `ErrEmptyPrompt`, `ErrEmptyBaseURL`, `ErrEmptyModel`, `ErrNilHTTPClient`).
  - [Added] Struct DTO payload `internal/infrastructure/ollama/models.go` (`GenerateRequest`, `GenerateResponse`, `ErrorResponse`).
  - [Added] Sanitasi prompt `internal/infrastructure/ollama/sanitize.go` (`SanitizePrompt`) yang membersihkan seluruh format mention Discord (`<@...>`, `<@!...>`, `<#...>`, `<@&...>`), menormalisasi spasi, dan memotong prompt panjang pada batas 2000 rune multi-byte secara aman.
  - [Added] Unit test sanitasi table-driven `internal/infrastructure/ollama/sanitize_test.go` memvalidasi kasus batas (empty, whitespace, mention-only, emoji, Japanese multi-byte text, dan rune truncation).
  - [Added] HTTP client wrapper `internal/infrastructure/ollama/client.go` (`Client`, `NewClient`, `Generate`, `SetBackoff`) dengan fail-fast validation, context timeout propagation, exponential backoff retry (maksimal 2 retries untuk HTTP 5xx dan socket error), serta penolakan 4xx tanpa retry.
  - [Added] Unit test table-driven `internal/infrastructure/ollama/client_test.go` menggunakan in-memory `http.RoundTripper` memvalidasi sukses inferensi, validasi fail-fast, HTTP 4xx tanpa retry, recovery retry HTTP 5xx, retry exhaustion, timeout handling, dan connection refused.
  - [Changed] Memperbarui status `TICKET-1762261000` menjadi `Done` dan `PLAN-1762261000` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 2.1 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/internal/infrastructure/ollama/errors.go`, `../../../../radovid-bot-api/internal/infrastructure/ollama/models.go`, `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize.go`, `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize_test.go`, `../../../../radovid-bot-api/internal/infrastructure/ollama/client.go`, `../../../../radovid-bot-api/internal/infrastructure/ollama/client_test.go`, `tickets/TICKET-1762261000-ollama-http-client.md`, `plans/PLAN-1762261000-ollama-http-client.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 14:04:00] - Basic /radovid help and /radovid start Command Skeleton (TICKET-1762174835)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762174835` dan rencana `PLAN-1762174835` untuk registrasi slash command Discord root `/radovid` dengan subcommands `help` dan `start`, factory builder embed UI terstandardisasi, routing subcommand pada delivery layer, dan penanganan respons galat ephemeral.
- **Perubahan:**
  - [Added] Factory Discord embed `internal/infrastructure/discord/embeds.go` (`NewInfoEmbed`, `NewSuccessEmbed`, `NewWarningEmbed`, `NewErrorEmbed`, `NewNeutralEmbed`) dengan palet warna terstandar dari `design-system.md` (`ColorInfo`, `ColorSuccess`, `ColorWarning`, `ColorError`, `ColorNeutral`) dan footer bawaan.
  - [Added] Unit test table-driven `internal/infrastructure/discord/embeds_test.go` memverifikasi struktur, pewarnaan, dan footer embed.
  - [Added] Definisi command Discord `internal/delivery/discord/command_def.go` (`GetApplicationCommands`) yang mendeklarasikan root command `/radovid` dengan subcommand `help` dan `start`.
  - [Added] Handler `internal/delivery/discord/radovid_help.go` (`HandleRadovidHelp`) yang merespons dengan embed informatif mengenai panduan bot dan daftar perintah.
  - [Added] Handler `internal/delivery/discord/radovid_start.go` (`HandleRadovidStart`) yang merespons dengan embed inisiasi sesi permainan 2 pemain.
  - [Added] Dispatcher subcommand dan helper pesan galat `internal/delivery/discord/handler.go` (`HandleRadovidCommand`, `RespondEphemeralError`) yang mengirim respons kesalahan ephemeral (`discordgo.MessageFlagsEphemeral`).
  - [Added] Unit test table-driven `internal/delivery/discord/command_test.go` memvalidasi definisi perintah, penanganan interaksi `nil`, opsi kosong, subcommand tak dikenal, dan error handling ephemeral.
  - [Changed] Memperbarui `cmd/bot/main.go` untuk mendaftarkan command `/radovid` ke router serta mendaftarkan slash command ke Discord API Gateway saat bot menerima event `Ready` (mendukung mode dev via `DISCORD_GUILD_ID` atau global jika kosong).
  - [Changed] Memperbarui status `TICKET-1762174835` menjadi `Done` dan `PLAN-1762174835` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 1.5 menjadi `[COMPLETED]` — Menandai selesainya seluruh Fase 1 (Foundation).
- **Path File:** `../../../../radovid-bot-api/internal/infrastructure/discord/embeds.go`, `../../../../radovid-bot-api/internal/infrastructure/discord/embeds_test.go`, `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`, `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`, `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go`, `../../../../radovid-bot-api/internal/delivery/discord/handler.go`, `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`, `../../../../radovid-bot-api/cmd/bot/main.go`, `tickets/TICKET-1762174835-basic-commands-skeleton.md`, `plans/PLAN-1762174835-basic-commands-skeleton.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 12:14:00] - Environment Configuration with Fail-Fast Validation (TICKET-1762174775)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762174775` dan rencana `PLAN-1762174775` untuk implementasi parser environment variable berbasis standard library dengan validasi fail-fast saat startup.
- **Perubahan:**
  - [Added] Modul konfigurasi `config/config.go` (`LoadConfig`) yang memvalidasi keberadaan `DISCORD_BOT_TOKEN` dan `SQLITE_DB_PATH` secara fail-fast, mendukung alias nama variabel (`DISCORD_TOKEN`, `DATABASE_PATH`, `DISCORD_APP_ID`), memparsing timeout dan port numerik dengan fallback aman, serta menetapkan default non-secret (`DefaultOllamaBaseURL`, `DefaultOllamaModel`, `DefaultOllamaTimeoutSeconds`, `DefaultAdminDashboardPort`, `DefaultLogLevel`).
  - [Added] Unit test table-driven `config/config_test.go` memvalidasi kegagalan fail-fast pada token atau path kosong/whitespace, keberhasilan pemuatan konfigurasi lengkap, toleransi alias, dan penanganan input numerik tidak valid.
  - [Changed] Memperbarui `cmd/bot/main.go` mengintegrasikan `config.LoadConfig()` di urutan pertama startup, dilanjutkan inisialisasi SQLite database dan migrasi otomatis sebelum gateway Discord dibuka.
  - [Changed] Memperbarui status `TICKET-1762174775` menjadi `Done` dan `PLAN-1762174775` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 1.4 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/config/config.go`, `../../../../radovid-bot-api/config/config_test.go`, `../../../../radovid-bot-api/cmd/bot/main.go`, `tickets/TICKET-1762174775-env-config-fail-fast.md`, `plans/PLAN-1762174775-env-config-fail-fast.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 11:30:00] - SQLite Database Initialization & Migration System (TICKET-1762174715)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762174715` dan rencana `PLAN-1762174715` untuk implementasi koneksi SQLite pure-Go (`modernc.org/sqlite`), konfigurasi PRAGMA WAL & pool single-writer, sistem runner migrasi DDL otomatis idempotent, dan skema awal tabel `users`.
- **Perubahan:**
  - [Added] Inisialisasi basis data `internal/infrastructure/database/sqlite.go` (`NewSQLiteDB`) dengan pembuatan direktori otomatis, pengaturan pool `SetMaxOpenConns(1)`, serta penegakan PRAGMA `journal_mode=WAL`, `busy_timeout=5000`, `foreign_keys=ON`, dan `synchronous=NORMAL`.
  - [Added] Engine runner migrasi otomatis `internal/infrastructure/database/migration.go` (`RunMigrations`) dengan tabel pelacak `schema_migrations`, sorting berkas SQL numerik, dan pembungkusan transaksi atomik per file.
  - [Added] Berkas migrasi awal `migrations/001_initial_schema.sql` untuk pembuatan tabel `users` (id UUID, discord_id unique, username, created_at epoch ms, updated_at epoch ms) dan indeks `idx_users_discord_id`.
  - [Added] Kontrak port repository domain `internal/domain/user/repository.go` (`Repository` interface).
  - [Added] Unit test komprehensif `internal/infrastructure/database/sqlite_test.go` memvalidasi inisialisasi PRAGMA, validasi input fail-fast, idempotensi eksekusi ganda, dan pembatalan transaksi (*rollback*) saat terjadi galat SQL.
  - [Added] Berkas template lingkungan `.env` dan `.env.example` di root modul `radovid-bot-api`.
  - [Changed] Memperbarui status `TICKET-1762174715` menjadi `Done` dan `PLAN-1762174715` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 1.3 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/internal/infrastructure/database/sqlite.go`, `../../../../radovid-bot-api/internal/infrastructure/database/migration.go`, `../../../../radovid-bot-api/internal/infrastructure/database/sqlite_test.go`, `../../../../radovid-bot-api/migrations/001_initial_schema.sql`, `../../../../radovid-bot-api/internal/domain/user/repository.go`, `../../../../radovid-bot-api/.env`, `../../../../radovid-bot-api/.env.example`, `tickets/TICKET-1762174715-sqlite-initialization-migration.md`, `plans/PLAN-1762174715-sqlite-initialization-migration.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 07:41:00] - Discord Bot Connection & Command Framework (TICKET-1762174655)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762174655` dan rencana `PLAN-1762174655` untuk implementasi koneksi Discord Gateway, adapter sesi DiscordGo, dan framework router slash command.
- **Perubahan:**
  - [Added] Adapter infrastruktur Discord `internal/infrastructure/discord/adapter.go` membungkus session DiscordGo dengan fail-fast validation token dan lifecycle method (`Open`, `Close`, `AddHandler`, `Session`).
  - [Added] Unit test adapter `internal/infrastructure/discord/adapter_test.go` memvalidasi error handling token kosong dan registrasi handler.
  - [Added] Interface signature handler `internal/delivery/discord/handler.go` (`CommandHandler`).
  - [Added] Dispatcher slash command `internal/delivery/discord/router.go` (`CommandRouter`) dengan context timeout default 5 detik (`CommandExecutionTimeout`), penanganan event `InteractionApplicationCommand`, dan fallback logging untuk unhandled command.
  - [Added] Unit test table-driven `internal/delivery/discord/router_test.go` memvalidasi dispatching perintah, timeout propagation, pengabaian event non-command, dan penanganan error tanpa panic.
  - [Changed] Memperbarui `cmd/bot/main.go` mengintegrasikan adapter Discord, router command, listener event Ready, dan penanganan sinyal graceful shutdown (`os.Interrupt`, `SIGTERM`, `SIGINT`).
  - [Changed] Memperbarui status `TICKET-1762174655` menjadi `Done` dan `PLAN-1762174655` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 1.2 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/internal/infrastructure/discord/adapter.go`, `../../../../radovid-bot-api/internal/infrastructure/discord/adapter_test.go`, `../../../../radovid-bot-api/internal/delivery/discord/handler.go`, `../../../../radovid-bot-api/internal/delivery/discord/router.go`, `../../../../radovid-bot-api/internal/delivery/discord/router_test.go`, `../../../../radovid-bot-api/cmd/bot/main.go`, `tickets/TICKET-1762174655-discord-bot-connection.md`, `plans/PLAN-1762174655-discord-bot-connection.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 07:19:00] - Setup Go Project Structure (TICKET-1762174595)
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Eksekusi tiket `TICKET-1762174595` dan rencana `PLAN-1762174595` untuk inisialisasi arsitektur Clean Architecture serta instalasi dependensi inti pada modul Go.
- **Perubahan:**
  - [Added] Inisialisasi struktur folder Clean Architecture (`cmd/bot`, `internal/domain`, `internal/usecase`, `internal/infrastructure`, `internal/delivery`, `migrations`, `web/templates`, `config`).
  - [Added] Penambahan dependensi pure-Go `github.com/bwmarrin/discordgo`, `modernc.org/sqlite`, dan `github.com/google/uuid` ke `go.mod` dan `go.sum` tanpa ketergantungan toolchain CGO (`CGO_ENABLED=0`).
  - [Added] Entitas domain inti `internal/domain/user/entity.go`, `internal/domain/session/entity.go`, dan `internal/domain/game/entity.go` beserta unit test table-driven `*_test.go` (100% PASS).
  - [Added] Entrypoint minimal `cmd/bot/main.go` yang sukses dikompilasi dengan `CGO_ENABLED=0` dan dieksekusi.
  - [Added] Konfigurasi `.gitignore` untuk mengabaikan file build, database lokal, dan environment file di codebase `radovid-bot-api`.
  - [Changed] Memperbarui status `TICKET-1762174595` menjadi `Done` dan `PLAN-1762174595` menjadi `Completed`.
  - [Changed] Memperbarui roadmap pada `docs/development-planning.md` item 1.1 menjadi `[COMPLETED]`.
- **Path File:** `../../../../radovid-bot-api/go.mod`, `../../../../radovid-bot-api/go.sum`, `../../../../radovid-bot-api/.gitignore`, `../../../../radovid-bot-api/cmd/bot/main.go`, `../../../../radovid-bot-api/internal/domain/user/entity.go`, `../../../../radovid-bot-api/internal/domain/user/entity_test.go`, `../../../../radovid-bot-api/internal/domain/session/entity.go`, `../../../../radovid-bot-api/internal/domain/session/entity_test.go`, `../../../../radovid-bot-api/internal/domain/game/entity.go`, `../../../../radovid-bot-api/internal/domain/game/entity_test.go`, `tickets/TICKET-1762174595-setup-go-project-structure.md`, `plans/PLAN-1762174595-setup-go-project-structure.md`, `docs/development-planning.md`, `CHANGELOG.md`

---

### [2026-11-04 02:35:00] - Development Planning: Generate Tickets and Plans for All 4 Phases
> **Trigger:** Autonomous Planning | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Pembuatan 32 pasang berkas Tiket (`tickets/TICKET-[EPOCH]-[judul].md`) dan Rencana Teknis (`plans/PLAN-[EPOCH]-[judul].md`) untuk seluruh 4 fase pada roadmap pengembangan Radovid Bot.

**Perubahan:**
- [Added] Menghasilkan 5 pasang berkas tiket & rencana untuk Fase 1 (Foundation): Setup Go structure, Discord connection, SQLite & migration, Env config fail-fast, dan Basic slash commands skeleton.
- [Added] Menghasilkan 7 pasang berkas tiket & rencana untuk Fase 2 (Core Game Engine): Ollama HTTP client, Game session state machine, AI GM prompts, Would You Rather, Taboo, 20 Questions, dan Discord embed templates.
- [Added] Menghasilkan 8 pasang berkas tiket & rencana untuk Fase 3 (Speaking & Writing): AI Feedback system, Job Interview, Roleplay, One Minute Challenge, Story Relay, Daily 5-Sentence Diary, Five Word Challenge, dan Difficulty level system.
- [Added] Menghasilkan 12 pasang berkas tiket & rencana untuk Fase 4 (Gamification & Dashboard): XP & Level system, Streak tracking, Achievements, /radovid progress command, HTMX Admin Dashboard, Secret Mission, Grammar Boss Fight, Sentence Transformation Race, Movie Review, News Reporter, Design Your Dream House, dan Daily Challenge notifications.
- [Changed] Memperbarui status seluruh 32 item di `docs/development-planning.md` menjadi `[CREATED]`.

**Files Affected:**
`nodes/radovid-bot/tickets/*` (32 berkas tiket), `nodes/radovid-bot/plans/*` (32 berkas rencana teknis), `nodes/radovid-bot/docs/development-planning.md`, `nodes/radovid-bot/CHANGELOG.md`

**Next Action:** Meminta persetujuan developer untuk memulai eksekusi Fase 1 — `TICKET-1762174595` / `PLAN-1762174595`: Setup Go Project Structure.

---

### [2026-11-03 23:48:00] - Rename: Node & System Name Alignment to Radovid Bot
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Penyesuaian nama sistem dan node menjadi `radovid-bot` sesuai instruksi developer.

**Perubahan:**
- [Changed] Mengubah nama node dari `nodes/veylo-bot/` menjadi `nodes/radovid-bot/`
- [Changed] Memperbarui seluruh referensi dokumen global (`prd.md`, `design-system.md`, diagram-diagram) dan dokumen node (`system-design.md`, `development-planning.md`, `project-context.md`, `CHANGELOG.md`) menggunakan nama `Radovid Bot` dan slash command `/radovid`

**Files Affected:**
`nodes/radovid-bot/`, `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`, `nodes/radovid-bot/docs/system-design.md`, `nodes/radovid-bot/docs/development-planning.md`, `nodes/radovid-bot/docs/diagrams/*`

---

### [2026-11-03 23:16:00] - Initialization: Radovid Bot Node Inception
> **Trigger:** Startup Prompt (Single-Project Environment) | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Inisialisasi perdana node `radovid-bot` dalam ekosistem AI Orchestrator untuk proyek Radovid Bot — AI-powered Discord English learning companion.

**Perubahan:**
- [Added] Duplikasi `nodes/_template/` menjadi `nodes/radovid-bot/` (Mode Operasional: MODE 1 — Autonomous Planning)
- [Added] Penulisan `nodes/radovid-bot/main.md` dengan MODE 1 terpilih di baris pertama
- [Added] Pemindaian codebase greenfield (`radovid-bot-api/`, hanya `go.mod`)
- [Added] Penulisan `nodes/radovid-bot/guidelines/project-context.md` (hasil pindai + konvensi lokal: Clean Architecture, interface-first, UUID PK, epoch ms timestamps, slash commands only, Ollama timeout wajib, input sanitization, table-driven tests)
- [Added] Penulisan `nodes/radovid-bot/docs/system-design.md` (arsitektur Clean Architecture, ERD, API contracts)
- [Added] Penulisan `nodes/radovid-bot/docs/development-planning.md` (4 fase, 30 item backlog, pasangan tiket & rencana dengan epoch timestamps)
- [Added] Pembuatan diagram PlantUML: `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`, `nodes/radovid-bot/docs/diagrams/erd.puml`, `nodes/radovid-bot/docs/diagrams/flowchart.puml`, `nodes/radovid-bot/docs/diagrams/state-diagram.puml`
- [Added] Pembaruan `global-docs/prd.md` dengan spesifikasi PRD Radovid Bot
- [Added] Pembaruan `global-docs/design-system.md` dengan design system Radovid Bot
- [Added] Penulisan `nodes/radovid-bot/CHANGELOG.md` (file ini)

**Files Affected:**
`nodes/radovid-bot/main.md`, `nodes/radovid-bot/guidelines/project-context.md`, `nodes/radovid-bot/docs/system-design.md`, `nodes/radovid-bot/docs/development-planning.md`, `nodes/radovid-bot/docs/diagrams/erd.puml`, `nodes/radovid-bot/docs/diagrams/flowchart.puml`, `nodes/radovid-bot/docs/diagrams/state-diagram.puml`, `nodes/radovid-bot/CHANGELOG.md`, `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`

**Next Action:** Mulai eksekusi Fase 1 — `TICKET-1762174595` / `PLAN-1762174595`: Setup Go Project Structure.
