---
id: TICKET-1762347520
plan_id: PLAN-1762347520
title: Game 5: Roleplay
status: Todo
priority: High
labels: [Game, Speaking, Roleplay, Backend, Discord]
---

# Deskripsi
Sebagai Player Radovid Bot,
Saya ingin melakukan simulasi percakapan interaktif berbasis giliran (turn-based) dalam berbagai skenario kehidupan sehari-hari (seperti di restoran, bandara, hotel, atau klinik dokter),
Sehingga saya dapat melatih kelancaran percakapan spontan dalam bahasa Inggris sementara AI memberikan koreksi tata bahasa secara halus tanpa merusak imersi situasi.

Game 5: Roleplay menyajikan skenario situasional realistis di mana AI mengambil persona karakter lawan bicara (misalnya pelayan restoran, petugas imigrasi, atau dokter). Pengguna berdialog bolak-balik hingga maksimal 10 giliran atau hingga objektif skenario tercapai. Sistem dirancang untuk memberikan koreksi gramatikal minor secara non-intrusif pada elemen visual pendukung, kemudian menyajikan evaluasi bahasa menyeluruh di akhir sesi.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1762347520-game-roleplay.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [ ] Tersedia katalog skenario sehari-hari (Restaurant, Airport Check-in, Hotel Reservation, Doctor's Appointment, Shopping Inquiry)
- [ ] AI Game Master menginisialisasi skenario pembuka dengan konteks yang jelas, tujuan misi percakapan, serta persona karakter yang konsisten
- [ ] Dialog berlangsung secara turn-based (pengguna memasukkan kalimat ucapan via tombol balasan / modal Discord)
- [ ] Koreksi tata bahasa dan frasa disajikan secara halus (subtle correction) pada footer embed atau side-note tanpa memutus dialog persona AI
- [ ] Sesi berakhir secara otomatis saat tujuan skenario terselesaikan atau telah mencapai batas maksimum 10 putaran dialog
- [ ] Embed penutup sesi menyajikan ringkasan performa dialog, daftar koreksi grammar yang terkumpul, dan skor naturalness percakapan

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/domain/game/roleplay.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay.go`
- `../../../../radovid-bot-api/internal/usecase/game/roleplay_test.go`
- `../../../../radovid-bot-api/internal/infrastructure/ollama/prompts/roleplay.tmpl`
- `../../../../radovid-bot-api/internal/delivery/discord/game_roleplay.go`
- `../../../../radovid-bot-api/internal/delivery/discord/embeds.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*
- **Status:** Pending Execution

