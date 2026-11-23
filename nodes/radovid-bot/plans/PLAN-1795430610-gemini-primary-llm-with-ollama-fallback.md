---
id: PLAN-1795430610
ticket_id: TICKET-1795430610
title: Implement Gemini Flash as Primary LLM with Ollama Fallback
status: Completed
created_at: 2026-11-23
---

# Rencana Implementasi Teknis: Gemini Flash Primary LLM dengan Ollama Fallback

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1795430610](../tickets/TICKET-1795430610-gemini-primary-llm-with-ollama-fallback.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:**
  1. Menghadirkan paket klien baru `internal/infrastructure/gemini` untuk berinteraksi dengan Google AI Studio REST API (`v1beta/models/{model}:generateContent`) yang mengimplementasikan interface `gamemaster.LLMClient`.
  2. Menghadirkan wrapper `FallbackClient` di `internal/infrastructure/llm` yang menampung `primary` dan `fallback` LLM client.
  3. Mengintegrasikan variabel konfigurasi `GEMINI_API_KEY` dan `GEMINI_MODEL` (default: `gemini-2.5-flash`) pada `config/config.go`.
  4. Menyambungkan `FallbackClient` di `cmd/bot/main.go` sehingga seluruh 14 usecase game, `GameMaster`, `TabooEvaluator`, dan `DiaryUseCase` langsung menikmati akselerasi Gemini Flash tanpa perlu merubah satupun layer usecase atau delivery.
- **Hasil Pemetaan Ketergantungan (Dependency Analysis):**
  - **Consumer Interface:**
    - Seluruh usecase game mengonsumsi `gamemaster.LLMClient`:
      ```go
      type LLMClient interface {
          Generate(ctx context.Context, prompt string, system string) (string, error)
      }
      ```
  - **Komponen Terdampak:**
    - `config/config.go`: Menambahkan field konfigurasi Gemini.
    - `cmd/bot/main.go`: Instansiasi klien Gemini, Ollama, dan perakitan ke dalam `FallbackClient`.
  - **Potensi Efek Samping & Mitigasi:**
    - *Network Failure:* Jika Gemini API mengalami gangguan jaringan atau rate limit (HTTP 429), `FallbackClient` otomatis memanggil Ollama (Qwen lokal).
    - *Missing API Key:* Jika `GEMINI_API_KEY` tidak diisi di `.env`, bot tetap dapat berjalan menggunakan Ollama tunggal tanpa panik/crash.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/gemini/client.go` | Implementasi Gemini Flash API client native Go (`net/http`) |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/gemini/client_test.go` | Unit test untuk Gemini Flash API client |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client.go` | Implementasi multi-provider fallback wrapper |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/llm/fallback_client_test.go` | Unit test untuk fallback wrapper |
| `MODIFY` | `../../../../radovid-bot-api/config/config.go` | Penambahan variabel `GEMINI_API_KEY` dan `GEMINI_MODEL` |
| `MODIFY` | `../../../../radovid-bot-api/config/config_test.go` | Unit test konfigurasi Gemini |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Integrasi `FallbackClient` sebagai penyedia LLM utama |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)

### Fase 1: Gemini Client Implementation (TDD)
1. **RED Test:** Buat `client_test.go` di `internal/infrastructure/gemini`:
   - Validasi parameter (API key, model, timeout).
   - Mock HTTP request/response format Gemini v1beta (`generateContent`).
   - Ekstraksi teks dari `candidates[0].content.parts[0].text`.
   - Penanganan error HTTP non-200 (400, 429, 500).
2. **GREEN Implementation:** Buat `client.go` di `internal/infrastructure/gemini`:
   - Implementasikan struct `Client` dengan method `Generate(ctx context.Context, prompt string, system string) (string, error)`.
   - Gunakan `application/json` config saat prompt memerlukan JSON.

### Fase 2: Fallback Client Implementation (TDD)
1. **RED Test:** Buat `fallback_client_test.go` di `internal/infrastructure/llm`:
   - Primary berhasil -> Mengembalikan output primary tanpa memanggil fallback.
   - Primary gagal -> Otomatis memanggil fallback dan mengembalikan output fallback.
   - Keduanya gagal -> Mengembalikan error komposit.
2. **GREEN Implementation:** Buat `fallback_client.go` di `internal/infrastructure/llm`.

### Fase 3: Konfigurasi & Wiring Main Application
1. Perbarui `config/config.go` dan `config/config_test.go` untuk memuat `GEMINI_API_KEY` dan `GEMINI_MODEL`.
2. Perbarui `cmd/bot/main.go` untuk menyuntikkan `FallbackClient` ke seluruh usecase bot.

### Fase 4: Verifikasi & Audit Delivery Gate
1. Jalankan `go test -race ./...` di `radovid-bot-api`.
2. Verifikasi kepatuhan Zero-Comment Policy (`coding.md`).
3. Jalankan `go vet ./...` dan `go build ./cmd/bot`.
4. Perbarui tiket, rencana, dan `CHANGELOG.md`.

---

## 5. Pratinjau Desain Kode (Focus Diff Preview)

### Klien Gemini Native REST:
```go
type Client struct {
    apiKey     string
    model      string
    timeout    time.Duration
    httpClient *http.Client
}

func (c *Client) Generate(ctx context.Context, prompt string, system string) (string, error) {
    // POST https://generativelanguage.googleapis.com/v1beta/models/{c.model}:generateContent?key={c.apiKey}
}
```

### Multi-Provider Fallback:
```go
type FallbackClient struct {
    primary  gamemaster.LLMClient
    fallback gamemaster.LLMClient
}

func (c *FallbackClient) Generate(ctx context.Context, prompt string, system string) (string, error) {
    if c.primary != nil {
        resp, err := c.primary.Generate(ctx, prompt, system)
        if err == nil && strings.TrimSpace(resp) != "" {
            return resp, nil
        }
    }
    return c.fallback.Generate(ctx, prompt, system)
}
```
