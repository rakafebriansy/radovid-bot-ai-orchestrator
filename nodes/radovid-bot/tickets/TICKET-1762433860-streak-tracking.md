---
id: TICKET-1762433860
plan_id: PLAN-1762433860
title: Streak Tracking (Daily Activity)
status: Todo
priority: High
labels: [Backend, Go, Gamification, Domain]
---

# Deskripsi
Sebagai Pengguna Discord / Pembelajar Aktif,
Saya ingin bot mencatat hari-hari berturut-turut saya belajar bahasa Inggris dan memberikan bonus XP atas konsistensi saya,
Sehingga saya termotivasi untuk hadir dan berlatih setiap hari tanpa terputus.

Streak tracking adalah mekanisme retensi pengguna utama pada Radovid Bot. Streak dihitung berdasarkan jarak waktu aktivitas terakhir (`last_activity_at`) dalam milidetik Epoch. Aturan bisnis menetapkan:
1. Aktivitas berulang pada hari kalender yang sama tidak menambah streak (streak tetap).
2. Aktivitas pada hari berikutnya (selisih antara 24 hingga 48 jam atau beda tanggal kalender berturut-turut) menambah streak sebesar 1 (+1).
3. Jika pengguna tidak beraktivitas lebih dari 48 jam, streak direset kembali menjadi 1.
4. Setiap kenaikan streak memberikan bonus XP harian: $\text{streak\_days} \times 5$ XP.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433860-streak-tracking.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Logika perhitungan status streak (`EvaluateStreak`) membedakan 3 kondisi: `SameDay` (streak tetap), `ConsecutiveDay` (streak bertambah 1), dan `Expired` (streak reset ke 1)
- [ ] Streak reset terjadi jika jarak waktu antara aktivitas saat ini dan `last_activity_at` melampaui 48 jam (172.800.000 ms)
- [ ] Formula kalkulasi bonus XP harian: $\text{streak\_days} \times 5$ XP diberikan ketika pengguna mencapai status `ConsecutiveDay` atau memulai hari baru
- [ ] Use case `RecordDailyActivity` memperbarui `streak_days` dan `last_activity_at` di tabel `user_progress` serta memicu pemberian bonus XP melalui modul `AwardXP`
- [ ] Penanganan zona waktu menggunakan standar UTC murni untuk mencegah pergeseran streak akibat daylight saving time atau perbedaan zona lokal
- [ ] Unit testing menyeluruh terhadap kasus batas: aktivitas pertama kali (streak 0 -> 1), aktivitas jeda 23 jam 59 menit (same calendar day vs consecutive), batas 47 jam 59 menit vs 48 jam 01 menit

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/progress/streak.go`
- `../../../../radovid-bot-api/internal/domain/progress/streak_test.go`
- `../../../../radovid-bot-api/internal/usecase/progress/record_daily_activity.go`
- `../../../../radovid-bot-api/internal/usecase/progress/record_daily_activity_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/sqlite/progress_repository.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
