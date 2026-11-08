---
id: PLAN-1762520360
ticket_id: TICKET-1762520360
title: "Fix Multiplayer Lobby Starting Prematurely with Single Player on Host Re-invocation"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix Multiplayer Lobby Premature Single-Player Start

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520360](../tickets/TICKET-1762520360-fix-multiplayer-lobby-single-player-start.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #7)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Testing](../../../global-guidelines/testing.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mencegah sesi lobby multiplayer (`would_you_rather`, `taboo`, `twenty_questions`) beralih ke status `active` hanya dengan 1 pemain ketika Host mengeksekusi ulang perintah `/radovid play <game>`.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `handlePlayWYR` (`command_play_wyr.go`), `handlePlayTaboo` (`command_play_taboo.go`), `handlePlay20Q` (`command_play_20q.go`).
  - Komponen Terdampak: `internal/delivery/discord/command_play_wyr.go`, `command_play_taboo.go`, `command_play_20q.go`.
  - Potensi Efek Samping: Untuk 20 Questions (yang mendukung solo atau co-op), jika Host memanggil ulang saat masih `pending` dengan 1 pemain, perlu konsisten dengan perilaku lobby multiplayer atau ditangani dengan jelas.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Menolak `ErrParticipantAlreadyExists` dengan pesan ephemeral bahwa pemain sudah di lobby menunggu lawan |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go` | Menolak `ErrParticipantAlreadyExists` dengan pesan ephemeral bahwa pemain sudah di lobby menunggu lawan |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go` | Menolak `ErrParticipantAlreadyExists` dengan pesan ephemeral bahwa pemain sudah di lobby menunggu lawan |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go` | Menambahkan unit test ketika Host menjalankan perintah kedua kali di lobby pending |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go` | Menambahkan unit test ketika Host menjalankan perintah kedua kali di lobby pending |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test di `command_play_wyr_test.go` dan `command_play_taboo_test.go` di mana `host-1` memanggil `/radovid play` dua kali berturut-turut.
   - Jalankan `go test` dan buktikan tes **GAGAL (RED)** karena sesi berubah menjadi `active` dengan 1 pemain pada kode lama.
2. **Langkah 2 (Implementasi Inti):**
   - Di `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go`: ketika `errors.Is(joinErr, session.ErrParticipantAlreadyExists)`, kirimkan pesan ephemeral *"You have already joined this lobby. Waiting for another player to run the command to start!"* dan hentikan eksekusi sebelum `StartSession`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Jalankan seluruh unit test di `internal/delivery/discord/...` hingga 100% PASS.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
if joinErr := h.sessionManager.JoinSession(ctx, activeSess.ID, userID); joinErr != nil {
	if errors.Is(joinErr, session.ErrParticipantAlreadyExists) {
		return respondEphemeralError(s, i, "You are already in the lobby! Waiting for another player to join.")
	}
	return respondEphemeralError(s, i, fmt.Sprintf("Failed to join session: %v", joinErr))
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain kedua dengan `userID` berbeda tetap dapat bergabung secara normal dan langsung memulai sesi (`StartSession`).
- **Strategi Rollback:** Revert perubahan pada `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go`.

