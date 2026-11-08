---
id: PLAN-1762433800
ticket_id: TICKET-1762433800
title: XP Calculation & Level System
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: XP Calculation & Level System

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762433800](../tickets/TICKET-1762433800-xp-calculation-level-system.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun domain service kalkulasi level kuadratik, skema logging mutasi XP (`user_xp_log`), dan use case `AwardXP` dengan integritas transaksional SQLite guna mendukung sistem progres gamifikasi Radovid Bot.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler akhir sesi game (`internal/delivery/discord`), modul feedback diary harian, dan streak bonus processor.
  - Komponen Terdampak: `internal/domain/progress`, `internal/usecase/progress`, skema database `user_progress`, serta tabel audit baru `user_xp_log`.
  - Potensi Efek Samping: Operasi write ke SQLite harus berjalan dalam transaksi atomik (`BEGIN IMMEDIATE TRANSACTION`) guna menghindari inkonsistensi saldo XP dan status level jika terjadi kegagalan proses.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/migrations/009_create_user_xp_log.sql` | Migration DDL tabel riwayat transaksi XP |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/entity.go` | Definisi struct XPLog, UserProgress, Difficulty multiplier |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/calculator.go` | Logika pure Go untuk kalkulasi level, threshold XP, dan multiplier |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/repository.go` | Port interface untuk persistensi progres dan XP log |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/progress/calculator_test.go` | Unit test komprehensif rumus level dan multiplier |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/award_xp.go` | Use case orkestrasi penambahan XP, deteksi level-up, dan audit log |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/award_xp_test.go` | Unit test use case penambahan XP dengan mock repository |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/sqlite/progress_repository.go` | Implementasi query atomik update progress dan insert XP log |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat berkas pengujian kalkulator level `internal/domain/progress/calculator_test.go`.
   - Definisikan ekspektasi konversi XP ke Level: 0-99 XP = Level 1, 100-399 XP = Level 2, 400-899 XP = Level 3, dst.
   - Jalankan `go test ./...` dan pastikan hasil RED (compile error karena simbol belum ada).
2. **Langkah 2 (Implementasi Domain & Calculator):**
   - Implementasikan fungsi murni `CalculateLevel(totalXP int) int`, `XPRequiredForLevel(level int) int`, dan `ApplyMultiplier(baseXP int, diff Difficulty) int` pada `calculator.go`.
   - Buat migrasi SQL `009_create_user_xp_log.sql` dengan indeks foreign key ke `users(id)`.
3. **Langkah 3 (Use Case & Transaksi SQLite):**
   - Susun interface `ProgressRepository` dengan metode atomik `AddXPAndLog(ctx context.Context, log XPLog, newTotalXP int, newLevel int) error`.
   - Implementasikan transaksi SQLite di repository adapter.
   - Buat use case `AwardXP` yang memvalidasi input, menghitung XP baru, mendeteksi jika terjadi kenaikan level (`newLevel > oldLevel`), lalu menyimpan state secara aman.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji batas ekstrem: input XP negatif (harus ditolak), overflow integer, dan lonjakan beberapa level sekaligus (*multi-level jump*).
   - Pastikan Zero-Comment Policy dan tidak ada magic number (gunakan konstanta `BaseLevelMultiplier = 100`).

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/progress/calculator.go
package progress

import "math"

const LevelScaleFactor = 100

func CalculateLevel(totalXP int) int {
	if totalXP <= 0 {
		return 1
	}
	level := int(math.Floor(math.Sqrt(float64(totalXP) / float64(LevelScaleFactor))))
	if level < 1 {
		return 1
	}
	return level
}

func XPRequiredForLevel(level int) int {
	if level <= 1 {
		return 0
	}
	return level * level * LevelScaleFactor
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Nilai XP bernilai negatif atau nol tidak boleh mengubah state dan mengembalikan `ErrInvalidXPAmount`.
- **Strategi Rollback:** Seluruh mutasi `user_progress` dan `user_xp_log` dibungkus dalam blok `tx.BeginTx()`. Bila salah satu operasi gagal, panggil `tx.Rollback()` untuk menjamin data tetap sinkron.
