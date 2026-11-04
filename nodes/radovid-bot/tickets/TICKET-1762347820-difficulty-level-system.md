---
id: TICKET-1762347820
plan_id: PLAN-1762347820
title: Difficulty Level System
status: Todo
priority: High
labels: [Settings, User, Core, Architecture, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin dapat mengatur tingkat kemahiran bahasa Inggris saya (Beginner, Intermediate, Advanced) melalui perintah `/radovid settings`,
Sehingga AI Game Master menyesuaikan tingkat kerumitan pertanyaan, kekayaan kosakata, serta gaya umpan balik bahasa agar sesuai dengan kapasitas pemahaman saya.

Sistem Radovid Bot membutuhkan mekanisme penyesuaian tingkat kemahiran (Difficulty Level System) yang persisten di tingkat profil pengguna. Level kesulitan ini mempengaruhi seluruh game dan modul evaluasi:
1. **Beginner (A1-A2):** Kosakata dasar, kalimat pendek, nada feedback sangat suportif, toleran terhadap kesalahan minor.
2. **Intermediate (B1-B2):** Percakapan natural, ungkapan umum & idiomatis, feedback berimbang antara apresiasi dan koreksi gramatikal.
3. **Advanced (C1-C2):** Kosakata kompleks, topik abstrak/profesional, evaluasi ketat mencakup kolokasi, kesesuaian register, dan kealamian gaya bahasa.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347820-difficulty-level-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Definisi tipe enum `Difficulty` (`Beginner`, `Intermediate`, `Advanced`) dengan metode validasi dan pemetaan CEFR di domain user
- [ ] Skema database `users` diperbarui via migration SQL untuk menyimpan kolom `difficulty` dengan nilai default `Beginner`
- [ ] Slash command `/radovid settings` menampilkan antarmuka konfigurasi tingkat kesulitan dengan interaksi Discord Select Menu / Buttons
- [ ] Pembaruan profil difficulty tersimpan persisten ke SQLite dan aktif seketika untuk semua sesi game berikutnya
- [ ] Seluruh prompt template Ollama (`{{.Difficulty}}`) menyertakan instruksi penyesuaian gaya respons AI berdasarkan level profil pengguna
- [ ] Unit test mencakup validasi parsing nilai difficulty, transisi level, dan fallback default jika input tidak dikenali

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/009_add_difficulty_to_users.sql`
- `../../../../radovid-bot-api/internal/domain/user/difficulty.go`
- `../../../../radovid-bot-api/internal/domain/user/repository.go`
- `../../../../radovid-bot-api/internal/usecase/user/settings.go`
- `../../../../radovid-bot-api/internal/usecase/user/settings_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/user_repository.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_settings.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

