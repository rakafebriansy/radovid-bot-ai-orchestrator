---
id: TICKET-1762434040
plan_id: PLAN-1762434040
title: HTMX Admin Dashboard
status: Done
priority: High
labels: [Frontend, Backend, HTMX, HTTP, Monitoring]
---

# Deskripsi
Sebagai Operator Bot / Administrator Sistem,
Saya ingin memiliki antarmuka web dashboard internal berbasis HTMX yang ringan, aman, dan tanpa ketergantungan JavaScript framework eksternal yang rumit,
Sehingga saya dapat memantau sesi permainan aktif, riwayat pengguna, performa LLM Ollama lokal, dan status kesehatan bot secara real-time.

Admin dashboard internal dibangun menggunakan kombinasi pustaka standar Go `net/http` dan template HTML dengan ekstensi HTMX untuk pembaruan komponen secara parsial (*partial re-rendering*). Seluruh endpoint web berada di bawah proteksi HTTP Basic Authentication dengan kredensial rahasia dari variabel lingkungan (`ADMIN_DASHBOARD_SECRET`). Aset web (HTML templates dan CSS/JS statis) dikemas langsung ke dalam biner Go menggunakan compiler directive `//go:embed`.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434040-htmx-admin-dashboard.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Server HTTP tertanam berjalan pada goroutine mandiri mendengarkan port dari `ADMIN_DASHBOARD_PORT` (default: 8080)
- [x] Middleware HTTP Basic Auth memvalidasi format `username:password` dari env var `ADMIN_DASHBOARD_SECRET`, mengembalikan status 401 Unauthorized dengan header `WWW-Authenticate: Basic` jika otentikasi gagal
- [x] Seluruh aset web (HTML templates di `web/templates/` dan aset statis di `web/static/`) di-embed ke binary Go via `embed.FS` tanpa ketergantungan file lokal pada waktu runtime
- [x] Route web dashboard tersedia:
  - `GET /admin` — Tampilan ringkasan metrik (total sesi, pengguna terdaftar, rata-rata XP)
  - `GET /admin/sessions` — Tabel sesi game dengan dukungan filter status (pending/active/completed) via HTMX partial swap
  - `GET /admin/users` — Tabel daftar pengguna, level saat ini, streak aktif, dan total XP
  - `GET /admin/health` — Status kesehatan bot, konektivitas SQLite WAL, dan ketersediaan Ollama API lokal
- [x] HTMX polling atau tombol *refresh* memperbarui blok data spesifik secara asinkron tanpa *full-page reload*
- [x] Unit test pada Basic Auth middleware (skenario: tanpa auth header, password salah, format kredensial valid)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/http/admin_handler.go`
- `../../../../radovid-bot-api/internal/delivery/http/admin_handler_test.go`
- `../../../../radovid-bot-api/internal/delivery/http/middleware.go`
- `../../../../radovid-bot-api/internal/delivery/http/middleware_test.go`
- `../../../../radovid-bot-api/internal/delivery/http/router.go`
- `../../../../radovid-bot-api/internal/usecase/admin/dashboard_stats.go`
- `../../../../radovid-bot-api/internal/usecase/admin/dashboard_stats_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/admin_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/admin_repository_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/client.go`
- `../../../../radovid-bot-api/web/embed.go`
- `../../../../radovid-bot-api/web/templates/base.html`
- `../../../../radovid-bot-api/web/templates/dashboard.html`
- `../../../../radovid-bot-api/web/templates/sessions.html`
- `../../../../radovid-bot-api/web/templates/users.html`
- `../../../../radovid-bot-api/web/templates/health.html`
- `../../../../radovid-bot-api/web/static/htmx.min.js`
- `../../../../radovid-bot-api/web/static/style.css`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Done
- **Waktu Eksekusi:** 2026-11-07 22:12:00
- **Rangkuman Eksekusi:**
  1. Middleware `BasicAuthMiddleware` dibangun di `internal/delivery/http/middleware.go` dengan `crypto/subtle.ConstantTimeCompare` untuk mencegah timing attack, format `user:password`, status 401 dan header `WWW-Authenticate: Basic realm="Radovid Admin"`. Unit test diuji di `middleware_test.go` (100% PASS).
  2. Usecase `DashboardStatsUseCase` diimplementasikan di `internal/usecase/admin/dashboard_stats.go` mengelola `GetOverview`, `GetSessions`, `GetUsers`, dan `GetHealth`. Unit test diuji di `dashboard_stats_test.go` (100% PASS).
  3. Repositori `AdminRepository` diimplementasikan di `internal/infrastructure/database/admin_repository.go` melayani agregasi metrik, daftar sesi terfilter, daftar pengguna bersortir XP, dan ping SQLite WAL. Unit test diuji di `admin_repository_test.go` (100% PASS).
  4. Template HTML (`base.html`, `dashboard.html`, `sessions.html`, `users.html`, `health.html`) dan static assets (`htmx.min.js`, `style.css`) dikemas via `//go:embed` pada `web/embed.go`.
  5. Handler HTTP `AdminHandler` dan `NewRouter` diimplementasikan di `internal/delivery/http/`, mendukung dual-mode rendering: layout penuh HTML5 saat navigasi browser biasa dan partial template swap saat header `HX-Request: true` aktif.
  6. Server HTTP tertanam diintegrasikan ke `cmd/bot/main.go` pada goroutine mandiri dengan graceful shutdown via `httpServer.Shutdown`.
  7. Seluruh rangkaian test suite (23 package) lulus 100%, `go vet` 0 error, `go build ./cmd/bot` berhasil, dan Zero-Comment Policy dipatuhi penuh.
