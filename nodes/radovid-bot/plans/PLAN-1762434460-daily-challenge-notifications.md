---
id: PLAN-1762434460
ticket_id: TICKET-1762434460
title: Daily Challenge Notifications
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Daily Challenge Notifications

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434460](../tickets/TICKET-1762434460-daily-challenge-notifications.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun background scheduler berbasis goroutine terkelola (`time.Timer` + `time.Ticker`) untuk mengirimkan pemberitahuan tantangan belajar harian otomatis ke channel Discord setiap pagi, terintegrasi dengan siklus hidup *graceful shutdown* aplikasi.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Goroutine inisiator di `cmd/bot/main.go` saat startup sistem.
  - Komponen Terdampak: `internal/domain/notification`, `internal/usecase/notification`, adapter pengirim pesan Discord, dan konfigurasi lingkungan.
  - Potensi Efek Samping: Penggunaan time ticker berpotensi menimbulkan *goroutine leak* jika channel context tidak di-listen atau ticker tidak di-stop saat aplikasi menerima sinyal terminasi.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/config/config.go` | Penambahan variabel konfigurasi `DAILY_CHALLENGE_HOUR` & `DAILY_CHALLENGE_CHANNEL_ID` |
| `MODIFY` | `../../../../radovid-bot-api/config/config_test.go` | Unit test konfigurasi jam harian dan channel ID tantangan |
| `CREATE` | `../../../../radovid-bot-api/migrations/011_create_daily_challenge_dispatches.sql` | Skema tabel SQLite `daily_challenge_dispatches` untuk idempotensi pengiriman |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/notification/entity.go` | Model DailyChallenge, ChallengeType, DispatchLog, pool tantangan harian deterministik |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/notification/entity_test.go` | Unit test validasi DailyChallenge dan rotasi tantangan harian |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/dispatch_repository.go` | Implementasi SQLite DispatchRepository untuk pengecekan idempotensi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/dispatch_repository_test.go` | Unit test operasi basis data dispatch repository |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Format embed visual `BuildDailyChallengeEmbed` dan tombol `BuildDailyChallengeComponents` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test embed tantangan harian dan tombol Accept Challenge |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/notification_sender.go` | Adapter pengiriman pesan kompleks Discord dan handler interaksi Accept Challenge |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/notification_sender_test.go` | Unit test pengiriman pesan Discord dan validasi interaksi |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler.go` | Background worker scheduler, kalkulasi delay UTC, dan eksekusi dispatch |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/notification/daily_challenge_scheduler_test.go` | Unit test kalkulasi jadwal jam UTC, respons cancel context, dan idempotensi |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Menjalankan scheduler dalam background goroutine dengan sinkronisasi `sync.WaitGroup` dan graceful shutdown |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/usecase/notification/daily_challenge_scheduler_test.go`.
   - Rancang unit test fungsi `CalculateInitialDelay(now time.Time, targetHour int) time.Duration`:
     - Jika waktu saat ini pukul 05:00 UTC dan target pukul 07:00 UTC, delay harus 2 jam.
     - Jika waktu saat ini pukul 09:00 UTC dan target pukul 07:00 UTC, delay harus 22 jam (keesokan harinya).
   - Pastikan tes berstatus RED.
2. **Langkah 2 (Domain & Scheduler Calculation):**
   - Implementasikan fungsi murni `CalculateInitialDelay` di `internal/usecase/notification/daily_challenge_scheduler.go`.
   - Buat struct `DailyChallenge`: `Title`, `Description`, `Category`, `SuggestedCommand`, `DateUTC`.
3. **Langkah 3 (Worker Loop & Graceful Context):**
   - Bangun struktur worker `DailyChallengeScheduler` dengan metode `Start(ctx context.Context, wg *sync.WaitGroup)`.
   - Gunakan `time.NewTimer` untuk initial delay, kemudian beralih ke `time.NewTicker(24 * time.Hour)`.
   - Dengarkan `ctx.Done()` di setiap percabangan `select` untuk menjamin worker berhenti seketika saat aplikasi dimatikan.
   - Panggil `notification_sender.go` untuk mengeksekusi `discordgo.ChannelMessageSendEmbed`.
4. **Langkah 4 (Delivery Gate & Config Binding):**
   - Daftarkan env var baru pada `config/config.go` dengan validasi jam rentang 0-23.
   - Sambungkan lifecycle scheduler ke `main.go` dengan `defer wg.Done()`.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/usecase/notification/daily_challenge_scheduler.go
package notification

import (
	"context"
	"sync"
	"time"
)

func CalculateInitialDelay(now time.Time, targetHour int) time.Duration {
	nowUTC := now.UTC()
	nextTarget := time.Date(nowUTC.Year(), nowUTC.Month(), nowUTC.Day(), targetHour, 0, 0, 0, time.UTC)
	if !nextTarget.After(nowUTC) {
		nextTarget = nextTarget.Add(24 * time.Hour)
	}
	return nextTarget.Sub(nowUTC)
}

func (s *DailyChallengeScheduler) Run(ctx context.Context, wg *sync.WaitGroup) {
	defer wg.Done()
	delay := CalculateInitialDelay(time.Now(), s.targetHour)
	timer := time.NewTimer(delay)
	defer timer.Stop()

	select {
	case <-ctx.Done():
		return
	case <-timer.C:
		s.dispatchChallenge(ctx)
	}

	ticker := time.NewTicker(24 * time.Hour)
	defer ticker.Stop()

	for {
		select {
		case <-ctx.Done():
			return
		case <-ticker.C:
			s.dispatchChallenge(ctx)
		}
	}
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Channel ID tidak valid atau bot kehilangan permission `SendMessages` di text channel target. Error di-log sebagai peringatan tanpa men-crash bot atau menghentikan timer siklus berikutnya.
- **Strategi Rollback:** Pengiriman pesan mencatat timestamp tanggal UTC sukses terakhir di memory/storage; jika bot restart beberapa menit setelah pengiriman, pengecekan tanggal mencegah pesan terkirim ulang.
