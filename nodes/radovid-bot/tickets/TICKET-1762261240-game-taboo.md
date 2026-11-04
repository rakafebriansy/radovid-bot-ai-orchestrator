---
id: TICKET-1762261240
plan_id: PLAN-1762261240
title: "Game 2: Taboo"
status: Todo
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
- [ ] AI Game Master menghasilkan 1 kata target (*target word*) dan 5 kata terlarang (*taboo words*) dalam bahasa Inggris sesuai level kesulitan sesi.
- [ ] Informasi kata target dan daftar kata terlarang dikirimkan secara rahasia kepada pemain yang sedang bertugas mendeskripsikan (*describer*) melalui pesan ephemeral.
- [ ] Pemain mengirimkan teks deskripsi menggunakan form input teks Discord Modal (TextInput Component, Paragraph style).
- [ ] Sistem evaluator memeriksa deskripsi melalui 2 lapis pengujian:
  1. Pemeriksaan deterministik: Deteksi kata terlarang berbasis word boundary regex / tokenization (case-insensitive).
  2. Evaluasi semantik AI: Menilai apakah deskripsi akurat dan cukup informatif untuk merepresentasikan kata target.
- [ ] Sistem penilaian (scoring system): Pemain memperoleh skor jika tidak melanggar kata terlarang dan akurasi deskripsi memenuhi standar kelulusan.
- [ ] Pergantian giliran (*turn-based switching*) yang tertib antar kedua pemain dalam sesi permainan.
- [ ] Discord Embed menampilkan ringkasan ronde: apakah ada pelanggaran kata terlarang, skor yang didapat, dan saran perbaikan kalimat bahasa Inggris dari AI.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/taboo_entity.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`

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
