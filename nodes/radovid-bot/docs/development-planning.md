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
| 2.2 | Docker Containerization & Ollama Service Orchestration | `1762261030` | `[COMPLETED]` | `TICKET-1762261030` / `PLAN-1762261030` |
| 2.3 | Game Session State Machine | `1762261060` | `[COMPLETED]` | `TICKET-1762261060` / `PLAN-1762261060` |
| 2.4 | Database Query Builder Utility | `1762261090` | `[COMPLETED]` | `TICKET-1762261090` / `PLAN-1762261090` |
| 2.5 | AI Game Master Prompt System | `1762261120` | `[COMPLETED]` | `TICKET-1762261120` / `PLAN-1762261120` |
| 2.6 | Game 1: Would You Rather? | `1762261180` | `[COMPLETED]` | `TICKET-1762261180` / `PLAN-1762261180` |
| 2.7 | Game 2: Taboo | `1762261240` | `[COMPLETED]` | `TICKET-1762261240` / `PLAN-1762261240` |
| 2.8 | Game 3: 20 Questions | `1762261300` | `[COMPLETED]` | `TICKET-1762261300` / `PLAN-1762261300` |
| 2.9 | Discord Embed Templates (Game UI) | `1762261360` | `[COMPLETED]` | `TICKET-1762261360` / `PLAN-1762261360` |

### Detail Item Fase 2

#### 2.1 Ollama HTTP Client Wrapper
- **Deskripsi:** HTTP client untuk Ollama Generate API dengan timeout wajib, retry logic (max 2 retries), dan sanitasi input sebelum dikirim.
- **Acceptance Criteria:**
  - [x] Client menggunakan `context.Context` dengan timeout dari `OLLAMA_TIMEOUT_SECONDS`
  - [x] Input sanitization: hapus Discord mention, strip markdown manipulatif, max 2000 karakter
  - [x] Retry dengan exponential backoff untuk error 5xx
  - [x] Return typed error untuk timeout vs connection refused vs model error
  - [x] Unit test table-driven untuk sanitization dan error handling

#### 2.2 Docker Containerization & Ollama Service Orchestration
- **Deskripsi:** Kontainerisasi aplikasi bot Go (multi-stage build) dan orkestrasi layanan Ollama offline beserta otomasi penarikan model Gemma 4b via Docker Compose.
- **Acceptance Criteria:**
  - [ ] Multi-stage `Dockerfile` untuk memproduksi biner Go minimalis dan aman
  - [ ] Berkas `.dockerignore` untuk mengabaikan git, data lokal, dan secrets
  - [ ] `docker-compose.yml` terintegrasi dengan 3 layanan: `ollama`, `ollama-pull`, dan `radovid-bot`
  - [ ] Otomasi penarikan model `gemma3:4b` pada inisialisasi awal container Ollama
  - [ ] Persistensi volume untuk model Ollama dan SQLite database lokal

#### 2.3 Game Session State Machine
- **Deskripsi:** State machine untuk siklus hidup sesi game: PENDING → ACTIVE → COMPLETED → CANCELLED, dengan persistence ke SQLite.
- **State Transitions:**
  ```
  PENDING → ACTIVE (saat semua player join)
  ACTIVE → COMPLETED (saat game selesai normal)
  ACTIVE → CANCELLED (saat timeout atau player meninggalkan session)
  PENDING → CANCELLED (saat host membatalkan)
  ```
- **Acceptance Criteria:**
  - [x] Tabel `game_sessions` dengan UUID PK, state, game_type, dan epoch timestamps
  - [x] Tabel `session_participants` untuk relasi user ↔ session
  - [x] Transisi state tervalidasi (tidak bisa dari COMPLETED ke ACTIVE)
  - [x] Session timeout handler via goroutine ticker
  - [x] Repository interface + SQLite implementation

#### 2.4 Database Query Builder Utility
- **Deskripsi:** Pustaka utilitas SQL query builder ringan untuk operasi batch/bulk (insert, update, delete, select-in) berbasis standard library Go dengan sanitasi identifier.
- **Acceptance Criteria:**
  - [x] Fungsi `BuildBulkInsertQuery` untuk multi-row INSERT berparameter
  - [x] Fungsi `BuildBatchUpdateQuery` untuk UPDATE dengan WHERE IN
  - [x] Fungsi `BuildCaseUpdateQuery` untuk update nilai berbeda per primary key via CASE WHEN
  - [x] Fungsi `BuildBatchDeleteQuery` untuk DELETE dengan WHERE IN
  - [x] Fungsi `BuildSelectInQuery` untuk SELECT dengan WHERE IN
  - [x] Validasi ketat nama identifier (tabel & kolom) mencegah SQL injection
  - [x] Integrasi ke `session_repository.go` dan unit test 100% PASS

#### 2.5 AI Game Master Prompt System
- **Deskripsi:** Sistem prompt template untuk AI Game Master yang bisa generate scenarios, questions, dan hints berdasarkan game type dan difficulty.
- **Acceptance Criteria:**
  - [x] Prompt templates disimpan sebagai embedded files (`//go:embed`)
  - [x] Template mendukung variabel: `{{.GameType}}`, `{{.Difficulty}}`, `{{.Context}}`
  - [x] Struct `GameMasterRequest` dan `GameMasterResponse` terdefinisi di domain layer
  - [x] Response parsing untuk extract pertanyaan/skenario dari output Ollama
  - [x] Fallback jika Ollama tidak tersedia (preset static questions)

#### 2.6 Game 1: Would You Rather?
- **Deskripsi:** Game paling sederhana — AI generate dua pilihan, player memilih via Discord button, AI merespons dengan komentar dan follow-up question.
- **Acceptance Criteria:**
  - [x] `/radovid play would-you-rather` memulai sesi
  - [x] AI generate dua opsi dalam English, ditampilkan sebagai Discord Embed + 2 buttons
  - [x] Player memilih tombol, bot mencatat pilihan dan generate komentar AI
  - [x] Sesi berlanjut minimal 3 ronde, kemudian ada opsi lanjut/selesai
  - [x] Hasil akhir ditampilkan sebagai embed dengan ringkasan pilihan

#### 2.7 Game 2: Taboo
- **Deskripsi:** Game vocabulary — AI generate kata target + daftar kata terlarang, player harus mendeskripsikan tanpa kata terlarang, AI menilai apakah deskripsi valid.
- **Acceptance Criteria:**
  - [x] AI generate kata target dan 5 kata terlarang
  - [x] Player mengirim teks deskripsi via modal Discord
  - [x] AI menilai apakah kata terlarang digunakan dan apakah deskripsi cukup jelas
  - [x] Score system berdasarkan jumlah kata berhasil dideskripsikan dalam waktu yang ditentukan
  - [x] Turn-based: giliran per player dalam sesi multiplayer

#### 2.8 Game 3: 20 Questions
- **Deskripsi:** AI "memikirkan" suatu benda/orang/tempat, player punya 20 pertanyaan yes/no untuk menebak.
- **Acceptance Criteria:**
  - [x] AI generate topik rahasia dan menyimpan di session state (tidak visible ke player)
  - [x] Player mengirim pertanyaan via message atau modal, AI menjawab "Yes", "No", atau "Sometimes"
  - [x] Counter pertanyaan tersisa ditampilkan di setiap response embed
  - [x] Game berakhir saat player menebak benar atau 20 pertanyaan habis
  - [x] AI memberikan penjelasan singkat di akhir game

#### 2.9 Discord Embed Templates (Game UI)
- **Deskripsi:** Library embed builder yang konsisten untuk semua tampilan game: session info, round display, hasil, error.
- **Acceptance Criteria:**
  - [x] `EmbedBuilder` struct dengan method chaining untuk title, description, fields, color, footer
  - [x] Preset warna: biru untuk info, hijau untuk success, merah untuk error, kuning untuk warning
  - [x] Template fungsi untuk: session start, round display, result, error ephemeral
  - [x] Warna dan format konsisten di semua game
  - [x] Unit test untuk embed builder

---

## Fase 3: Speaking & Writing Activities

> **Tujuan:** Implementasi game-game speaking dan writing yang memerlukan AI feedback lebih kompleks, termasuk grammar correction dan vocabulary feedback.

**Prasyarat:** Fase 2 selesai, Ollama client dan game session state machine stabil.

### Backlog Fase 3

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 3.1 | AI Feedback System (Grammar & Vocabulary) | `1762347400` | `[COMPLETED]` | `TICKET-1762347400` / `PLAN-1762347400` |
| 3.2 | Game 4: Job Interview | `1762347460` | `[COMPLETED]` | `TICKET-1762347460` / `PLAN-1762347460` |
| 3.3 | Game 5: Roleplay | `1762347520` | `[COMPLETED]` | `TICKET-1762347520` / `PLAN-1762347520` |
| 3.4 | Game 6: One Minute Challenge | `1762347580` | `[COMPLETED]` | `TICKET-1762347580` / `PLAN-1762347580` |
| 3.5 | Game 7: Story Relay | `1762347640` | `[COMPLETED]` | `TICKET-1762347640` / `PLAN-1762347640` |
| 3.6 | Game 8: Daily 5-Sentence Diary | `1762347700` | `[COMPLETED]` | `TICKET-1762347700` / `PLAN-1762347700` |
| 3.7 | Game 9: Five Word Writing Challenge | `1762347760` | `[COMPLETED]` | `TICKET-1762347760` / `PLAN-1762347760` |
| 3.8 | Difficulty Level System | `1762347820` | `[COMPLETED]` | `TICKET-1762347820` / `PLAN-1762347820` |

### Detail Item Fase 3

#### 3.1 AI Feedback System (Grammar & Vocabulary)
- **Deskripsi:** Sistem feedback AI yang dapat menganalisis teks tulisan/ucapan user dan memberikan koreksi grammar, saran vocabulary, dan skor komunikasi.
- **Acceptance Criteria:**
  - [x] `FeedbackRequest` struct dengan teks input, konteks game, dan difficulty level
  - [x] `FeedbackResponse` struct dengan grammar corrections, vocabulary suggestions, dan overall score (1-10)
  - [x] Prompt template khusus untuk feedback mode (berbeda dari game master mode)
  - [x] Parsing structured output dari Ollama (JSON atau delimited format)
  - [x] Fallback response jika parsing gagal (generic encouragement)

#### 3.2 Game 4: Job Interview
- **Deskripsi:** Roleplay — AI bertindak sebagai interviewer, mengajukan pertanyaan interview dalam English, player menjawab, AI memberikan feedback profesional.
- **Acceptance Criteria:**
  - [x] Player memilih bidang pekerjaan (Tech, Marketing, Design, dll.) via select menu
  - [x] AI generate pertanyaan interview yang realistis sesuai bidang
  - [x] Player menjawab via Discord modal (teks panjang)
  - [x] AI memberikan feedback per jawaban: apa yang baik, apa yang perlu diperbaiki
  - [x] Sesi terdiri dari 5 pertanyaan, diakhiri dengan overall assessment

#### 3.3 Game 5: Roleplay
- **Deskripsi:** AI generate situasi sehari-hari (di restoran, di bandara, melapor kehilangan, dll.), player dan AI saling bercakap-cakap dalam English.
- **Acceptance Criteria:**
  - [x] AI generate skenario pembuka dengan konteks situasi
  - [x] Dialog turn-based: AI bertindak sebagai karakter dalam situasi
  - [x] AI secara subtle mengoreksi grammar besar tanpa memutus alur percakapan
  - [x] Sesi berakhir setelah situasi "terselesaikan" atau setelah 10 giliran
  - [x] Ringkasan koreksi grammar di akhir sesi

#### 3.4 Game 6: One Minute Challenge
- **Deskripsi:** AI generate topik random, player berbicara (menulis dalam konteks Discord) tentang topik tersebut selama satu menit — fokus pada fluency bukan accuracy.
- **Acceptance Criteria:**
  - [x] AI generate topik menarik dan unexpected (contoh: "Why cats are secretly spies")
  - [x] Timer 60 detik ditampilkan di embed (countdown via Discord message edit atau field update)
  - [x] Player mengirim teks respons setelah timer selesai
  - [x] AI memberikan feedback pada word count, variety, dan coherence

#### 3.5 Game 7: Story Relay
- **Deskripsi:** Collaborative storytelling — AI memulai cerita, setiap player (atau AI dalam single-player) menambahkan satu-dua kalimat, AI melanjutkan dan mengevaluasi.
- **Acceptance Criteria:**
  - [x] AI generate opening paragraph cerita
  - [x] Player menambahkan kelanjutan via modal (minimal 1 kalimat)
  - [x] AI melanjutkan cerita berdasarkan kontribusi player dan menjaga koherensi
  - [x] AI highlight kesalahan grammar signifikan setelah setiap giliran
  - [x] Cerita berakhir setelah round yang ditentukan dengan ending yang memuaskan

#### 3.6 Game 8: Daily 5-Sentence Diary
- **Deskripsi:** Player menulis 5 kalimat diary dalam English tentang hari mereka, AI memberikan grammar feedback dan vocabulary enrichment suggestions.
- **Acceptance Criteria:**
  - [x] `/radovid diary` memunculkan Discord modal dengan 5 field teks
  - [x] AI menganalisis setiap kalimat secara individual dan memberikan feedback spesifik
  - [x] AI menyarankan 3 kata baru yang relevan dengan konteks diary
  - [x] Respons disimpan di database (tabel `daily_diaries`) dengan timestamp
  - [x] Daily reminder notification support (terkoneksi dan siap diintegrasikan pada sistem penjadwalan Fase 4)

#### 3.7 Game 9: Five Word Writing Challenge
- **Deskripsi:** AI generate 5 kata random (kata benda, kata kerja, kata sifat campuran), player harus membuat satu paragraf yang menggunakan kelima kata tersebut.
- **Acceptance Criteria:**
  - [x] AI generate 5 kata yang diverse dan menarik
  - [x] Player menulis paragraf via modal
  - [x] AI verifikasi semua 5 kata digunakan dan berikan feedback pada kualitas tulisan
  - [x] Score berdasarkan: semua kata terpakai, koherensi paragraf, kualitas grammar

#### 3.8 Difficulty Level System
- **Deskripsi:** Sistem level kesulitan (Beginner/Intermediate/Advanced) yang mempengaruhi kompleksitas pertanyaan AI, vocabulary yang digunakan, dan toleransi error feedback.
- **Acceptance Criteria:**
  - [x] Enum `Difficulty` dengan nilai `Beginner`, `Intermediate`, `Advanced`
  - [x] Player bisa set preferensi difficulty via `/radovid settings`
  - [x] Difficulty disimpan di tabel `users` dan persisten lintas sesi
  - [x] Prompt template menerima difficulty sebagai parameter yang mengubah instruksi AI
  - [x] Feedback lebih encouraging untuk Beginner, lebih teknis untuk Advanced

---

## Fase 4: Gamification & Dashboard

> **Tujuan:** XP system, progress tracking, achievements, admin dashboard HTMX, dan melengkapi semua game yang tersisa.

**Prasyarat:** Fase 3 selesai, AI feedback system stabil dan teruji.

### Backlog Fase 4

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 4.1 | XP Calculation & Level System | `1762433800` | `[COMPLETED]` | `TICKET-1762433800` / `PLAN-1762433800` |
| 4.2 | Streak Tracking (Daily Activity) | `1762433860` | `[COMPLETED]` | `TICKET-1762433860` / `PLAN-1762433860` |
| 4.3 | Achievements System | `1762433920` | `[COMPLETED]` | `TICKET-1762433920` / `PLAN-1762433920` |
| 4.4 | `/radovid progress` Command | `1762433980` | `[COMPLETED]` | `TICKET-1762433980` / `PLAN-1762433980` |
| 4.5 | HTMX Admin Dashboard | `1762434040` | `[COMPLETED]` | `TICKET-1762434040` / `PLAN-1762434040` |
| 4.6 | Game 10: Secret Mission | `1762434100` | `[COMPLETED]` | `TICKET-1762434100` / `PLAN-1762434100` |
| 4.7 | Game 11: Grammar Boss Fight | `1762434160` | `[COMPLETED]` | `TICKET-1762434160` / `PLAN-1762434160` |
| 4.8 | Game 12: Sentence Transformation Race | `1762434220` | `[COMPLETED]` | `TICKET-1762434220` / `PLAN-1762434220` |
| 4.9 | Game 13: Movie Review | `1762434280` | `[COMPLETED]` | `TICKET-1762434280` / `PLAN-1762434280` |
| 4.10 | Game 14: News Reporter | `1762434340` | `[COMPLETED]` | `TICKET-1762434340` / `PLAN-1762434340` |
| 4.11 | Game 15: Design Your Dream House | `1762434400` | `[COMPLETED]` | `TICKET-1762434400` / `PLAN-1762434400` |
| 4.12 | Daily Challenge Notifications | `1762434460` | `[COMPLETED]` | `TICKET-1762434460` / `PLAN-1762434460` |

### Detail Item Fase 4 (Ringkasan)

#### 4.1 XP Calculation & Level System
- XP diberikan per aktivitas selesai, dengan multiplier berdasarkan difficulty
- Formula XP to next level: `level^2 * 100`
- Tabel `user_xp_log` mencatat setiap XP gain dengan sumber aktivitas
- **Acceptance Criteria:**
  - [x] Migration DDL `008_create_user_progress.sql` dan `009_create_user_xp_log.sql`
  - [x] Formula `CalculateLevel`, `XPRequiredForLevel`, dan `ApplyMultiplier` (Easy 1.0x, Medium 1.5x, Hard 2.0x)
  - [x] Use case `AwardXP` dengan transaksi atomik SQLite
  - [x] Deteksi event level-up via `LevelUpResult.LeveledUp`
  - [x] Unit test 100% PASS mencakup batas nol, multi-level jump, dan input negatif

#### 4.2 Streak Tracking
- Streak increment jika user melakukan aktivitas dalam 24 jam dari aktivitas terakhir
- Streak reset jika lewat 48 jam tanpa aktivitas
- Streak bonus XP: `streak_days * 5` XP per hari
- **Acceptance Criteria:**
  - [x] Logika `EvaluateStreak` membedakan `StreakSameDay`, `StreakIncremented`, dan `StreakReset`
  - [x] Streak reset ke 1 jika selisih waktu melampaui 48 jam (172.800.000 ms)
  - [x] Kalkulasi bonus XP harian: `streak_days * 5` XP pada `StreakIncremented`
  - [x] Use case `RecordDailyActivity` memperbarui `streak_days` & `last_activity_at` di tabel `user_progress` dan memicu `AwardXP`
  - [x] Standar UTC murni untuk konsistensi tanggal global
  - [x] Unit test komprehensif mencakup first activity, same day, consecutive day, batas 48 jam, dan clock drift

#### 4.3 Achievements System
- Achievement terdiri dari: `id`, `name`, `description`, `icon_emoji`, `xp_reward`, `trigger_condition`
- Achievement awal: First Game, 7-Day Streak, 30-Day Streak, Grammar Master, Vocabulary Champion, Boss Slayer
- Achievement notification via Discord embed perayaan
- **Acceptance Criteria:**
  - [x] Katalog statis 6 achievement dengan XP reward (`first_game`, `streak_7`, `streak_30`, `grammar_master`, `vocab_champion`, `boss_slayer`)
  - [x] Engine `EvaluateAchievements` mengevaluasi statistik pengguna dan memfilter lencana yang sudah diperoleh
  - [x] Idempotensi terjamin via `UNIQUE(user_id, achievement_type)` di SQLite dan pengecekan use case
  - [x] Bonus XP otomatis dihadiahkan via `AwardXPUseCase` saat lencana baru terbuka
  - [x] Embed builder `BuildAchievementUnlockedEmbed` menyajikan emoji, nama, deskripsi, dan bonus XP
  - [x] Unit test 100% PASS mencakup seluruh kriteria milestone dan pencegahan duplikasi lencana

#### 4.4 `/radovid progress` Command
- Embed menampilkan: level, total XP, XP to next level, current streak, total games, achievements unlocked
- Visual progress bar menggunakan Unicode block characters
- **Acceptance Criteria:**
  - [x] Perintah slash `/radovid progress` terdaftar di router Discord command framework
  - [x] Embed Discord menyajikan visualisasi statistik: Level, Total XP, progress bar XP, streak (🔥), dan daftar lencana
  - [x] Tersedia utilitas generator Unicode Progress Bar murni Go yang fleksibel dan aman dari overflow/divisi nol
  - [x] Menampilkan deretan lencana pencapaian (Achievements) yang telah dibuka atau placeholder edukatif jika belum ada
  - [x] Penanganan pengguna baru secara otomatis membuat profil inisial (Level 1, 0 XP, 0 Streak) tanpa error
  - [x] Unit testing 100% PASS pada generator progress bar, use case, dan handler Discord

#### 4.5 HTMX Admin Dashboard
- HTTP server embedded di binary (port dari env var `ADMIN_DASHBOARD_PORT`, default: 8080)
- Route: `/admin` (overview), `/admin/sessions` (session history & filter), `/admin/users` (user list), `/admin/health` (system status)
- HTMX untuk refresh data tanpa page reload
- Basic Auth dari env var `ADMIN_DASHBOARD_SECRET` (format: `user:password`)
- Templates & static assets di-embed via `//go:embed templates/* static/*`
- **Acceptance Criteria:**
  - [x] Server HTTP tertanam berjalan pada goroutine mandiri mendengarkan port dari `ADMIN_DASHBOARD_PORT`
  - [x] Middleware HTTP Basic Auth memvalidasi format `username:password` dari `ADMIN_DASHBOARD_SECRET`
  - [x] Seluruh aset web (HTML templates dan aset statis) di-embed ke binary Go via `embed.FS`
  - [x] Endpoint `/admin`, `/admin/sessions`, `/admin/users`, dan `/admin/health` tersedia dan berfungsi
  - [x] HTMX partial swap memperbarui blok data spesifik secara asinkron tanpa full-page reload
  - [x] Unit test 100% PASS pada middleware Basic Auth, use case, dan HTTP handler

#### 4.6 Game 10: Secret Mission
- **Deskripsi:** Player mendapat misi rahasia dalam English (kata atau pola tata bahasa target), harus melaksanakan (melalui teks diskusi publik) tanpa ketahuan — AI menilai akurasi tata bahasa dan keluwesan (stealth/naturalness).
- **Acceptance Criteria:**
  - [x] Game Type `secret_mission` terdaftar dalam enum entitas game dan sub-command `/radovid play secret-mission`
  - [x] Generator topik diskusi publik dan misi rahasia unik (kata/tata bahasa target)
  - [x] Misi rahasia dikirim secara privat menggunakan Discord Ephemeral interaction response
  - [x] Pemain mengirimkan kalimat/paragraf jawaban melalui Discord Modal
  - [x] AI mengevaluasi input pemain dengan parameter: target_present, grammar_score, stealth_naturalness_score, critique_notes
  - [x] Perolehan total score dan XP reward dihitung secara proporsional
  - [x] Unit test 100% PASS mencakup domain, parsing respons AI, use case, dan delivery handler

#### 4.7 Game 11: Grammar Boss Fight
- **Deskripsi:** Player bertarung melawan Boss Grammar dalam mode RPG di mana pemain harus memperbaiki kalimat yang rusak untuk memberikan serangan (damage). AI mengevaluasi perbaikan (Critical, Standard, Miss) dan Boss melancarkan serangan balasan jika serangan meleset.
- **Acceptance Criteria:**
  - [x] Game Type `grammar_boss_fight` terdaftar di enum game dan sub-command `/radovid play grammar-boss-fight`
  - [x] Entitas Boss Game menyimpan status state: `boss_name`, `max_hp`, `current_hp`, `player_lives`, `current_round`, dan riwayat pertarungan
  - [x] Template prompt Ollama Gemma 3 4b dan generator kalimat dengan kesalahan tata bahasa tersembunyi sesuai tingkat kesulitan (Easy, Medium, Hard)
  - [x] AI evaluator membandingkan koreksi pemain dengan kalimat rusak asli dan mengembalikan kalkulasi damage (0, 10-15, 25-35) serta penjelasan edukatif
  - [x] Tampilan visual HP Boss disajikan menggunakan Unicode Bar grafis (`[████████░░] 80/100 HP`) pada Discord Embed interaktif
  - [x] Kondisi akhir terkelola dengan baik: Kemenangan (Boss HP <= 0) memberikan bonus XP besar + achievement `boss_slayer`, atau Kekalahan (pemain kehabisan nyawa/ronde)
  - [x] Unit testing 100% PASS terhadap kalkulasi damage, state machine HP & lives, dan parsing format evaluasi pertarungan

#### 4.8 Game 12: Sentence Transformation Race
- **Deskripsi:** Balapan transformasi kalimat cepat (*Sentence Transformation Race*) di mana AI memberikan kalimat acuan dan instruksi transformasi tata bahasa (seperti aktif ke pasif, langsung ke tidak langsung, pergeseran tenses, atau conditional sentences) di bawah batasan waktu yang kompetitif.
- **Acceptance Criteria:**
  - [x] Game Type `sentence_transformation` terdaftar di enum game dan menu `/radovid play transformation-race`
  - [x] AI Game Master mengenerate tantangan kalimat awal beserta kategori transformasi yang valid sesuai level kesulitan
  - [x] Timer countdown visual ditampilkan pada embed ronde (misal: 45 detik) dengan pencatatan timestamp waktu mulai dan waktu submit respon pemain
  - [x] AI evaluator memverifikasi ketepatan gramatikal hasil transformasi, ekuivalensi semantik (makna kalimat tidak berubah), dan mengembalikan skor akurasi (1-100)
  - [x] Rumus penilaian mengombinasikan akurasi sintaksis dan faktor kecepatan: $\text{RoundScore} = \text{Accuracy} \times \left(1.0 + \frac{\text{TimeRemaining}}{\text{TotalTime}} \times 0.5\right)$
  - [x] Embed hasil ronde menyajikan perbandingan kalimat asli, transformasi pemain, kunci transformasi ideal dari AI, dan rincian skor
  - [x] Unit test algoritma skoring kecepatan, parser response evaluasi AI, dan validasi jenis transformasi

#### 4.9 Game 13: Movie Review
- **Deskripsi:** Diskusi dan ulasan film terstruktur 3 fase (*Plot Premise & Setting*, *Character & Acting Analysis*, *Critical Verdict & Recommendation*) untuk melatih diskursus analitis dan memperkaya perbendaharaan kata evaluatif.
- **Acceptance Criteria:**
  - [x] Game Type `movie_review` terdaftar di enum game dan menu slash command `/radovid play movie-review`
  - [x] State sesi mencatat 3 fase review sekuensial: `PhasePremise`, `PhaseCharacters`, `PhaseVerdict`
  - [x] AI Game Master (Ollama Gemma 3 4b) menyesuaikan pertanyaan panduan berdasarkan judul film dan genre yang diinput oleh pemain
  - [x] Pemain mengirimkan respon setiap fase melalui Discord Modal input teks panjang
  - [x] Evaluasi akhir AI menghasilkan laporan komprehensif: skor koherensi, evaluasi ekspresi opini, daftar 3-5 kosakata sinematik alternatif (misal: *captivating*, *monotonous*, *thought-provoking*), dan skor keseluruhan (1-10)
  - [x] Integrasi dengan modul progres: memberikan base XP plus bonus proporsional berdasarkan kualitas ulasan
  - [x] Unit testing terhadap alur transisi fase, parser respon evaluasi ulasan film, dan validasi kelengkapan argumen

#### 4.10 Game 14: News Reporter
- **Deskripsi:** Simulasi peliputan berita breaking news di mana AI memberikan memo fakta peristiwa mentah 5W1H (Who, What, When, Where, Why, How), pemain menyusun naskah siaran berita formal melalui Discord Modal, dan AI mengevaluasi kelengkapan fakta 5W1H, laras bahasa formal (*formal register*), tata bahasa, serta menyajikan model naskah ideal (*exemplary news script*).
- **Acceptance Criteria:**
  - [x] Game Type `news_reporter` terdaftar di enum game dan pilihan menu `/radovid play news-reporter`
  - [x] AI Game Master (Ollama Gemma 3 4b) menghasilkan skenario breaking news yang bervariasi lengkap dengan poin fakta 5W1H
  - [x] Pemain menginput naskah siaran melalui Discord Modal interaktif
  - [x] AI evaluator memvalidasi kelengkapan fakta 5W1H (mendeteksi fakta yang tercakup dan missing facts)
  - [x] AI memberikan skor register formal (1-10) dan akurasi tata bahasa (1-10) dengan analisis konstruksi jurnalistik
  - [x] Menyajikan revisi model broadcast berita yang ideal (*exemplary news script*) sebagai bahan perbandingan belajar pemain
  - [x] Unit testing 100% PASS terhadap model berita 5W1H, use case orkestrasi, fallback deterministik, format embed Discord, dan handler modal

#### 4.11 Game 15: Design Your Dream House
- **Deskripsi:** Roleplay kreatif dan penulisan deskriptif terpandu — pemain mendeskripsikan rumah impian secara bertahap (Exterior, Interior Layout, Signature Sanctuary) dalam bahasa Inggris, AI memindai penggunaan preposisi spasial (*prepositions of place*) dan kosakata estetika arsitektur, memberikan saran kata sifat pengayaan nuansa (*evocative adjectives*), serta merangkum deskripsi ke dalam narasi sintesis arsitektural.
- **Acceptance Criteria:**
  - [x] Game Type `design_dream_house` terdaftar di enum game dan pilihan menu `/radovid play dream-house`
  - [x] Sesi game mengelola 3 tahapan desain terstruktur: Eksterior & Lanskap, Tata Ruang Spasial Interior, dan Ruang Khusus Personal
  - [x] Pemain menginput konsep deskriptif tiap tahapan melalui formulir Discord Modal
  - [x] AI Game Master (Ollama Gemma 3 4b) memindai penggunaan preposisi spasial dan istilah estetika arsitektural
  - [x] AI evaluator menghasilkan umpan balik terstruktur: skor deskripsi spasial (1-10), skor kekayaan kosakata sensorik (1-10), daftar 3 kata sifat arsitektural lanjutan baru, dan rangkuman sintetis (*architectural synthesis narration*)
  - [x] Penambahan reward XP gamifikasi berdasarkan kelengkapan deskripsi dan kekayaan leksikal
  - [x] Unit testing 100% PASS terhadap alur state mesin tahapan desain, ekstraksi terminologi spasial, format embed Discord, parser respons AI, dan use case orkestrasi

#### 4.12 Daily Challenge Notifications
- **Deskripsi:** Background worker goroutine terkelola yang mengirimkan notifikasi tantangan bahasa Inggris harian (*Daily Challenge*) ke channel Discord publik setiap pagi sesuai jadwal waktu UTC yang dikonfigurasi, dilengkapi kartu embed interaktif, tombol aksi "Accept Challenge", dan idempotensi pengiriman persisten.
- **Acceptance Criteria:**
  - [x] Tersedia background scheduler worker yang berjalan dalam goroutine dengan konteks *graceful shutdown* (`context.Context`)
  - [x] Pengaturan waktu harian dikonfigurasi melalui variabel lingkungan `DAILY_CHALLENGE_HOUR` (format jam 0-23 UTC) dan channel tujuan `DAILY_CHALLENGE_CHANNEL_ID`
  - [x] Algoritma scheduler menghitung durasi waktu tunda awal (*initial delay*) hingga jadwal jam target berikutnya tercapai, lalu mengaktifkan interval perulangan `time.NewTicker(24 * time.Hour)`
  - [x] Idempotensi pengiriman persisten via SQLite `daily_challenge_dispatches`: jika bot di-restart pada hari yang sama setelah pesan terkirim, scheduler tidak mengirimkan pesan duplikat
  - [x] Embed Discord tantangan harian menyajikan kartu interaktif dengan tema harian (Word of the Day, Grammar Quick Fix, Story Prompt, Sanctuary Room, News Dispatch, Dynamic Syntax, Weekly Reflection), panduan, dan tombol aksi "Accept Challenge"
  - [x] Scheduler berhenti dengan bersih (*clean exit*) saat sinyal shutdown (SIGTERM/SIGINT) diterima tanpa menyisakan *dangling goroutines*
  - [x] Unit testing 100% PASS terhadap kalkulasi waktu tunda scheduler, penanganan pembatalan konteks, idempotensi, embed presentation, dan pengiriman pesan Discord

---

## Fase 5: Hardening & Bug Remediation (Audit Hunt `hunt-001-2026-11-08`)

> **Tujuan:** Remediasi seluruh 10 temuan bug dan celah ketahanan sistem dari sesi eksplorasi kualitas [`hunt-001-2026-11-08.md`](../hunt/hunt-001-2026-11-08.md) (Kehilangan Data -> Keamanan -> Alur Bisnis Patah -> Kosmetik/UI).

### Backlog Fase 5

| # | Judul | Epoch Timestamp | Status | Pasangan |
|---|-------|-----------------|--------|---------|
| 5.1 | [BUG] Progression Loop Terputus (XP, Streak & Achievement Tidak Tercatat Saat Game Selesai) | `1762520000` | `[COMPLETED]` | `TICKET-1762520000` / `PLAN-1762520000` |
| 5.2 | [BUG] Inkonsistensi Format User Primary Key (`usr-<discordID>` vs `<discordID>`) Memicu Foreign Key Error | `1762520060` | `[COMPLETED]` | `TICKET-1762520060` / `PLAN-1762520060` |
| 5.3 | [BUG] Evaluasi Streak Harian Bertambah Meski Pengguna Melewatkan 1 Hari Kalender Penuh | `1762520120` | `[COMPLETED]` | `TICKET-1762520120` / `PLAN-1762520120` |
| 5.4 | [BUG] Hardcoded Fallback Secret `'admin:admin'` pada HTMX Admin Dashboard | `1762520180` | `[COMPLETED]` | `TICKET-1762520180` / `PLAN-1762520180` |
| 5.5 | [BUG] Pembajakan Giliran oleh Penonton (Spectator Turn Hijack) pada Tombol Would You Rather | `1762520240` | `[COMPLETED]` | `TICKET-1762520240` / `PLAN-1762520240` |
| 5.6 | [BUG] `TimeoutTicker` Tidak Pernah Dijalankan di `main.go` (Channel Discord Terkunci Permanen) | `1762520300` | `[COMPLETED]` | `TICKET-1762520300` / `PLAN-1762520300` |
| 5.7 | [BUG] Lobby Multiplayer Memulai Game 2-Pemain dengan 1 Pemain Saat Host Menjalankan Command Kedua Kali | `1762520360` | `[COMPLETED]` | `TICKET-1762520360` / `PLAN-1762520360` |
| 5.8 | [BUG] Preferensi Difficulty Pengguna (`/radovid settings`) Diabaikan di Seluruh Mini-Game | `1762520420` | `[COMPLETED]` | `TICKET-1762520420` / `PLAN-1762520420` |
| 5.9 | [BUG] Global Mutex Freeze Saat Request HTTP LLM & Kebocoran Memori Status Game (`uc.states`) | `1762520480` | `[COMPLETED]` | `TICKET-1762520480` / `PLAN-1762520480` |
| 5.10 | [BUG] Pemotongan Byte Mentah UTF-8 pada Template Embed & Clock Drift pada Daily Scheduler | `1762520540` | `[COMPLETED]` | `TICKET-1762520540` / `PLAN-1762520540` |

---

## Catatan Pengerjaan

> **[MODE 1 — Autonomous Planning]** Setiap item di atas harus dieksekusi sebagai pasangan `TICKET-[EPOCH]` + `PLAN-[EPOCH]`. Buat pasangan tersebut sebelum memulai implementasi, paparkan rencana ke developer, dan minta persetujuan eksplisit sebelum mutasi state apapun.

**Aturan Centang Status:**
- `[CREATED]` — belum dimulai
- `[~] In Progress` — sedang dikerjakan (sertakan epoch sesi aktif)
- `[x] Done` — selesai, delivery gate lulus, tiket & rencana ditandai Done/Completed
