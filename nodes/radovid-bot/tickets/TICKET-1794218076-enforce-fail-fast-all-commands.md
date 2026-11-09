---
id: TICKET-1794218076
plan_id: PLAN-1794218076
title: Enforce Fail-Fast Policy Across All Radovid Bot Commands
status: Done
priority: High
labels: [Backend, Go, Discord, FailFast, Architecture, Quality]
---

# Deskripsi
Sebagai Pengembang dan Pengguna Radovid Bot,
Saya ingin seluruh perintah Discord dan use case AI menerapkan kebijakan Fail-Fast secara ketat tanpa silent fallback atau data mock palsu saat LLM/layanan eksternal gagal,
Sehingga pengguna menerima status dan pesan galat yang jujur, akurat, dan tidak disesatkan oleh umpan balik tata bahasa palsu (seperti mencentang benar kalimat yang salah saat AI timeout).

Konteks Masalah:
Sesuai pedoman `global-guidelines/coding.md` Seksi 2.1, sistem dilarang membuat silent fallback atau dummy defaults pada kode produksi. Sebelumnya, ketika AI Ollama mengalami kendala koneksi atau batas waktu (timeout), beberapa use case (seperti `/radovid diary`, game evaluasi, dan game master) secara diam-diam memanggil data fallback statis (`buildFallbackDiaryAnalysis`, `buildFallbackResponse`, `ollama.GetFallbackResponse`). Hal ini menyebabkan `/radovid diary` meloloskan kalimat dengan tata bahasa keliru dan memberi centang hijau seolah-olah kalimat tersebut benar.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794218076-enforce-fail-fast-all-commands.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Seluruh fungsi fallback diam-diam (`buildFallbackDiaryAnalysis`, `buildFallbackResponse`, dan pemanggilan `ollama.GetFallbackResponse` pada kegagalan runtime) dihapus dari use case produksi.
- [x] Ketika pemanggilan LLM gagal, timeout, atau menghasilkan JSON tidak valid, use case wajib melempar runtime error eksplisit bertipe `error`.
- [x] Handler Discord delivery menangkap error tersebut dan menampilkan pesan galat yang transparan dan jujur kepada pengguna melalui embed error / balasan ephemeral (`AI language evaluation service is currently unavailable. Please try again later.`).
- [x] Prompt `daily_diary.tmpl` diperketat agar melakukan audit tata bahasa yang tegas dan objektif untuk setiap kalimat.
- [x] Seluruh unit test suite diperbarui agar memverifikasi pola Fail-Fast (ekspektasi error eksplisit saat LLM gagal, bukan hasil fallback palsu).
- [x] Kebijakan Zero-Comment (`coding.md` Seksi 1.1) dipatuhi 100% pada seluruh berkas Go yang dimodifikasi.
- [x] Seluruh unit test lulus (`go test -race ./...`) tanpa regresi.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/usecase/feedback/diary.go`
- `../../../../radovid-bot-api/internal/usecase/feedback/ai_feedback.go`
- `../../../../radovid-bot-api/internal/usecase/game/game_master.go`
- `../../../../radovid-bot-api/internal/usecase/game/movie_review.go`
- `../../../../radovid-bot-api/internal/usecase/game/news_reporter.go`
- `../../../../radovid-bot-api/internal/usecase/game/dream_house.go`
- `../../../../radovid-bot-api/internal/usecase/game/five_word.go`
- `../../../../radovid-bot-api/internal/usecase/game/one_minute.go`
- `../../../../radovid-bot-api/internal/usecase/game/boss_fight.go`
- `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator.go`
- `../../../../radovid-bot-api/internal/usecase/game/transformation.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/daily_diary.tmpl`
- Unit tests terkait di `radovid-bot-api/internal/...`
- `CHANGELOG.md`

## AI Execution Log & Output

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment dipatuhi pada seluruh berkas Go baru/modifikasi, no silent fallback/mock data di kode produksi, no magic numbers, no direct DDL, safe file ops)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Semua TIDAK: tidak menyentuh DB produksi/live, bukti konkret terminal disertakan, tanpa melonggarkan test assertion, repositori bersih dari artefak sampah)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): PASS — Terbukti pada kode lama, `buildFallbackDiaryAnalysis` meloloskan kalimat salah dan memberi centang hijau palsu karena menyetel `Corrected = Original` saat LLM error/timeout.
  - O-08 (Edge Case Atlas): PASS (Sumbu: LLM error, LLM timeout, malformed JSON response, empty response, user input edge cases)
  - O-09 (Damage Assessment): PASS — Bersih. Tidak ada mutasi skema atau data persistensi rusak.
  - O-10 (Full Suite Metrics): PASS — 100% test packages passed (`go test -count=1 ./...` dan `go test -race ./...` di seluruh paket `radovid-bot-api`).
  - O-11 (Real Thing Driven): PASS — Diverifikasi via unit test mock LLM failure dan template rendering check.
  - O-12 (Shipping Env Check): PASS — Penanganan error seragam di seluruh Discord delivery layer.
  - Hunt Loop Final Round: PASS (0 temuan kegagalan baru di putaran verifikasi terakhir).
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (tidak terdapat direktori `.graphify` pada codebase `radovid-bot-api`).
  - CHANGELOG.md: PASS (dicatat pada entri teratas `[2026-11-09 17:05:00]`).
  - Commit & PR Sanity: PASS (steril dari istilah orchestrator).
  - O-16 Untested List: Interaksi live WebSocket ke server Discord aktual dan gateway Ollama GPU fisik (memerlukan instance Discord bot running dan endpoint Ollama aktif secara live).
