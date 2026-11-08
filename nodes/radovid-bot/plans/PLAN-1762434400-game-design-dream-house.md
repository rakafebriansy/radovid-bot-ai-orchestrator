---
id: PLAN-1762434400
ticket_id: TICKET-1762434400
title: "Game 15: Design Your Dream House"
status: Completed
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
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/dream_house.go` | Model HouseDesignPhase, HouseDesignState, ArchitecturalAssessment, skoring, XP, dan spatial prepositions detection |
| `CREATE` | `../../../../radovid-bot-api/internal/domain/game/dream_house_test.go` | Unit test domain model, validasi deskripsi (15-2000 chars), spatial prepositions, skoring, XP |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity.go` | Pendaftaran konstanta `GameTypeDesignDreamHouse = "design_dream_house"` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/entity_test.go` | Unit test validasi GameTypeDesignDreamHouse |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/dream_house.tmpl` | Prompt template Ollama pembuat evaluasi arsitektur dan sintesis hunian |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/embed_test.go` | Unit test rendering prompt dream house |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser.go` | Parser respons terstruktur `parseDreamHouse` dan `ParseDreamHouseEvaluation` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/parser_test.go` | Unit test parsing output AI dream house |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback.go` | Fallback data evaluasi arsitektur deterministik `dreamHouseFallbacks` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/fallback_test.go` | Unit test fallback dream house |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/dream_house.go` | Use case orkestrasi 3 tahapan desain rumah, evaluasi LLM, dan perolehan XP |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/game/dream_house_test.go` | Unit test use case, transisi fase, pemanggilan Ollama, skoring, fallback, dan perolehan XP |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates.go` | Embed builder fase perancangan (`BuildDreamHousePhaseEmbed`) dan hasil akhir (`BuildDreamHouseEvaluationEmbed`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/discord/game_templates_test.go` | Unit test format embed dan komponen interaksi dream house Discord |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Pendaftaran subcommand slash `dream-house` dengan opsi `theme` dan `difficulty` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_test.go` | Unit test definisi subcommand slash dream house |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Routing subcommand `dream-house` ke `HandlePlayDreamHouse` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/game_handler.go` | Injeksi `dreamHouseUseCase` ke `GameHandler` dan delegator component/modal |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house.go` | Handler play, modal open, dan modal submit tiap fase perancangan rumah |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/discord/game_dream_house_test.go` | Unit test delivery handler Discord dream house |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Inisialisasi use case dan registrasi handler interaction prefix `dreamhouse:` |

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
