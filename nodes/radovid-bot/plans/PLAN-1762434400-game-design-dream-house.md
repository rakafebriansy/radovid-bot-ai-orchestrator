---
id: PLAN-1762434400
ticket_id: TICKET-1762434400
title: "Game 15: Design Your Dream House"
status: Draft
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: Game 15: Design Your Dream House

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434400](../tickets/TICKET-1762434400-game-design-dream-house.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun mini-game komunikasi deskriptif kreatif "Design Your Dream House" (`design_dream_house`) dengan alur tahapan bertingkat (Exterior, Interior Layout, Signature Sanctuary), pemanfaatan prompting Ollama Gemma 3 4b untuk mendeteksi kosakata spasial dan estetika, serta perancangan narasi sintesis arsitektural.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Handler perintah `/radovid start` dan modal submit di `internal/delivery/discord`.
  - Komponen Terdampak: `internal/domain/game`, `internal/usecase/game`, pustaka prompt `internal/infrastructure/ollama`, dan modul audit XP.
  - Potensi Efek Samping: Karena game bersifat multi-step, state sementara antar tahapan disimpan dalam memori sesi aktif atau tabel `game_rounds` agar pemain dapat melanjutkan tahapan tanpa kehilangan data sebelumnya.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/dream_house.go` | Model HouseDesignPhase, HouseDesignState, ArchitecturalAssessment |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeDesignDreamHouse = "design_dream_house"` |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/dream_house_game.go` | Use case alur tahapan perancangan, evaluasi kosakata spasial, dan feedback |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/dream_house_game_test.go` | Unit test transisi tahapan desain dan evaluasi kosakata spasial |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts.go` | Prompt pemandu perancangan rumah dan prompt kurator arsitektur |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_start.go` | Pendaftaran game type design dream house pada command start |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handlers.go` | Modal form input tahapan eksterior, interior, dan sanctuary |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/embeds.go` | Format embed visual arsitektur dan sintesis presentasi hunian |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/usecase/game/dream_house_game_test.go`.
   - Siapkan skenario pengujian: verifikasi transisi urutan tahapan (Exterior -> Interior -> Sanctuary -> Final Recap) dan pendeteksian preposisi spasial (misal: "adjacent", "overlooking").
   - Pastikan tes berstatus RED.
2. **Langkah 2 (Domain Modeling):**
   - Definisikan tipe enum: `HousePhaseExterior`, `HousePhaseInterior`, `HousePhaseSanctuary`, `HousePhaseFinished`.
   - Buat struct `HouseDesignSubmission`: `ExteriorDesc`, `InteriorDesc`, `SanctuaryDesc`.
   - Buat struct `ArchitecturalAssessment`: `SpatialPrepositionsUsed []string`, `AestheticScore int`, `SpatialScore int`, `AdvancedVocabulary []string`, `ArchitecturalSummary string`.
3. **Langkah 3 (Use Case & Prompt Orchestration):**
   - Susun template prompt kurasi arsitektur di `prompts.go` yang menginstruksikan Gemma 3 4b mengekstrak preposisi spasial dan menghasilkan umpan balik estetika berbobot.
   - Implementasikan method `ProcessPhaseSubmission(ctx, sessionID, phase, text)` di use case layer.
   - Panggil `AwardXP` di fase penutup dengan bonus ekstra jika pemain berhasil menyematkan $\ge 4$ preposisi spasial akurat.
4. **Langkah 4 (Delivery Gate & Format Visual):**
   - Format embed Discord dengan nuansa warna alam/bumi (Earth tone/Emerald) untuk mencerminkan keindahan desain arsitektur.
   - Sajikan bagian "Architect's Synthesis" yang merangkum keseluruhan rumah impian pemain dalam satu narasi puitis yang inspiratif.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/domain/game/dream_house.go
package game

type HouseDesignPhase int

const (
	HousePhaseExterior HouseDesignPhase = iota
	HousePhaseInterior
	HousePhaseSanctuary
	HousePhaseFinished
)

type ArchitecturalAssessment struct {
	SpatialScore         int      `json:"spatial_score"`
	AestheticScore       int      `json:"aesthetic_score"`
	PrepositionsFound    []string `json:"prepositions_found"`
	EnrichmentAdjectives []string `json:"enrichment_adjectives"`
	ArchitecturalSummary string   `json:"architectural_summary"`
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Pemain mengirim deskripsi yang tidak berkaitan dengan arsitektur/rumah (misal: lirik lagu atau teks acak). AI evaluator mengenali penyimpangan tema dan memberikan pengingat sopan untuk memfokuskan deskripsi pada elemen spasial hunian.
- **Strategi Rollback:** Seluruh deskripsi tiap tahapan disimpan ke record `game_rounds` di basis data SQLite; jika koneksi bot terputus sementara di tengah permainan, pemain dapat melanjutkan tahapan yang sedang aktif.
