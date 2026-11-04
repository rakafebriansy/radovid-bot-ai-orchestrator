---
id: PLAN-1762433860
ticket_id: TICKET-1762433860
title: Streak Tracking (Daily Activity)
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Streak Tracking (Daily Activity)

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762433860](../tickets/TICKET-1762433860-streak-tracking.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengimplementasikan domain rule pelacakan streak aktivitas harian pengguna, pendeteksian kedaluwarsa interval 48 jam, dan pemicu bonus XP harian tanpa menimbulkan balapan data (*data race*) antar event interaksi.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Selesai game session, submit daily diary, atau penyelesaian daily challenge.
  - Komponen Terdampak: `internal/domain/progress`, `internal/usecase/progress`, `internal/infrastructure/sqlite/progress_repository.go`.
  - Potensi Efek Samping: Selisih perhitungan jam harus didasarkan pada perbandingan tanggal UTC murni atau interval waktu epoch millisecond untuk memastikan pengguna global tidak dirugikan oleh perbedaan offset lokal.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/streak.go` | Domain service kalkulasi streak dan status aktivitas harian |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/streak_test.go` | Unit test variasi interval waktu streak (Same Day, Consecutive, Expired) |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/record_daily_activity.go` | Use case evaluasi streak dan orkestrasi bonus XP |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/record_daily_activity_test.go` | Unit test flow update streak dengan mock repository |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/progress_repository.go` | Penambahan method update streak dan last_activity_at |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/domain/progress/streak_test.go`.
   - Siapkan skenario matriks waktu: aktivitas berjarak 2 jam di hari yang sama, aktivitas berjarak 25 jam (hari berikutnya), aktivitas berjarak 49 jam (>48 jam).
   - Pastikan tes gagal (RED).
2. **Langkah 2 (Domain Implementation):**
   - Definisikan tipe `StreakEvaluationResult`: `NewStreak int`, `BonusXP int`, `Status StreakStatus` (StatusSameDay, StatusIncremented, StatusReset).
   - Terapkan fungsi `EvaluateStreak(currentStreak int, lastActivityAt int64, now int64) StreakEvaluationResult`.
   - Gunakan perbandingan hari kalender UTC: jika `now` dan `lastActivityAt` jatuh pada tanggal UTC yang sama -> StatusSameDay. Jika `now - lastActivityAt > 48 * time.Hour` -> StatusReset (streak kembali ke 1). Jika beda 1 hari kalender UTC -> StatusIncremented.
3. **Langkah 3 (Use Case Integration):**
   - Buat usecase `RecordDailyActivity(ctx context.Context, userID string, now int64) (*ActivityResult, error)`.
   - Ambil data progres terkini, hitung streak, simpan perubahan nilai streak & `last_activity_at`.
   - Jika ada `BonusXP > 0`, delegasikan ke use case `AwardXP` dengan source `streak_bonus`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Verifikasi boundary test: tepat 48 jam (172.800.000 ms) vs 48 jam 1 ms.
   - Periksa pemanggilan konkurensi (optimistic lock / serial write) di SQLite adapter.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/progress/streak.go
package progress

import "time"

type StreakStatus int

const (
	StreakSameDay StreakStatus = iota
	StreakIncremented
	StreakReset
)

const StreakBonusMultiplier = 5

func EvaluateStreak(currentStreak int, lastActivityAt int64, now int64) (int, int, StreakStatus) {
	if lastActivityAt == 0 {
		return 1, 1 * StreakBonusMultiplier, StreakIncremented
	}

	lastTime := time.UnixMilli(lastActivityAt).UTC()
	nowTime := time.UnixMilli(now).UTC()

	diff := nowTime.Sub(lastTime)
	if diff > 48*time.Hour {
		return 1, 1 * StreakBonusMultiplier, StreakReset
	}

	lastYear, lastMonth, lastDay := lastTime.Date()
	nowYear, nowMonth, nowDay := nowTime.Date()

	if lastYear == nowYear && lastMonth == nowMonth && lastDay == nowDay {
		return currentStreak, 0, StreakSameDay
	}

	newStreak := currentStreak + 1
	return newStreak, newStreak * StreakBonusMultiplier, StreakIncremented
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Nilai `now` yang lebih lampau daripada `last_activity_at` (akibat sinkronisasi clock NTP mundur) harus ditangani dengan mengembalikan status `StreakSameDay` tanpa melakukan mutasi streak atau memberikan bonus ganda.
- **Strategi Rollback:** Jika kalkulasi streak gagal disimpan ke database, pembatalan transaksi menjaga `last_activity_at` tetap pada nilai sebelumnya sehingga pengguna tidak kehilangan kesempatan menjaga streak di percobaan ulang.
