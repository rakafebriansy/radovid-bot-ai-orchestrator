---
id: TICKET-1762520240
plan_id: PLAN-1762520240
title: "[BUG] Pembajakan Giliran oleh Penonton (Spectator Turn Hijack) pada Tombol Would You Rather"
status: Done
priority: Medium
severity: Security/Privacy
labels: [Bug, Security, Discord, Game, Authorization]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Pada `internal/delivery/discord/interaction_wyr_button.go` baris 91-132, handler interaksi tombol komponen `wyr:next:<sessionID>` (Next Round) dan `wyr:finish:<sessionID>` (Finish Game) tidak memvalidasi apakah pengguna Discord yang menekan tombol (`extractUserID(i)`) merupakan peserta sah yang terdaftar di dalam sesi permainan tersebut (`session.Participants`). Akibatnya, penonton luar (*spectator*) di text channel publik yang sama dapat membajak jalannya permainan dengan melompati ronde atau mengakhiri sesi permainan milik pemain lain secara sepihak.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762520240-fix-wyr-spectator-button-hijack.md)

## 2. Langkah Reproduksi (Steps to Reproduce)
1. Pemain A dan Pemain B memulai sesi permainan Would You Rather dan menyelesaikan voting ronde 3.
2. Tombol `Next Round` dan `Finish Game` muncul pada pesan embed di channel publik.
3. Pengguna C (bukan peserta sesi) menekan tombol `Finish Game` (`wyr:finish:<sessionID>`).
4. Bot langsung menghentikan permainan milik Pemain A dan B.

## 3. Kondisi Ekspektasi vs Aktual
- **Ekspektasi (Expected):** Hanya peserta yang terdaftar di dalam `session.Participants` pada sesi WYR aktif yang diizinkan menekan tombol `Next Round` dan `Finish Game`. Penonton luar harus ditolak dengan pesan ephemeral.
- **Aktual (Actual):** Bot menerima klik tombol `wyr:next:` dan `wyr:finish:` dari user manapun di channel tanpa memeriksa daftar partisipan sesi.

---

## 4. Kriteria Penyelesaian (Acceptance Criteria)
- [x] Aksi `wyr:next:` dan `wyr:finish:` memverifikasi bahwa `userID` pengklik tombol adalah peserta sah dalam sesi tersebut (`session.Participants`).
- [x] Jika pengguna bukan peserta sesi, bot menolak interaksi dengan pesan error ephemeral tanpa mengubah state ronde atau mengakhiri sesi.
- [x] **Unit test baru ditambahkan dan TERBUKTI GAGAL pada kode lama sebelum fix (O-07).**
- [x] Seluruh test suite lulus 100% tanpa regresi fitur lain (*no regression*).
- [x] Pemeriksaan kerusakan data historis telah tuntas (Damage Assessment - O-09).
- [x] Lolos verifikasi penuh [Delivery Gate](../../../global-guidelines/delivery-gate.md).
- [x] Pesan komit steril dari istilah orchestrator pada repositori Project/Node.

## 5. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go`

---

## 6. AI Execution Log & Output

- **Penyebab Akar (Root Cause):**
  - Pada `HandleButtonInteraction` (`internal/delivery/discord/interaction_wyr_button.go`), percabangan `case "next":` dan `case "finish":` langsung memanggil `h.wyrUseCase.StartRound` atau `h.wyrUseCase.CompleteSession` tanpa memeriksa apakah `userID` pengguna yang menekan tombol terdaftar di dalam `session.Participants`.
- **Langkah Perbaikan:**
  1. Menambahkan method `IsParticipant(ctx context.Context, sessionID, userID string) (bool, error)` pada `WouldYouRatherUseCase` (`internal/usecase/game/would_you_rather.go`) untuk memverifikasi keikutsertaan via `sessionRepo.GetParticipants`.
  2. Menambahkan helper `isWYRParticipant(ctx context.Context, sessionID, userID string) bool` pada `GameHandler` (`internal/delivery/discord/interaction_wyr_button.go`) yang menginspeksi peserta sesi melalui `sessionManager` dan/atau `wyrUseCase`.
  3. Memasang guard `if !h.isWYRParticipant(ctx, sessionID, userID)` pada aksi `next` dan `finish` sebelum ronde dimajukan atau sesi diselesaikan, mengembalikan pesan ephemeral `"Only active participants in this game session can control the round."` jika interaksi berasal dari pengguna luar (spectator).
  4. Menambahkan test suite `TestHandleButtonInteraction_SpectatorRejected` di `interaction_wyr_button_test.go` dan `TestWouldYouRatherUseCase_IsParticipant` di `would_you_rather_test.go`.
- **Ringkasan Berkas Terdampak:**
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`: Menambahkan otorisasi participant pada `case "next"` dan `case "finish"` serta helper `isWYRParticipant`.
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`: Menambahkan test penolakan interaksi tombol oleh spectator serta verifikasi partisipan sah dapat melanjutkan/mengakhiri sesi.
  - `radovid-bot-api/internal/usecase/game/would_you_rather.go`: Menambahkan method `IsParticipant`.
  - `radovid-bot-api/internal/usecase/game/would_you_rather_test.go`: Menambahkan unit test untuk method `IsParticipant`.

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `cd radovid-bot-api && go test -v -run TestHandleButtonInteraction_SpectatorRejected ./internal/delivery/discord/...`
- **Output Terminal Aktual:**
  ```text
  === RUN   TestHandleButtonInteraction_SpectatorRejected
  === RUN   TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_advance_round_via_next_button
  === RUN   TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_terminate_session_via_finish_button
  === RUN   TestHandleButtonInteraction_SpectatorRejected/participant_can_advance_round_via_next_button
  === RUN   TestHandleButtonInteraction_SpectatorRejected/participant_can_terminate_session_via_finish_button
  --- PASS: TestHandleButtonInteraction_SpectatorRejected (0.00s)
      --- PASS: TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_advance_round_via_next_button (0.00s)
      --- PASS: TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_terminate_session_via_finish_button (0.00s)
      --- PASS: TestHandleButtonInteraction_SpectatorRejected/participant_can_advance_round_via_next_button (0.00s)
      --- PASS: TestHandleButtonInteraction_SpectatorRejected/participant_can_terminate_session_via_finish_button (0.00s)
  PASS
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	0.498s
  ```

### Verifikasi Regresi (Disiplin O-07)
- [x] **Tes baru terbukti GAGAL pada kode lama sebelum perbaikan:**
  - Perintah: `cd radovid-bot-api && go test -v -run TestHandleButtonInteraction_SpectatorRejected ./internal/delivery/discord/...`
  - Pesan Kegagalan:
    ```text
    === RUN   TestHandleButtonInteraction_SpectatorRejected
    === RUN   TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_advance_round_via_next_button
        interaction_wyr_button_test.go:275: vulnerability reproduced: spectator was able to start next round
    === RUN   TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_terminate_session_via_finish_button
        interaction_wyr_button_test.go:289: vulnerability reproduced: spectator was able to finish session
    --- FAIL: TestHandleButtonInteraction_SpectatorRejected (0.00s)
        --- FAIL: TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_advance_round_via_next_button (0.00s)
        --- FAIL: TestHandleButtonInteraction_SpectatorRejected/spectator_cannot_terminate_session_via_finish_button (0.00s)
    FAIL
    ```
- [x] **Tes baru LULUS setelah perbaikan kode diterapkan.**

### Penilaian Kerusakan Data Masa Lalu (Damage Assessment - O-09)
- **Status Data Historis:** Bersih.
- **Rincian Temuan:** Celah keamanan merupakan otorisasi interaksi runtime pada layer delivery Discord. Database tidak menyimpan state yang korup, dan kontrol otorisasi yang baru mencegah manipulasi state game di masa mendatang tanpa memerlukan migrasi skema atau manipulasi data historis.

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Hard Gate (HG-1 s/d HG-4):** PASSED. Kode teruji bersih, build berhasil (`go build ./cmd/bot`), tidak ada lint/vet error (`go vet ./...`), seluruh test suite lulus dengan race detector (`go test -race ./...`).
- **Must Probe Status (MP-1 s/d MP-7):** PASSED. Validasi otorisasi mencakup penolakan spectator dan penerimaan participant sah untuk aksi `next` maupun `finish`.
- **Daftar Belum Diuji (Untested List - O-16):** NONE. Seluruh alur otorisasi telah diuji secara komprehensif.
