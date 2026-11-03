# Radovid Bot — System Design

> **Radovid Bot** is an AI-powered Discord English learning companion. Two players per session engage in 15 game activities managed by an AI Game Master running on a local Ollama (Gemma 3 4b) instance.

---

## Table of Contents

1. [Software Architecture](#1-software-architecture)
2. [Directory Structure](#2-directory-structure)
3. [Tech Stack](#3-tech-stack)
4. [Logika & Alur Komponen](#4-logika--alur-komponen)
5. [Data Modeling](#5-data-modeling)
6. [API Contracts](#6-api-contracts)
7. [Deployment](#7-deployment)
8. [Third-Party Services](#8-third-party-services)
9. [Environment Variables](#9-environment-variables)
10. [Security & Error Handling](#10-security--error-handling)

---

## 1. Software Architecture

Radovid Bot menggunakan **Clean Architecture** dengan pola **Hexagonal (Ports & Adapters)**, memisahkan business logic dari infrastruktur eksternal secara tegas.

```
┌─────────────────────────────────────────────────────────┐
│                    Delivery Layer                        │
│   ┌─────────────────────┐  ┌──────────────────────┐    │
│   │  Discord Slash Cmds │  │   HTMX HTTP Dashboard│    │
│   └────────┬────────────┘  └──────────┬───────────┘    │
└────────────┼──────────────────────────┼────────────────┘
             │                          │
┌────────────▼──────────────────────────▼────────────────┐
│                    Use Case Layer                        │
│   ┌──────────┐  ┌──────────────┐  ┌────────────────┐  │
│   │  Game UC │  │  Feedback UC │  │  Progress UC   │  │
│   └──────────┘  └──────────────┘  └────────────────┘  │
└────────────────────────┬───────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────┐
│                    Domain Layer                          │
│   ┌──────────┐  ┌──────────────┐  ┌────────────────┐  │
│   │   Game   │  │     User     │  │    Session     │  │
│   │ Entities │  │   Entities   │  │   Entities     │  │
│   └──────────┘  └──────────────┘  └────────────────┘  │
└────────────────────────┬───────────────────────────────┘
                         │
┌────────────────────────▼───────────────────────────────┐
│                 Infrastructure Layer                     │
│   ┌──────────┐  ┌──────────────┐  ┌────────────────┐  │
│   │  SQLite  │  │Ollama Client │  │  DiscordGo     │  │
│   │   Repos  │  │ (HTTP/REST)  │  │  Event Adapter │  │
│   └──────────┘  └──────────────┘  └────────────────┘  │
└────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Tanggung Jawab |
|---|---|
| **Delivery** | Menerima input dari Discord / HTTP, menerjemahkan ke Use Case calls, memformat respons |
| **Use Case** | Orkestrasi business logic, tidak bergantung pada framework apapun |
| **Domain** | Pure Go structs + business rules, zero external dependencies |
| **Infrastructure** | Implementasi konkret dari repository interfaces, Ollama HTTP client, DiscordGo adapter |

---

## 2. Directory Structure

Proyeksi struktur direktori untuk codebase **greenfield** (dimulai hanya dengan `go.mod`):

```
radovid-bot-api/
├── cmd/
│   └── bot/
│       └── main.go                  # Entrypoint: init config, DI, start bot
├── internal/
│   ├── domain/
│   │   ├── game/
│   │   │   ├── entity.go            # Game, GameType, Difficulty structs
│   │   │   ├── repository.go        # GameRepository interface (port)
│   │   │   └── errors.go            # Domain-level errors
│   │   ├── user/
│   │   │   ├── entity.go            # User, UserProgress structs
│   │   │   ├── repository.go        # UserRepository interface
│   │   │   └── errors.go
│   │   └── session/
│   │       ├── entity.go            # GameSession, SessionParticipant, GameRound structs
│   │       ├── repository.go        # SessionRepository interface
│   │       └── errors.go
│   ├── usecase/
│   │   ├── game/
│   │   │   ├── start_session.go     # Use case: /radovid start
│   │   │   ├── join_session.go      # Use case: /radovid join
│   │   │   ├── end_session.go       # Use case: /radovid end
│   │   │   └── game_master.go       # AI Game Master orchestration
│   │   ├── feedback/
│   │   │   ├── ai_feedback.go       # Request AI feedback via Ollama
│   │   │   ├── diary.go             # Daily diary write + feedback
│   │   │   └── correction.go        # English correction logic
│   │   └── progress/
│   │       ├── get_progress.go      # Fetch user progress
│   │       ├── award_xp.go          # XP & leveling logic
│   │       └── streak.go            # Daily streak management
│   ├── infrastructure/
│   │   ├── sqlite/
│   │   │   ├── db.go                # SQLite connection + WAL setup
│   │   │   ├── migrations.go        # Run migration files
│   │   │   ├── user_repository.go   # UserRepository implementation
│   │   │   ├── session_repository.go
│   │   │   └── game_repository.go
│   │   ├── ollama/
│   │   │   ├── client.go            # HTTP client for Ollama REST API
│   │   │   ├── prompts.go           # Prompt templates per game type
│   │   │   └── models.go            # Request/response structs
│   │   └── discord/
│   │       ├── adapter.go           # DiscordGo session wrapper
│   │       └── embeds.go            # Discord embed builders
│   └── delivery/
│       ├── discord/
│       │   ├── handler.go           # Main slash command dispatcher
│       │   ├── radovid_start.go       # /radovid start handler
│       │   ├── radovid_join.go        # /radovid join handler
│       │   ├── radovid_end.go         # /radovid end handler
│       │   ├── radovid_progress.go    # /radovid progress handler
│       │   ├── radovid_feedback.go    # /radovid feedback handler
│       │   ├── radovid_diary.go       # /radovid diary handler
│       │   └── radovid_help.go        # /radovid help handler
│       └── http/
│           ├── admin_handler.go     # HTMX dashboard handlers
│           ├── middleware.go        # Basic auth middleware
│           └── router.go            # HTTP route registration
├── web/
│   ├── templates/
│   │   ├── base.html               # Base layout template
│   │   ├── dashboard.html          # /admin index
│   │   ├── sessions.html           # /admin/sessions
│   │   ├── users.html              # /admin/users
│   │   └── stats.html              # /admin/stats
│   └── static/
│       ├── style.css
│       └── htmx.min.js
├── migrations/
│   ├── 001_create_users.sql
│   ├── 002_create_game_sessions.sql
│   ├── 003_create_session_participants.sql
│   ├── 004_create_game_rounds.sql
│   ├── 005_create_user_progress.sql
│   ├── 006_create_achievements.sql
│   ├── 007_create_daily_diaries.sql
│   └── 008_create_feedback_logs.sql
├── config/
│   └── config.go                   # Config struct + load + validation
├── go.mod
└── go.sum
```

---

## 3. Tech Stack

| Komponen | Teknologi | Keterangan |
|---|---|---|
| **Language** | Go 1.26.7 | Single binary, cross-compile ke Linux |
| **Discord** | `github.com/bwmarrin/discordgo` | Slash commands, interactions, embeds |
| **Database** | SQLite via `modernc.org/sqlite` | Pure Go (tanpa CGO), WAL mode |
| **AI / LLM** | Ollama HTTP API (`localhost:11434`) | Model: `gemma3:4b`, dijalankan lokal |
| **Dashboard** | HTMX + `net/http` (stdlib) | Internal admin UI, tidak butuh JS framework |
| **Linter** | `golangci-lint` | Konfigurasi via `.golangci.yml` |
| **Testing** | `go test` (built-in) | Unit tests per layer; integration test opsional |
| **Config** | `os.Getenv` + fail-fast validation | Tidak ada dependency eksternal |
| **Process Mgmt** | `systemd` | Service unit file untuk Linux server |

### Rationale Pilihan

- **`modernc.org/sqlite`** dipilih atas `mattn/go-sqlite3` untuk menghindari kebutuhan CGO, sehingga binary dapat di-cross-compile tanpa C toolchain.
- **HTMX** dipilih atas React/Vue karena dashboard bersifat internal dan read-heavy, sehingga cukup dengan server-rendered HTML + partial updates.
- **Ollama lokal** memastikan tidak ada latensi jaringan eksternal dan tidak ada biaya API cloud.

---

## 4. Logika & Alur Komponen

Lihat diagram detail di: [diagrams/flowchart.puml](diagrams/flowchart.puml)

### Ringkasan Alur Command Processing

```
User Discord Input
       │
       ▼
[Discord Gateway Event]
       │ DiscordGo receives InteractionCreate event
       ▼
[Delivery: discord/handler.go]
       │ Route ke handler berdasarkan command name + subcommand
       ▼
[Use Case Layer]
       │ Validasi input, orchestrasi business logic
       ├──► [Domain] Buat/update entities, terapkan business rules
       ├──► [Infrastructure: SQLite] Simpan/baca data
       └──► [Infrastructure: Ollama] Generate prompt/feedback (async dengan timeout)
       │
       ▼
[Delivery: format respons]
       │ Bangun Discord embed atau plain message
       ▼
[Discord API: InteractionResponse]
```

### Alur AI Game Master

1. **Prompt Generation**: Use case `game_master.go` membangun prompt berdasarkan `game_type`, `difficulty`, dan riwayat ronde sebelumnya.
2. **Ollama Call**: HTTP POST ke `localhost:11434/api/generate` dengan context timeout (default 30 detik).
3. **Response Parsing**: Response di-stream dari Ollama, di-aggregate menjadi teks lengkap.
4. **Feedback**: Setelah user merespons, teks dikirim kembali ke Ollama untuk koreksi dan feedback bahasa Inggris.

---

## 5. Data Modeling

Lihat ERD lengkap di: [diagrams/erd.puml](diagrams/erd.puml)

Semua kolom datetime menggunakan **Epoch Milliseconds (BIGINT)** untuk kemudahan perbandingan dan portabilitas.

### Skema Entitas

#### `users`
```sql
CREATE TABLE users (
    id          TEXT PRIMARY KEY,           -- UUID v4
    discord_id  TEXT UNIQUE NOT NULL,       -- Discord Snowflake ID
    username    TEXT NOT NULL,
    created_at  BIGINT NOT NULL,            -- Epoch ms
    updated_at  BIGINT NOT NULL
);
```

#### `game_sessions`
```sql
CREATE TABLE game_sessions (
    id          TEXT PRIMARY KEY,
    game_type   TEXT NOT NULL,              -- e.g. 'word_chain', 'story_builder'
    status      TEXT NOT NULL DEFAULT 'pending', -- pending|active|completed|cancelled
    channel_id  TEXT NOT NULL,
    guild_id    TEXT NOT NULL,
    created_at  BIGINT NOT NULL,
    ended_at    BIGINT                      -- NULL jika belum selesai
);
```

#### `session_participants`
```sql
CREATE TABLE session_participants (
    session_id  TEXT NOT NULL REFERENCES game_sessions(id),
    user_id     TEXT NOT NULL REFERENCES users(id),
    role        TEXT NOT NULL DEFAULT 'player', -- player|host
    joined_at   BIGINT NOT NULL,
    PRIMARY KEY (session_id, user_id)
);
```

#### `game_rounds`
```sql
CREATE TABLE game_rounds (
    id           TEXT PRIMARY KEY,
    session_id   TEXT NOT NULL REFERENCES game_sessions(id),
    round_number INT NOT NULL,
    prompt       TEXT NOT NULL,             -- Prompt yang digenerate Ollama
    started_at   BIGINT NOT NULL,
    ended_at     BIGINT
);
```

#### `user_progress`
```sql
CREATE TABLE user_progress (
    user_id         TEXT PRIMARY KEY REFERENCES users(id),
    total_xp        INT NOT NULL DEFAULT 0,
    current_level   INT NOT NULL DEFAULT 1,
    streak_days     INT NOT NULL DEFAULT 0,
    last_activity_at BIGINT,
    updated_at      BIGINT NOT NULL
);
```

#### `achievements`
```sql
CREATE TABLE achievements (
    id               TEXT PRIMARY KEY,
    user_id          TEXT NOT NULL REFERENCES users(id),
    achievement_type TEXT NOT NULL,         -- e.g. 'first_game', 'streak_7'
    earned_at        BIGINT NOT NULL
);
```

#### `daily_diaries`
```sql
CREATE TABLE daily_diaries (
    id          TEXT PRIMARY KEY,
    user_id     TEXT NOT NULL REFERENCES users(id),
    content     TEXT NOT NULL,              -- Tulisan user
    ai_feedback TEXT,                       -- Feedback dari Ollama
    created_at  BIGINT NOT NULL
);
```

#### `feedback_logs`
```sql
CREATE TABLE feedback_logs (
    id             TEXT PRIMARY KEY,
    session_id     TEXT REFERENCES game_sessions(id),
    user_id        TEXT NOT NULL REFERENCES users(id),
    original_text  TEXT NOT NULL,
    corrected_text TEXT,
    notes          TEXT,                    -- Penjelasan koreksi dari Ollama
    created_at     BIGINT NOT NULL
);
```

---

## 6. API Contracts

### Discord Slash Commands

Semua commands berada di bawah grup `/radovid`.

| Command | Parameter | Deskripsi |
|---|---|---|
| `/radovid start` | `game_type` (required), `difficulty` (optional: easy/medium/hard) | Membuat game session baru di channel saat ini |
| `/radovid join` | — | Bergabung ke session aktif di channel |
| `/radovid end` | — | Mengakhiri session aktif (hanya host atau admin) |
| `/radovid progress` | — | Menampilkan progress belajar user (XP, level, streak) |
| `/radovid feedback` | — | Meminta AI feedback dari session terakhir |
| `/radovid diary` | `content` (required, modal input) | Menulis daily diary + mendapat feedback |
| `/radovid help` | `game_type` (optional) | Menampilkan panduan game |

### Game Types

| Game Type Slug | Deskripsi |
|---|---|
| `word_chain` | Sambung kata berbahasa Inggris |
| `story_builder` | Bangun cerita bersama kalimat per kalimat |
| `debate` | Debat topik random, AI jadi juri |
| `vocabulary_quiz` | Tebak arti/sinonim kosakata |
| `roleplay` | Roleplay scenario dalam bahasa Inggris |
| `fill_blank` | Lengkapi kalimat yang hilang |
| `pronunciation_check` | Submit teks, AI cek grammar & naturalness |
| *(+8 game types lainnya)* | — |

### HTMX Dashboard (HTTP)

| Endpoint | Method | Deskripsi |
|---|---|---|
| `/admin` | GET | Dashboard utama |
| `/admin/sessions` | GET | Daftar semua game sessions + filter status |
| `/admin/sessions/{id}` | GET | Detail session (HTMX partial) |
| `/admin/users` | GET | Daftar users + progress |
| `/admin/stats` | GET | Statistik agregat (total sessions, XP, dll) |

Semua endpoint dashboard dilindungi oleh **HTTP Basic Auth** (lihat [#10-security--error-handling](#10-security--error-handling)).

---

## 7. Deployment

### Target Environment

- **OS**: Linux (Ubuntu 22.04+ atau Debian Bullseye+)
- **Server**: VPS atau bare metal
- **Binary**: Single statically-linked Go binary
- **Database**: SQLite file di `./data/radovid.db` (atau path dari env var)
- **AI**: Ollama berjalan sebagai service terpisah di port `11434`

### Proses Build

```bash
# Build untuk Linux amd64
GOOS=linux GOARCH=amd64 go build -o radovid-bot ./cmd/bot/

# Upload ke server
scp radovid-bot user@server:/opt/radovid/

# Upload migrations
scp -r migrations/ user@server:/opt/radovid/
```

### Systemd Service Unit

```ini
# /etc/systemd/system/radovid.service
[Unit]
Description=Radovid Discord Bot
After=network.target ollama.service
Requires=ollama.service

[Service]
Type=simple
User=radovid
WorkingDirectory=/opt/radovid
EnvironmentFile=/opt/radovid/.env
ExecStart=/opt/radovid/radovid-bot
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Startup Sequence (di `main.go`)

1. Load & validate environment variables (fail-fast jika wajib tidak ada)
2. Inisialisasi SQLite connection + jalankan migrations
3. Inisialisasi Ollama HTTP client + health check ping
4. Inisialisasi DiscordGo session + register slash commands
5. Start HTTP server untuk dashboard (goroutine)
6. Block pada Discord gateway connection

---

## 8. Third-Party Services

| Service | Jenis | Penggunaan | Auth |
|---|---|---|---|
| **Discord API** | Cloud (external) | Bot gateway, slash commands, embeds | Bot Token (Bearer) |
| **Ollama** | Lokal (self-hosted) | Generate game prompts, feedback, koreksi | Tidak ada (localhost only) |

> **Tidak ada ketergantungan pada cloud AI API** (OpenAI, Gemini API, dll). Semua inferensi berjalan lokal via Ollama.

---

## 9. Environment Variables

| Variable | Wajib | Default | Deskripsi |
|---|---|---|---|
| `DISCORD_BOT_TOKEN` | ✅ | — | Bot token dari Discord Developer Portal |
| `DISCORD_APPLICATION_ID` | ✅ | — | Application ID untuk register slash commands |
| `DISCORD_GUILD_ID` | ❌ | — | Jika diset, slash commands hanya didaftarkan ke guild ini (dev mode) |
| `SQLITE_DB_PATH` | ✅ | `./data/radovid.db` | Path ke file SQLite database |
| `OLLAMA_BASE_URL` | ✅ | `http://localhost:11434` | Base URL Ollama server |
| `OLLAMA_MODEL` | ✅ | `gemma3:4b` | Nama model Ollama yang digunakan |
| `OLLAMA_TIMEOUT_SECONDS` | ❌ | `30` | Timeout dalam detik untuk setiap request ke Ollama |
| `ADMIN_DASHBOARD_PORT` | ❌ | `8080` | Port untuk HTMX admin dashboard |
| `ADMIN_DASHBOARD_SECRET` | ✅ | — | Secret untuk HTTP Basic Auth dashboard (format: `user:password`) |
| `LOG_LEVEL` | ❌ | `info` | Level logging: `debug`, `info`, `warn`, `error` |

### Contoh `.env`

```env
DISCORD_BOT_TOKEN=your_token_here
DISCORD_APPLICATION_ID=123456789012345678
DISCORD_GUILD_ID=987654321098765432
SQLITE_DB_PATH=./data/radovid.db
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=gemma3:4b
OLLAMA_TIMEOUT_SECONDS=30
ADMIN_DASHBOARD_PORT=8080
ADMIN_DASHBOARD_SECRET=admin:supersecretpassword
LOG_LEVEL=info
```

---

## 10. Security & Error Handling

### Security Measures

| Area | Mekanisme |
|---|---|
| **Discord Command Auth** | Verifikasi `guild_id` dan `channel_id` di setiap command handler |
| **Dashboard Auth** | HTTP Basic Auth middleware; secret dari env var; HTTPS dianjurkan di depan reverse proxy |
| **Input Sanitization** | Semua input user di-trim dan dibatasi panjang maksimum sebelum dikirim ke Ollama |
| **Ollama Isolation** | Ollama hanya listen di `localhost`; tidak diekspos ke publik |
| **SQLite Concurrency** | WAL (Write-Ahead Logging) mode diaktifkan saat inisialisasi |
| **Session Ownership** | Hanya session host (atau admin) yang bisa `/radovid end` |
| **Rate Limiting** | Per-user cooldown menggunakan in-memory map + mutex (minimal implementasi MVP) |

### Error Handling Strategy

```
Error Level    │ Strategi
───────────────┼──────────────────────────────────────────────────
Startup Error  │ Fail-fast: log.Fatal + exit(1) jika env vars wajib tidak ada
               │ atau SQLite/Ollama tidak bisa diinisialisasi
               │
Domain Error   │ Return typed error (e.g. ErrSessionNotFound, ErrGameAlreadyActive)
               │ Delivery layer map ke pesan Discord yang user-friendly
               │
Ollama Timeout │ Context timeout per request; jika timeout, kirim fallback message
               │ ke user di Discord ("AI sedang sibuk, coba lagi sebentar")
               │
SQLite Error   │ Log error + return 500-equivalent; jangan ekspos detail ke user
               │
Discord API    │ Retry sekali dengan backoff; log jika tetap gagal
Error          │
```

### Fail-Fast Validation (startup)

```go
// config/config.go
func Load() (*Config, error) {
    required := []string{
        "DISCORD_BOT_TOKEN",
        "DISCORD_APPLICATION_ID",
        "SQLITE_DB_PATH",
        "OLLAMA_BASE_URL",
        "OLLAMA_MODEL",
        "ADMIN_DASHBOARD_SECRET",
    }
    for _, key := range required {
        if os.Getenv(key) == "" {
            return nil, fmt.Errorf("required environment variable %s is not set", key)
        }
    }
    // ... load rest of config
}
```

---

*Dokumen ini adalah living document. Perbarui saat ada perubahan arsitektur signifikan.*
