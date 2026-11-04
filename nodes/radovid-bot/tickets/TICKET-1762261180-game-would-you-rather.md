---
id: TICKET-1762261180
plan_id: PLAN-1762261180
title: "Game 1: Would You Rather?"
status: Todo
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
- [ ] Command `/radovid play would-you-rather` memulai sesi permainan atau masuk ke antrean lobi aktif.
- [ ] AI Game Master menghasilkan pertanyaan dilema dengan dua pilihan (Option A dan Option B) murni dalam bahasa Inggris.
- [ ] Antarmuka Discord menampilkan Discord Embed pertanyaan beserta dua Discord Interactive Buttons (Komponen Tipe 2) untuk memilih Opsi A atau Opsi B.
- [ ] Bot mencatat pilihan dari pemain yang terdaftar dalam sesi dan mencegah vote ganda pada ronde yang sama.
- [ ] Setelah pemain memberikan pilihan, AI Game Master memberikan komentar follow-up berbahasa Inggris terhadap pilihan pemain.
- [ ] Sesi permainan berjalan minimal 3 ronde berturut-turut sebelum menampilkan pilihan "Lanjut Ronde Berikutnya" atau "Selesai".
- [ ] Menampilkan ringkasan hasil akhir game berisi komparasi pilihan kedua pemain beserta ulasan karakter AI Game Master.
- [ ] Migrasi database `004_create_game_rounds.sql` untuk mencatat riwayat prompt, pilihan, dan status ronde.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/004_create_game_rounds.sql`
- `../../../../radovid-bot-api/internal/domain/game/wyr_entity.go`
- `../../../../radovid-bot-api/internal/domain/game/round_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/round_repository.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Pending execution.
- **Ringkasan File Terpengaruh:**
  - Pending execution.
- **Catatan & Keputusan Arsitektural:**
  - Pending execution.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v ./internal/domain/game/... ./internal/usecase/game/...`
- **Output Terminal:**
  ```text
  Pending execution.
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** Pending
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** Pending
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): Pending
  - O-08 (Edge Case Atlas): Pending
  - O-09 (Damage Assessment): Pending
  - O-10 (Full Suite Metrics): Pending
  - O-11 (Real Thing Driven): Pending
  - O-12 (Shipping Env Check): Pending
  - Hunt Loop Final Round: Pending
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Pending
  - CHANGELOG.md: Pending
  - Commit Message Sanity: Pending
  - O-16 Untested List: Pending
