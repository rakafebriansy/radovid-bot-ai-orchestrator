# CHANGELOG — Radovid Bot Node

> **Format:** Descending (terbaru di atas). Setiap entry WAJIB menggunakan struktur baku dari `../../global-docs/templates/changelog_entry_template.md`.

---

### [2026-11-03 23:48:00] - Rename: Node & System Name Alignment to Radovid Bot
> **Trigger:** Prompt Driven | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Penyesuaian nama sistem dan node menjadi `radovid-bot` sesuai instruksi developer.

**Perubahan:**
- [Changed] Mengubah nama node dari `nodes/veylo-bot/` menjadi `nodes/radovid-bot/`
- [Changed] Memperbarui seluruh referensi dokumen global (`prd.md`, `design-system.md`, diagram-diagram) dan dokumen node (`system-design.md`, `development-planning.md`, `project-context.md`, `CHANGELOG.md`) menggunakan nama `Radovid Bot` dan slash command `/radovid`

**Files Affected:**
`nodes/radovid-bot/`, `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`, `nodes/radovid-bot/docs/system-design.md`, `nodes/radovid-bot/docs/development-planning.md`, `nodes/radovid-bot/docs/diagrams/*`

---

### [2026-11-03 23:16:00] - Initialization: Radovid Bot Node Inception
> **Trigger:** Startup Prompt (Single-Project Environment) | **Branch:** `main` | **Repo:** `radovid-bot-ai-orchestrator`

- **Konteks:** Inisialisasi perdana node `radovid-bot` dalam ekosistem AI Orchestrator untuk proyek Radovid Bot — AI-powered Discord English learning companion.

**Perubahan:**
- [Added] Duplikasi `nodes/_template/` menjadi `nodes/radovid-bot/` (Mode Operasional: MODE 1 — Autonomous Planning)
- [Added] Penulisan `nodes/radovid-bot/main.md` dengan MODE 1 terpilih di baris pertama
- [Added] Pemindaian codebase greenfield (`radovid-bot-api/`, hanya `go.mod`)
- [Added] Penulisan `nodes/radovid-bot/guidelines/project-context.md` (hasil pindai + konvensi lokal: Clean Architecture, interface-first, UUID PK, epoch ms timestamps, slash commands only, Ollama timeout wajib, input sanitization, table-driven tests)
- [Added] Penulisan `nodes/radovid-bot/docs/system-design.md` (arsitektur Clean Architecture, ERD, API contracts)
- [Added] Penulisan `nodes/radovid-bot/docs/development-planning.md` (4 fase, 30 item backlog, pasangan tiket & rencana dengan epoch timestamps)
- [Added] Pembuatan diagram PlantUML: `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`, `nodes/radovid-bot/docs/diagrams/erd.puml`, `nodes/radovid-bot/docs/diagrams/flowchart.puml`, `nodes/radovid-bot/docs/diagrams/state-diagram.puml`
- [Added] Pembaruan `global-docs/prd.md` dengan spesifikasi PRD Radovid Bot
- [Added] Pembaruan `global-docs/design-system.md` dengan design system Radovid Bot
- [Added] Penulisan `nodes/radovid-bot/CHANGELOG.md` (file ini)

**Files Affected:**
`nodes/radovid-bot/main.md`, `nodes/radovid-bot/guidelines/project-context.md`, `nodes/radovid-bot/docs/system-design.md`, `nodes/radovid-bot/docs/development-planning.md`, `nodes/radovid-bot/docs/diagrams/erd.puml`, `nodes/radovid-bot/docs/diagrams/flowchart.puml`, `nodes/radovid-bot/docs/diagrams/state-diagram.puml`, `nodes/radovid-bot/CHANGELOG.md`, `global-docs/prd.md`, `global-docs/design-system.md`, `global-docs/diagrams/user-journey.puml`, `global-docs/diagrams/system-activity.puml`

**Next Action:** Mulai eksekusi Fase 1 — `TICKET-1762174595` / `PLAN-1762174595`: Setup Go Project Structure.
