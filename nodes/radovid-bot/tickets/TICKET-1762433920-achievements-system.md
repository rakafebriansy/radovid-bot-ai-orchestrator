---
id: TICKET-1762433920
plan_id: PLAN-1762433920
title: Achievements System
status: Todo
priority: High
labels: [Backend, Go, Gamification, Domain, Discord]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Bahasa Inggris,
Saya ingin mendapatkan lencana penghargaan (achievements/badges) ketika mencapai pencapaian tertentu (seperti menyelesaikan game pertama, mempertahankan streak 7 hari berturut-turut, atau menguasai tata bahasa),
Sehingga saya merasa diapresiasi dan memiliki tujuan jangka panjang dalam ekosistem bot.

Sistem achievement memberikan penghargaan atas pencapaian milestone penting pengguna. Setiap achievement memiliki atribut unik (ID, nama, deskripsi, emoji icon, XP reward, dan trigger condition). Ketika aktivitas pengguna memicu kondisi pembukaan lencana baru yang belum pernah diraih, bot akan:
1. Menyimpan data lencana ke tabel `achievements`.
2. Menghadiahkan bonus XP langsung ke profil pengguna via use case `AwardXP`.
3. Membangun dan mengirimkan notifikasi perayaan achievement ke pengguna via pesan Discord.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433920-achievements-system.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Tersedia katalog achievement statis di domain layer mencakup: `first_game` (First Steps - 50 XP), `streak_7` (Week Warrior - 150 XP), `streak_30` (Monthly Devotion - 500 XP), `grammar_master` (Syntax Sorcerer - 200 XP), `vocab_champion` (Lexicon Master - 200 XP), `boss_slayer` (Dragon Tamer - 250 XP)
- [ ] Engine evaluator (`EvaluateAchievements`) mengevaluasi statistik pengguna pasca-aktivitas dan hanya mengembalikan daftar lencana baru yang belum pernah diraih
- [ ] Idempotensi terjamin: pengguna tidak dapat memperoleh achievement yang sama lebih dari satu kali (dicegah pada level logika aplikasi dan SQLite unique constraint)
- [ ] Pemberian bonus XP otomatis dipicu untuk setiap lencana baru yang terbuka
- [ ] Format embed Discord notifikasi achievement (`BuildAchievementUnlockedEmbed`) menyajikan emoji, nama lencana, deskripsi, serta bonus XP yang didapat
- [ ] Unit testing menyeluruh untuk evaluator kondisi milestone (kondisi tercapai vs belum tercapai, filter duplicate achievements)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/achievement/entity.go`
- `../../../../radovid-bot-api/internal/domain/achievement/catalog.go`
- `../../../../radovid-bot-api/internal/domain/achievement/evaluator.go`
- `../../../../radovid-bot-api/internal/domain/achievement/repository.go`
- `../../../../radovid-bot-api/internal/domain/achievement/evaluator_test.go`
- `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock.go`
- `../../../../radovid-bot-api/internal/usecase/achievement/check_and_unlock_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/achievement_repository.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
