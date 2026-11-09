---
id: TICKET-1794189452
plan_id: PLAN-1794189452
title: Add Notice in Settings Embed that Difficulty Changes Affect Subsequent Games
status: Done
priority: Low
labels: [Discord, Infrastructure, Settings, UX, Clarification]
---

# Deskripsi
Sebagai pemain Radovid Bot,
Saya ingin melihat pemberitahuan yang jelas pada tampilan `/radovid settings` bahwa perubahan tingkat kesulitan hanya akan berpengaruh pada sesi permainan berikutnya,
Sehingga saya memahami bahwa permainan yang sedang aktif di channel akan tetap berjalan dengan tingkat kesulitan lama sampai selesai.

Konteks masalah:
Saat ini embed `BuildSettingsEmbed` menampilkan profil pemain, level saat ini, dan panduan CEFR, namun belum menyertakan catatan bahwa pergantian level kesulitan bersifat *subsequent-session* (hanya berlaku mulai game baru selanjutnya) dan tidak serta-merta mengubah state round game yang sedang berlangsung di channel.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794189452-settings-next-game-notice.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] Embed `/radovid settings` memuat field pemberitahuan (`ℹ️ Notice`) yang menyatakan secara eksplisit bahwa perubahan level kemahiran berlaku mulai sesi permainan berikutnya.
- [x] Panjang field pemberitahuan berada di bawah batas Discord 1024 karakter.
- [x] Unit test `TestBuildSettingsEmbed` memvalidasi keberadaan field pemberitahuan sesi berikutnya.
- [x] Dokumen `manual-testing-playbook.md` (Seksi 4.2 `/radovid settings`) diperbarui mencakup field pemberitahuan baru pada daftar tampilan.
- [x] Kebijakan Zero-Comment pada seluruh berkas Go tetap terjaga 100%.
- [x] Seluruh unit test suite lulus (`go test -race ./...`) dan build binary berhasil.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `docs/manual-testing-playbook.md`
- `CHANGELOG.md`

---

## AI Execution Log & Output
- **Status:** Done
- **Execution Date:** 2026-11-09

### Delivery Gate Verification
- **Code & Data Hygiene (CG-1 s/d CG-5):** PASS (Zero-comment pada seluruh berkas Go, no magic numbers, no direct DDL, field value terukur rapi)
- **Hard Gate Testing (HG-1 s/d HG-4):** PASS (Bukan DB prod, bukti konkret pengujian lulus 100%, tanpa melonggarkan tes, repositori bersih dari artefak build)
- **Must Probe Status:**
  - O-07 (Fail on Old Code): PASS (Terbukti gagal saat pengujian Red phase dengan `expected settings embed to have a notice field explaining that changes apply to the next game`)
  - O-08 (Edge Case Atlas): PASS (Field Discord berada jauh di bawah batas 1024 karakter; user nil dicek aman)
  - O-09 (Damage Assessment): PASS / N/A (Tidak ada data lama yang rusak atau termutasi)
  - O-10 (Full Suite Metrics): PASS (25 paket pass, 0 fail, 0 race)
  - O-11 (Real Thing Driven): PASS (Binary bot berhasil dikompilasi via `go build ./cmd/bot`)
  - O-12 (Shipping Env Check): PASS (Pure Go tanpa OS dependency)
- **Orchestrator Sync & Quality Locks:**
  - Graphify Update: N/A (direktori .graphify tidak ada di proyek)
