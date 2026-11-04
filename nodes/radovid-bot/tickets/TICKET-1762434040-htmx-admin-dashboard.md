---
id: TICKET-1762434040
plan_id: PLAN-1762434040
title: HTMX Admin Dashboard
status: Todo
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
- [ ] Server HTTP tertanam berjalan pada goroutine mandiri mendengarkan port dari `ADMIN_DASHBOARD_PORT` (default: 8080)
- [ ] Middleware HTTP Basic Auth memvalidasi format `username:password` dari env var `ADMIN_DASHBOARD_SECRET`, mengembalikan status 401 Unauthorized dengan header `WWW-Authenticate: Basic` jika otentikasi gagal
- [ ] Seluruh aset web (HTML templates di `web/templates/` dan aset statis di `web/static/`) di-embed ke binary Go via `embed.FS` tanpa ketergantungan file lokal pada waktu runtime
- [ ] Route web dashboard tersedia:
  - `GET /admin` — Tampilan ringkasan metrik (total sesi, pengguna terdaftar, rata-rata XP)
  - `GET /admin/sessions` — Tabel sesi game dengan dukungan filter status (pending/active/completed) via HTMX partial swap
  - `GET /admin/users` — Tabel daftar pengguna, level saat ini, streak aktif, dan total XP
  - `GET /admin/health` — Status kesehatan bot, konektivitas SQLite WAL, dan ketersediaan Ollama API lokal
- [ ] HTMX polling atau tombol *refresh* memperbarui blok data spesifik secara asinkron tanpa *full-page reload*
- [ ] Unit test pada Basic Auth middleware (skenario: tanpa auth header, password salah, format kredensial valid)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/http/admin_handler.go`
- `../../../../radovid-bot-api/internal/delivery/http/middleware.go`
- `../../../../radovid-bot-api/internal/delivery/http/router.go`
- `../../../../radovid-bot-api/internal/delivery/http/middleware_test.go`
- `../../../../radovid-bot-api/internal/delivery/http/admin_handler_test.go`
- `../../../../radovid-bot-api/internal/usecase/admin/dashboard_stats.go`
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
- **Status:** Pending Execution
