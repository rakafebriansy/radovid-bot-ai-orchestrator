---
id: TICKET-1794307528
plan_id: PLAN-1794307528
title: Fix Multiplayer Lobby Timeout on Player 2 Join and Add Stuck Session Reset Action
status: Done
priority: High
severity: Broken Flow
labels: [Bug, Discord, Multiplayer, Session, Game, Architecture]
---

# Laporan Bug

> **Aturan Relative Path:** Seluruh referensi berkas di dalam tiket ini WAJIB menggunakan relative path dari posisi berkas tiket aktif.

## 1. Deskripsi Masalah
Pada game multiplayer (Would You Rather, Taboo, dan 20 Questions), ketika Player 2 bergabung ke dalam lobby pending (`/radovid play <game>`), handler Discord delivery langsung memanggil use case untuk memulai ronde pertama (`StartRound` / `StartGame`) yang memanggil LLM Ollama tanpa melakukan *defer interaction* (`discordgo.InteractionResponseDeferredChannelMessageWithSource`).

Karena pembuatan konten game AI lokal membutuhkan waktu 4–8 detik, interaksi Discord melewati batas waktu maksimal 3 detik (*timeout*) dan menghasilkan pesan *"The application did not respond"*. Akibatnya:
1. Pesan soal ronde 1 beserta komponen tombol pilihan (`Option A` / `Option B`) **tidak pernah terkirim** ke channel Discord.
2. Namun, sesi game di SQLite sudah terlanjur berstatus `active` karena dipicu oleh `StartSession`.
3. Channel Discord terkunci (*stuck*) dalam status `active` selama TTL 15 menit. Setiap kali ada pemain yang memanggil `/radovid play <game>`, bot menolak dengan pesan *"A game is already active in this channel! Cast your vote on the active question above."*, padahal tidak ada soal yang dapat dipilih di atas.
4. Tidak ada mekanisme pembatalan (*cancel/reset*) interaktif bagi pemain untuk mereset sesi yang macet tersebut.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1794307528-fix-multiplayer-lobby-timeout-and-stuck-session-reset.md)
*(Rincian arsitektur, flow sequence, dan langkah teknis mendalam berada di dokumen rencana di atas).*

## 2. Kriteria Penerimaan (Acceptance Criteria)
- [x] Handler `HandlePlayCommand` pada `command_play_wyr.go` segera melakukan deferral interaksi (`discordgo.InteractionResponseDeferredChannelMessageWithSource`) begitu `StartSession` sukses sebelum memanggil `wyrUseCase.StartRound`, dan mengirim soal Ronde 1 via `s.FollowupMessageCreate`.
- [x] Handler `HandlePlayTaboo` pada `command_play_taboo.go` dan `HandlePlayTQ` pada `command_play_20q.go` menerapkan deferral serupa sebelum memanggil `StartGame` agar kebal terhadap batas waktu 3 detik Discord.
- [x] Handler tombol `HandleButtonInteraction` pada `interaction_wyr_button.go` menerapkan `InteractionResponseDeferredMessageUpdate` pada aksi `vote` (saat kedua pemain selesai voting dan AI membuat commentary via `WillVoteFinishRound`), `next` (saat membuat soal ronde berikutnya), dan `finish` (saat membuat rangkuman akhir), lalu memperbarui pesan via `InteractionResponseEdit`.
- [x] Ketika sesi di channel berstatus `active` dan pengguna memanggil `/radovid play <game>`:
  - Bot menyertakan tombol aksi **"Cancel Stuck Game & Reset"** (`CustomID: "<game>:cancel:<sessionID>"`).
  - Saat tombol tersebut diklik oleh pemain, sesi yang macet dibatalkan (`session.StatusCancelled`) dan bot mengonfirmasi bahwa channel siap untuk membuat game baru.
- [x] Jika `StartRound` atau `StartGame` mengalami kegagalan AI, sesi otomatis dibatalkan (*rollback* status) dan pesan galat ephemeral dikirim via followup agar channel tidak terkunci.
- [x] Unit test baru ditambahkan dan **TERBUKTI GAGAL pada kode lama sebelum fix (Disiplin O-07)**.
- [x] Seluruh unit test suite lulus 100% (`go test -race ./...`) tanpa regresi.
- [x] Kebijakan Zero-Comment (`coding.md` Seksi 1.1) dipatuhi 100% pada seluruh berkas Go yang dimodifikasi.

## 3. Target Lingkup Berkas (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
- `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go`
- `CHANGELOG.md`

---

## 4. AI Execution Log & Output
1. **Bukti Kegagalan Kode Lama (TDD O-07 / RED Phase):**
   ```
   === RUN   TestHandlePlayWYR_ActiveSessionShowsCancelButton
       command_play_wyr_test.go:318: expected cancel button with custom ID 'wyr:cancel:sess-active-1' in payload, got: {"type":4,"data":{"tts":false,"content":"","components":null,"embeds":[{"title":"Error","description":"A game is already active in this channel! Cast your vote on the active question above.","color":15680580,"footer":{"text":"Radovid Bot — English Learning Companion"}}],"flags":64}}
   --- FAIL: TestHandlePlayWYR_ActiveSessionShowsCancelButton (0.00s)
   === RUN   TestHandlePlayWYR_Player2JoinDefersAndFollowsUp
       command_play_wyr_test.go:395: expected at least 2 requests (deferral callback + followup), got 1
   --- FAIL: TestHandlePlayWYR_Player2JoinDefersAndFollowsUp (0.00s)
   === RUN   TestHandleButtonInteraction_CancelStuckSession
       interaction_wyr_button_test.go:429: expected session status to be cancelled, got active
   --- FAIL: TestHandleButtonInteraction_CancelStuckSession (0.00s)
   FAIL
   ```
2. **Implementasi & Verifikasi Sukses (GREEN Phase):**
   - Ditambahkan `WillVoteFinishRound` pada `internal/usecase/game/would_you_rather.go` untuk mendeteksi vote terakhir secara presisi sebelum deferral.
   - Diperbarui `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go` dengan `InteractionResponseDeferredChannelMessageWithSource` dan pengiriman followup embed pertanyaan/putaran saat pemain kedua bergabung.
   - Ditambahkan tombol bahaya merah `Cancel Stuck Game & Reset` (`<game>:cancel:<sessionID>`) saat channel mendeteksi sesi aktif yang macet, serta ditangani di masing-masing handler komponen.
   - Diperbarui `interaction_wyr_button.go` dengan `InteractionResponseDeferredMessageUpdate` untuk `vote` terakhir, `next`, dan `finish` sebelum panggilan LLM, lalu diedit via `InteractionResponseEdit`.
3. **Hasil Pengujian Keseluruhan (100% Pass with -race):**
   ```
   ok  github.com/rakafebriansy/radovid-bot/internal/delivery/discord 1.717s
   ok  github.com/rakafebriansy/radovid-bot/internal/usecase/game 0.450s
   ...
   ALL PACKAGES PASSED (100%)
   ```
4. **Kepatuhan Kebijakan:**
   - Zero-Comment Policy (§1.1): 0 baris komentar pada seluruh kode Go baru/modifikasi.
   - Static analysis: `go vet ./...` lulus dengan 0 warning.


