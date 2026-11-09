---
id: PLAN-1794187566
ticket_id: TICKET-1794187566
title: Implementation Plan - Make Radovid Help Command Response Ephemeral
status: Completed
priority: Medium
---

# Rencana Implementasi: Mengubah Respon `/radovid help` Menjadi Ephemeral

## 1. Analisis Arsitektur & Masalah
Saat ini, `HandleRadovidHelp` di `internal/delivery/discord/radovid_help.go` membalas interaksi dengan:
```go
return s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
    Type: discordgo.InteractionResponseChannelMessageWithSource,
    Data: &discordgo.InteractionResponseData{
        Embeds: []*discordgo.MessageEmbed{embed},
    },
})
```
Karena tidak memiliki flag `discordgo.MessageFlagsEphemeral`, pesan tersebut dicetak di channel secara publik. Jika dipanggil berulang kali oleh user yang sama, channel akan dipenuhi embed bantuan yang panjang.

Discord API menyediakan flag `MessageFlagsEphemeral` (nilai: 64) yang membatasi visibilitas pesan hanya untuk pengguna pemanggil (*Only you can see this*), seperti pada `/radovid settings` dan `/radovid progress`.

## 2. Rencana Perubahan Teknis
1. Di `radovid-bot-api/internal/delivery/discord/radovid_help.go`:
   - Tambahkan fungsi helper `BuildHelpResponseData() *discordgo.InteractionResponseData`.
   - Set `Flags: discordgo.MessageFlagsEphemeral` pada `InteractionResponseData`.
   - Gunakan `BuildHelpResponseData()` di dalam `HandleRadovidHelp`.
2. Di `radovid-bot-api/internal/delivery/discord/radovid_help_test.go`:
   - Tambahkan unit test `TestBuildHelpResponseData_EphemeralFlag`.
   - Verifikasi bahwa flag ephemeral bernilai tepat `discordgo.MessageFlagsEphemeral` dan embed di dalamnya valid.
3. Di `radovid-bot-ai-orchestrator/nodes/radovid-bot/docs/manual-testing-playbook.md`:
   - Ubah tipe respon `/radovid help` di tabel seksi 3 menjadi `Ephemeral Embed`.
   - Perbarui seksi 4.1 agar memaparkan tampilan *Only you can see this* dan uji coba spam/privasi.

## 3. Rencana Pengujian TDD (Red-Green Cycle)
1. **Fase RED:** Tulis test `TestBuildHelpResponseData_EphemeralFlag` pada `radovid_help_test.go`.
2. Jalankan `go test -v -run TestBuildHelpResponseData ./internal/delivery/discord` -> Dipastikan gagal kompilasi (**RED**).
3. **Fase GREEN:** Implementasikan `BuildHelpResponseData()` pada `radovid_help.go`.
4. Jalankan kembali unit test -> Dipastikan lolos (**GREEN**).
5. Jalankan `go test -race ./...` di `radovid-bot-api` untuk memastikan tidak ada regresi.
