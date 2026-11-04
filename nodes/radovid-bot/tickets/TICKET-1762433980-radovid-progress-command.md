---
id: TICKET-1762433980
plan_id: PLAN-1762433980
title: /radovid progress Command
status: Todo
priority: High
labels: [Frontend, Discord, SlashCommand, Gamification]
---

# Deskripsi
Sebagai Pengguna Discord,
Saya ingin menjalankan perintah slash `/radovid progress` untuk melihat ringkasan visual statistik belajar bahasa Inggris saya,
Sehingga saya dapat memantau level saat ini, progres XP menuju level berikutnya melalui progress bar grafis Unicode, jumlah streak aktif, dan daftar pencapaian yang telah saya raih.

Perintah `/radovid progress` adalah antarmuka visual utama bagi pengguna untuk menginspeksi kemajuan belajar mereka. Tampilan harus disajikan dalam format Discord Embed yang bersih, elegan, dan informatif, memuat indikator level, progress bar Unicode (misal: `[████████░░░░░░] 56%`), streak hari berturut-turut, total sesi game yang diselesaikan, dan deretan lencana penghargaan yang berhasil dibuka.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762433980-radovid-progress-command.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Perintah slash `/radovid progress` terdaftar di router Discord command framework
- [ ] Embed Discord menyajikan visualisasi statistik: Level, Total XP, XP saat ini vs XP menuju level berikutnya, Current Streak (dengan emoji 🔥), dan Total Games Completed
- [ ] Tersedia utilitas generator Unicode Progress Bar murni Go yang fleksibel (panjang bar dapat dikonfigurasi, menghitung persentase akurat 0% - 100%)
- [ ] Menampilkan daftar lencana pencapaian (Achievements) yang telah dibuka berupa deretan ikon emoji dan nama lencana, atau placeholder edukatif jika belum ada
- [ ] Penanganan pengguna baru (user yang belum pernah bermain) secara otomatis membuat profil inisial (Level 1, 0 XP, 0 Streak) tanpa menampilkan error
- [ ] Respons dikirimkan dalam format ephemeral atau channel embed sesuai preferensi konteks pemanggilan
- [ ] Unit testing pada generator progress bar (uji nilai 0%, 50%, 100%, pembagian dengan nol, dan nilai batas overflow)

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_progress.go`
- `../../../../radovid-bot-api/internal/delivery/discord/handler.go`
- `../../../../radovid-bot-api/internal/delivery/discord/progress_bar.go`
- `../../../../radovid-bot-api/internal/delivery/discord/progress_bar_test.go`
- `../../../../radovid-bot-api/internal/usecase/progress/get_progress.go`
- `../../../../radovid-bot-api/internal/usecase/progress/get_progress_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution
