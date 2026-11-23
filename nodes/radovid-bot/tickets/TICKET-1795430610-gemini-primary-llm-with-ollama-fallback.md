---
id: TICKET-1795430610
plan_id: PLAN-1795430610
title: Implement Gemini Flash as Primary LLM with Ollama Fallback
status: Done
priority: High
labels: [AI, Infrastructure, Gemini, LLM, Fallback, Reliability, Cost-Optimization]
---

# Deskripsi
Sebagai Developer dan Pengguna Server Discord Radovid Bot,
Saya ingin bot menggunakan Gemini Flash (Google AI Studio) sebagai Primary LLM provider dan otomatis melakukan fallback ke Ollama (Qwen lokal) saat layanan cloud gagal atau offline,
Sehingga generasi konten permainan dan evaluasi bahasa Inggris menjadi sangat andal, cepat (< 1 detik), bebas timeout Discord, dan berbiaya Rp 0 melalui Free Tier resmi.

### Konteks & Latar Belakang Masalah
1. Selama ini bot hanya mengandalkan instance lokal Ollama (`Qwen` / `Gemma`).
2. Layanan lokal Ollama sering kali lambat (cold-start 10-30 detik), memakan memori/VRAM besar, dan gagal merespons interaksi Discord tepat waktu (gateway timeout).
3. Google AI Studio menyediakan Gemini 1.5 / 2.0 / 2.5 Flash secara gratis (15 RPM / 1.500 RPD) dengan latency super cepat (300ms - 800ms) dan dukungan native Structured JSON Output.
4. Tiket ini menambahkan klien Gemini API berbasis HTTP native Go yang mengimplementasikan `gamemaster.LLMClient`, serta membungkusnya dalam `FallbackClient` berjenjang:
   - Primary: Gemini Flash.
   - Fallback: Ollama lokal (Qwen).
   - Tertiary: Offline static catalog (`fallback.go`) yang sudah ada.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1795430610-gemini-primary-llm-with-ollama-fallback.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] **Gemini Infrastructure Client:**
  - [x] Implementasi paket `internal/infrastructure/gemini` dengan `Client` yang memenuhi kontrak antarmuka `gamemaster.LLMClient`.
  - [x] Mendukung HTTP request ke endpoint Google AI Studio (`generateContent`) dengan timeout context dan sanitasi prompt.
  - [x] Mendukung ekstraksi response candidate teks dan deteksi galat HTTP (4xx, 429 rate limit, 5xx).
  - [x] Menggunakan native standard library `net/http` murni tanpa dependensi eksternal CGO/heavy SDK.
- [x] **Multi-Provider Fallback Wrapper:**
  - [x] Implementasi `FallbackClient` di `internal/infrastructure/llm` yang membungkus `primary` (`gemini.Client`) dan `fallback` (`ollama.Client`).
  - [x] Jika `primary` sukses menghasilkan teks, kembalikan hasil tanpa memanggil fallback.
  - [x] Jika `primary` gagal (error, timeout, 429), catat log peringatan dan otomatis panggil `fallback`.
  - [x] Jika `primary` tidak dikonfigurasi (misal API key kosong), fallback langsung aktif sebagai single provider tanpa galat runtime.
- [x] **Konfigurasi Lingkungan:**
  - [x] `config/config.go` mendukung `GEMINI_API_KEY` dan `GEMINI_MODEL` (default `gemini-2.5-flash`).
  - [x] `cmd/bot/main.go` menginisialisasi `FallbackClient` dan menyuntikkannya ke `GameMaster`, evaluator, dan seluruh usecase game.
- [x] **Testing & Higienitas Kode:**
  - [x] Unit test komprehensif untuk `gemini.Client`, `FallbackClient`, dan `config.LoadConfig`.
  - [x] Seluruh unit test lulus 100% dengan `go test -race ./...`.
  - [x] 100% mematuhi Zero-Comment Policy (`coding.md`).

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/config/config.go`
- `../../../../radovid-bot-api/config/config_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/gemini/client.go`
- `../../../../radovid-bot-api/internal/infrastructure/gemini/client_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client.go`
- `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output

- **Langkah Teknis Tereksekusi:**
  1. Melakukan evaluasi kebutuhan branch baru vs `main` bersama user sesuai protokol Master Entrypoint `main.md`; disetujui bekerja langsung pada branch `main`.
  2. Mengimplementasikan `internal/infrastructure/gemini/client.go` dan suite test `client_test.go` menggunakan pola TDD (RED -> GREEN). Klien mengimplementasikan antarmuka `gamemaster.LLMClient` via Google AI Studio REST API (`v1beta/models/{model}:generateContent?key={apiKey}`), mendukung timeout context, pembersihan prompt, konfigurasi structured JSON output, dan penanganan galat HTTP.
  3. Mengimplementasikan `internal/infrastructure/llm/fallback_client.go` dan suite test `fallback_client_test.go` menggunakan TDD. Fallback wrapper mengeksekusi Primary LLM terlebih dahulu; jika Primary gagal (HTTP error/timeout/rate limit), ia mencatat log peringatan dan mengalihkan generasi secara transparan ke Fallback LLM (Ollama lokal).
  4. Menambahkan field konfigurasi `GEMINI_API_KEY` dan `GEMINI_MODEL` (default `gemini-2.5-flash`) pada `config/config.go` dan unit test pada `config/config_test.go`.
  5. Menghubungkan `gemini.NewClient` dan `llm.NewFallbackClient` di `cmd/bot/main.go`. Menyuntikkan `llmClient` ke `GameMaster`, `TabooEvaluator`, `DiaryUseCase`, dan 14 usecase game (`wyrUseCase`, `tqUseCase`, `interviewUseCase`, `roleplayUseCase`, `oneMinuteUseCase`, `storyRelayUseCase`, `fiveWordUseCase`, `secretMissionUseCase`, `bossFightUseCase`, `transformationUseCase`, `movieReviewUseCase`, `newsReporterUseCase`, `dreamHouseUseCase`).
  6. Memvalidasi kepatuhan Zero-Comment Policy (`coding.md` 1.1) pada seluruh file Go yang diubah/dibuat; memastikan tidak ada komentar `//` maupun `/* */`.
  7. Menjalankan verifikasi penuh `go test -race ./...`, `go vet ./...`, dan `go build ./cmd/bot`.
- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/config/config.go`
  - `radovid-bot-api/config/config_test.go`
  - `radovid-bot-api/internal/infrastructure/gemini/client.go`
  - `radovid-bot-api/internal/infrastructure/gemini/client_test.go`
  - `radovid-bot-api/internal/infrastructure/llm/fallback_client.go`
  - `radovid-bot-api/internal/infrastructure/llm/fallback_client_test.go`
  - `radovid-bot-api/cmd/bot/main.go`
- **Catatan & Keputusan Arsitektural:**
  - Arsitektur sepenuhnya decoupled: usecase layer mengonsumsi `gamemaster.LLMClient` sehingga pergantian provider atau penyisipan fallback wrapper tidak memerlukan perubahan pada logic game sama sekali.
  - Backward compatibility terjamin: jika `GEMINI_API_KEY` tidak diatur, bot secara otomatis berjalan menggunakan `ollamaClient` tunggal tanpa error runtime.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -race ./... && go vet ./... && go build ./cmd/bot && rm -f bot`
- **Output Terminal:**
  ```text
  ?   	github.com/rakafebriansy/radovid-bot/cmd/bot	[no test files]
  ok  	github.com/rakafebriansy/radovid-bot/config	1.483s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/diary	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/gemini	1.931s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/llm	2.320s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	(cached)
  ?   	github.com/rakafebriansy/radovid-bot/web	[no test files]
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASSED (100% Zero-Comment Policy dipenuhi, tidak ada file tertinggal atau binary bocor).
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASSED (Seluruh suite test lulus dengan flag `-race`).
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):** PASSED
  - O-07 (Fail on Old Code): Terbukti test gagal saat belum ada implementasi klien/wrapper.
  - O-08 (Edge Case Atlas): Menguji HTTP 429 rate limit, 500 server error, empty candidates, malformed JSON, primary fail -> fallback success, both fail.
  - O-09 (Damage Assessment): Tidak ada dampak regresi ke usecase layer karena kontrak antarmuka identik.
  - O-10 (Full Suite Metrics): 100% test lulus.
- **Pilar 4: Tata Kelola Dokumen & Git (GG-1 s/d GG-5):** PASSED (Dokumen tiket dan rencana tersinkronisasi, changelog diperbarui).

### Daftar Belum Diuji (Untested List - O-16)
- Panggilan jaringan langsung ke Google AI Studio dengan API key riil di lingkungan produksi (diuji secara deterministik via mock HTTP RoundTripper di suite test).
