---
id: PLAN-1762520240
ticket_id: TICKET-1762520240
title: "Fix Spectator Button Hijack on Would You Rather Round Controls"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix Spectator Button Hijack on WYR

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520240](../tickets/TICKET-1762520240-fix-wyr-spectator-button-hijack.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #5)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Security Guidelines](../../../global-guidelines/security.md) | [Coding Standards](../../../global-guidelines/coding.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menambahkan pemeriksaan otorisasi kepemilikan/kepesertaan sesi pada aksi tombol `wyr:next:` dan `wyr:finish:` di `internal/delivery/discord/interaction_wyr_button.go` sehingga hanya pemain yang terdaftar di `session.Participants` yang dapat melanjutkan ronde berikutnya atau mengakhiri sesi permainan.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `HandleWYRButton` -> `GameHandler.handleWYRButton`.
  - Komponen Terdampak: `internal/delivery/discord/interaction_wyr_button.go`, `internal/delivery/discord/command_play_wyr_test.go`.
  - Potensi Efek Samping: Tidak ada; aksi `vote_a` dan `vote_b` sudah memvalidasi partisipan di dalam `wyrUseCase.RecordVote`, sehingga pemeriksaan ini menyelaraskan perilaku `next` dan `finish` dengan `vote`.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go` | Memvalidasi bahwa `userID` terdaftar sebagai participant sesi sebelum mengeksekusi `next` atau `finish` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go` | Menambahkan unit test penolakan klik tombol `wyr:next` dan `wyr:finish` oleh spectator non-peserta |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Buat unit test di `command_play_wyr_test.go` di mana sesi WYR dibuat oleh `user-1` dan `user-2`, lalu `spectator-99` menekan tombol `wyr:finish:<sessionID>`.
   - Jalankan `go test` dan buktikan tes **GAGAL (RED)** karena sesi justru berakhir pada kode lama.
2. **Langkah 2 (Implementasi Inti):**
   - Di `interaction_wyr_button.go`, sebelum mengeksekusi cabang `action == "next"` atau `action == "finish"`, ambil data sesi dari `h.sessionManager` (atau helper `isSessionParticipant`) dan periksa apakah `userID` ada di dalam `sess.Participants`.
   - Jika bukan peserta, kirimkan respons ephemeral `discordInfra.ErrorEmbed("Access Denied", "Only active participants in this game session can control the round.")`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh unit test WYR lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
sess, err := h.sessionManager.GetSession(ctx, sessionID)
if err != nil {
	return respondEphemeralError(s, i, "Session not found.")
}
if !isParticipant(sess.Participants, userID) {
	return respondEphemeralError(s, i, "Only participants of this game session can use this button.")
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Sesi sudah selesai atau dibatalkan sebelum tombol ditekan ditangani dengan pesan error ephemeral yang ramah pengguna.
- **Strategi Rollback:** Revert perubahan pada `interaction_wyr_button.go`.

