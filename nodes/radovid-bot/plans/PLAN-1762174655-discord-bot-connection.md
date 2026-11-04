---
id: PLAN-1762174655
ticket_id: TICKET-1762174655
title: Discord Bot Connection & Command Framework
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Discord Bot Connection & Command Framework

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762174655](../tickets/TICKET-1762174655-discord-bot-connection.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md) | [Error Handling Guidelines](../../../global-guidelines/error-handling.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun modul koneksi Discord Gateway menggunakan pustaka DiscordGo (`github.com/bwmarrin/discordgo`), menyediakan adapter infrastruktur untuk lifecycle sesi, merancang router modular untuk slash command (`InteractionCreate`), dan mengimplementasikan mekanisme graceful shutdown melalui penangkapan sinyal OS (`SIGINT`, `SIGTERM`).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: `cmd/bot/main.go` memanggil inisialisasi adapter Discord dan memasang command router sebelum memulai gateway listener.
  - Komponen Terdampak: `internal/infrastructure/discord/adapter.go` (session management), `internal/delivery/discord/router.go` (dispatching), dan seluruh command handlers di `internal/delivery/discord/`.
  - Potensi Efek Samping: Kegagalan koneksi ke Discord Gateway dapat menghambat startup bot; goroutine leak dapat terjadi jika handler interaksi tidak mengontrol siklus pembatalan via `context.Context`.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/adapter.go` | Wrapper session DiscordGo untuk lifecycle koneksi dan registrasi command |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/router.go` | Router dispatching command interaction dengan mapping nama command ke handler |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/handler.go` | Interface dan tipe signature command handler beserta context builder |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Menghubungkan adapter, router, serta channel graceful shutdown |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Rancang unit test untuk `router.go` pada `internal/delivery/discord/router_test.go` guna memverifikasi command matching dan fallback handling untuk command yang tidak dikenal.
   - Buktikan tes gagal sebelum definisi struct router diimplementasikan (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Buat `internal/infrastructure/discord/adapter.go` yang membungkus `*discordgo.Session`, mengonfigurasi intents, dan menyediakan method `Open()`, `Close()`, dan `RegisterCommand()`.
   - Buat `internal/delivery/discord/router.go` yang menyimpan `map[string]CommandHandler` dan method `HandleInteraction(s *discordgo.Session, i *discordgo.InteractionCreate)`.
   - Buat `internal/delivery/discord/handler.go` yang mendefinisikan tipe `CommandHandler func(ctx context.Context, s *discordgo.Session, i *discordgo.InteractionCreate) error`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Integrasikan lifecycle di `cmd/bot/main.go` dengan `signal.Notify(stopChan, os.Interrupt, syscall.SIGTERM)` dan `defer adapter.Close()`.
   - Pastikan context dibuat dengan timeout default (misal 5 detik) untuk setiap eksekusi slash command interaction agar tidak memblokir event loop DiscordGo.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go test ./internal/delivery/discord/...` untuk memastikan routing bekerja sesuai ekspektasi.
   - Uji penanganan sinyal simulasi untuk memastikan `s.Close()` dipanggil tanpa deadlock.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/delivery/discord/router.go
package discord

import (
	"context"
	"log"

	"github.com/bwmarrin/discordgo"
)

type CommandHandler func(ctx context.Context, s *discordgo.Session, i *discordgo.InteractionCreate) error

type CommandRouter struct {
	handlers map[string]CommandHandler
}

func NewCommandRouter() *CommandRouter {
	return &CommandRouter{
		handlers: make(map[string]CommandHandler),
	}
}

func (r *CommandRouter) Register(name string, handler CommandHandler) {
	r.handlers[name] = handler
}

func (r *CommandRouter) HandleInteraction(s *discordgo.Session, i *discordgo.InteractionCreate) {
	if i.Type != discordgo.InteractionApplicationCommand {
		return
	}

	commandName := i.ApplicationCommandData().Name
	handler, exists := r.handlers[commandName]
	if !exists {
		log.Printf("Unhandled slash command: %s", commandName)
		return
	}

	ctx := context.Background()
	if err := handler(ctx, s, i); err != nil {
		log.Printf("Error executing command %s: %v", commandName, err)
	}
}
```

```go
// cmd/bot/main.go (Graceful Shutdown)
stop := make(chan os.Signal, 1)
signal.Notify(stop, os.Interrupt, syscall.SIGTERM)
<-stop

log.Println("Shutting down gracefully...")
if err := discordAdapter.Close(); err != nil {
    log.Printf("Error closing Discord session: %v", err)
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Interaksi selain `InteractionApplicationCommand` (seperti autocomplete atau modal submit di masa depan) diabaikan secara aman oleh router tanpa panic. Jika token Discord invalid, `Open()` langsung mengembalikan error tanpa infinite retry loop.
- **Strategi Rollback:** Revert berkas yang dimodifikasi melalui `git checkout -- cmd/bot/main.go` dan hapus file baru di package `delivery/discord` dan `infrastructure/discord`.
