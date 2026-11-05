---
id: PLAN-1762261120
ticket_id: TICKET-1762261120
title: AI Game Master Prompt System
status: Completed
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: AI Game Master Prompt System

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261120](../tickets/TICKET-1762261120-ai-game-master-prompts.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun sistem prompt terintegrasi untuk AI Game Master dengan pemisahan domain contract (`internal/domain/gamemaster`), manajemen berkas prompt template menggunakan Go 1.26 `//go:embed` filesystem (`internal/infrastructure/ollama/prompts`), tokenizer/parser output LLM berbasis ekspresi reguler dan delimiter, serta penyedia cadangan statis (*static fallback questions*) ketika LLM offline atau memproduksi format tak valid.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler dan use case game interaktif (`would_you_rather.go`, `taboo.go`, `twenty_questions.go`).
  - Komponen Terdampak: Layer infrastructure Ollama, usecase `game_master.go`, template prompt files.
  - Potensi Efek Samping: Template parsing failure pada runtime jika sintaks Go template tidak valid; dicegah melalui unit test compile-time embed validation.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/gamemaster/entity.go` | Domain struct GameMasterRequest, GameMasterResponse, GameType enum |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/gamemaster/repository.go` | Interface GameMasterService (Port) |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed.go` | Embed variable FS `//go:embed *.tmpl` dan renderer fungsi |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/would_you_rather.tmpl` | Prompt template game Would You Rather |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/taboo.tmpl` | Prompt template game Taboo |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/twenty_questions.tmpl` | Prompt template game 20 Questions |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser ekstraksi output teks LLM ke GameMasterResponse |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parser dengan berbagai variasi gaya output LLM |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Preset curated static questions per game type & difficulty |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test ketersediaan data fallback |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/game_master.go` | Orkestrator pemanggilan prompt, client Ollama, parser, dan fallback |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/game_master_test.go` | Unit test GameMaster usecase dengan mock Ollama client |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Domain Model Definition):**
   - Tulis `internal/domain/gamemaster/entity.go` yang memuat struct `GameMasterRequest` (GameType, Difficulty, Round, History) dan `GameMasterResponse` (Question, Options, TabooWords, SecretWord, IsFallback).
   - Tulis `repository.go` untuk antarmuka service LLM.
2. **Langkah 2 (Embedded Prompt Templates):**
   - Buat berkas template `.tmpl` di `internal/infrastructure/ollama/prompts/` dengan instruksi delimiter ketat (misal: `QUESTION:`, `OPTION_A:`, `OPTION_B:`).
   - Buat `embed.go` menggunakan `//go:embed *.tmpl` dengan fungsi `RenderPrompt(gameType string, data any) (string, error)`.
3. **Langkah 3 (Red Test Parser & Implementasi):**
   - Buat `parser_test.go` yang menguji input terpotong, markdown bold (`**QUESTION:**`), list bernomor, dan variasi spasi.
   - Implementasikan `parser.go` hingga tes GREEN.
4. **Langkah 4 (Static Fallback Provider):**
   - Buat `fallback.go` yang menyediakan setidaknya 10 bank pertanyaan per game type (Would You Rather, Taboo, 20 Questions) per level (Easy, Medium, Hard).
   - Tulis `fallback_test.go` untuk memverifikasi tidak ada list kosong.
5. **Langkah 5 (Integrasi Use Case & Fallback Graceful Handling):**
   - Implementasikan `game_master.go`: saat `ollamaClient.Generate` mengembalikan error atau parser gagal, sistem log warning lalu mengambil respons dari `fallback.go` dengan `IsFallback: true`.
   - Jalankan seluruh rangkaian tes hingga lulus.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package gamemaster

type Difficulty string

const (
	DifficultyEasy   Difficulty = "easy"
	DifficultyMedium Difficulty = "medium"
	DifficultyHard   Difficulty = "hard"
)

type GameMasterRequest struct {
	GameType     string
	Difficulty   Difficulty
	Context      string
	RoundNumber  int
	History      []string
}

type GameMasterResponse struct {
	Scenario    string
	Question    string
	Options     []string
	TabooWords  []string
	SecretWord  string
	RawOutput   string
	IsFallback  bool
}
```

```go
package prompts

import (
	"bytes"
	"embed"
	"fmt"
	"text/template"
)

//go:embed *.tmpl
var promptFS embed.FS

var tmplCache = template.Must(template.ParseFS(promptFS, "*.tmpl"))

func RenderPrompt(templateName string, data any) (string, error) {
	var buf bytes.Buffer
	err := tmplCache.ExecuteTemplate(&buf, templateName+".tmpl", data)
	if err != nil {
		return "", fmt.Errorf("failed to render prompt template %s: %w", templateName, err)
	}
	return buf.String(), nil
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - LLM menghasilkan halusinasi / tidak mengikuti format delimiter: Parser melakukan fallback ke regex secondary match. Jika tetap gagal mengekstrak elemen wajib, sistem langsung mengaktifkan `fallback.go` demi menjaga kelancaran game.
  - Template name tidak terdaftar: `RenderPrompt` mengembalikan error eksplisit (fail-fast), bukan string kosong.
- **Strategi Rollback:**
  - Kembalikan perubahan kode melalui Git. Komponen ini tidak memutasi database.
