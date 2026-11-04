---
id: PLAN-1762174595
ticket_id: TICKET-1762174595
title: Setup Go Project Structure
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Setup Go Project Structure

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762174595](../tickets/TICKET-1762174595-setup-go-project-structure.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md) | [Dependencies Standards](../../../global-guidelines/dependencies.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun kerangka direktori Clean Architecture (Hexagonal / Ports & Adapters) pada modul `github.com/rakafebriansy/radovid-bot` di folder `radovid-bot-api`, memasang dependensi dasar (`github.com/bwmarrin/discordgo`, `modernc.org/sqlite`, `github.com/google/uuid`), dan memastikan build Go pure-Go tanpa dependensi CGO.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Entrypoint `cmd/bot/main.go` yang akan memuat dan merangkai lapisan `config`, `infrastructure`, `usecase`, dan `delivery`.
  - Komponen Terdampak: Seluruh paket dalam `radovid-bot-api` yang akan dibangun pada fase-fase berikutnya.
  - Potensi Efek Samping: Struktur paket menentukan visibilitas simbol; domain layer harus tetap terisolasi tanpa mengimpor delivery atau infrastructure layer (Zero External Dependencies pada domain).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/go.mod` | Menambahkan dependensi `discordgo`, `modernc.org/sqlite`, dan `uuid` |
| `CREATE` | `../../../../radovid-bot-api/cmd/bot/main.go` | Scaffold entrypoint aplikasi minimal dengan logging startup |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/user/entity.go` | Definisi entitas domain User awal |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/session/entity.go` | Definisi entitas domain GameSession awal |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Definisi enum dan entitas Game Type dasar |
| `CREATE` | `../../../../radovid-bot-api/migrations/.gitkeep` | Direktori penyimpanan berkas migrasi SQL |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/.gitkeep` | Direktori adapter database SQLite |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/discord/.gitkeep` | Direktori wrapper session DiscordGo |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/.gitkeep` | Direktori client Ollama HTTP |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/.gitkeep` | Direktori command router dan interaction handler |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/.gitkeep` | Direktori dashboard HTTP handler |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Periksa keutuhan `go.mod` di `radovid-bot-api`.
   - Jalankan verifikasi `go build ./...` awal untuk memastikan workspace Go terdeteksi dengan benar.
2. **Langkah 2 (Implementasi Inti):**
   - Jalankan pengunduhan dependensi: `go get github.com/bwmarrin/discordgo`, `go get modernc.org/sqlite`, dan `go get github.com/google/uuid`.
   - Buat direktori-direktori lapisan Clean Architecture sesuai spesifikasi di `docs/system-design.md`.
   - Letakkan entitas domain inti pada `internal/domain/user/entity.go`, `internal/domain/session/entity.go`, dan `internal/domain/game/entity.go` dengan tipe data waktu Epoch Milliseconds (`int64`).
   - Buat file `cmd/bot/main.go` sederhana yang mengimpor domain dan mencetak status inisialisasi.
3. **Langkah 3 (Refactor & Integrasi):**
   - Jalankan `go mod tidy` untuk merapikan `go.mod` dan menghasilkan `go.sum`.
   - Pastikan tidak ada dependensi yang membutuhkan compiler GCC/Clang dengan memverifikasi build menggunakan `CGO_ENABLED=0 go build ./cmd/bot`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go vet ./...` untuk memastikan seluruh deklarasi struct dan package valid.
   - Verifikasi repositori bersih dan tidak ada file biner atau artefak sementara yang ter-commit.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// cmd/bot/main.go
package main

import (
	"fmt"
	"log"

	_ "modernc.org/sqlite"
	_ "github.com/bwmarrin/discordgo"
	_ "github.com/google/uuid"
)

func main() {
	log.Println("Radovid Bot API initialized successfully (Clean Architecture scaffold)")
	fmt.Println("Ready for Phase 1 module integration")
}
```

```go
// internal/domain/user/entity.go
package user

type User struct {
	ID        string `json:"id"`
	DiscordID string `json:"discord_id"`
	Username  string `json:"username"`
	CreatedAt int64  `json:"created_at"`
	UpdatedAt int64  `json:"updated_at"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Penggunaan `modernc.org/sqlite` murni Go mencegah kegagalan kompilasi cross-platform pada mesin tanpa compiler C (GCC/Clang) atau arsitektur Linux musl.
- **Strategi Rollback:** Jika terjadi kerusakan dependensi pada `go.mod`, kembalikan ke state awal melalui `git checkout go.mod` dan hapus direktori kosong yang dibuat.
