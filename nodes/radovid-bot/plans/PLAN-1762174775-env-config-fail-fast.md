---
id: PLAN-1762174775
ticket_id: TICKET-1762174775
title: Environment Configuration with Fail-Fast Validation
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Environment Configuration with Fail-Fast Validation

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762174775](../tickets/TICKET-1762174775-env-config-fail-fast.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md) | [Security Guidelines](../../../global-guidelines/security.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun paket `config` mandiri yang mengekstrak dan memvalidasi konfigurasi lingkungan runtime menggunakan standard library Go (`os.Getenv`), mematuhi prinsip fail-fast jika kredensial wajib tidak disediakan, mengamankan token agar tidak memiliki nilai default hardcoded, serta menyediakan fallback yang terprediksi untuk konfigurasi non-sensitif.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Entrypoint `cmd/bot/main.go` di baris pertama eksekusi sebelum modul lain diinisialisasi.
  - Komponen Terdampak: `internal/infrastructure/database` (menerima `DatabasePath`), `internal/infrastructure/discord` (menerima `DiscordToken`, `AppID`, `GuildID`), dan `internal/infrastructure/ollama` (menerima `OllamaBaseURL`, `OllamaModel`, `OllamaTimeout`).
  - Potensi Efek Samping: Binary langsung berhenti (*terminate*) dengan status code non-zero jika konfigurasi wajib tidak valid; ini adalah perilaku yang diharapkan demi mencegah aplikasi berjalan dalam status tidak aman.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/config/config.go` | Struct `Config`, pemetaan env var, validasi fail-fast, dan toleransi alias |
| `CREATE` | `../../../../radovid-bot-api/config/config_test.go` | Unit test skenario variabel lengkap, hilang, dan alias variabel |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Menggantikan logging dummy dengan pemanggilan `config.Load()` dan penanganan error |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Tulis unit test table-driven pada `config/config_test.go` yang menguji kegagalan fungsi ketika env var `DISCORD_TOKEN` dan `DATABASE_PATH` tidak ada.
   - Uji parsing string numerik untuk `OLLAMA_TIMEOUT_SECONDS` beserta fallback default.
   - Buktikan tes gagal sebelum implementasi paket selesai (RED).
2. **Langkah 2 (Implementasi Inti):**
   - Definisikan struct `Config`:
     - `DiscordToken`, `DiscordAppID`, `DiscordGuildID` (string)
     - `DatabasePath` (string)
     - `OllamaBaseURL`, `OllamaModel` (string)
     - `OllamaTimeoutSeconds` (int)
     - `LogLevel` (string)
   - Tulis fungsi `LoadConfig() (*Config, error)`:
     - Gunakan helper untuk membaca nilai dengan opsi alias (misal `getEnvWithFallback("DISCORD_TOKEN", "DISCORD_BOT_TOKEN")`).
     - Terapkan validasi ketiadaan nilai wajib: jika token atau path DB kosong, kembalikan error eksplisit: `missing required environment variable: DISCORD_TOKEN`.
     - Tetapkan default non-secret: URL Ollama `http://localhost:11434`, model `gemma3:4b`, timeout `30` detik, log level `info`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pasang pemanggilan `cfg, err := config.LoadConfig()` di awal `cmd/bot/main.go`. Jika `err != nil`, catat pesan error dan panggil `os.Exit(1)`.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Jalankan `go test -v ./config/...` dan periksa kepatuhan 100% pada skenario positif dan negatif.
   - Verifikasi tidak ada hardcoded secret di dalam seluruh kode atau file commit.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// config/config.go
package config

import (
	"errors"
	"fmt"
	"os"
	"strconv"
	"strings"
)

type Config struct {
	DiscordToken         string
	DiscordAppID         string
	DiscordGuildID       string
	DatabasePath         string
	OllamaBaseURL        string
	OllamaModel          string
	OllamaTimeoutSeconds int
	LogLevel             string
}

func LoadConfig() (*Config, error) {
	token := getEnv("DISCORD_TOKEN", "DISCORD_BOT_TOKEN")
	if strings.TrimSpace(token) == "" {
		return nil, errors.New("fail-fast: missing required environment variable: DISCORD_TOKEN")
	}

	dbPath := getEnv("DATABASE_PATH", "SQLITE_DB_PATH")
	if strings.TrimSpace(dbPath) == "" {
		return nil, errors.New("fail-fast: missing required environment variable: DATABASE_PATH")
	}

	timeoutStr := getEnv("OLLAMA_TIMEOUT_SECONDS")
	timeout := 30
	if parsed, err := strconv.Atoi(timeoutStr); err == nil && parsed > 0 {
		timeout = parsed
	}

	return &Config{
		DiscordToken:         token,
		DiscordAppID:         getEnv("DISCORD_APP_ID", "DISCORD_APPLICATION_ID"),
		DiscordGuildID:       getEnv("DISCORD_GUILD_ID"),
		DatabasePath:         dbPath,
		OllamaBaseURL:        getEnvDefault("OLLAMA_BASE_URL", "http://localhost:11434"),
		OllamaModel:          getEnvDefault("OLLAMA_MODEL", "gemma3:4b"),
		OllamaTimeoutSeconds: timeout,
		LogLevel:             getEnvDefault("LOG_LEVEL", "info"),
	}, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Nilai whitespace kosong (`"   "`) dianggap kosong melalui `strings.TrimSpace` untuk mencegah nilai token dummy yang tidak valid. String timeout non-numerik (misal `"abc"` atau `"-5"`) ditangani aman dengan fallback ke default 30 detik tanpa menimbulkan panic.
- **Strategi Rollback:** Kode konfigurasi murni stateless; pengembalian ke implementasi sebelumnya dapat dilakukan dengan `git checkout -- config/`.
