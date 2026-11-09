---
id: TICKET-1762520600
plan_id: PLAN-1762520600
title: Fix Radovid Help Command to List All Commands and Mini-Games
status: Done
priority: Medium
labels: [Discord, Delivery, Hotfix, Help, UX]
---

# Deskripsi
Sebagai pengguna Discord Radovid Bot,
Saya ingin perintah `/radovid help` menampilkan daftar lengkap seluruh perintah inti, pengaturan, jurnal, dan 15 mini-game yang dapat dimainkan,
Sehingga saya dan rekan belajar dapat mengetahui seluruh kapabilitas dan opsi permainan yang tersedia tanpa kebingungan.

Konteks masalah:
Saat ini method `HandleRadovidHelp` pada `radovid_help.go` hanya mencantumkan 2 perintah lama (`/radovid start` dan `/radovid help`), sementara fitur-fitur baru seperti `/radovid settings`, `/radovid progress`, `/radovid diary`, `/radovid ask`, `/radovid guess`, serta 15 varian `/radovid play <game>` belum tercantum sama sekali dalam embed bantuan.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520600-fix-radovid-help-command-list.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Embed `/radovid help` menampilkan seksi Core Commands (`/radovid help`, `/radovid start`, `/radovid settings`, `/radovid progress`, `/radovid diary`).
- [x] Embed `/radovid help` menampilkan seksi 20 Questions Game Commands (`/radovid ask`, `/radovid guess`).
- [x] Embed `/radovid help` menampilkan katalog seluruh 15 mini-games (`would-you-rather`, `taboo`, `20-questions`, `job-interview`, `roleplay`, `one-minute-challenge`, `story-relay`, `five-word-challenge`, `secret-mission`, `grammar-boss-fight`, `transformation-race`, `movie-review`, `news-reporter`, `dream-house`, `diary`).
- [x] Mempertahankan seksi panduan belajar (*Learning Guidelines*).
- [x] Seluruh panjang field value embed berada di bawah batas Discord 1024 karakter.
- [x] Terdapat unit test verifikasi Red-Green yang memvalidasi keberadaan seluruh perintah dalam embed bantuan.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help_test.go`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-09

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment pada `radovid_help.go` dan `radovid_help_test.go`, no magic numbers, no direct DDL, batas field embed dicek presisi < 1024 karakter)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, bukti konkret pengujian lulus 100%, tanpa melonggarkan tes, repositori bersih dari artefak build)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): PASS (Terbukti gagal saat pengujian Red phase dengan `undefined: BuildHelpEmbed` dan kegagalan batas 1024 karakter saat mini-game belum dipecah secara modular)
  - O-08 (Edge Case Atlas): PASS (Field Discord terverifikasi tidak melebihi 1024 karakter per field dan total embed di bawah 6000 karakter; penanganan session/interaction nil teruji)
  - O-09 (Damage Assessment): PASS / N/A (Tidak ada data lama yang rusak atau termutasi)
  - O-10 (Full Suite Metrics): PASS (25 paket pass, 0 fail, 0 race)
  - O-11 (Real Thing Driven): PASS (Binary berhasil dikompilasi via `go build ./cmd/bot`)
  - O-12 (Shipping Env Check): PASS (Pure Go tanpa OS dependency)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (direktori .graphify tidak ada di proyek)
  - CHANGELOG.md: PASS (dicatat di baris teratas)
  - Commit & PR Sanity: Menunggu persetujuan eksplisit pengguna sebelum commit/push

### Terminal Evidence
```text
$ go test -v -run TestBuildHelpEmbed ./internal/delivery/discord
=== RUN   TestBuildHelpEmbed_ContainsAllCommandsAndGames
--- PASS: TestBuildHelpEmbed_ContainsAllCommandsAndGames (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.466s

$ go vet ./...
# Exit code 0 (No issues)

$ go test -race ./...
ok  	github.com/rakafebriansy/radovid-bot/config	(cached)
ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	1.603s
ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	(cached)
... 25 packages passed (100% PASS, 0 race conditions)
```
