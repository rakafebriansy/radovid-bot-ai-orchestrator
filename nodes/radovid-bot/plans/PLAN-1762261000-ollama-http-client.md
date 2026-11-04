---
id: PLAN-1762261000
ticket_id: TICKET-1762261000
title: Ollama HTTP Client Wrapper
status: Completed
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Ollama HTTP Client Wrapper

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261000](../tickets/TICKET-1762261000-ollama-http-client.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun package wrapper HTTP client yang tangguh dan thread-safe untuk berkomunikasi dengan Offline Ollama Generate API (`/api/generate`) lokal berbasis model Gemma 4b (`gemma3:4b` / `gemma:4b`). Wrapper mengisolasi detail jaringan dan HTTP transport dari domain/usecase layer, menyediakan context propagation dengan boundary timeout, retry backoff eksponensial (maksimal 2 retries untuk HTTP 5xx dan socket errors), serta sanitasi input deterministik (strip mention Discord, markdown control sequence injection, batas 2000 karakter UTF-8).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: `internal/usecase/game/game_master.go`, game activity use cases (Would You Rather, Taboo, 20 Questions).
  - Komponen Terdampak: `internal/infrastructure/ollama`, layer orkestrasi game.
  - Potensi Efek Samping: Latensi I/O blocking jika timeout tidak diterapkan dengan ketat; pemborosan memori jika payload stream tidak ditangani secara batching (`stream: false`). Integritas komputasi sepenuhnya offline tanpa bergantung pada koneksi internet publik.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/errors.go` | Mendefinisikan typed domain errors untuk kegagalan Ollama |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/models.go` | Struct DTO untuk GenerateRequest dan GenerateResponse |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize.go` | Fungsi sanitasi prompt: strip Discord mentions, length limit 2000 rune |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/sanitize_test.go` | Unit test sanitasi prompt dengan kasus batas edge-case |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/client.go` | Implementasi Ollama Client dengan retry backoff & context timeout |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/client_test.go` | Unit test client menggunakan httptest.Server (sukses, timeout, 5xx retry) |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test Sanitasi):**
   - Buat `internal/infrastructure/ollama/sanitize_test.go` yang menguji berbagai varian Discord mention (`<@123>`, `<@!456>`, `<#789>`, `<@&012>`), markdown exploit, dan teks panjang > 2000 karakter.
   - Buktikan tes gagal (RED).
2. **Langkah 2 (Implementasi Sanitasi & Error Types):**
   - Buat `sanitize.go` dengan fungsi regex-based mention stripper dan rune truncation.
   - Buat `errors.go` dengan sentinel error types (`ErrTimeout`, `ErrConnectionRefused`, `ErrOllamaAPI`, `ErrEmptyPrompt`).
   - Jalankan tes hingga berstatus hijau (GREEN).
3. **Langkah 3 (Red Test HTTP Client & Retry Mechanism):**
   - Buat `internal/infrastructure/ollama/client_test.go` menggunakan `httptest.NewServer` untuk mensimulasikan respons 200, respon 500 berturut-turut, respon lambat melebihi deadline timeout, dan bad payload.
   - Buktikan tes client berstatus RED.
4. **Langkah 4 (Implementasi Client & Backoff Loop):**
   - Tulis `models.go` untuk representasi payload Ollama API.
   - Tulis `client.go` dengan loop retry maksimal 2 kali bertahap (500ms, 1000ms), context cancellation check, dan `stream: false`.
   - Pastikan tes client berstatus GREEN.
5. **Langkah 5 (Refactor & Delivery Gate):**
   - Pastikan kode mematuhi Zero-Comment Policy, semua pesan error berbahasa Inggris, dan validasi fail-fast saat inisialisasi client (URL tidak boleh kosong).

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package ollama

import (
	"bytes"
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"net/http"
	"regexp"
	"strings"
	"time"
	"unicode/utf8"
)

var (
	ErrTimeout           = errors.New("ollama request timed out")
	ErrConnectionRefused = errors.New("ollama connection refused")
	ErrOllamaAPI         = errors.New("ollama api returned error")
	ErrEmptyPrompt       = errors.New("prompt cannot be empty")
)

var mentionRegex = regexp.MustCompile(`<@!?[0-9]+>|<#[0-9]+>|<@&[0-9]+>`)

func SanitizePrompt(input string) (string, error) {
	trimmed := strings.TrimSpace(input)
	if trimmed == "" {
		return "", ErrEmptyPrompt
	}
	clean := mentionRegex.ReplaceAllString(trimmed, "")
	clean = strings.TrimSpace(clean)
	if clean == "" {
		return "", ErrEmptyPrompt
	}
	if utf8.RuneCountInString(clean) > 2000 {
		runes := []rune(clean)
		clean = string(runes[:2000])
	}
	return clean, nil
}

type Client struct {
	baseURL    string
	model      string
	timeout    time.Duration
	httpClient *http.Client
}

func NewClient(baseURL, model string, timeout time.Duration, httpClient *http.Client) (*Client, error) {
	if strings.TrimSpace(baseURL) == "" {
		return nil, errors.New("ollama base url is required")
	}
	if strings.TrimSpace(model) == "" {
		return nil, errors.New("ollama model is required")
	}
	if httpClient == nil {
		httpClient = &http.Client{}
	}
	return &Client{
		baseURL:    strings.TrimRight(baseURL, "/"),
		model:      model,
		timeout:    timeout,
		httpClient: httpClient,
	}, nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - Prompt berisi 100% Discord mentions: Ditangani oleh `SanitizePrompt` dengan mengembalikan `ErrEmptyPrompt` alih-alih mengirim string kosong ke LLM.
  - Multi-byte UTF-8 character pada batas 2000 rune: Pemotongan berbasis `[]rune` mencegah korupsi byte UTF-8.
  - Server Ollama hang / unreach: HTTP client menggunakan Context dengan deadline pasti sehingga goroutine tidak bocor.
- **Strategi Rollback:**
  - Hapus package `internal/infrastructure/ollama` jika terjadi kegagalan integrasi besar atau batalkan commit melalui Git. Tidak ada efek persistensi DB pada tahap ini.
