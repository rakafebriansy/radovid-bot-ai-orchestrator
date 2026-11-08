---
id: TICKET-1762520360
plan_id: PLAN-1762520360
title: "[BUG] Lobby Multiplayer Memulai Game 2-Pemain dengan 1 Pemain Saat Host Menjalankan Command Kedua Kali"
status: Done
priority: Medium
severity: Broken Flow
labels: [Bug, Discord, Multiplayer, Session, Game]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Pada tiga game multiplayer (Would You Rather, Taboo, dan 20 Questions), ketika terdapat sesi berstatus `pending` (`SessionStatusWaitingForPlayers`) di channel dan Host (pembuat sesi) menjalankan kembali perintah `/radovid play <game>`, metode `JoinSession` mengembalikan `session.ErrParticipantAlreadyExists`. Namun, handler delivery di `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go` mengabaikan error tersebut (`if joinErr != nil && !errors.Is(joinErr, session.ErrParticipantAlreadyExists)`) dan langsung mengeksekusi `StartSession`. Akibatnya, sesi multiplayer dimulai paksa hanya dengan 1 pemain (Host sendiri), merusak mekanisme giliran lawan dan mengunci pemain kedua di luar.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520360-fix-multiplayer-lobby-single-player-start.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. User A menjalankan `/radovid play taboo` di channel Discord (lobby `pending` terbentuk dengan 1 pemain).
2. User A menjalankan kembali `/radovid play taboo` di channel yang sama.
3. Bot mengabaikan `ErrParticipantAlreadyExists` dan langsung memulai sesi `active` hanya dengan User A seorang diri.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Jika Host (atau peserta yang sudah tergabung di lobby `pending`) memanggil ulang perintah `/radovid play <game>` saat jumlah peserta belum mencukupi minimal 2 pemain, bot harus menolak dengan pesan informasi ephemeral bahwa pengguna sudah berada di lobby dan masih menunggu pemain lawan untuk bergabung.
- **Aktual (Actual):** Sesi 2 pemain langsung dimulai dengan 1 pemain saja.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] Ketika `JoinSession` mengembalikan `session.ErrParticipantAlreadyExists`, handler tidak memulai sesi jika jumlah peserta `< 2`, melainkan mengembalikan pesan ephemeral bahwa pemain sudah terdaftar di lobby dan sedang menunggu lawan.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`

---

## 6. AI Execution Log & Output

- **Penyebab Akar (Root Cause):**
  - Pada `command_play_wyr.go:286`, `command_play_taboo.go:224`, dan `command_play_20q.go:105`, kondisi error joining sengaja meloloskan `session.ErrParticipantAlreadyExists` (`if joinErr != nil && !errors.Is(joinErr, session.ErrParticipantAlreadyExists)`). Ketika host atau pemain yang telah bergabung memanggil ulang slash command, eksekusi terus berlanjut ke `StartSession` sehingga game multiplayer 2-pemain dimulai hanya dengan 1 orang.
- **Langkah Perbaikan:**
  1. Pada `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go`, cegah kelanjutan eksekusi ke `StartSession` jika `joinErr != nil`. Apabila `errors.Is(joinErr, session.ErrParticipantAlreadyExists)`, kirimkan pesan error ephemeral: `"You have already joined this lobby! Waiting for another player to join."`.
  2. Menambahkan unit test TDD (O-07) pada `command_play_wyr_test.go`, `command_play_taboo_test.go`, dan `command_play_20q_test.go` yang memvalidasi bahwa re-invokasi host menolak transisi game ke `active`, mempertahankan sesi pada `pending`, dan baru bertransisi ke `active` saat pemain kedua sah mengeksekusi perintah.
- **Ringkasan Berkas Terdampak:**
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr.go`: Menolak `ErrParticipantAlreadyExists` secara ephemeral pada lobby Would You Rather.
  - `radovid-bot-api/internal/delivery/discord/command_play_taboo.go`: Menolak `ErrParticipantAlreadyExists` secara ephemeral pada lobby Taboo.
  - `radovid-bot-api/internal/delivery/discord/command_play_20q.go`: Menolak `ErrParticipantAlreadyExists` secara ephemeral pada lobby 20 Questions.
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`: Unit test re-invokasi host WYR.
  - `radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`: Unit test re-invokasi host Taboo.
  - `radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`: Unit test re-invokasi host 20Q.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -v -run "TestHandlePlay(WYR|Taboo|TQ)_HostReinvocation" ./internal/delivery/discord/...`
- **Output Terminal Aktual:**
  ```text
  === RUN   TestHandlePlayTQ_HostReinvocationDoesNotStartSinglePlayer
  --- PASS: TestHandlePlayTQ_HostReinvocationDoesNotStartSinglePlayer (0.00s)
  === RUN   TestHandlePlayTaboo_HostReinvocationDoesNotStartSinglePlayer
  --- PASS: TestHandlePlayTaboo_HostReinvocationDoesNotStartSinglePlayer (0.00s)
  === RUN   TestHandlePlayWYR_HostReinvocationDoesNotStartSinglePlayer
  --- PASS: TestHandlePlayWYR_HostReinvocationDoesNotStartSinglePlayer (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.502s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `cd radovid-bot-api && go test -v -run TestHandlePlayWYR_HostReinvocationDoesNotStartSinglePlayer ./internal/delivery/discord/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestHandlePlayWYR_HostReinvocationDoesNotStartSinglePlayer
        command_play_wyr_test.go:210: vulnerability reproduced: session was started with only 1 player (host)
    --- FAIL: TestHandlePlayWYR_HostReinvocationDoesNotStartSinglePlayer (0.00s)
    FAIL
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** Masalah hanya terjadi pada state transisi lobby runtime dan tidak memengaruhi skema atau merusak data historis di SQLite.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):** PASSED. Build bot berhasil (`go build ./cmd/bot`), tidak ada lint/vet error (`go vet ./...`), seluruh test suite lulus dengan race detector (`go test -race ./...`).
- **Must Probe Status (MP-1 s/d MP-7):** PASSED. Seluruh skenario lobby multiplayer (1 pemain mencoba start ulang ditolak, 2 pemain berhasil start) teruji penuh pada 3 varian game (WYR, Taboo, 20Q).
- **Daftar Belum Diuji (Untested List - O-16):** NONE.
