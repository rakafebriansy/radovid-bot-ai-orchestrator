---
id: PLAN-1762520120
ticket_id: TICKET-1762520120
title: "Fix Streak Calendar Day Validation in Progress Domain"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix Streak Calendar Day Validation

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520120](../tickets/TICKET-1762520120-fix-streak-calendar-day-validation.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #3)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Database Date/Time](../../../global-guidelines/database.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Memperbaiki algoritma `EvaluateStreak` di `internal/domain/progress/streak.go` agar menghitung selisih tanggal kalender UTC secara presisi (`daysDiff`), sehingga pengguna yang melewatkan satu hari kalender penuh (meskipun jarak jam `< 48 jam`, seperti Senin 23:50 UTC ke Rabu 00:10 UTC) tetap mengalami reset streak ke `1`.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `RecordDailyActivityUseCase.RecordDailyActivity` di `internal/usecase/progress/record_daily_activity.go`.
  - Komponen Terdampak: `internal/domain/progress/streak.go`, `internal/domain/progress/streak_test.go`.
  - Potensi Efek Samping: Tetap mempertahankan aturan bahwa selisih waktu `>= 48 jam` pada kondisi apapun juga memicu `StreakReset`.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/progress/streak.go` | Menambahkan normalisasi tanggal kalender UTC (`Truncate(24 * time.Hour)`) untuk memvalidasi hari berturut-turut (`calendarDaysDiff == 1`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/progress/streak_test.go` | Menambahkan test case batas lintas tengah malam yang melewati 1 hari kalender dalam kurun `< 48 jam` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Tambahkan kasus uji `Skipped calendar day within 25 hours (Mon 23:50 to Wed 00:10)` di `streak_test.go`.
   - Jalankan `go test ./internal/domain/progress/...` dan buktikan tes **GAGAL (RED)** karena mengembalikan `StreakIncremented` (6) alih-alih `StreakReset` (1).
2. **Langkah 2 (Implementasi Inti):**
   - Di `EvaluateStreak`, hitung awal hari UTC (`time.Date(y, m, d, 0, 0, 0, 0, time.UTC)`) untuk `lastTime` dan `nowTime`.
   - Jika tanggal sama (`daysDiff == 0`), kembalikan `StreakSameDay`.
   - Jika `diff >= streakResetThreshold` ATAU `daysDiff > 1`, kembalikan `1, 0, StreakReset`.
   - Jika `daysDiff == 1` dan `diff < streakResetThreshold`, kembalikan `currentStreak + 1, bonusXP, StreakIncremented`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh test existing dan baru di `internal/domain/progress` dan `internal/usecase/progress` lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
lastDayUTC := time.Date(lastTime.Year(), lastTime.Month(), lastTime.Day(), 0, 0, 0, 0, time.UTC)
nowDayUTC := time.Date(nowTime.Year(), nowTime.Month(), nowTime.Day(), 0, 0, 0, 0, time.UTC)
daysDiff := int(nowDayUTC.Sub(lastDayUTC) / (24 * time.Hour))

if daysDiff == 0 {
	return currentStreak, 0, StreakSameDay
}

if diff >= streakResetThreshold || daysDiff > 1 {
	return 1, 0, StreakReset
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Tahun kabisat (28 Feb -> 1 Mar pada tahun non-kabisat vs 28 Feb -> 29 Feb -> 1 Mar pada tahun kabisat) ditangani secara akurat oleh `time.Date` UTC.
- **Strategi Rollback:** Revert perubahan pada `streak.go`.

