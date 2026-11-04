---
id: PLAN-1762434280
ticket_id: TICKET-1762434280
title: "Game 13: Movie Review"
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 13: Movie Review

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434280](../tickets/TICKET-1762434280-game-movie-review.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Mengembangkan alur aktivitas edukatif "Movie Review" (`movie_review`) dengan state machine 3 fase interaktif, integrasi dialog terpandu bersama Ollama Gemma 3 4b, serta sistem evaluasi kecakapan ekspresi opini dan kosakata sinematik.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler `/radovid start` dan modal interaksi di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, prompt templates di `internal/infrastructure/ollama`, dan modul audit log XP.
  - Potensi Efek Samping: Panjang teks submission pada fase ulasan film memerlukan penanganan Discord Modal textarea (hingga 4000 karakter) dan splitting prompt yang tepat agar tidak melampaui context window lokal Ollama.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/movie_review.go` | Model ReviewPhase, ReviewSession, VocabularyRecommendation |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeMovieReview = "movie_review"` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/movie_review_game.go` | Use case state machine 3 fase ulasan dan evaluasi wacana kritis |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/movie_review_game_test.go` | Unit test transisi fase dan pengolahan skor ulasan |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go` | Prompt bertingkat untuk pemandu ulasan film dan analizer opini |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Pendaftaran sub-perintah start tipe game movie review |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go` | Modal submission handler untuk input teks review per fase |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Format embed kartu pertanyaan film dan laporan evaluasi akhir |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/usecase/game/movie_review_game_test.go`.
   - Uji logika transisi fase: dari `PhasePremise` -> `PhaseCharacters` -> `PhaseVerdict` -> `PhaseCompleted`. Pastikan tidak dapat melompati fase secara acak.
   - Pastikan tes berstatus RED.
2. **Langkah 2 (Domain Modeling):**
   - Buat enum fase: `MovieReviewPhasePremise`, `MovieReviewPhaseCharacters`, `MovieReviewPhaseVerdict`, `MovieReviewPhaseCompleted`.
   - Buat struct data review: `MovieReviewState` (`MovieTitle`, `Genre`, `CurrentPhase`, `Answers map[string]string`, `FinalScore int`).
3. **Langkah 3 (Use Case & Prompt Engineering):**
   - Susun prompt dinamis di `prompts.go` yang mengambil judul film pengguna dan merumuskan pertanyaan tajam nan kontekstual.
   - Bangun prompt evaluasi akhir yang mengurai respon pengguna menjadi: skor kosakata evaluatif, pemakaian idiom/frasa penghubung, koreksi tata bahasa, dan 3 rekomendasi kosakata lanjutan.
   - Integrasikan hasil evaluasi dengan sistem XP (`AwardXP`) berbasis metrik kualitas.
4. **Langkah 4 (Delivery Gate & Formatting):**
   - Pastikan embed Discord hasil ulasan film tersusun rapi dengan bidang (fields) terpisah untuk Analisis Sinopsis, Karakter, dan Opini Kritis.
   - Verifikasi isolasi domain dari dependensi DiscordGo.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/movie_review.go
package game

type MovieReviewPhase int

const (
	MovieReviewPhasePremise MovieReviewPhase = iota
	MovieReviewPhaseCharacters
	MovieReviewPhaseVerdict
	MovieReviewPhaseCompleted
)

type MovieReviewAssessment struct {
	CoherenceScore      int      `json:"coherence_score"`
	VocabularyScore     int      `json:"vocabulary_score"`
	OpinionPhrasing     string   `json:"opinion_phrasing"`
	RecommendedWords    []string `json:"recommended_words"`
	GrammarImprovements []string `json:"grammar_improvements"`
	OverallFeedback     string   `json:"overall_feedback"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain menginput judul film fiktif atau aneh. Prompt AI diarahkan untuk bersikap agnostik terhadap kebenaran fakta film riil dan memfokuskan penilaian murni pada struktur argumen dan bahasa Inggris pengguna.
- **Strategi Rollback:** Jika pemain menutup modal tanpa submit, status sesi tetap berada di fase terakhir dengan tombol interaktif "Lanjutkan Ulasan" agar pemain tidak kehilangan progres diskusi.
