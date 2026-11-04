---
id: PLAN-1762174835
ticket_id: TICKET-1762174835
title: Basic /radovid help dan /radovid start Command Skeleton
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Basic /radovid help dan /radovid start Command Skeleton

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762174835](../tickets/TICKET-1762174835-basic-commands-skeleton.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md) | [UI and Assets Guidelines](../../../global-guidelines/ui-and-assets.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mendefinisikan dan mendaftarkan slash command root `/radovid` beserta subcommands `help` dan `start` pada Discord Application Command API, menyediakan builder pesan Discord Embed terstandardisasi dengan palet tema warna Radovid, mengurai opsi subcommand pada level delivery layer, serta memastikan seluruh kegagalan atau validasi dikirimkan secara ephemeral agar tidak mencemari kanal percakapan umum.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Router `internal/delivery/discord/router.go` meneruskan eksekusi ke `HandleRadovidCommand` saat command bernama `radovid` diterima.
  - Komponen Terdampak: `internal/delivery/discord/radovid_help.go`, `internal/delivery/discord/radovid_start.go`, serta pembuat UI `internal/infrastructure/discord/embeds.go`.
  - Potensi Efek Samping: Propagasi slash command global membutuhkan waktu hingga 1 jam di Discord, oleh karena itu konfigurasi dev mode via `DISCORD_GUILD_ID` wajib diprioritaskan untuk pembaruan instan selama siklus pengembangan lokal.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Definisi struct `discordgo.ApplicationCommand` untuk `/radovid` dan subcommands |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go` | Handler untuk subcommand `help` yang membangun dan mengirim embed panduan |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Handler untuk subcommand `start` yang mengirim embed skeleton sesi game |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/embeds.go` | Factory builder untuk Discord Embed (Info, Success, Error, Game Card) |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/handler.go` | Dispatcher internal subcommand dan helper pengiriman respons ephemeral |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Tulis unit test untuk `embeds.go` guna memastikan builder menghasilkan warna tema yang benar, judul, deskripsi, dan footer tanpa memicu pointer dereference pada field opsional.
   - Buktikan tes gagal sebelum implementasi builder (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Buat `command_def.go` yang mendeklarasikan struktur command:
     - Name: `radovid`
     - Description: `Radovid Bot - AI-powered English Learning Companion`
     - Options: Subcommand `help` (deskripsi: `Show Radovid Bot guide and commands`) dan `start` (deskripsi: `Start a new English learning game session`).
   - Buat `embeds.go` dengan fungsi:
     - `NewInfoEmbed(title, description string) *discordgo.MessageEmbed` (Warna Blurple / `#5865F2`)
     - `NewErrorEmbed(errMessage string) *discordgo.MessageEmbed` (Warna Merah / `#ED4245`)
   - Tulis `radovid_help.go` yang membalas interaksi dengan embed panduan: instruksi bot, daftar 15 mode game, dan tips interaksi.
   - Tulis `radovid_start.go` yang membalas interaksi dengan embed skeleton: status sesi pending, pemain host, dan persiapan integrasi ke engine Fase 2.
3. **Langkah 3 (Refactor & Integrasi):**
   - Di `handler.go`, baca subcommand via `i.ApplicationCommandData().Options[0].Name`. Rutekan ke `HandleRadovidHelp` atau `HandleRadovidStart`.
   - Tambahkan fungsi helper `RespondEphemeralError(s, i, message)` yang membalas dengan flag `discordgo.MessageFlagsEphemeral`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go test -v ./internal/delivery/discord/...` dan `go test -v ./internal/infrastructure/discord/...`.
   - Uji batas waktu interaksi: pastikan respons dikirim dalam rentang 3 detik awal sebelum gateway Discord memicu timeout token.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/delivery/discord/command_def.go
package discord

import "github.com/bwmarrin/discordgo"

func GetApplicationCommands() []*discordgo.ApplicationCommand {
	return []*discordgo.ApplicationCommand{
		{
			Name:        "radovid",
			Description: "AI-powered Discord English Learning Companion",
			Options: []*discordgo.ApplicationCommandOption{
				{
					Type:        discordgo.ApplicationCommandOptionSubCommand,
					Name:        "help",
					Description: "Show available commands and how to play",
				},
				{
					Type:        discordgo.ApplicationCommandOptionSubCommand,
					Name:        "start",
					Description: "Start a new 2-player English learning game session",
				},
			},
		},
	}
}
```

```go
// internal/delivery/discord/radovid_help.go
package discord

import (
	"context"

	"github.com/bwmarrin/discordgo"
	infraDiscord "github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord"
)

func HandleRadovidHelp(ctx context.Context, s *discordgo.Session, i *discordgo.InteractionCreate) error {
	embed := infraDiscord.NewInfoEmbed(
		"🏰 Radovid Bot — English Learning Companion",
		"Welcome to Radovid Bot! Sharpen your English conversational skills through 15 interactive AI-driven games.",
	)
	embed.Fields = []*discordgo.MessageEmbedField{
		{
			Name:   "🎮 /radovid start",
			Value:  "Start a new 2-player English game session with an AI Game Master.",
			Inline: false,
		},
		{
			Name:   "📖 /radovid help",
			Value:  "Display this navigation guide and rules.",
			Inline: false,
		},
	}

	return s.InteractionRespond(i.Interaction, &discordgo.InteractionResponse{
		Type: discordgo.InteractionResponseChannelMessageWithSource,
		Data: &discordgo.InteractionResponseData{
			Embeds: []*discordgo.MessageEmbed{embed},
		},
	})
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Jika opsi subcommand kosong atau tidak dikenali, kirim respons ephemeral ramah pengguna ("Unknown subcommand") tanpa membiarkan interaksi mengalami unhandled hanging state. Bila interaksi gagal dijawab sebelum deadline 3 detik, pastikan error logged.
- **Strategi Rollback:** Hapus berkas command dan kembalikan router ke state awal via `git checkout -- internal/delivery/discord/handler.go`.
