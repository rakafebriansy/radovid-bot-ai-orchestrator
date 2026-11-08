---
id: PLAN-1762520540
ticket_id: TICKET-1762520540
title: "Fix UTF-8 Rune Truncation in Discord Templates & Eliminate Scheduler Clock Drift"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix UTF-8 Rune Truncation & Scheduler Clock Drift

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520540](../tickets/TICKET-1762520540-fix-utf8-truncation-and-scheduler-drift.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #10)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [UI & Assets](../../../global-guidelines/ui-and-assets.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:**
  1. Mengganti seluruh operasi byte slicing (`str[:max] + "..."`) di `internal/infrastructure/discord/game_templates.go` dengan helper berbasis rune `truncateString(str, maxLen)` agar karakter multibyte UTF-8 tidak pernah terbelah.
  2. Memperbarui loop `DailyChallengeScheduler.Run` di `internal/usecase/notification/daily_challenge_scheduler.go` agar menghitung ulang delay ke target jam UTC berikutnya di setiap iterasi apabila menggunakan jadwal harian standar (`customInterval <= 0`).
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: Fungsi builder embed di `game_templates.go` dan worker `DailyChallengeScheduler.Run`.
  - Komponen Terdampak: `internal/infrastructure/discord/game_templates.go`, `internal/usecase/notification/daily_challenge_scheduler.go`.
  - Potensi Efek Samping: Tetap mendukung `customInterval > 0` yang digunakan oleh unit test untuk pengujian siklus cepat milidetik.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Mengganti seluruh pemotongan byte mentah dengan `truncateString` berbasis `[]rune` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Menambahkan unit test validasi `utf8.ValidString` pada pemotongan string berisi karakter emoji multibyte |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler.go` | Menghitung ulang `CalculateInitialDelay` pada setiap siklus harian ketika `customInterval <= 0` |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler_test.go` | Menambahkan unit test verifikasi kalkulasi ulang jadwal harian bebas drift |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test di `game_templates_test.go` yang menyusun string sepanjang 495 karakter ASCII diikuti emoji 4-byte `🔥` (`\xf0\x9f\x94\xa5`), lalu memanggil `BuildDreamHouseSummaryEmbed` dan memeriksa `utf8.ValidString` pada field embed.
   - Jalankan `go test ./internal/infrastructure/discord/...` dan buktikan tes **GAGAL (RED)** karena `preview[:497]` memotong emoji `🔥` menjadi 2 byte rusak.
2. **Langkah 2 (Implementasi Inti):**
   - Ganti seluruh 12 lokasi pemotongan byte mentah di `game_templates.go` dengan panggilan `truncateString(..., max)`.
   - Perbarui `DailyChallengeScheduler.Run` agar menggunakan `timer.Reset(nextDelay)` berbasis `CalculateInitialDelay(s.nowFunc(), s.targetHour)` ketika `s.customInterval <= 0`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh test di `internal/infrastructure/discord` dan `internal/usecase/notification` lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
exteriorPreview := truncateString(strings.TrimSpace(data.ExteriorConcept), 300)
interiorPreview := truncateString(strings.TrimSpace(data.InteriorConcept), 300)
sanctuaryPreview := truncateString(strings.TrimSpace(data.SanctuaryConcept), 300)
synthesisText := truncateString(strings.TrimSpace(data.SynthesisNarration), 900)
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Teks yang seluruhnya terdiri dari emoji atau karakter CJK (Chinese/Japanese/Korean) dipotong tepat pada batas jumlah rune tanpa merusak encoding UTF-8.
- **Strategi Rollback:** Revert perubahan pada `game_templates.go` dan `daily_challenge_scheduler.go`.

