---
id: TICKET-1762261360
plan_id: PLAN-1762261360
title: Discord Embed Templates
status: Todo
priority: High
labels: [Frontend, UI, Discord, Architecture, Helpers]
---

# Deskripsi
Sebagai pengembang fitur bot dan pemain Radovid Bot di Discord,
Saya ingin memiliki komponen reusable `EmbedBuilder` dan pustaka template embed standar untuk antarmuka game (session start, round display, result summary, dan error ephemeral) dengan palet warna yang konsisten,
Sehingga seluruh tampilan permainan dan interaksi visual bot memiliki estetika terpadu, ramah pengguna, bebas dari duplikasi kode pembangunan pesan Discord, serta mematuhi batasan karakter Discord API.

Latar belakang teknis: Tampilan visual bot pada Discord dibangun melalui struktur `discordgo.MessageEmbed`. Tanpa builder helper terpusat dengan fluent interface dan palet warna standar, pembangunan embed di berbagai command akan menghasilkan inkonsistensi visual, duplikasi logika boilerplate, dan risiko runtime error akibat pelanggaran batasan karakter Discord (seperti title > 256 karakter atau description > 4096 karakter).

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261360-discord-embed-templates.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Tersedia struct `EmbedBuilder` dengan pola method chaining (fluent interface) untuk membangun objek `discordgo.MessageEmbed`.
- [ ] Definisi konstanta palet warna baku: Info (Blue), Success (Green), Error (Red), Warning (Yellow), dan AI Game Master (Purple).
- [ ] Template fungsi siap pakai untuk skenario UI game inti:
  - `SessionStartEmbed`: Tampilan pembukaan lobi / sesi permainan baru.
  - `RoundDisplayEmbed`: Tampilan ronde aktif dengan nomor ronde, instruksi, dan status progres.
  - `ResultSummaryEmbed`: Tampilan papan skor, pemenang, dan evaluasi ringkasan AI Game Master.
  - `ErrorEphemeralEmbed`: Format standar pesan galat yang dikirimkan secara ephemeral.
- [ ] Pemangkasan otomatis (*safe truncation*) terhadap batas maksimum karakter Discord API (Title max 256, Description max 4096, Field Value max 1024, Total karakter max 6000) untuk mencegah penolakan payload dari Discord Gateway.
- [ ] Unit test komprehensif untuk pengujian method chaining, batas karakter ekstrem, dan integritas template fungsi.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/discord/palette.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/embeds.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/embeds_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`

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
- **Perintah Pengujian:** `go test -v ./internal/infrastructure/discord/...`
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
