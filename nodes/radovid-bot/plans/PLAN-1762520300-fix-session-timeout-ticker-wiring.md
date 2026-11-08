---
id: PLAN-1762520300
ticket_id: TICKET-1762520300
title: "Wire Session TimeoutTicker in main.go & Auto-Expire Stale Channel Sessions"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Wire Session TimeoutTicker & Auto-Expire Stale Sessions

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520300](../tickets/TICKET-1762520300-fix-session-timeout-ticker-wiring.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #6)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Testing](../../../global-guidelines/testing.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengintegrasikan `TimeoutTicker` ke dalam siklus hidup aplikasi di `cmd/bot/main.go` serta menambahkan evaluasi kedaluwarsa otomatis pada `SessionManager.CreateSession` agar sesi lama yang ditinggalkan pengguna (> 30 menit) tidak mengunci channel Discord selamanya.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `cmd/bot/main.go` dan `SessionManager.CreateSession` di `internal/usecase/session/session_manager.go`.
  - Komponen Terdampak: `cmd/bot/main.go`, `internal/usecase/session/session_manager.go`, `internal/usecase/session/session_manager_test.go`.
  - Potensi Efek Samping: Saat bot baru dinyalakan kembali setelah mati, menjalankan `CleanupExpiredSessions` di awal serta di dalam `CreateSession` menjamin channel langsung bersih tanpa harus menunggu tick pertama.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi `sessionUsecase.NewTimeoutTicker`, memanggil `Start(schedulerCtx)`, dan `Stop()` saat graceful shutdown |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/session/session_manager.go` | Membatalkan sesi yang sudah melewati batas usia kedaluwarsa (`DefaultSessionTTL = 30 * time.Minute`) saat `CreateSession` memeriksa sesi aktif di channel |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/session/session_manager_test.go` | Menambahkan unit test pembuatan sesi baru pada channel yang memiliki sesi menggantung yang sudah kedaluwarsa |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Tambahkan unit test di `session_manager_test.go` di mana sebuah sesi dibuat dengan `CreatedAt` 2 jam yang lalu dan masih berstatus `active`, kemudian `CreateSession` dipanggil untuk channel yang sama.
   - Jalankan `go test ./internal/usecase/session/...` dan buktikan tes **GAGAL (RED)** karena mengembalikan `ErrActiveSessionAlreadyInChannel`.
2. **Langkah 2 (Implementasi Inti):**
   - Pada `SessionManager.CreateSession`, jika `existing != nil`, periksa apakah `now - existing.CreatedAt >= DefaultSessionTTL` (30 menit). Jika sudah kedaluwarsa, batalkan sesi lama tersebut via `UpdateSessionStatus` ke `SessionStatusCancelled` dan izinkan pembuatan sesi baru.
   - Pada `cmd/bot/main.go`, inisialisasi `TimeoutTicker` dengan interval 1 menit dan timeout 30 menit, jalankan `timeoutTicker.Start(schedulerCtx)`, dan panggil `timeoutTicker.Stop()` saat shutdown.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh test di `internal/usecase/session` lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
if existing != nil {
	if now-existing.CreatedAt >= DefaultSessionTimeoutMs {
		_ = m.repo.UpdateSessionStatus(ctx, existing.ID, existing.Status, domain.SessionStatusCancelled, &now)
	} else {
		return nil, domain.ErrActiveSessionAlreadyInChannel
	}
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Sesi dengan `CreatedAt` di masa depan atau baru berjalan `< 30 menit` tetap dilindungi dan tidak dibatalkan.
- **Strategi Rollback:** Revert perubahan pada `session_manager.go` dan `main.go`.

