---
id: TICKET-1762520540
plan_id: PLAN-1762520540
title: "[BUG] Pemotongan Byte Mentah UTF-8 pada Template Embed & Clock Drift pada Daily Scheduler"
status: Done
priority: Low
severity: Wording/UI
labels: [Bug, Discord, UI, UTF8, Scheduler]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Terdapat dua anomali pada pemformatan teks embed dan penjadwalan waktu harian:
1. **Pemotongan Karakter Multibyte UTF-8:** Pada `internal/infrastructure/discord/game_templates.go` (baris 550, 1197, 1204, 1335, 1343, 1394, 1401, 1408, 1459, 1467, 1475, 1483), pemangkasan teks panjang menggunakan pemotongan byte mentah (`preview[:497] + "..."`) alih-alih pemotongan berbasis `[]rune` (`truncateString`). Jika karakter UTF-8 multibyte (seperti emoji 4-byte atau karakter beraksen) berada di batas byte ke-497, urutan byte terbelah dan menghasilkan string UTF-8 tidak valid yang ditolak oleh Discord API dengan `HTTP 400 Bad Request`.
2. **Cumulative Clock Drift pada `DailyChallengeScheduler`:** Pada `internal/usecase/notification/daily_challenge_scheduler.go` (baris 131-148), setelah `timer` pertama selesai, scheduler beralih ke `time.NewTicker(24 * time.Hour)` statis. Akibat waktu eksekusi pengiriman pesan Discord dan pergeseran clock OS, waktu pengiriman harian akan bergeser secara kumulatif menjauhi jam target UTC awal.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520540-fix-utf8-truncation-and-scheduler-drift.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Buat string input yang memiliki karakter emoji 4-byte (misal `🔥`) tepat pada posisi byte 495-498.
2. Panggil `BuildDreamHouseEvaluationEmbed` di `game_templates.go`.
3. Periksa `utf8.ValidString` pada field embed yang dihasilkan: bernilai `false` (UTF-8 korup).

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Seluruh pemotongan teks pada `game_templates.go` menggunakan pemotongan berbasis `[]rune` (`truncateString`) sehingga `utf8.ValidString` selalu `true`. Selain itu, `DailyChallengeScheduler` menghitung ulang durasi ke target jam UTC berikutnya menggunakan `CalculateInitialDelay` di setiap siklus.
- **Aktual (Actual):** Pemotongan byte mentah merusak karakter UTF-8 multibyte dan ticker 24 jam mengalami *drift*.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] Seluruh pemotongan string di `internal/infrastructure/discord/game_templates.go` menggunakan helper berbasis rune (`truncateString`) dan menghasilkan string UTF-8 yang 100% valid (`utf8.ValidString == true`).
- [x] `DailyChallengeScheduler` menghitung ulang delay ke jam target berikutnya pada setiap siklus harian agar bebas dari *cumulative clock drift*.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go`
- `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go`
- `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler.go`
- `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler_test.go`

---

## 6. AI Execution Log & Output
*Bagian ini WAJIB diisi oleh AI Agent SAAT dan SETELAH memperbaiki bug.*

- **Penyebab Akar (Root Cause):**
  1. Pada `game_templates.go`, terdapat 12 lokasi pemotongan string panjang yang menggunakan pemotongan byte mentah (`[:n] + "..."`). Saat karakter UTF-8 multibyte (seperti emoji 4-byte `🔥`) berada tepat di batas byte tersebut, deretan byte UTF-8 terbelah dan menghasilkan string dengan urutan byte tidak valid (`utf8.ValidString == false`).
  2. Pada `daily_challenge_scheduler.go`, loop penjadwalan beralih ke `time.NewTicker(24 * time.Hour)` statis setelah penundaan awal selesai. Karena durasi pengiriman HTTP ke Discord dan latency eksekusi ditambahkan ke setiap siklus, jam pengiriman harian bergeser secara kumulatif menjauhi jam target UTC yang ditentukan.
- **Langkah Perbaikan:**
  1. Menambahkan unit test `TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation` pada `game_templates_test.go` dan `TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle` pada `daily_challenge_scheduler_test.go`.
  2. Menjalankan pengujian terhadap kode lama dan membuktikan kegagalan (**RED**) pada kedua skenario pengujian.
  3. Mengganti seluruh 12 ekspresi pemotongan byte mentah di `game_templates.go` dengan fungsi helper berbasis rune `truncateString(...)` dari `embeds.go`.
  4. Memperbarui method `Run` pada `daily_challenge_scheduler.go` agar menghitung ulang `s.initialDelay(time.Now(), s.targetHour)` di setiap iterasi harian standar (`s.tickInterval == 24*time.Hour`), sehingga waktu pengiriman terkunci tepat pada target jam UTC tanpa drift kumulatif.
  5. Menjalankan seluruh pengujian unit test dan race detector (`go test -race ./...`) memastikan 100% lulus tanpa regresi.
- **Ringkasan Berkas Terdampak:**
  - `internal/infrastructure/discord/game_templates.go`
  - `internal/infrastructure/discord/game_templates_test.go`
  - `internal/usecase/notification/daily_challenge_scheduler.go`
  - `internal/usecase/notification/daily_challenge_scheduler_test.go`

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -race -v ./internal/infrastructure/discord/... ./internal/usecase/notification/...`
- **Output Terminal Aktual:**
  ```text
=== RUN   TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation
--- PASS: TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation (0.00s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	1.688s

=== RUN   TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle
--- PASS: TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle (0.03s)
PASS
ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	1.904s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `go test -v -run "TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation|TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle" ./internal/infrastructure/discord/... ./internal/usecase/notification/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation
        game_templates_test.go:1188: corrupted invalid UTF-8 string found in embed field "🏡 Phase 1: Exterior & Setting": "> *AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xf0...*"
    --- FAIL: TestBuildDreamHouseSummaryEmbed_UTF8EmojiTruncation (0.00s)

    === RUN   TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle
        daily_challenge_scheduler_test.go:330: expected initialDelay to be recalculated on each cycle to avoid clock drift, called only 1 times
    --- FAIL: TestDailyChallengeScheduler_Run_RecalculatesDelayEachCycle (0.03s)
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** Pemformatan template embed dan penjadwalan scheduler bersifat stateless/in-memory, tidak ada data historis database yang mengalami kerusakan atau anomali.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):**
  - HG-1 (Zero Comment Policy): LULUS. Seluruh berkas Go bersih tanpa komentar `//` atau `/* */`.
  - HG-2 (Clean Build & Vet): LULUS. `go vet ./...` dan `go build ./cmd/bot` berhasil tanpa error.
  - HG-3 (Race Free Tests): LULUS. `go test -race ./...` lulus 100%.
  - HG-4 (Working on Main Approved): LULUS. Telah dikonfirmasi dan disetujui langsung pada branch `main`.
- **Must Probe Status (MP-1 s/d MP-7):**
  - MP-1 (Regression check): LULUS. Seluruh suite pengujian lulus 100%.
  - MP-2 (Code Hygiene): LULUS. Zero comments di codebase Go.
  - MP-3 (TDD Red-Green cycle): LULUS. Dibuktikan dengan tangkapan kegagalan sebelum fix dan kelulusan sesudah fix.
  - MP-4 (UTF-8 safety): LULUS. `utf8.ValidString == true` pada seluruh field embed Discord.
  - MP-5 (Scheduler precision): LULUS. Kalkulasi ulang initial delay mencegah akumulasi drift.
  - MP-6 (Context cancellation): LULUS. `timer.Stop()` dipanggil dengan benar saat context dibatalkan.
  - MP-7 (Documentation sync): LULUS. Tiket, Plan, Development Planning, dan CHANGELOG disinkronkan.
- **Daftar Belum Diuji (Untested List - O-16):** Nihil.
