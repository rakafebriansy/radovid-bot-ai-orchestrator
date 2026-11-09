---
id: PLAN-1794188809
ticket_id: TICKET-1794188809
title: Implementation Plan - Remove Deprecated Radovid Start Command
status: Completed
priority: Medium
---

# Rencana Implementasi: Penghapusan Perintah Legacy `/radovid start`

## 1. Analisis Arsitektur & Depresiasi
Perintah `/radovid start` merupakan artefak dari Fase 1 fondasi bot yang bertindak sebagai skeleton inisialisasi sesi sebelum mini-games dan sistem domain aktif.
Pada arsitektur saat ini:
- Seluruh 15 mini-game memiliki inisialisasi sesi otomatis masing-masing melalui `/radovid play <game>`.
- Fitur solo writing diinisialisasi melalui `/radovid diary`.
- Utilitas pengguna diatur melalui `/radovid settings`, `/radovid progress`, dan `/radovid help`.
Dengan demikian, `/radovid start` tidak lagi memiliki peran fungsional dan keberadaannya berpotensi menyesatkan pengguna akhir.

## 2. Rencana Perubahan Teknis
1. **`internal/delivery/discord/command_def.go`:**
   - Hapus entri subcommand `start` dari daftar opsi `radovid` ApplicationCommand (jumlah subcommand inti berkurang dari 8 menjadi 7).
2. **`internal/delivery/discord/radovid_start.go`:**
   - Hapus berkas ini sepenuhnya.
3. **`internal/delivery/discord/handler.go`:**
   - Hapus `case "start":` dari switch dispatching `HandleRadovidCommand`.
   - Perbarui pesan fallback tanpa opsi menjadi `Please specify a subcommand, e.g., /radovid help or /radovid play`.
4. **`internal/delivery/discord/radovid_help.go`:**
   - Hapus baris `"` + "`/radovid start` — Initialize a new English learning session.\n" dari `BuildHelpEmbed()`.
5. **`internal/delivery/discord/command_test.go` & `radovid_help_test.go`:**
   - Perbarui ekspektasi jumlah subcommands di `command_test.go` menjadi 7 dan hapus pengujian `start`.
   - Perbarui `radovid_help_test.go` untuk memverifikasi bahwa `/radovid start` tidak lagi muncul di embed.
6. **`nodes/radovid-bot/docs/manual-testing-playbook.md`:**
   - Sesuaikan judul & tabel dari 21 menjadi 20 perintah.
   - Hapus seksi 4.2 (`/radovid start`) dan re-number subseksi di seksi 4.

## 3. Rencana Pengujian TDD
1. **Fase RED:** Ubah ekspektasi `command_test.go` dan `radovid_help_test.go` untuk menghapus `start` dan menuntut tidak adanya `start`. Jalankan test untuk membuktikan kegagalan (**RED**).
2. **Fase GREEN:** Terapkan pembersihan kode pada `command_def.go`, `handler.go`, `radovid_help.go`, dan hapus `radovid_start.go`. Jalankan test (**GREEN**).
3. Jalankan `go test -race ./...` dan `go vet ./...` untuk memastikan seluruh test suite lulus tanpa regresi.
