---
id: TICKET-1762434460
plan_id: PLAN-1762434460
title: Daily Challenge Notifications
status: Done
priority: High
labels: [Backend, Worker, Discord, Notification, Automation]
---

# Deskripsi
Sebagai Pengguna Discord / Komunitas Pembelajar Bahasa Inggris,
Saya ingin bot secara otomatis mengirimkan tantangan bahasa Inggris harian (*Daily Challenge*) ke channel Discord setiap pagi pada waktu yang telah ditentukan,
Sehingga saya dan anggota server mendapatkan pengingat rutin untuk berlatih dan dapat segera memulai sesi belajar bersama.

Pemberitahuan tantangan harian dijalankan oleh *background worker* mandiri berbasis Goroutine dan `time.Ticker` di dalam biner bot. Setiap pagi pada jam yang dikonfigurasi melalui variabel lingkungan (`DAILY_CHALLENGE_HOUR`, default: pukul 07:00 UTC), worker akan:
1. Memilih tantangan harian secara rotasi atau acak dari repositori tantangan (atau meminta Ollama men-generate tantangan tematik harian).
2. Membangun pesan Discord Embed kaya informasi (*Daily Challenge Embed*) yang memuat judul tantangan, instruksi singkat, dan tombol pintas interaktif untuk langsung memulai aktivitas.
3. Mengirimkan pesan ke text channel Discord yang ditentukan oleh `DAILY_CHALLENGE_CHANNEL_ID`.
4. Mencatat log pengiriman untuk mencegah duplikasi pengiriman pesan pada hari yang sama saat bot mengalami restart.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762434460-daily-challenge-notifications.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Tersedia background scheduler worker yang berjalan dalam goroutine dengan konteks *graceful shutdown* (`context.Context`)
- [x] Pengaturan waktu harian dikonfigurasi melalui variabel lingkungan `DAILY_CHALLENGE_HOUR` (format jam 0-23 UTC) dan channel tujuan `DAILY_CHALLENGE_CHANNEL_ID`
- [x] Algoritma scheduler menghitung durasi waktu tunda awal (*initial delay*) hingga jadwal jam target berikutnya tercapai, lalu mengaktifkan interval perulangan `time.NewTicker(24 * time.Hour)`
- [x] Idempotensi pengiriman: jika bot di-restart pada hari yang sama setelah pesan terkirim, scheduler tidak mengirimkan pesan duplikat (menggunakan state flag atau tabel riwayat pengiriman)
- [x] Embed Discord tantangan harian menyajikan kartu interaktif dengan tema harian (misal: "Word of the Day", "Grammar Quick Fix", "Story Prompt"), panduan, dan tombol aksi "Accept Challenge"
- [x] Scheduler berhenti dengan bersih (*clean exit*) saat sinyal shutdown (SIGTERM/SIGINT) diterima tanpa menyisakan *dangling goroutines*
- [x] Unit testing terhadap kalkulasi waktu tunda scheduler dan penanganan pembatalan konteks

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/config/config.go`
- `../../../../radovid-bot-api/config/config_test.go`
- `../../../../radovid-bot-api/migrations/011_create_daily_challenge_dispatches.sql`
- `../../../../radovid-bot-api/internal/domain/notification/entity.go`
- `../../../../radovid-bot-api/internal/domain/notification/entity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/dispatch_repository.go`
- `../../../../radovid-bot-api/internal/infrastructure/database/dispatch_repository_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/notification_sender.go`
- `../../../../radovid-bot-api/internal/delivery/discord/notification_sender_test.go`
- `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler.go`
- `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler_test.go`
- `../../../../radovid-bot-api/cmd/bot/main.go`

---

## AI Execution Log & Output
- **Status:** Done
- **Tanggal Eksekusi:** 2026-11-08
- **Ringkasan Delivery Gate:**
  - 100% PASS seluruh test suite di 24 package `radovid-bot-api` (`go test -count=1 ./...`).
  - Linter `go vet ./...` bersih tanpa warning/error.
  - Kompilasi biner `go build ./cmd/bot` berhasil tanpa error.
  - Mematuhi *Zero-Comment Policy* pada seluruh berkas Go baru maupun hasil modifikasi.
  - Graceful shutdown worker goroutine terkoordinasi dengan `context.Context` dan `sync.WaitGroup`.
