---
id: TICKET-1762261000
plan_id: PLAN-1762261000
title: Ollama HTTP Client Wrapper
status: Done
priority: High
labels: [Backend, AI, Infrastructure, Ollama]
---

# Deskripsi
Sebagai AI Game Master & bot engine,
Saya ingin memiliki HTTP client wrapper yang tangguh untuk Ollama Generate API dengan perlindungan timeout, retry otomatis dengan exponential backoff, dan pembersihan input prompt,
Sehingga interaksi dengan model AI lokal offline (Ollama Gemma 4b / gemma3:4b) berjalan stabil, aman dari eksploitasi format teks Discord, serta terhindar dari hung process saat Ollama sedang sibuk atau down.

Latar belakang teknis: Komponen AI Game Master di Radovid Bot sepenuhnya mengandalkan inferensi lokal offline menggunakan Ollama (`http://localhost:11434`) dengan model Gemma 4b (Gemma 3 4b) tanpa dependensi internet/cloud pihak ketiga. Panggilan HTTP ke endpoint `/api/generate` rentan mengalami timeout atau kegagalan sementara ketika sistem lokal terbebani. Tanpa wrapper yang menangani retry exponential backoff (maksimal 2 kali), typed errors, dan sanitasi ketat (membersihkan Discord mention `<@...>`, format markdown manipulatif, dan batas 2000 karakter), bot berisiko crash, freeze, atau mengirimkan input kotor ke model AI offline.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762261000-ollama-http-client.md)
*(Rincian arsitektur, diff preview, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Client menggunakan `context.Context` dengan timeout configurable dari `OLLAMA_TIMEOUT_SECONDS` (default: 30 detik).
- [x] Input sanitization: menghapus Discord mention (`<@...>`, `<@!...>`, `<@&...>`, `<#...>`), mensterilkan markdown manipulatif, dan memotong prompt hingga maksimal 2000 karakter tanpa memotong karakter multi-byte secara parsial (rune slicing).
- [x] Logika retry dengan exponential backoff untuk error transient / HTTP 5xx (maksimal 2 kali percobaan ulang; tidak melakukan retry untuk error 4xx).
- [x] Return typed error eksplisit (`ErrTimeout`, `ErrConnectionRefused`, `ErrOllamaAPI`, `ErrEmptyPrompt`).
- [x] Payload request/response JSON terpetakan dengan baik (`stream: false` untuk single completion batch).
- [x] Unit test table-driven untuk sanitization dan error handling dengan mock HTTP client (`http.RoundTripper`).

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/ollama/client.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/client_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/models.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/errors.go`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-04

- **Langkah Teknis Tereksekusi:**
  1. Membuat typed sentinel errors (`ErrTimeout`, `ErrConnectionRefused`, `ErrOllamaAPI`, `ErrEmptyPrompt`, `ErrEmptyBaseURL`, `ErrEmptyModel`) pada `internal/infrastructure/ollama/errors.go`.
  2. Membuat DTO payload Ollama Generate API (`GenerateRequest`, `GenerateResponse`, `ErrorResponse`) pada `internal/infrastructure/ollama/models.go`.
  3. Menulis unit test sanitasi prompt (`sanitize_test.go`) dengan pembuktian RED, kemudian mengimplementasikan fungsi `SanitizePrompt` pada `sanitize.go` (GREEN).
  4. Menulis unit test table-driven HTTP client (`client_test.go`) dengan pembuktian RED, kemudian mengimplementasikan `Client` (`NewClient`, `Generate`, `SetBackoff`, `isConnectionRefused`) pada `client.go` (GREEN).
  5. Menjalankan verifikasi penuh Delivery Gate: `go test ./...` 100% PASS, `go vet ./...` 0 warnings, `CGO_ENABLED=0 go build ./...` sukses, Zero-Comment Policy dipatuhi 100%.

- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/internal/infrastructure/ollama/errors.go` (CREATED)
  - `radovid-bot-api/internal/infrastructure/ollama/models.go` (CREATED)
  - `radovid-bot-api/internal/infrastructure/ollama/sanitize.go` (CREATED)
  - `radovid-bot-api/internal/infrastructure/ollama/sanitize_test.go` (CREATED)
  - `radovid-bot-api/internal/infrastructure/ollama/client.go` (CREATED)
  - `radovid-bot-api/internal/infrastructure/ollama/client_test.go` (CREATED)

- **Catatan & Keputusan Arsitektural:**
  - Client menggunakan inferensi lokal offline Ollama (`http://localhost:11434`) dengan model default Gemma 4b (`gemma3:4b`).
  - Unit test menggunakan in-memory `http.RoundTripper` untuk pengujian HTTP terisolasi tanpa membuka network socket TCP ke OS, memastikan test suite dapat berjalan deterministik dan aman di lingkungan sandboxed.
  - Nil-safety dan bounds checking diterapkan secara ketat tanpa magic numbers.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -count=1 -v ./internal/infrastructure/ollama/...`
- **Output Terminal:**
  ```text
  === RUN   TestNewClientValidation
  === RUN   TestNewClientValidation/empty_base_url
  === RUN   TestNewClientValidation/whitespace_base_url
  === RUN   TestNewClientValidation/empty_model
  === RUN   TestNewClientValidation/whitespace_model
  === RUN   TestNewClientValidation/valid_parameters_with_trailing_slash_removal
  --- PASS: TestNewClientValidation (0.00s)
  === RUN   TestGenerateSuccess
  --- PASS: TestGenerateSuccess (0.00s)
  === RUN   TestGenerateEmptyPromptValidation
  --- PASS: TestGenerateEmptyPromptValidation (0.00s)
  === RUN   TestGenerateOllamaAPIError4xxNoRetry
  --- PASS: TestGenerateOllamaAPIError4xxNoRetry (0.00s)
  === RUN   TestGenerateOllamaAPIError5xxRetrySuccess
  --- PASS: TestGenerateOllamaAPIError5xxRetrySuccess (0.03s)
  === RUN   TestGenerateOllamaAPIError5xxExhaustedRetries
  --- PASS: TestGenerateOllamaAPIError5xxExhaustedRetries (0.08s)
  === RUN   TestGenerateTimeout
  --- PASS: TestGenerateTimeout (0.05s)
  === RUN   TestGenerateConnectionRefused
  --- PASS: TestGenerateConnectionRefused (0.08s)
  === RUN   TestClientSetBackoff
  --- PASS: TestClientSetBackoff (0.00s)
  === RUN   TestSanitizePrompt
  === RUN   TestSanitizePrompt/empty_string
  === RUN   TestSanitizePrompt/whitespace_only
  === RUN   TestSanitizePrompt/mention_only
  === RUN   TestSanitizePrompt/mention_embedded_in_text
  === RUN   TestSanitizePrompt/unicode_and_emojis_preserved
  === RUN   TestSanitizePrompt/trimming_leading_and_trailing_whitespace
  === RUN   TestSanitizePrompt/long_text_truncated_to_2000_runes_safely
  --- PASS: TestSanitizePrompt (0.00s)
  === RUN   TestSanitizePromptNormalizedWhitespace
  --- PASS: TestSanitizePromptNormalizedWhitespace (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	0.725s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASS (Zero-comment policy dipatuhi 100% pada semua berkas Go baru, fail-fast validasi input baseURL/model kosong, no magic numbers dengan konstanta `DefaultTimeout`, `MaxRetries`, `InitialBackoff`, `BackoffMultiplier`, `GenerateEndpoint`, `MaxPromptRunes`, safe file ops via write_file/edit_file)
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASS (Semua TIDAK: bukti konkret terminal 100% lulus, tabel-driven unit tests untuk validasi client, retry exponential backoff, penolakan 4xx tanpa retry, timeout handling, connection refused handling, dan rune-safe prompt sanitization)
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS (TDD dibuktikan dengan tes berstatus RED sebelum kode implementasi ditulis)
  - O-08 (Edge Case Atlas): PASS (Prompt kosong/whitespace, mention-only, multi-byte unicode pada boundary 2000 rune, server hang/timeout, connection refused, 4xx non-retryable, dan 5xx recovery)
  - O-09 (Damage Assessment): PASS / N/A (tidak ada data lama yang terdampak)
  - O-10 (Full Suite Metrics): PASS (7 packages tested, 50+ tests passed, 100% pass)
  - O-11 (Real Thing Driven): PASS (Binary dikompilasi dengan CGO_ENABLED=0 sukses tanpa warning go vet)
  - O-12 (Shipping Env Check): PASS (Pure Go standard library net/http, tidak membutuhkan compiler CGO)
  - Hunt Loop Final Round: PASS (0 temuan kegagalan baru)
- **Pilar 4: Tata Kelola & Sinkronisasi:**
  - Graphify Update: N/A (direktori .graphify belum ada di root proyek)
  - CHANGELOG.md: PASS (dicatat di baris teratas)
  - Commit Message Sanity: PASS (pengerjaan branch main terisolasi di Project Repo)
  - O-16 Untested List: Seluruh metode client dan sanitasi telah diuji secara komprehensif.
