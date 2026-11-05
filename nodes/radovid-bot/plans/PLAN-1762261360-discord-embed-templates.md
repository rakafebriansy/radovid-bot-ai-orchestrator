---
id: PLAN-1762261360
ticket_id: TICKET-1762261360
title: Discord Embed Templates
status: Completed
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Discord Embed Templates

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261360](../tickets/TICKET-1762261360-discord-embed-templates.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun reusable `EmbedBuilder` helper dan template generator antarmuka Discord game pada package `internal/infrastructure/discord`. Builder ini menerapkan pattern fluent interface (method chaining), memvalidasi batas kapasitas payload Discord (Title <= 256, Description <= 4096, Fields <= 25), serta menyediakan fungsi template siap pakai (`SessionStartEmbed`, `RoundDisplayEmbed`, `ResultSummaryEmbed`, `ErrorEphemeralEmbed`) dengan palet warna baku untuk menjamin konsistensi UI di seluruh game Fase 2.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Delivery layer handlers (`internal/delivery/discord/...`), subcommands game Would You Rather, Taboo, dan 20 Questions.
  - Komponen Terdampak: Layer delivery Discord, visual output bot.
  - Potensi Efek Samping: Discord API 400 Bad Request jika batas karakter terlampaui; dicegah melalui safe truncation logic di dalam method builder.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/palette.go` | Konstanta hex color palette UI (Info, Success, Error, Warning, GM Accent) |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/embeds.go` | Struct EmbedBuilder fluent API dengan validasi panjang teks aman |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/embeds_test.go` | Unit test EmbedBuilder (truncation, fields limit, method chaining) |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Factory fungsi untuk tampilan start, round, result summary, dan error |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test validasi struktur objek embed yang dihasilkan template |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan Palet Warna & Red Test EmbedBuilder):**
   - Buat `palette.go` yang mendeklarasikan konstanta integer warna: `ColorInfo = 0x3498DB`, `ColorSuccess = 0x2ECC71`, `ColorError = 0xE74C3C`, `ColorWarning = 0xF1C40F`, `ColorGMAccent = 0x9B59B6`.
   - Buat `embeds_test.go` untuk menguji chaining, penambahan field, dan pemotongan teks yang melebihi batas Discord API.
   - Buktikan tes RED.
2. **Langkah 2 (Implementasi EmbedBuilder):**
   - Tulis `embeds.go` dengan pemotongan teks aman menggunakan slicing rune (`[]rune`) jika melebihi batas batas:
     - Title dipotong maksimal 256 karakter.
     - Description dipotong maksimal 4096 karakter.
     - Field value dipotong maksimal 1024 karakter.
     - Fields dibatasi maksimal 25 item.
   - Pastikan tes `embeds_test.go` berstatus GREEN.
3. **Langkah 3 (Red Test Game Templates & Factory Functions):**
   - Buat `game_templates_test.go` untuk memverifikasi output fungsi generator template:
     - `SessionStartEmbed` menghasilkan embed ber-author Radovid, color Info, dan field peserta.
     - `RoundDisplayEmbed` menghasilkan embed ber-color GM Accent, title nomor ronde, dan prompt deskripsi.
     - `ResultSummaryEmbed` menghasilkan embed ber-color Success, ringkasan skor, dan pesan feedback.
     - `ErrorEphemeralEmbed` menghasilkan embed ber-color Error.
   - Buktikan tes RED.
4. **Langkah 4 (Implementasi Game Templates):**
   - Tulis `game_templates.go` menggunakan fungsi builder yang sudah teruji.
   - Pastikan tes `game_templates_test.go` berstatus GREEN.
5. **Langkah 5 (Refactor & Delivery Gate):**
   - Pastikan kode mematuhi Zero-Comment Policy, tidak ada duplikasi warna numerik magic number, dan seluruh tes lulus 100%.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package discord

import (
	"unicode/utf8"

	"github.com/bwmarrin/discordgo"
)

const (
	ColorInfo     = 0x3498DB
	ColorSuccess  = 0x2ECC71
	ColorError    = 0xE74C3C
	ColorWarning  = 0xF1C40F
	ColorGMAccent = 0x9B59B6
)

type EmbedBuilder struct {
	embed *discordgo.MessageEmbed
}

func NewEmbedBuilder() *EmbedBuilder {
	return &EmbedBuilder{
		embed: &discordgo.MessageEmbed{
			Color: ColorInfo,
		},
	}
}

func (b *EmbedBuilder) SetTitle(title string) *EmbedBuilder {
	if utf8.RuneCountInString(title) > 256 {
		runes := []rune(title)
		title = string(runes[:253]) + "..."
	}
	b.embed.Title = title
	return b
}

func (b *EmbedBuilder) SetDescription(desc string) *EmbedBuilder {
	if utf8.RuneCountInString(desc) > 4096 {
		runes := []rune(desc)
		desc = string(runes[:4093]) + "..."
	}
	b.embed.Description = desc
	return b
}

func (b *EmbedBuilder) SetColor(color int) *EmbedBuilder {
	b.embed.Color = color
	return b
}

func (b *EmbedBuilder) AddField(name, value string, inline bool) *EmbedBuilder {
	if len(b.embed.Fields) >= 25 {
		return b
	}
	if utf8.RuneCountInString(value) > 1024 {
		runes := []rune(value)
		value = string(runes[:1021]) + "..."
	}
	b.embed.Fields = append(b.embed.Fields, &discordgo.MessageEmbedField{
		Name:   name,
		Value:  value,
		Inline: inline,
	})
	return b
}

func (b *EmbedBuilder) Build() *discordgo.MessageEmbed {
	return b.embed
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - String prompt dari LLM melebihi 4096 karakter: `SetDescription` memotong string secara aman pada 4093 rune dan menyisipkan elipsis `"..."`, menjaga integritas encoding UTF-8 dan mencegah Discord API error 400.
  - Jumlah ronde atau daftar pemain menghasilkan lebih dari 25 fields: `AddField` mengabaikan penambahan field ke-26 dan seterusnya alih-alih melempar panic.
- **Strategi Rollback:**
  - Hapus atau kembalikan berkas pada package `internal/infrastructure/discord` melalui Git.
