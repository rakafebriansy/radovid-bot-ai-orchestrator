---
id: TICKET-1762261120
plan_id: PLAN-1762261120
title: AI Game Master Prompt System
status: Done
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
- [x] Definisi domain struct `GameMasterRequest` dan `GameMasterResponse` pada layer domain (`internal/domain/gamemaster`).
- [x] File prompt template ter-embed ke dalam binary menggunakan `//go:embed` pada `internal/infrastructure/ollama/prompts`.
- [x] Template engine mendukung injeksi variabel dinamis: `{{.GameType}}`, `{{.Difficulty}}`, `{{.Context}}`, `{{.RoundNumber}}`.
- [x] Output parser yang tangguh: mampu mengekstrak pertanyaan, opsi, kata target, dan kata terlarang dari output mentah Ollama meskipun terdapat variasi markdown formatting (`**`, `##`, list item).
- [x] Katalog bank soal cadangan (*static fallback questions*) per jenis game dan tingkat kesulitan yang aktif otomatis jika Ollama down atau parsing gagal.
- [x] Penanda eksplisit `IsFallback: true` pada response jika menggunakan bank soal cadangan.
- [x] Unit test komprehensif untuk perenderan template prompt, ketahanan parser terhadap respons anomali, dan aktivasi fallback.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/gamemaster/entity.go`
- `../../../../radovid-bot-api/internal/domain/gamemaster/repository.go`
- `../../../../radovid-bot-api/internal/domain/gamemaster/entity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go`
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
  1. Membuat domain model `GameMasterRequest`, `GameMasterResponse`, dan enum `Difficulty` beserta validasi `Validate()` pada `internal/domain/gamemaster/entity.go` dan antarmuka `LLMClient` & `GameMasterService` pada `repository.go`.
  2. Menyusun template prompt disiplin (`would_you_rather.tmpl`, `taboo.tmpl`, `twenty_questions.tmpl`) dan perender berbasis embed FS `internal/infrastructure/ollama/prompts/embed.go`.
  3. Mengimplementasikan parser teks toleran markdown `internal/infrastructure/ollama/parser.go` yang mengekstrak pertanyaan, opsi A/B, secret word, dan taboo words.
  4. Menyediakan katalog kurasi bank soal statis `internal/infrastructure/ollama/fallback.go` yang mencakup 3 jenis game untuk level Easy, Medium, dan Hard dengan flag otomatis `IsFallback: true`.
  5. Mengimplementasikan use case `internal/usecase/game/game_master.go` yang mengorkestrasi perenderan prompt, inferensi Ollama, parsing, dan aktivasi graceful fallback ketika LLM offline/gagal.
  6. Menguji menyeluruh di layer domain, infrastructure, dan usecase dengan 100% test pass.
- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/internal/domain/gamemaster/entity.go` (Created)
  - `radovid-bot-api/internal/domain/gamemaster/repository.go` (Created)
  - `radovid-bot-api/internal/domain/gamemaster/entity_test.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/prompts/embed.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/prompts/would_you_rather.tmpl` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/prompts/taboo.tmpl` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/prompts/twenty_questions.tmpl` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/parser.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/parser_test.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/fallback.go` (Created)
  - `radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` (Created)
  - `radovid-bot-api/internal/usecase/game/game_master.go` (Created)
  - `radovid-bot-api/internal/usecase/game/game_master_test.go` (Created)
- **Catatan & Keputusan Arsitektural:**
  - Pemanfaatan `//go:embed` menjaga biner mandiri tanpa dependensi filesystem eksternal saat dideploy di Docker container.
  - Arsitektur graceful degradation menjamin gameplay Discord tidak pernah macet (*zero interruption*) meskipun Ollama lokal mengalami lonjakan beban atau kegagalan koneksi.
  - Zero-Comment Policy diterapkan secara konsisten di seluruh kode Go baru.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -v ./internal/domain/gamemaster/... ./internal/infrastructure/ollama/... ./internal/usecase/game/...`
- **Output Terminal:**
  ```text
  === RUN   TestDifficultyIsValid
  --- PASS: TestDifficultyIsValid (0.00s)
  === RUN   TestGameMasterRequestValidate
  --- PASS: TestGameMasterRequestValidate (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	0.509s
  === RUN   TestNewClientValidation
  --- PASS: TestNewClientValidation (0.00s)
  === RUN   TestGenerateSuccess
  --- PASS: TestGenerateSuccess (0.00s)
  === RUN   TestGetFallbackResponse
  --- PASS: TestGetFallbackResponse (0.00s)
  === RUN   TestParseGameMasterOutput_Validation
  --- PASS: TestParseGameMasterOutput_Validation (0.00s)
  === RUN   TestParseWouldYouRather
  --- PASS: TestParseWouldYouRather (0.00s)
  === RUN   TestParseTaboo
  --- PASS: TestParseTaboo (0.00s)
  === RUN   TestParseTwentyQuestions
  --- PASS: TestParseTwentyQuestions (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	1.122s
  === RUN   TestRenderPrompt
  --- PASS: TestRenderPrompt (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	1.847s
  === RUN   TestNewGameMaster
  --- PASS: TestNewGameMaster (0.00s)
  === RUN   TestGenerateGameContent_Validation
  --- PASS: TestGenerateGameContent_Validation (0.00s)
  === RUN   TestGenerateGameContent_Success
  --- PASS: TestGenerateGameContent_Success (0.00s)
  === RUN   TestGenerateGameContent_LLMErrorFallback
  --- PASS: TestGenerateGameContent_LLMErrorFallback (0.00s)
  === RUN   TestGenerateGameContent_UnparseableOutputFallback
  --- PASS: TestGenerateGameContent_UnparseableOutputFallback (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	1.307s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASS (Clean Architecture, Zero-Comment Policy, Pure Go).
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASS (`go test -count=1 ./...` 100% pass, `go vet` clean, `CGO_ENABLED=0 go build ./...` clean).
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS (Validasi request fail-fast dan parser menolak format tak valid).
  - O-08 (Edge Case Atlas): PASS (Markdown bold/italics tokens, missing options, fallback deterministic cycling).
  - O-09 (Damage Assessment): PASS (Tidak ada dampak regresi ke package lain).
  - O-10 (Full Suite Metrics): PASS (Seluruh package proyek lulus tes).
  - O-11 (Real Thing Driven): PASS (Template parsing nyata menggunakan embed FS).
  - O-12 (Shipping Env Check): PASS (Biner Go CGO_ENABLED=0 terkompilasi bersih).
  - Hunt Loop Final Round: PASS (Semua kasus kegagalan LLM memicu fallback dengan aman).
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: Terverifikasi (direktori `.graphify` belum diinisialisasi).
  - CHANGELOG.md: Updated.
  - Commit Message Sanity: Sesuai konvensi tiket.
  - O-16 Untested List: Nihil (seluruh fungsi teruji dengan unit test).
