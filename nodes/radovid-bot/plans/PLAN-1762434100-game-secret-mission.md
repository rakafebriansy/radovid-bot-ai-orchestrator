---
id: PLAN-1762434100
ticket_id: TICKET-1762434100
title: "Game 10: Secret Mission"
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 10: Secret Mission

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434100](../tickets/TICKET-1762434100-game-secret-mission.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun alur mini-game "Secret Mission" (`secret_mission`) yang memanfaatkan prompting terarah Ollama Gemma 3 4b untuk mendistribusikan misi target rahasia kepada peserta via Discord Ephemeral interaction dan mengevaluasi naturalness serta akurasi tata bahasa secara terstruktur.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Discord interaction dispatcher (`internal/delivery/discord/radovid_start.go` dan `game_handlers.go`).
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, `internal/infrastructure/ollama`, Discord message builder.
  - Potensi Efek Samping: Interaksi ephemeral mengharuskan penyimpanan state misi rahasia di memori sesi aktif atau database ronde agar dapat diverifikasi saat submit jawaban publik.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/secret_mission.go` | Model domain misi rahasia, target kosa kata, skor naturalitas |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeSecretMission = "secret_mission"` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/secret_mission_game.go` | Use case orkestrasi fase briefing, submission, dan evaluasi AI |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/secret_mission_game_test.go` | Unit test alur state game dan parsing evaluasi JSON |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go` | Penambahan prompt generator misi dan prompt evaluator naturalness |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Opsi game type `secret_mission` di slash command start |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go` | Modal submit respons dan dispatcher tombol ambil misi |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Embed briefing publik dan embed kartu misi ephemeral |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat unit test pada `internal/usecase/game/secret_mission_game_test.go`.
   - Simulasikan evaluasi output JSON dari Ollama: verifikasi string JSON valid dipetakan ke struct `SecretMissionEvaluation` dengan skor akurat.
   - Pastikan tes gagal (RED).
2. **Langkah 2 (Domain & Prompt Engineering):**
   - Tambahkan enum `GameTypeSecretMission` di `internal/domain/game/entity.go`.
   - Definisikan struct domain `SecretMissionRound` (`Topic`, `PlayerTargets map[string]string`, `Submissions map[string]string`).
   - Buat template prompt pada `prompts.go` yang menginstruksikan Gemma 3 4b menghasilkan JSON terstruktur: `{ "target_used": true, "grammar_score": 8, "naturalness_score": 9, "feedback": "..." }`.
3. **Langkah 3 (Use Case & Flow Execution):**
   - Implementasikan metode `GenerateMission(ctx, difficulty)` dan `EvaluateSubmission(ctx, topic, target, text)`.
   - Pasang tombol Discord "View Secret Mission" yang membalas dengan Ephemeral response berisi target kata/tata bahasa pemain tersebut.
   - Sediakan Discord modal interaktif untuk submit kalimat jawaban.
4. **Langkah 4 (Award XP & Delivery Gate):**
   - Hitung reward XP dari performa evaluasi: $\text{BaseXP} + (\text{grammar\_score} + \text{naturalness\_score}) \times 3$, lalu panggil `AwardXP`.
   - Verifikasi fail-safe jika Ollama mengembalikan teks non-JSON (regex fallback extractor).

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/secret_mission.go
package game

type SecretMissionAssignment struct {
	PlayerID     string `json:"player_id"`
	TargetWord   string `json:"target_word"`
	GrammarFocus string `json:"grammar_focus"`
}

type SecretMissionEvaluation struct {
	TargetUsed       bool   `json:"target_used"`
	GrammarScore     int    `json:"grammar_score"`
	NaturalnessScore int    `json:"naturalness_score"`
	Feedback         string `json:"feedback"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain mengirim kalimat kosong atau spam karakter acak. Sistem memvalidasi minimal 5 kata bermakna sebelum mengirim ke Ollama untuk menghemat resource inferensi.
- **Strategi Rollback:** Jika Ollama mengalami timeout selama evaluasi (melebihi 30 detik), berikan pesan peringatan ramah dan izinkan pemain melakukan *retry evaluation* tanpa membatalkan ronde.
