---
id: PLAN-1762433980
ticket_id: TICKET-1762433980
title: /radovid progress Command
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: /radovid progress Command

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762433980](../tickets/TICKET-1762433980-radovid-progress-command.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun handler Discord slash command `/radovid progress`, use case `GetProgressSummary`, utilitas render Unicode progress bar, dan Discord embed builder untuk menyajikan statistik belajar pengguna secara menyeluruh.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord Interaction router (`internal/delivery/discord/handler.go`).
  - Komponen Terdampak: Use case `progress`, repositori user progress dan achievements, Discord embed builder.
  - Potensi Efek Samping: Jika profil pengguna belum ada di basis data (pengguna baru yang langsung mengetik command progress), sistem harus menginisialisasi record dasar tanpa mengembalikan pesan error fatal.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_progress.go` | Handler slash command `/radovid progress` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/progress_bar.go` | Generator string Unicode progress bar |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/progress_bar_test.go` | Unit test visualisasi progress bar |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/get_progress.go` | Use case pengumpulan metrik progres dan achievements pengguna |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/progress/get_progress_test.go` | Unit test query profil progres |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/handler.go` | Registrasi router sub-command `/radovid progress` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Fungsi pembuatan embed visualisasi statistik pengguna |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/delivery/discord/progress_bar_test.go`.
   - Uji pembuatan bar 10 karakter: rasio 0.0 -> `[░░░░░░░░░░] 0%`, rasio 0.5 -> `[█████░░░░░] 50%`, rasio 1.0 -> `[██████████] 100%`.
   - Jalankan `go test` dan buktikan status RED.
2. **Langkah 2 (Implementasi Progress Bar Utilitas):**
   - Implementasikan fungsi murni `RenderProgressBar(current, target int, length int) string`.
   - Tangani proteksi pembagian dengan nol (`target <= 0`) dan pembatasan persentase maksimal 100%.
3. **Langkah 3 (Use Case & Delivery Command Integration):**
   - Buat use case `GetProgressSummary(ctx, discordUserID, username)` yang mengambil data `user_progress` dan `achievements`. Jika user belum terdaftar, lakukan *lazy creation* user & progress default.
   - Buat handler `HandleProgressCommand(s *discordgo.Session, i *discordgo.InteractionCreate)` di `radovid_progress.go`.
   - Hubungkan subcommand `progress` pada router slash command di `handler.go`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji tampilan visual embed dengan avatar user Discord, warna embed dinamis berdasarkan level (misal: Hijau Pemula, Biru Menengah, Emas Ahli).
   - Pastikan response time Discord interaction di bawah 3 detik (langsung jawab tanpa perlu deferral interaction bila query SQLite cepat).

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/delivery/discord/progress_bar.go
package discord

import (
	"fmt"
	"strings"
)

func RenderProgressBar(current, target int, length int) string {
	if target <= 0 {
		return fmt.Sprintf("[%s] 0%%", strings.Repeat("░", length))
	}
	if current < 0 {
		current = 0
	}
	ratio := float64(current) / float64(target)
	if ratio > 1.0 {
		ratio = 1.0
	}
	filledCount := int(ratio * float64(length))
	emptyCount := length - filledCount
	percentage := int(ratio * 100)

	return fmt.Sprintf("[%s%s] %d%%", strings.Repeat("█", filledCount), strings.Repeat("░", emptyCount), percentage)
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pengguna dengan total XP sangat besar atau level tinggi tidak boleh menyebabkan perhitungan persentase level saat ini bernilai negatif atau `NaN`.
- **Strategi Rollback:** Jika terjadi galat pembacaan database, kirimkan respons interaksi ephemeral Discord dengan pesan kesalahan ramah (*friendly error*) tanpa mematikan sesi gateway bot.
