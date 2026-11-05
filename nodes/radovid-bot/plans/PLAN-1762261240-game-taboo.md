---
id: PLAN-1762261240
ticket_id: TICKET-1762261240
title: "Game 2: Taboo"
status: Completed
created_at: 2026-11-04T00:15:00+07:00
---

# Rencana Implementasi Teknis: Game 2 - Taboo

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762261240](../tickets/TICKET-1762261240-game-taboo.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun modul game "Taboo" dengan mekanisme turn-based 2-player di Discord. Modul ini mencakup orkestrasi pemanggilan AI Game Master untuk menghasilkan target word dan 5 taboo words, pengiriman kartu taboo via ephemeral interaction, penangkapan deskripsi pemain melalui Discord Modal (`discordgo.InteractionModalSubmit`), serta implementasi evaluator hibrida (verifikasi pelanggaran kata deterministik via word-boundary regex + penilaian semantik AI terhadap kecukupan petunjuk deskripsi).
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord Interaction Router (`handler.go`), `/radovid play taboo`.
  - Komponen Terdampak: `internal/usecase/game/taboo.go`, `internal/usecase/game/taboo_evaluator.go`, Discord Modal Handlers.
  - Potensi Efek Samping: False positives pada deteksi kata terlarang jika pencocokan string bersifat naif (misal: "cat" mencocokkan "catch"); dicegah dengan strict word-boundary tokenization (`\bword\b`).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
> **Peringatan Relative Path:** Gunakan relative path untuk seluruh referensi berkas di bawah ini.

| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/taboo_entity.go` | Domain struct TabooCard, TabooSubmission, TabooEvaluation |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator.go` | Detektor deterministik taboo words & semantik akurasi Ollama |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/taboo_evaluator_test.go` | Unit test edge-case deteksi taboo words (tanda baca, kapitalisasi, jamak) |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/taboo.go` | Core usecase alur giliran, kartu rahasia, penilaian, dan skor |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/taboo_test.go` | Integration-level usecase test untuk turn-switching antar pemain |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go` | Handler slash command dan tombol pembuka modal deskripsi |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go` | Handler Discord Modal submit event untuk memproses input deskripsi |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Domain Model & Evaluator Red Test):**
   - Buat `taboo_entity.go` yang memodelkan kartu taboo, submission pemain, dan skor.
   - Buat `taboo_evaluator_test.go` untuk menguji kata-kata terlarang dengan berbagai variasi casing (`ANIMAL`, `animal`), tanda baca (`animal,`), dan varian jamak.
   - Buktikan tes evaluator RED.
2. **Langkah 2 (Implementasi Detektor Deterministik & Evaluasi AI):**
   - Tulis `taboo_evaluator.go` dengan fungsi `CheckViolations(description string, tabooWords []string) []string`.
   - Tambahkan fungsi evaluasi semantik yang memanggil Ollama untuk menilai apakah deskripsi cukup merepresentasikan target word tanpa menyebutkannya.
   - Pastikan tes evaluator GREEN.
3. **Langkah 3 (Red Test Taboo Use Case):**
   - Tulis `taboo_test.go` untuk mensimulasikan alur permainan:
     - Giliran 1: Pemain A adalah describer, menerima kartu ephemeral.
     - Pemain A submit modal: Deskripsi valid tanpa kata terlarang -> Mendapat poin, giliran berganti ke Pemain B.
     - Giliran 2: Pemain B submit modal: Menggunakan 1 kata terlarang -> Terdeteksi, 0 poin, penjelasan pelanggaran.
   - Buktikan tes RED.
4. **Langkah 4 (Implementasi Use Case Taboo):**
   - Selesaikan implementasi `taboo.go` dengan turn management dan state updates.
   - Pastikan semua skenario tes usecase GREEN.
5. **Langkah 5 (Implementasi Discord Modal Interaction):**
   - Buat `command_play_taboo.go` yang menampilkan kartu game dan tombol "Open Description Form" (hanya bisa diklik describer).
   - Buat `interaction_taboo_modal.go` untuk menangani event `InteractionTypeModalSubmit`, membaca field input teks, mengeksekusi use case, dan mengirim pesan respon ke channel.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

```go
package game

import (
	"fmt"
	"regexp"
	"strings"
)

type TabooCard struct {
	TargetWord string
	TabooWords []string
}

type TabooEvaluation struct {
	Violations  []string
	IsAccurate  bool
	Score       int
	Feedback    string
}

type TabooEvaluator struct{}

func NewTabooEvaluator() *TabooEvaluator {
	return &TabooEvaluator{}
}

func (e *TabooEvaluator) CheckViolations(description string, tabooWords []string) []string {
	var violations []string
	cleanDesc := strings.ToLower(description)
	for _, word := range tabooWords {
		pattern := fmt.Sprintf(`(?i)\b%s\b`, regexp.QuoteMeta(strings.TrimSpace(word)))
		matched, _ := regexp.MatchString(pattern, cleanDesc)
		if matched {
			violations = append(violations, word)
		}
	}
	return violations
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:**
  - Pemain yang bukan describer mencoba membuka modal atau mengirim deskripsi: Divalidasi di usecase; request ditolak dengan pesan ephemeral `ErrNotYourTurn`.
  - Input teks kosong atau whitespace pada modal Discord: Modal validator menolak submission dengan error interaktif "Description cannot be empty".
  - Penggunaan tanda baca rapat (contoh: `"animal."` atau `"(animal)"`): Regex `\b` (word-boundary) menjamin kata tetap terdeteksi sebagai pelanggaran tanpa terlewat.
- **Strategi Rollback:**
  - Kembalikan perubahan kode melalui git revert. Status sesi yang aktif dapat dibatalkan manual melalui slash command `/radovid end`.
