---
id: TICKET-1794188809
plan_id: PLAN-1794188809
title: Remove Deprecated Radovid Start Command
status: Done
priority: Medium
labels: [Discord, Delivery, Deprecation, Cleanup, Refactor]
---

# Deskripsi
Sebagai pengembang dan pengguna Radovid Bot,
Saya ingin menghapus perintah legacy `/radovid start`,
Sehingga pengguna tidak bingung dengan alur inisialisasi awal Fase 1 yang sudah usang dan dapat langsung menggunakan perintah game nyata (`/radovid play <game>`), jurnal (`/radovid diary`), pengaturan (`/radovid settings`), maupun progres (`/radovid progress`).

Konteks masalah:
Perintah `/radovid start` dibuat pada Fase 1 sebagai placeholder awal ("Pending Player 2 (Session skeleton initialized)"). Sejak seluruh 15 mini-game dan fitur sistem pembelajaran interaktif selesai diimplementasikan, perintah `/radovid start` menjadi usang, redundan, dan membingungkan pengguna.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794188809-remove-deprecated-radovid-start-command.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Subcommand `start` dihapus dari definisi Discord application command pada `command_def.go`.
- [x] Handler `HandleRadovidStart` dan berkas `radovid_start.go` dihapus dari codebase.
- [x] Routing `case "start":` pada `handler.go` dihapus dan pesan fallback kosong diperbarui menjadi contoh `/radovid help or /radovid play`.
- [x] Perintah `/radovid start` dihapus dari katalog bantuan `BuildHelpEmbed()` pada `radovid_help.go`.
- [x] Seluruh unit test (`command_test.go`, `radovid_help_test.go`) diperbarui untuk memastikan `start` tidak lagi terdaftar dan tidak muncul di embed.
- [x] Dokumentasi `manual-testing-playbook.md` dibersihkan dari seluruh referensi `/radovid start` (jumlah perintah resmi disesuaikan menjadi 20).
- [x] Kebijakan Zero-Comment pada seluruh berkas Go tetap terjaga 100%.
- [x] Seluruh unit test suite lulus (`go test -race ./...`) dan build binary berhasil.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/command_def.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` (Dihapus)
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_test.go`
- `docs/manual-testing-playbook.md`
- `CHANGELOG.md`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-09

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment pada seluruh berkas Go, no magic numbers, no direct DDL, penghapusan kode mati 100% tuntas)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, bukti konkret pengujian lulus 100%, tanpa melonggarkan tes, repositori bersih dari artefak build)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): PASS (Terbukti gagal saat pengujian Red phase dengan `expected 7 subcommands, got 8` dan `expected help embed not to contain deprecated '/radovid start'`)
  - O-08 (Edge Case Atlas): PASS (Penghapusan start tidak mengganggu dispatching subcommand lain; fallback error command kosong tervalidasi)
  - O-09 (Damage Assessment): PASS / N/A (Tidak ada data lama yang rusak atau termutasi)
  - O-10 (Full Suite Metrics): PASS (25 paket pass, 0 fail, 0 race)
  - O-11 (Real Thing Driven): PASS (Binary bot berhasil dikompilasi via `go build ./cmd/bot`)
  - O-12 (Shipping Env Check): PASS (Pure Go tanpa OS dependency)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (direktori .graphify tidak ada di proyek)
