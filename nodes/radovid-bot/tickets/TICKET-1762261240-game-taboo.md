---
id: TICKET-1762261240
plan_id: PLAN-1762261240
title: "Game 2: Taboo"
status: Done
priority: High
labels: [Backend, Game, Feature, Discord, NLP, Ollama]
---

# Deskripsi
Sebagai pemain yang ingin memperkaya kosakata dan ketangkasan berbahasa Inggris,
Saya ingin memainkan game "Taboo" di mana AI Game Master menentukan satu kata target beserta 5 kata terlarang yang tidak boleh digunakan dalam deskripsi, dan saya mengirimkan deskripsi melalui Discord Modal,
Sehingga saya terlatih menggunakan sinonim, parafrasa, dan struktur kalimat alternatif dalam bahasa Inggris tanpa bergantung pada kata-kata klise.

Latar belakang teknis: Game 2 ("Taboo") menguji pemahaman leksikal dan kemampuan parafrase pemain. Sistem memerlukan generator target word dan 5 taboo words dari AI Game Master, penyampaian kata rahasia secara aman (ephemeral message) kepada describer, form input teks paragraf via Discord Modal interaction, sistem evaluasi ganda (deteksi kata terlarang deterministik + evaluasi semantik AI terhadap kualitas deskripsi), dan pencatatan skor per giliran.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261240-game-taboo.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [x] AI Game Master menghasilkan 1 kata target (*target word*) dan 5 kata terlarang (*taboo words*) dalam bahasa Inggris sesuai level kesulitan sesi.
- [x] Informasi kata target dan daftar kata terlarang dikirimkan secara rahasia kepada pemain yang sedang bertugas mendeskripsikan (*describer*) melalui pesan ephemeral.
- [x] Pemain mengirimkan teks deskripsi menggunakan form input teks Discord Modal (TextInput Component, Paragraph style).
- [x] Sistem evaluator memeriksa deskripsi melalui 2 lapis pengujian:
  1. Pemeriksaan deterministik: Deteksi kata terlarang berbasis word boundary regex / tokenization (case-insensitive).
  2. Evaluasi semantik AI: Menilai apakah deskripsi akurat dan cukup informatif untuk merepresentasikan kata target.
- [x] Sistem penilaian (scoring system): Pemain memperoleh skor jika tidak melanggar kata terlarang dan akurasi deskripsi memenuhi standar kelulusan.
- [x] Pergantian giliran (*turn-based switching*) yang tertib antar kedua pemain dalam sesi permainan.
- [x] Discord Embed menampilkan ringkasan ronde: apakah ada pelanggaran kata terlarang, skor yang didapat, dan saran perbaikan kalimat bahasa Inggris dari AI.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/entity.go`
- `../../../../radovid-bot-api/internal/domain/game/taboo_entity.go`
- `../../../../radovid-bot-api/internal/domain/game/taboo_entity_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/router.go`
- `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Menambahkan tipe permainan `GameTypeTaboo = "taboo"` pada `internal/domain/game/entity.go`.
  2. Mengimplementasikan domain model Taboo: `TabooCard`, `TabooEvaluation`, `TabooTurn`, `TabooSessionState`, `TabooSummary`, dan sentinel errors di `internal/domain/game/taboo_entity.go` beserta unit test di `taboo_entity_test.go`.
  3. Mengimplementasikan `TabooEvaluator` dua lapis (deteksi kata terlarang berbasis word boundary regex case-insensitive + evaluasi akurasi semantik AI dengan graceful fallback) di `internal/usecase/game/taboo_evaluator.go` beserta unit test di `taboo_evaluator_test.go`.
  4. Mengimplementasikan core use case `TabooUseCase` dengan turn-based player switching, isolasi kartu rahasia per pemain, scoring, dan ringkasan sesi di `internal/usecase/game/taboo.go` beserta unit test di `taboo_test.go`.
  5. Menambahkan subcommand `/radovid play taboo` pada `command_def.go` dan router Discord Modal (`RegisterModalPrefix`) pada `router.go`.
  6. Mengimplementasikan delivery adapter `command_play_taboo.go` (lobby/start dan embed kartu/ronde) serta `interaction_taboo_modal.go` (modal input form paragraph, button kart rahasia ephemeral, dan evaluasi hasil ronde).
  7. Menghubungkan seluruh dependensi baru pada `cmd/bot/main.go` dan memverifikasi unit test 100% lulus tanpa komentar kode (Zero-Comment Policy).
- **Ringkasan File Terpengaruh:**
  - `internal/domain/game/entity.go`
  - `internal/domain/game/taboo_entity.go` & `taboo_entity_test.go`
  - `internal/usecase/game/taboo_evaluator.go` & `taboo_evaluator_test.go`
  - `internal/usecase/game/taboo.go` & `taboo_test.go`
  - `internal/delivery/discord/command_def.go`
  - `internal/delivery/discord/router.go`
  - `internal/delivery/discord/game_handler.go`
  - `internal/delivery/discord/command_play_taboo.go` & `command_play_taboo_test.go`
  - `internal/delivery/discord/interaction_taboo_modal.go` & `interaction_taboo_modal_test.go`
  - `cmd/bot/main.go`
- **Catatan & Keputusan Arsitektural:**
  - Kartu rahasia (target word dan taboo words) hanya dapat diakses melalui tombol ephemeral interaction khusus describer, sehingga pemain lain tidak dapat melihat bocoran kata.
  - Regex deteksi kata terlarang menggunakan boundary `\b<word>\b` dengan `regexp.QuoteMeta` untuk mencegah false positive pada substring kata lain (contoh: "stone" tidak memicu pelanggaran pada "milestone").
  - Form deskripsi menggunakan Discord Modal tipe TextInput Paragraph untuk pengalaman pengetikan kalimat panjang yang nyaman di desktop maupun mobile.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v ./internal/domain/game/... ./internal/usecase/game/... ./internal/delivery/discord/...`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	0.558s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	0.382s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.575s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Terpenuhi (Zero-Comment Policy lolos, validasi input fail-fast, sanitasi regex).
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Terpenuhi (100% test pass, go vet bersih, build Go CGO_ENABLED=0 sukses).
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Terpenuhi
  - O-08 (Edge Case Atlas): Terpenuhi (non-describer click, whitespace description, substring false positives, modal submit handling)
  - O-09 (Damage Assessment): Nol dampak regresi pada Game 1 (Would You Rather)
  - O-10 (Full Suite Metrics): 100% pass di seluruh paket
  - O-11 (Real Thing Driven): Teruji integrasi komponen discordgo mock & in-memory state
  - O-12 (Shipping Env Check): Pure Go build
  - Hunt Loop Final Round: Bersih
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Terverifikasi (.graphify tidak ditemukan di root)
  - CHANGELOG.md: Terupdate
  - Commit Message Sanity: Menunggu instruksi commit eksplisit pengguna
  - O-16 Untested List: Seluruh komponen baru teruji secara komprehensif
