---
id: TICKET-1762261180
plan_id: PLAN-1762261180
title: "Game 1: Would You Rather?"
status: Done
priority: High
labels: [Backend, Game, Feature, Discord, Ollama]
---

# Deskripsi
Sebagai pemain Discord di server komunitas bahasa Inggris,
Saya ingin memainkan game interaktif "Would You Rather?" yang dipandu oleh AI Game Master dengan tombol Discord dan komentar follow-up berbahasa Inggris,
Sehingga saya dan rekan belajar dapat melatih kemampuan berpikir kritis, mengekspresikan pilihan, dan memahami konteks percakapan dalam bahasa Inggris secara menyenangkan.

Latar belakang teknis: Game 1 ("Would You Rather?") adalah aktivitas pertama dalam ekosistem Radovid Bot. Game ini memvalidasi end-to-end loop: slash command trigger (`/radovid play would-you-rather`), pembuatan state sesi, pemanggilan AI Game Master untuk menghasilkan dua opsi dilematis berbahasa Inggris, interaksi tombol Discord (Component Buttons), pencatatan pilihan multi-pemain, komentar follow-up AI, serta progres minimal 3 ronde hingga layar ringkasan hasil.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261180-game-would-you-rather.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Command `/radovid play would-you-rather` memulai sesi permainan atau masuk ke antrean lobi aktif.
- [x] AI Game Master menghasilkan pertanyaan dilema dengan dua pilihan (Option A dan Option B) murni dalam bahasa Inggris.
- [x] Antarmuka Discord menampilkan Discord Embed pertanyaan beserta dua Discord Interactive Buttons (Komponen Tipe 2) untuk memilih Opsi A atau Opsi B.
- [x] Bot mencatat pilihan dari pemain yang terdaftar dalam sesi dan mencegah vote ganda pada ronde yang sama.
- [x] Setelah pemain memberikan pilihan, AI Game Master memberikan komentar follow-up berbahasa Inggris terhadap pilihan pemain.
- [x] Sesi permainan berjalan minimal 3 ronde berturut-turut sebelum menampilkan pilihan "Lanjut Ronde Berikutnya" atau "Selesai".
- [x] Menampilkan ringkasan hasil akhir game berisi komparasi pilihan kedua pemain beserta ulasan karakter AI Game Master.
- [x] Migrasi database `004_create_game_rounds.sql` untuk mencatat riwayat prompt, pilihan, dan status ronde.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/004_create_game_rounds.sql`
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/wyr_entity.go`
- `../../../../radovid-bot-api/internal/domain/game/wyr_entity_test.go`
- `../../../../radovid-bot-api/internal/domain/game/round_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/round_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/round_repository_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/router.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Membuat migrasi database `migrations/004_create_game_rounds.sql` yang mendefinisikan tabel `game_rounds` dan `round_votes` lengkap dengan relasi foreign key dan index.
  2. Menambahkan `GameTypeWouldYouRather` pada `internal/domain/game/entity.go`.
  3. Mengimplementasikan domain model `WYRRound`, `PlayerVote`, `WYRChoice` ("A" / "B"), `WYRSummary`, dan sentinel errors di `internal/domain/game/wyr_entity.go` beserta unit test di `wyr_entity_test.go`.
  4. Mendefinisikan interface `RoundRepository` di `internal/domain/game/round_repository.go` dan implementasi konkret SQLite di `internal/infrastructure/database/round_repository.go` dengan query terparameterisasi dan atomic transaction di `round_repository_test.go`.
  5. Mengimplementasikan `WouldYouRatherUseCase` di `internal/usecase/game/would_you_rather.go` mencakup `StartRound`, `SubmitVote`, `generateCommentary`, `GetSessionSummary`, dan `CompleteSession` dengan unit test komprehensif di `would_you_rather_test.go`.
  6. Mengintegrasikan Discord delivery layer: slash command definition `/radovid play would-you-rather` di `command_def.go`, message component button router di `router.go`, command handler di `command_play_wyr.go`, button click handler di `interaction_wyr_button.go`, bridge `GameHandler` di `game_handler.go`, dan wiring dependency injection di `cmd/bot/main.go`.
  7. Memverifikasi seluruh pengujian unit, formatting, vet, compilation build, dan Zero-Comment Policy.
- **Ringkasan File Terpengaruh:**
  - `migrations/004_create_game_rounds.sql`
  - `internal/domain/game/entity.go`
  - `internal/domain/game/wyr_entity.go` & `wyr_entity_test.go`
  - `internal/domain/game/round_repository.go`
  - `internal/infrastructure/database/round_repository.go` & `round_repository_test.go`
  - `internal/usecase/game/would_you_rather.go` & `would_you_rather_test.go`
  - `internal/delivery/discord/command_def.go`
  - `internal/delivery/discord/router.go`
  - `internal/delivery/discord/handler.go`
  - `internal/delivery/discord/game_handler.go`
  - `internal/delivery/discord/command_play_wyr.go` & `command_play_wyr_test.go`
  - `internal/delivery/discord/interaction_wyr_button.go` & `interaction_wyr_button_test.go`
  - `cmd/bot/main.go`
- **Catatan & Keputusan Arsitektural:**
  - Penyimpanan vote dinormalisasi ke tabel `round_votes` dengan primary key `(round_id, user_id)` untuk memastikan atomisitas dan mencegah vote ganda pada level database selain dari proteksi use case.
  - Button interactive discord menggunakan custom ID berstruktur `wyr:vote:<session_id>:<round_number>:<option>`, `wyr:next:<session_id>:<round_number>`, dan `wyr:finish:<session_id>` yang diproses secara modular melalui `RegisterComponentPrefix`.
  - Zero-Comment Policy dipatuhi 100% pada seluruh berkas Go dan SQL baru/terubah.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -count=1 ./internal/domain/game/... ./internal/usecase/game/... ./internal/delivery/discord/... ./internal/infrastructure/database/...`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	0.159s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	0.254s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.228s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	0.550s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Terpenuhi (Zero-Comment Policy lolos, parameterisasi SQL aman, validasi fail-fast pada domain/usecase/delivery).
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Terpenuhi (100% test pass, go vet bersih, CGO_ENABLED=0 go build sukses).
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Terpenuhi
  - O-08 (Edge Case Atlas): Terpenuhi (non-participant vote, double vote, finished round, session not active)
  - O-09 (Damage Assessment): Nol breaking change pada command lama (/radovid help & start)
  - O-10 (Full Suite Metrics): 100% pass di seluruh package
  - O-11 (Real Thing Driven): Teruji dengan real SQLite in-memory dan mock interaction
  - O-12 (Shipping Env Check): Pure Go build tanpa CGO
  - Hunt Loop Final Round: Bersih
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Terverifikasi (.graphify tidak ditemukan di root)
  - CHANGELOG.md: Terupdate
  - Commit Message Sanity: Menunggu instruksi commit eksplisit pengguna
  - O-16 Untested List: Tidak ada komponen inti yang tidak teruji
