---
id: TICKET-1762433800
plan_id: PLAN-1762433800
title: XP Calculation & Level System
status: Todo
priority: High
labels: [Backend, Go, Gamification, Database]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mendapatkan Experience Points (XP) dan naik level setiap kali menyelesaikan aktivitas belajar atau game di Radovid Bot,
Sehingga motivasi belajar saya meningkat melalui perkembangan level yang terukur dan kompetitif.

Fitur gamifikasi membutuhkan sistem kalkulasi XP yang adil dan transparan. Perolehan XP dipengaruhi oleh jenis aktivitas, tingkat kesulitan game (*difficulty multiplier*), serta performa jawaban. Seluruh riwayat perolehan XP harus dicatat dalam tabel audit log (`user_xp_log`) untuk integritas data dan mencegah eksploitasi, serta rumus kenaikan level menggunakan formula kuadratik konstan $\text{XP}_{\text{required}} = \text{level}^2 \times 100$.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433800-xp-calculation-level-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Tersedia skema migrasi database `009_create_user_xp_log.sql` untuk mencatat riwayat perolehan XP (id, user_id, amount, source_activity, multiplier, created_at)
- [ ] Formula kalkulasi kenaikan level diimplementasikan: $\text{Level}(XP) = \lfloor\sqrt{\frac{XP}{100}}\rfloor$ dan ambang batas level $N$: $N^2 \times 100$ XP
- [ ] Pengganda tingkat kesulitan (*difficulty multiplier*) diterapkan secara akurat: Easy (1.0x), Medium (1.5x), Hard (2.0x)
- [ ] Use case `AwardXP` secara atomik memperbarui `user_progress` (total_xp, current_level, updated_at) dan memasukkan baris baru pada `user_xp_log` dalam satu transaksi database SQLite
- [ ] Event / status level-up terdeteksi ketika akumulasi XP melampaui batas level saat ini dan mengembalikan payload informasi kenaikan level
- [ ] Unit test mencakup perhitungan batas level nol, kelipatan batas, multiplier desimal (pembulatan integer ke bawah / `math.Floor`), dan validasi input negatif

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/migrations/009_create_user_xp_log.sql`
- `../../../../radovid-bot-api/internal/domain/progress/entity.go`
- `../../../../radovid-bot-api/internal/domain/progress/calculator.go`
- `../../../../radovid-bot-api/internal/domain/progress/repository.go`
- `../../../../radovid-bot-api/internal/domain/progress/calculator_test.go`
- `../../../../radovid-bot-api/internal/usecase/progress/award_xp.go`
- `../../../../radovid-bot-api/internal/usecase/progress/award_xp_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/progress_repository.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
