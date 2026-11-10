---
id: PLAN-1794307528
ticket_id: TICKET-1794307528
title: Fix Multiplayer Lobby Timeout on Player 2 Join and Add Stuck Session Reset Action
status: Completed
priority: High
---

# Rencana Implementasi Teknis: Fix Multiplayer Lobby Timeout on Player 2 Join and Add Stuck Session Reset Action

## 1. Analisis Kebutuhan & Akar Masalah
Pada pengujian live game multiplayer Would You Rather di Discord, terjadi kondisi macet permanen (*stuck state*):
1. **Pemicu Utama (Discord Interaction 3-Second Timeout):**
   Ketika Host membuat lobby, pesan `Lobby Created` dikirim seketika. Namun saat Player 2 mengetik `/radovid play would-you-rather` untuk bergabung, `h.sessionManager.StartSession` mengubah status sesi menjadi `StatusActive`, lalu memanggil `wyrUseCase.StartRound` yang melakukan panggilan HTTP sinkron ke Ollama untuk membuat soal dilema.
   Ollama lokal membutuhkan waktu 4–8 detik untuk menyelesaikan inferensi. Karena tidak ada penundaan respon (*interaction deferral*), Discord memutus interaksi setelah 3 detik (`The application did not respond`). Akibatnya, `s.InteractionRespond` gagal, soal ronde 1 tidak pernah tampil di channel, sementara sesi di SQLite sudah terkunci dalam status `active`.
2. **Kondisi Terkunci (Deadlock Channel):**
   Ketika pemain mencoba memanggil ulang `/radovid play would-you-rather`, handler mendapati `activeSession.Status == StatusActive` dan mengembalikan pesan error ephemeral bahwa game sedang aktif di atas, padahal pesan soal tidak pernah ada. Channel terkunci selama 15 menit tanpa jalan keluar.
3. **Pola Serupa pada Taboo, 20 Questions, dan Button Handler:**
   Pola ketiadaan deferral ini juga ditemukan pada `command_play_taboo.go`, `command_play_20q.go`, serta pada aksi tombol `interaction_wyr_button.go` (pada aksi `next` round dan `finish` summary yang memanggil Ollama).

## 2. Strategi Perubahan Arsitektural
1. **Pola Deferred Channel Message pada Start Game:**
   Segera setelah `StartSession` berhasil mengunci 2 peserta, panggil:
   ```go
   _ = s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
       Type: discordgo.InteractionResponseDeferredChannelMessageWithSource,
   })
   ```
   Lalu jalankan `StartRound` / `StartGame`. Jika sukses, kirim embed soal dan tombol via `s.FollowupMessageCreate(i.Interaction, false, ...)`.
   Jika gagal, batalkan status sesi (*rollback* ke `StatusCancelled` agar channel bersih) dan kirim pesan galat ephemeral.
2. **Pola Deferred Message Update pada Button Interactions:**
   Pada `interaction_wyr_button.go` untuk aksi `next` dan `finish`, serta aksi `vote` (saat voter terakhir memicu AI commentary):
   Kirim:
   ```go
   _ = s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
       Type: discordgo.InteractionResponseDeferredMessageUpdate,
   })
   ```
   Lalu perbarui pesan melalui `s.InteractionResponseEdit` setelah AI selesai menghasilkan konten.
3. **Mekanisme Reset / Batal Sesi Macet (Stuck Session Rescue):**
   Pada `command_play_wyr.go` ketika `activeSession.Status == session.StatusActive`:
   Sajikan pesan ephemeral informatif disertai tombol bahaya merah (*danger button*):
   `[Cancel Stuck Game & Reset]` (`CustomID: "wyr:cancel:" + activeSession.ID`).
   Pada `interaction_wyr_button.go`, tangani `case "cancel"`:
   Panggil `h.sessionManager.CancelSession(ctx, sessionID)`. Kirim balasan publik bahwa sesi telah dibatalkan dan pemain dapat langsung memulai lobi baru.
4. **Kepatuhan Pedoman:**
   - Zero-Comment Policy (`coding.md` Seksi 1.1) pada seluruh berkas Go.
   - Penulisan unit test berbasis TDD yang membuktikan kegagalan pada kode lama (O-07).

## 3. Langkah Eksekusi Terinci
1. **Langkah 1 (TDD - Tahap RED):**
   - Tambahkan pengujian pada `command_play_wyr_test.go` yang memverifikasi bahwa saat `StartRound` sukses, balasan dikirimkan melalui deferred response (`InteractionResponseDeferredChannelMessageWithSource`) dan followup (`FollowupMessageCreate`), bukan langsung merespon sinkron.
   - Tambahkan pengujian untuk penanganan aksi `wyr:cancel:<sessionID>` pada `interaction_wyr_button_test.go`.
   - Buktikan tes gagal pada kode lama (O-07).
2. **Langkah 2 (Tahap GREEN):**
   - Perbarui `command_play_wyr.go`, `command_play_taboo.go`, dan `command_play_20q.go` dengan interaction deferral.
   - Perbarui `interaction_wyr_button.go` dengan deferred message update dan handler aksi `cancel`.
   - Perbarui pesan error aktif di `command_play_wyr.go` agar menyertakan tombol cancel/reset.
3. **Langkah 3 (Tahap REFACTOR & QUALITY CHECK):**
   - Jalankan `go test -race ./...` dan pastikan 100% lulus.
   - Jalankan `go vet ./...` dan verifikasi nol komentar pada berkas Go.
4. **Langkah 4 (Delivery Gate & CHANGELOG):**
   - Perbarui status tiket dan rencana menjadi `Done` / `Completed`.
   - Catat pembaruan pada `CHANGELOG.md`.

