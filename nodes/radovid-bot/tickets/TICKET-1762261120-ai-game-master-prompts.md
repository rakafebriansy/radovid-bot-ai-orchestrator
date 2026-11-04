---
id: TICKET-1762261120
plan_id: PLAN-1762261120
title: AI Game Master Prompt System
status: Todo
priority: High
labels: [Backend, AI, Domain, Ollama, Prompts]
---

# Deskripsi
Sebagai sistem orkestrasi game Radovid Bot,
Saya ingin memiliki arsitektur AI Game Master prompt system berbasis `//go:embed` dengan domain model terstandarisasi, parser output yang toleran terhadap format LLM, dan kurasi fallback static questions,
Sehingga game dapat menghasilkan skenario dan pertanyaan bahasa Inggris berkualitas tinggi, parsing data berjalan andal, serta permainan tetap berjalan mulus meskipun Ollama mengalami kegagalan atau timeout.

Latar belakang teknis: Interaksi antara LLM dan sistem game memerlukan prompt engineering yang disiplin dan deterministik. File template prompt perlu dikompilasi langsung ke dalam binary Go (`//go:embed`) agar tidak bergantung pada relative path filesystem saat deployment. Selain itu, respons teks dari LLM lokal terkadang mengandung variasi format (markdown bold, awalan angka, atau spasi berlebih) yang membutuhkan parser robust, serta fallback preset statis berkualitas ketika LLM tidak responsif.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261120-ai-game-master-prompts.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Definisi domain struct `GameMasterRequest` dan `GameMasterResponse` pada layer domain (`internal/domain/gamemaster`).
- [ ] File prompt template ter-embed ke dalam binary menggunakan `//go:embed` pada `internal/infrastructure/ollama/prompts`.
- [ ] Template engine mendukung injeksi variabel dinamis: `{{.GameType}}`, `{{.Difficulty}}`, `{{.Context}}`, `{{.RoundNumber}}`.
- [ ] Output parser yang tangguh: mampu mengekstrak pertanyaan, opsi, kata target, dan kata terlarang dari output mentah Ollama meskipun terdapat variasi markdown formatting (`**`, `##`, list item).
- [ ] Katalog bank soal cadangan (*static fallback questions*) per jenis game dan tingkat kesulitan yang aktif otomatis jika Ollama down atau parsing gagal.
- [ ] Penanda eksplisit `IsFallback: true` pada response jika menggunakan bank soal cadangan.
- [ ] Unit test komprehensif untuk perenderan template prompt, ketahanan parser terhadap respons anomali, dan aktivasi fallback.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/gamemaster/entity.go`
- `../../../../radovid-bot-api/internal/domain/gamemaster/repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/would_you_rather.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/taboo.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/twenty_questions.tmpl`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/game_master.go`
- `../../../../radovid-bot-api/internal/usecase/game/game_master_test.go`

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
- **Perintah Pengujian:** `go test -v ./internal/domain/gamemaster/... ./internal/infrastructure/ollama/... ./internal/usecase/game/...`
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
