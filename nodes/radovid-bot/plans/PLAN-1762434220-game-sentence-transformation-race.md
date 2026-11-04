---
id: PLAN-1762434220
ticket_id: TICKET-1762434220
title: "Game 12: Sentence Transformation Race"
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 12: Sentence Transformation Race

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434220](../tickets/TICKET-1762434220-game-sentence-transformation-race.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun mini-game "Sentence Transformation Race" (`sentence_transformation`) yang menuntut pemain mengubah struktur gramatikal kalimat berbahasa Inggris di bawah batas waktu tertentu, dengan penilaian kombinasi ketepatan sintaksis dan kecepatan pengiriman jawaban.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord interaction command `/radovid start` dan modal submit handler.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, kalkulator skor kecepatan, dan XP system.
  - Potensi Efek Samping: Jeda waktu jaringan (network latency) antara pengiriman Discord modal dan server harus diperhitungkan dengan toleransi *grace period* 2-3 detik agar tidak membatalkan jawaban valid di detik terakhir.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/transformation.go` | Model TransformationType, Challenge, PlayerSubmission, ScoreCalculator |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeSentenceTransformation = "sentence_transformation"` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/transformation_game.go` | Use case orkestrasi ronde transformasi, evaluasi semantik, dan skoring |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/transformation_game_test.go` | Unit test algoritma skoring kecepatan dan akurasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go` | Template prompt tantangan transformasi dan prompt evaluasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Registrasi opsi game type transformasi kalimat |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go` | Modal input respon dan pencatatan timestamp submit |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Embed tantangan transformasi dengan countdown visual dan papan hasil |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `internal/usecase/game/transformation_game_test.go`.
   - Uji rumus kalkulasi skor: akurasi 100 dengan sisa waktu 50% harus menghasilkan skor 125. Jawaban lewat batas waktu (expired) harus diberi skor 0.
   - Pastikan tes gagal (RED).
2. **Langkah 2 (Domain Modeling & Formula):**
   - Buat enum jenis transformasi: `ActiveToPassive`, `PassiveToActive`, `DirectToReported`, `ReportedToDirect`, `TenseShift`.
   - Implementasikan fungsi murni `CalculateSpeedScore(accuracy int, elapsedMs int64, durationMs int64) int`.
3. **Langkah 3 (Use Case & AI Prompts):**
   - Buat template prompt di `prompts.go` yang menghasilkan kalimat input, instruksi transformasi, dan contoh acuan.
   - Buat template prompt evaluasi yang mengembalikan JSON terstruktur: `{ "accuracy_score": 90, "is_semantically_equivalent": true, "ideal_transformation": "...", "feedback": "..." }`.
   - Orkestrasi flow: start round -> tampilkan prompt di embed -> terima submission via modal -> hitung selisih waktu -> panggil evaluasi Ollama -> award XP proporsional.
4. **Langkah 4 (Delivery Gate & Edge Cases):**
   - Tangani kasus timeout: jika waktu habis sebelum submit, tutup modal dan kirimkan ringkasan solusi ideal dari AI.
   - Pastikan toleransi pergeseran jam lokal dengan menggunakan epoch server murni.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/transformation.go
package game

type TransformationType string

const (
	TransformActiveToPassive  TransformationType = "active_to_passive"
	TransformDirectToReported TransformationType = "direct_to_reported"
	TransformTenseShift       TransformationType = "tense_shift"
)

func CalculateSpeedScore(accuracy int, elapsedMs, durationMs int64) int {
	if accuracy <= 0 || elapsedMs > durationMs {
		return 0
	}
	remainingRatio := float64(durationMs-elapsedMs) / float64(durationMs)
	if remainingRatio < 0 {
		remainingRatio = 0
	}
	multiplier := 1.0 + (remainingRatio * 0.5)
	return int(float64(accuracy) * multiplier)
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain mengirim kalimat identik dengan kalimat asal tanpa melakukan transformasi. Sistem harus mengenali kesamaan verbatim dan memberikan skor akurasi 0.
- **Strategi Rollback:** Bila model Ollama mengalami kelambatan inferensi saat menghitung evaluasi, waktu komputasi AI tidak boleh mengurangi poin pemain; penghitungan elapsed time dihentikan seketika saat event interaksi Discord diterima.
