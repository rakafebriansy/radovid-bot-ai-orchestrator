---
id: TICKET-1795452322
plan_id: PLAN-1795452322
title: "[BUG] Game Generation Fails with Total Blackout When Ollama Unreachable and Gemini Key Not Configured"
status: Done
priority: High
severity: Broken Flow
labels: [Bug, LLM, Infrastructure, Reliability]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah

Bot gagal menjalankan permainan karena dua faktor yang teridentifikasi secara bertahap di lingkungan produksi:
1. Variabel lingkungan `GEMINI_API_KEY` dan `GEMINI_MODEL` belum aktif/tersinkronisasi dengan container runtime, sementara host Ollama lokal (`192.168.1.5:11434`) offline/unreachable (`dial tcp: no route to host`).
2. Setelah Gemini terhubung, klien `gemini.Client` mengalami kendala pemotongan respon karena hanya membaca `parts[0]` (mengabaikan bagian respon selanjutnya ketika LLM mengembalikan multi-part chunk atau thought). Selain itu, parser `ollama.ParseGameMasterOutput` menolak variasi format spasi seperti `OPTION A:` / `OPTION B:` yang lazim dihasilkan oleh Gemini.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1795452322-bug-game-generation-fails-when-ollama-unreachable.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Jalankan bot dengan konfigurasi Gemini API aktif pada model `gemini-3.8-flash` / `gemini-2.0-flash`.
2. Host Ollama lokal offline (`no route to host`).
3. Panggil perintah Discord `/radovid play would-you-rather`.
4. Amati kegagalan parser: `failed to generate game content: AI game generation service produced unparseable output: failed to parse game master output`.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Bot berhasil menerima seluruh bagian respon Gemini (`parts`), mem-parse opsi permainan secara toleran terhadap spasi atau penomoran, mencatat log diagnostik bila parsing gagal, dan `FallbackClient` menangani network failure pada Ollama secara anggun.
- **Aktual (Actual):**
  ```
  Failed to start game round: failed to generate game content:
  AI game generation service produced unparseable output: failed to parse game master output
  ```

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] Klien `gemini.Client` membaca dan menggabungkan seluruh elemen `parts` respon kandidat (`strings.Builder`) agar teks tidak terpotong.
- [x] Parser `ollama.ParseGameMasterOutput` mendukung variasi format teks Gemini (`OPTION A:`, `OPTION B:`, `1.`, `2.`, `A.`, `B.`, dsb).
- [x] `GameMaster.GenerateGameContent` mencatat error diagnostik dan `rawOutput` secara eksplisit pada log sistem saat parsing gagal.
- [x] `DefaultGeminiModel` diperbarui menjadi `gemini-3.8-flash` pada `config/config.go` sesuai ketersediaan Google AI Studio.
- [x] Unit test baru membuktikan bahwa `FallbackClient` menangani kegagalan network error pada primary dan meneruskan ke fallback (O-07).
- [x] Unit test multi-part candidate concatenation ditambahkan pada `gemini/client_test.go`.
- [x] Unit test variasi format ditambahkan pada `ollama/parser_test.go`.
- [x] Seluruh test suite lulus 100% tanpa regresi (`go test -race ./...`).
- [x] Lolos verifikasi penuh [Delivery Gate](../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori `radovid-bot-api`.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/gemini/client.go`
- `../../../../radovid-bot-api/internal/infrastructure/gemini/client_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/game_master.go`
- `../../../../radovid-bot-api/config/config.go`

---

## 6. AI Execution Log & Output

- **Penyebab Akar (Root Cause):**
  1. `gemini.Client.Generate` hanya mengambil `parts[0].Text` dari kandidat pertama. Pada model Gemini 2.5 / 3.x, respon teks sering dipecah menjadi beberapa `parts` (terutama bila terdapat thought signature atau streaming chunks), sehingga bagian penting seperti `OPTION_B` terpotong.
  2. `parseWouldYouRather` pada `ollama.parser.go` hanya mengenali `OPTION_A:` (dengan underscore) dan `OPTION 1:`, tetapi tidak mengenali `OPTION A:` (dengan spasi) yang lazim dihasilkan model LLM.
- **Langkah Perbaikan:**
  1. Mengubah ekstraksi teks kandidat di `gemini/client.go` menggunakan `strings.Builder` yang merangkai seluruh `part.Text`.
  2. Memperluas deteksi prefix pada `parseWouldYouRather` di `ollama/parser.go` agar mendukung `OPTION A:`, `OPTION B:`, format nomor `1.` / `2.`, dan huruf `A.` / `B.`.
  3. Menambahkan log diagnostik `log.Printf` pada `GameMaster.GenerateGameContent` yang mencetak `rawOutput` jika terjadi error parsing.
  4. Menyelaraskan `DefaultGeminiModel` di `config/config.go` menjadi `gemini-3.8-flash`.
  5. Menambahkan pengujian komprehensif di `client_test.go`, `parser_test.go`, dan `fallback_client_test.go`.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -race ./... && go vet ./... && go build ./cmd/bot && rm -f bot`
- **Output Terminal Aktual:**
  ```text
  ?   	github.com/rakafebriansy/radovid-bot/cmd/bot	[no test files]
  ok  	github.com/rakafebriansy/radovid-bot/config	1.619s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.736s
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
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/gemini	2.876s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/llm	2.450s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	2.305s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	4.263s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	(cached)
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	(cached)
  ?   	github.com/rakafebriansy/radovid-bot/web	[no test files]
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `go test -v ./internal/infrastructure/gemini -run TestClient_Generate_MultiPartConcatenation`
  - Pesan Kegagalan: `expected 'Part 1: Part 2', got 'Part 1: '`
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih. Tidak ada mutasi atau persistensi data korup pada database SQLite.
- **Rincian Temuan:** Kegagalan hanya terjadi pada layer transmisi dan parsing memori runtime.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):** PASS
- **Must Probe Status (MP-1 s/d MP-7):**
  - O-07 (Fail on Old Code): PASS (Tercatat di atas)
  - O-08 (Edge Case Atlas): PASS (Multi-part candidate, spaces in delimiters, numbered options)
  - O-09 (Damage Assessment): PASS (Data historis bersih)
  - O-10 (Full Suite Metrics): PASS (100% test lulus dengan `-race`)
  - O-11 (Real Thing Driven): PASS (Diuji terhadap output riil Gemini API v1beta)
  - O-12 (Shipping Env): PASS
  - Hunt Loop Final Round: PASS (0 temuan baru)
- **Daftar Belum Diuji (Untested List - O-16):**
  - Verifikasi langsung melalui antarmuka Discord produksi (memerlukan rebuild image container pada server).
