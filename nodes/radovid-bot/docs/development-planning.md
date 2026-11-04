# Development Planning: Radovid Bot

> **[DOKUMEN AKTIF]** Peta jalan dan backlog pasangan tiket & rencana implementasi spesifik node `radovid-bot`. Diperbarui setiap kali ada penugasan baru atau perubahan prioritas.

---

## Ringkasan Proyek

**Nama:** Radovid Bot — AI-powered Discord English Learning Companion
**Module:** `github.com/rakafebriansy/radovid-bot`
**Stack:** Golang 1.26.7, DiscordGo, SQLite, Ollama Gemma 3 4b (lokal), HTMX Dashboard Admin
**Mode Operasional:** MODE 1 — Autonomous Planning
**Status Codebase:** Greenfield (hanya `go.mod`)

---

## Roadmap Empat Fase

```
Fase 1: Foundation          → Fase 2: Core Game Engine
(Bot + DB + Config)           (AI GM + Session + 3 Games)
         ↓                              ↓
Fase 4: Gamification       ← Fase 3: Speaking & Writing
(XP + Dashboard + Full Games)   (AI Feedback + 6 More Games)
```

---

## Fase 1: Foundation (Fondasi Bot & Infrastruktur)

> **Tujuan:** Setup proyek Go, koneksi Discord, database schema, konfigurasi environment — fondasi yang solid sebelum logika game apapun ditulis.

**Target Selesai:** Fase 1 harus menghasilkan binary yang bisa dijalankan, terhubung ke Discord, dan merespons slash command `/radovid help` dengan embed yang benar.

### Backlog Fase 1

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 1.1 | Setup Go Project Structure | `1762174595` | `[COMPLETED]` | `TICKET-1762174595` / `PLAN-1762174595` |
| 1.2 | Discord Bot Connection & Command Framework | `1762174655` | `[COMPLETED]` | `TICKET-1762174655` / `PLAN-1762174655` |
| 1.3 | SQLite Database Initialization & Migration System | `1762174715` | `[COMPLETED]` | `TICKET-1762174715` / `PLAN-1762174715` |
| 1.4 | Environment Configuration with Fail-Fast Validation | `1762174775` | `[COMPLETED]` | `TICKET-1762174775` / `PLAN-1762174775` |
| 1.5 | Basic `/radovid help` dan `/radovid start` Command Skeleton | `1762174835` | `[COMPLETED]` | `TICKET-1762174835` / `PLAN-1762174835` |

### Detail Item Fase 1

#### 1.1 Setup Go Project Structure
- **Deskripsi:** Scaffold struktur direktori Clean Architecture, install dependensi utama (DiscordGo, SQLite driver), verifikasi `go build` berhasil.
- **Struktur Target:**
  ```
  radovid-bot-api/
  ├── cmd/bot/main.go
  ├── internal/
  │   ├── delivery/discord/
  │   ├── usecase/
  │   ├── domain/
  │   └── infrastructure/
  │       ├── database/
  │       └── ollama/
  ├── migrations/
  ├── web/templates/
  ├── go.mod
  └── go.sum
  ```
- **Acceptance Criteria:**
  - [ ] Struktur direktori Clean Architecture tersedia
  - [ ] `go.mod` memiliki dependensi DiscordGo dan SQLite driver
  - [ ] `go build ./...` sukses tanpa error
  - [ ] Tidak ada CGO dependency (modernc.org/sqlite)

#### 1.2 Discord Bot Connection & Command Framework
- **Deskripsi:** Inisialisasi koneksi bot ke Discord via token, setup handler untuk slash command registration on ready, kerangka router command.
- **Acceptance Criteria:**
  - [ ] Bot berhasil login dan tampil online di Discord
  - [ ] Framework registrasi slash command tersedia dan modular
  - [ ] Handler untuk `InteractionCreate` event terdaftar
  - [ ] Graceful shutdown (SIGTERM/SIGINT) via `os.Signal`
  - [ ] Context propagation dari event Discord ke handler

#### 1.3 SQLite Database Initialization & Migration System
- **Deskripsi:** Setup koneksi SQLite, sistem migration numerik auto-run saat startup, schema awal untuk entitas User.
- **Acceptance Criteria:**
  - [ ] Koneksi SQLite berhasil (file-based, path dari env var)
  - [ ] Migration runner membaca file `migrations/*.sql` secara urutan numerik
  - [ ] Tabel `schema_migrations` mencatat migration yang telah dijalankan
  - [ ] Migration idempotent (tidak error jika dijalankan ulang)
  - [ ] `001_initial_schema.sql` membuat tabel `users` dengan UUID PK dan epoch timestamps

#### 1.4 Environment Configuration with Fail-Fast Validation
- **Deskripsi:** Baca semua env var wajib saat startup, fail-fast jika ada yang kosong, tidak ada hardcoded defaults untuk secrets.
- **Env Vars Wajib:**
  - `DISCORD_TOKEN` — token bot Discord
  - `DISCORD_APP_ID` — application ID Discord
  - `DISCORD_GUILD_ID` — guild ID (dev mode, kosong untuk global)
  - `DATABASE_PATH` — path file SQLite
  - `OLLAMA_BASE_URL` — base URL Ollama (default: `http://localhost:11434`)
  - `OLLAMA_MODEL` — nama model (default: `gemma3:4b`)
  - `OLLAMA_TIMEOUT_SECONDS` — timeout request Ollama (default: `30`)
- **Acceptance Criteria:**
  - [ ] Semua env var dimuat dari OS environment (bukan file .env di runtime)
  - [ ] Binary langsung exit dengan pesan jelas jika `DISCORD_TOKEN` atau `DATABASE_PATH` kosong
  - [ ] Nilai default hanya untuk non-secret vars (Ollama URL, model, timeout)
  - [ ] Struct `Config` tersedia dan di-pass via dependency injection

#### 1.5 Basic `/radovid help` dan `/radovid start` Command Skeleton
- **Deskripsi:** Implementasi dua slash command pertama sebagai validasi end-to-end pipeline: command registration → handler → Discord response.
- **Acceptance Criteria:**
  - [ ] `/radovid help` merespons dengan Discord Embed berisi daftar command yang tersedia
  - [ ] `/radovid start` merespons dengan Discord Embed placeholder (belum ada logika game)
  - [ ] Semua error response dikirim sebagai ephemeral
  - [ ] Command terdaftar di guild saat bot startup (dev mode)

---

## Fase 2: Core Game Engine (Mesin Game Inti)

> **Tujuan:** Implementasi AI Game Master, session management state machine, dan 3 game aktivitas pertama sebagai validasi sistem end-to-end.

**Prasyarat:** Fase 1 selesai 100% (binary berjalan, DB tersedia, Discord terhubung).

### Backlog Fase 2

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 2.1 | Ollama HTTP Client Wrapper | `1762261000` | `[COMPLETED]` | `TICKET-1762261000` / `PLAN-1762261000` |
| 2.2 | Game Session State Machine | `1762261060` | `[CREATED]` | `TICKET-1762261060` / `PLAN-1762261060` |
| 2.3 | AI Game Master Prompt System | `1762261120` | `[CREATED]` | `TICKET-1762261120` / `PLAN-1762261120` |
| 2.4 | Game 1: Would You Rather? | `1762261180` | `[CREATED]` | `TICKET-1762261180` / `PLAN-1762261180` |
| 2.5 | Game 2: Taboo | `1762261240` | `[CREATED]` | `TICKET-1762261240` / `PLAN-1762261240` |
| 2.6 | Game 3: 20 Questions | `1762261300` | `[CREATED]` | `TICKET-1762261300` / `PLAN-1762261300` |
| 2.7 | Discord Embed Templates (Game UI) | `1762261360` | `[CREATED]` | `TICKET-1762261360` / `PLAN-1762261360` |

### Detail Item Fase 2

#### 2.1 Ollama HTTP Client Wrapper
- **Deskripsi:** HTTP client untuk Ollama Generate API dengan timeout wajib, retry logic (max 2 retries), dan sanitasi input sebelum dikirim.
- **Acceptance Criteria:**
  - [ ] Client menggunakan `context.Context` dengan timeout dari `OLLAMA_TIMEOUT_SECONDS`
  - [ ] Input sanitization: hapus Discord mention, strip markdown manipulatif, max 2000 karakter
  - [ ] Retry dengan exponential backoff untuk error 5xx
  - [ ] Return typed error untuk timeout vs connection refused vs model error
  - [ ] Unit test table-driven untuk sanitization dan error handling

#### 2.2 Game Session State Machine
- **Deskripsi:** State machine untuk siklus hidup sesi game: PENDING → ACTIVE → COMPLETED → CANCELLED, dengan persistence ke SQLite.
- **State Transitions:**
  ```
  PENDING → ACTIVE (saat semua player join)
  ACTIVE → COMPLETED (saat game selesai normal)
  ACTIVE → CANCELLED (saat timeout atau player meninggalkan session)
  PENDING → CANCELLED (saat host membatalkan)
  ```
- **Acceptance Criteria:**
  - [ ] Tabel `game_sessions` dengan UUID PK, state, game_type, dan epoch timestamps
  - [ ] Tabel `session_players` untuk many-to-many user ↔ session
  - [ ] Transisi state tervalidasi (tidak bisa dari COMPLETED ke ACTIVE)
  - [ ] Session timeout handler via goroutine ticker
  - [ ] Repository interface + SQLite implementation

#### 2.3 AI Game Master Prompt System
- **Deskripsi:** Sistem prompt template untuk AI Game Master yang bisa generate scenarios, questions, dan hints berdasarkan game type dan difficulty.
- **Acceptance Criteria:**
  - [ ] Prompt templates disimpan sebagai embedded files (`//go:embed`)
  - [ ] Template mendukung variabel: `{{.GameType}}`, `{{.Difficulty}}`, `{{.Context}}`
  - [ ] Struct `GameMasterRequest` dan `GameMasterResponse` terdefinisi di domain layer
  - [ ] Response parsing untuk extract pertanyaan/skenario dari output Ollama
  - [ ] Fallback jika Ollama tidak tersedia (preset static questions)

#### 2.4 Game 1: Would You Rather?
- **Deskripsi:** Game paling sederhana — AI generate dua pilihan, player memilih via Discord button, AI merespons dengan komentar dan follow-up question.
- **Acceptance Criteria:**
  - [ ] `/radovid play would-you-rather` memulai sesi
  - [ ] AI generate dua opsi dalam English, ditampilkan sebagai Discord Embed + 2 buttons
  - [ ] Player memilih tombol, bot mencatat pilihan dan generate komentar AI
  - [ ] Sesi berlanjut minimal 3 ronde, kemudian ada opsi lanjut/selesai
  - [ ] Hasil akhir ditampilkan sebagai embed dengan ringkasan pilihan

#### 2.5 Game 2: Taboo
- **Deskripsi:** Game vocabulary — AI generate kata target + daftar kata terlarang, player harus mendeskripsikan tanpa kata terlarang, AI menilai apakah deskripsi valid.
- **Acceptance Criteria:**
  - [ ] AI generate kata target dan 5 kata terlarang
  - [ ] Player mengirim teks deskripsi via modal Discord
  - [ ] AI menilai apakah kata terlarang digunakan dan apakah deskripsi cukup jelas
  - [ ] Score system berdasarkan jumlah kata berhasil dideskripsikan dalam waktu yang ditentukan
  - [ ] Turn-based: giliran per player dalam sesi multiplayer

#### 2.6 Game 3: 20 Questions
- **Deskripsi:** AI "memikirkan" suatu benda/orang/tempat, player punya 20 pertanyaan yes/no untuk menebak.
- **Acceptance Criteria:**
  - [ ] AI generate topik rahasia dan menyimpan di session state (tidak visible ke player)
  - [ ] Player mengirim pertanyaan via message atau modal, AI menjawab "Yes", "No", atau "Sometimes"
  - [ ] Counter pertanyaan tersisa ditampilkan di setiap response embed
  - [ ] Game berakhir saat player menebak benar atau 20 pertanyaan habis
  - [ ] AI memberikan penjelasan singkat di akhir game

#### 2.7 Discord Embed Templates (Game UI)
- **Deskripsi:** Library embed builder yang konsisten untuk semua tampilan game: session info, round display, hasil, error.
- **Acceptance Criteria:**
  - [ ] `EmbedBuilder` struct dengan method chaining untuk title, description, fields, color, footer
  - [ ] Preset warna: biru untuk info, hijau untuk success, merah untuk error, kuning untuk warning
  - [ ] Template fungsi untuk: session start, round display, result, error ephemeral
  - [ ] Warna dan format konsisten di semua game
  - [ ] Unit test untuk embed builder

---

## Fase 3: Speaking & Writing Activities

> **Tujuan:** Implementasi game-game speaking dan writing yang memerlukan AI feedback lebih kompleks, termasuk grammar correction dan vocabulary feedback.

**Prasyarat:** Fase 2 selesai, Ollama client dan game session state machine stabil.

### Backlog Fase 3

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 3.1 | AI Feedback System (Grammar & Vocabulary) | `1762347400` | `[CREATED]` | `TICKET-1762347400` / `PLAN-1762347400` |
| 3.2 | Game 4: Job Interview | `1762347460` | `[CREATED]` | `TICKET-1762347460` / `PLAN-1762347460` |
| 3.3 | Game 5: Roleplay | `1762347520` | `[CREATED]` | `TICKET-1762347520` / `PLAN-1762347520` |
| 3.4 | Game 6: One Minute Challenge | `1762347580` | `[CREATED]` | `TICKET-1762347580` / `PLAN-1762347580` |
| 3.5 | Game 7: Story Relay | `1762347640` | `[CREATED]` | `TICKET-1762347640` / `PLAN-1762347640` |
| 3.6 | Game 8: Daily 5-Sentence Diary | `1762347700` | `[CREATED]` | `TICKET-1762347700` / `PLAN-1762347700` |
| 3.7 | Game 9: Five Word Writing Challenge | `1762347760` | `[CREATED]` | `TICKET-1762347760` / `PLAN-1762347760` |
| 3.8 | Difficulty Level System | `1762347820` | `[CREATED]` | `TICKET-1762347820` / `PLAN-1762347820` |

### Detail Item Fase 3

#### 3.1 AI Feedback System (Grammar & Vocabulary)
- **Deskripsi:** Sistem feedback AI yang dapat menganalisis teks tulisan/ucapan user dan memberikan koreksi grammar, saran vocabulary, dan skor komunikasi.
- **Acceptance Criteria:**
  - [ ] `FeedbackRequest` struct dengan teks input, konteks game, dan difficulty level
  - [ ] `FeedbackResponse` struct dengan grammar corrections, vocabulary suggestions, dan overall score (1-10)
  - [ ] Prompt template khusus untuk feedback mode (berbeda dari game master mode)
  - [ ] Parsing structured output dari Ollama (JSON atau delimited format)
  - [ ] Fallback response jika parsing gagal (generic encouragement)

#### 3.2 Game 4: Job Interview
- **Deskripsi:** Roleplay — AI bertindak sebagai interviewer, mengajukan pertanyaan interview dalam English, player menjawab, AI memberikan feedback profesional.
- **Acceptance Criteria:**
  - [ ] Player memilih bidang pekerjaan (Tech, Marketing, Design, dll.) via select menu
  - [ ] AI generate pertanyaan interview yang realistis sesuai bidang
  - [ ] Player menjawab via Discord modal (teks panjang)
  - [ ] AI memberikan feedback per jawaban: apa yang baik, apa yang perlu diperbaiki
  - [ ] Sesi terdiri dari 5 pertanyaan, diakhiri dengan overall assessment

#### 3.3 Game 5: Roleplay
- **Deskripsi:** AI generate situasi sehari-hari (di restoran, di bandara, melapor kehilangan, dll.), player dan AI saling bercakap-cakap dalam English.
- **Acceptance Criteria:**
  - [ ] AI generate skenario pembuka dengan konteks situasi
  - [ ] Dialog turn-based: AI bertindak sebagai karakter dalam situasi
  - [ ] AI secara subtle mengoreksi grammar besar tanpa memutus alur percakapan
  - [ ] Sesi berakhir setelah situasi "terselesaikan" atau setelah 10 giliran
  - [ ] Ringkasan koreksi grammar di akhir sesi

#### 3.4 Game 6: One Minute Challenge
- **Deskripsi:** AI generate topik random, player berbicara (menulis dalam konteks Discord) tentang topik tersebut selama satu menit — fokus pada fluency bukan accuracy.
- **Acceptance Criteria:**
  - [ ] AI generate topik menarik dan unexpected (contoh: "Why cats are secretly spies")
  - [ ] Timer 60 detik ditampilkan di embed (countdown via Discord message edit atau field update)
  - [ ] Player mengirim teks respons setelah timer selesai
  - [ ] AI memberikan feedback pada word count, variety, dan coherence

#### 3.5 Game 7: Story Relay
- **Deskripsi:** Collaborative storytelling — AI memulai cerita, setiap player (atau AI dalam single-player) menambahkan satu-dua kalimat, AI melanjutkan dan mengevaluasi.
- **Acceptance Criteria:**
  - [ ] AI generate opening paragraph cerita
  - [ ] Player menambahkan kelanjutan via modal (minimal 1 kalimat)
  - [ ] AI melanjutkan cerita berdasarkan kontribusi player dan menjaga koherensi
  - [ ] AI highlight kesalahan grammar signifikan setelah setiap giliran
  - [ ] Cerita berakhir setelah round yang ditentukan dengan ending yang memuaskan

#### 3.6 Game 8: Daily 5-Sentence Diary
- **Deskripsi:** Player menulis 5 kalimat diary dalam English tentang hari mereka, AI memberikan grammar feedback dan vocabulary enrichment suggestions.
- **Acceptance Criteria:**
  - [ ] `/radovid diary` memunculkan Discord modal dengan 5 field teks
  - [ ] AI menganalisis setiap kalimat secara individual dan memberikan feedback spesifik
  - [ ] AI menyarankan 3 kata baru yang relevan dengan konteks diary
  - [ ] Respons disimpan di database (tabel `diary_entries`) dengan timestamp
  - [ ] Daily reminder notification support (lihat Fase 4)

#### 3.7 Game 9: Five Word Writing Challenge
- **Deskripsi:** AI generate 5 kata random (kata benda, kata kerja, kata sifat campuran), player harus membuat satu paragraf yang menggunakan kelima kata tersebut.
- **Acceptance Criteria:**
  - [ ] AI generate 5 kata yang diverse dan menarik
  - [ ] Player menulis paragraf via modal
  - [ ] AI verifikasi semua 5 kata digunakan dan berikan feedback pada kualitas tulisan
  - [ ] Score berdasarkan: semua kata terpakai, koherensi paragraf, kualitas grammar

#### 3.8 Difficulty Level System
- **Deskripsi:** Sistem level kesulitan (Beginner/Intermediate/Advanced) yang mempengaruhi kompleksitas pertanyaan AI, vocabulary yang digunakan, dan toleransi error feedback.
- **Acceptance Criteria:**
  - [ ] Enum `Difficulty` dengan nilai `Beginner`, `Intermediate`, `Advanced`
  - [ ] Player bisa set preferensi difficulty via `/radovid settings`
  - [ ] Difficulty disimpan di tabel `users` dan persisten lintas sesi
  - [ ] Prompt template menerima difficulty sebagai parameter yang mengubah instruksi AI
  - [ ] Feedback lebih encouraging untuk Beginner, lebih teknis untuk Advanced

---

## Fase 4: Gamification & Dashboard

> **Tujuan:** XP system, progress tracking, achievements, admin dashboard HTMX, dan melengkapi semua game yang tersisa.

**Prasyarat:** Fase 3 selesai, AI feedback system stabil dan teruji.

### Backlog Fase 4

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 4.1 | XP Calculation & Level System | `1762433800` | `[CREATED]` | `TICKET-1762433800` / `PLAN-1762433800` |
| 4.2 | Streak Tracking (Daily Activity) | `1762433860` | `[CREATED]` | `TICKET-1762433860` / `PLAN-1762433860` |
| 4.3 | Achievements System | `1762433920` | `[CREATED]` | `TICKET-1762433920` / `PLAN-1762433920` |
| 4.4 | `/radovid progress` Command | `1762433980` | `[CREATED]` | `TICKET-1762433980` / `PLAN-1762433980` |
| 4.5 | HTMX Admin Dashboard | `1762434040` | `[CREATED]` | `TICKET-1762434040` / `PLAN-1762434040` |
| 4.6 | Game 10: Secret Mission | `1762434100` | `[CREATED]` | `TICKET-1762434100` / `PLAN-1762434100` |
| 4.7 | Game 11: Grammar Boss Fight | `1762434160` | `[CREATED]` | `TICKET-1762434160` / `PLAN-1762434160` |
| 4.8 | Game 12: Sentence Transformation Race | `1762434220` | `[CREATED]` | `TICKET-1762434220` / `PLAN-1762434220` |
| 4.9 | Game 13: Movie Review | `1762434280` | `[CREATED]` | `TICKET-1762434280` / `PLAN-1762434280` |
| 4.10 | Game 14: News Reporter | `1762434340` | `[CREATED]` | `TICKET-1762434340` / `PLAN-1762434340` |
| 4.11 | Game 15: Design Your Dream House | `1762434400` | `[CREATED]` | `TICKET-1762434400` / `PLAN-1762434400` |
| 4.12 | Daily Challenge Notifications | `1762434460` | `[CREATED]` | `TICKET-1762434460` / `PLAN-1762434460` |

### Detail Item Fase 4 (Ringkasan)

#### 4.1 XP Calculation & Level System
- XP diberikan per aktivitas selesai, dengan multiplier berdasarkan difficulty
- Formula XP to next level: `level^2 * 100`
- Tabel `user_xp_log` mencatat setiap XP gain dengan sumber aktivitas

#### 4.2 Streak Tracking
- Streak increment jika user melakukan aktivitas dalam 24 jam dari aktivitas terakhir
- Streak reset jika lewat 48 jam tanpa aktivitas
- Streak bonus XP: `streak_days * 5` XP per hari

#### 4.3 Achievements System
- Achievement terdiri dari: `id`, `name`, `description`, `icon_emoji`, `xp_reward`, `trigger_condition`
- Achievement awal: First Game, 7-Day Streak, 100 Games, Grammar Master, Vocabulary Champion
- Achievement notification via Discord DM atau ephemeral response

#### 4.4 `/radovid progress` Command
- Embed menampilkan: level, total XP, XP to next level, current streak, total games, achievements unlocked
- Visual progress bar menggunakan Unicode block characters

#### 4.5 HTMX Admin Dashboard
- HTTP server embedded di binary (port dari env var `DASHBOARD_PORT`)
- Route: `/` (overview), `/sessions` (session history), `/users` (user list), `/health` (system status)
- HTMX untuk refresh data tanpa page reload
- Basic Auth dari env var `DASHBOARD_USERNAME` / `DASHBOARD_PASSWORD`
- Templates di-embed via `//go:embed web/templates/*.html`

#### 4.6–4.11 Remaining Games
- **Secret Mission:** Player mendapat misi rahasia dalam English, harus melaksanakan (melalui teks) tanpa ketahuan — AI menilai.
- **Grammar Boss Fight:** AI generate kalimat salah grammar, player harus memperbaiki. Tiap perbaikan benar = damage ke "boss".
- **Sentence Transformation Race:** AI memberikan kalimat aktif, player transformasi ke pasif (atau sebaliknya), atau dari direct ke indirect speech.
- **Movie Review:** Player memilih film, AI bertanya pertanyaan review terstruktur, player menjawab dalam English, AI feedback pada ekspresi opini.
- **News Reporter:** AI generate "breaking news" scenario, player harus melaporkan berita tersebut dalam format siaran berita.
- **Design Your Dream House:** Roleplay kreatif — player mendeskripsikan rumah impian dalam English, AI membantu dengan vocabulary arsitektur dan memberikan visual description.

#### 4.12 Daily Challenge Notifications
- Goroutine dengan ticker harian (waktu dikonfigurasi via env var `DAILY_CHALLENGE_HOUR`)
- Kirim Discord channel message (bukan DM) dengan daily challenge embed
- Challenge dipilih secara rotasi atau random dari pool aktivitas tersedia

---

## Catatan Pengerjaan

> **[MODE 1 — Autonomous Planning]** Setiap item di atas harus dieksekusi sebagai pasangan `TICKET-[EPOCH]` + `PLAN-[EPOCH]`. Buat pasangan tersebut sebelum memulai implementasi, paparkan rencana ke developer, dan minta persetujuan eksplisit sebelum mutasi state apapun.

**Aturan Centang Status:**
- `[CREATED]` — belum dimulai
- `[~] In Progress` — sedang dikerjakan (sertakan epoch sesi aktif)
- `[x] Done` — selesai, delivery gate lulus, tiket & rencana ditandai Done/Completed
