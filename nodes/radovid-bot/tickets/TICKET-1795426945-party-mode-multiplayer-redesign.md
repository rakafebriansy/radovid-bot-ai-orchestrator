---
id: TICKET-1795426945
plan_id: PLAN-1795426945
title: Transform 2-Player Locked Games (WYR, Taboo, 20Q) into Channel-Wide Party Games
status: Done
priority: High
labels: [Game, Multiplayer, Party-Mode, Discord, UX, Architecture, Refactor]
---

# Deskripsi
Sebagai Komunitas dan Pengguna Server Discord Radovid Bot,
Saya ingin game multiplayer (Would You Rather, Taboo, dan 20 Questions) dapat dimainkan bersama secara terbuka oleh seluruh anggota channel tanpa terkunci pada lobi kaku 2 pemain,
Sehingga permainan dapat langsung dinikmati sebagai aktivitas pesta / diskusi kelompok (*party game*) yang interaktif, di mana materi dapat dikonsumsi bersama dan dipandu secara luwes oleh host.

### Konteks & Latar Belakang Masalah
Sebelumnya, tiga permainan multiplayer utama (`would-you-rather`, `taboo`, dan `20-questions`) menggunakan pola lobi pending (`StatusPending`) yang mewajibkan tepat 2 orang pemain terdaftar:
1. Permainan tidak dapat dimulai sama sekali jika tidak ada pemain ke-2 yang bergabung secara formal ke lobi.
2. Pada Would You Rather, bot mengharuskan kedua pemain mengklik tombol voting Option A / Option B, padahal esensi permainan di channel Discord sering kali digunakan sebagai pemantik obrolan/debat terbuka antar-anggota voice/text channel.
3. Pada Taboo dan 20 Questions, anggota server lain di channel yang sama terblokir dari berpartisipasi karena pembatasan partisipan lobi yang kaku.

Tiket ini merombak sistem permainan multiplayer tersebut menjadi **Channel-Wide Party Game**:
- **Sesi Instan:** Host yang memanggil `/radovid play <game>` langsung mengaktifkan sesi tanpa menunggu pemain ke-2.
- **Konsumsi Publik:** Topik, kartu, atau teka-teki disajikan ke channel untuk dinikmati dan didiskusikan oleh semua orang.
- **Kendali Host:** Hanya Host yang memegang wewenang navigasi (tombol *Next Round* dan *Finish & View Summary*).
- **Would You Rather Khusus:** Menghilangkan tombol pilih A/B; bot menyajikan teks topik/dilema untuk bahan diskusi bebas, dan host menekan tombol *Next* atau *Finish* sesuai alur obrolan.
- **20 Questions & Taboo:** Siapa saja di channel dapat ikut bertanya (`/radovid ask`) dan menebak (`/radovid guess` atau chat) secara kooperatif.

## Rencana Implementasi Teknis (Implementation Plan)
- **Tautan Rencana Teknis:** [Lihat Dokumen Rencana](../plans/PLAN-1795426945-party-mode-multiplayer-redesign.md)

## Acceptance Criteria (Kriteria Penerimaan)
- [x] **Would You Rather (Party Discussion Mode):**
  - [x] Pemanggilan `/radovid play would-you-rather` langsung memulai sesi (`StatusActive`) tanpa lobi tunggu 2 pemain.
  - [x] Embed pertanyaan menyajikan teks topik dilema secara bersih tanpa komponen tombol voting Option A / Option B.
  - [x] Komponen navigasi hanya menyediakan tombol `Next: Round X` (jika belum mencapai target) dan `Finish & View Summary`.
  - [x] Hanya Host yang berwenang menekan tombol `Next` dan `Finish`; interaksi dari pengguna lain ditolak dengan pesan ephemeral informatif.
  - [x] Menekan `Finish & View Summary` mengakhiri sesi, menampilkan rangkuman topik yang telah dibahas, dan mencatat progres/XP untuk Host.
- [x] **Taboo (Party Guessing Mode):**
  - [x] `/radovid play taboo` langsung aktif seketika; Host bertindak sebagai pembicara/describer.
  - [x] Kata rahasia dan daftar kata terlarang hanya dapat dilihat oleh Host melalui kartu rahasia (pesan ephemeral).
  - [x] Host memegang kontrol tombol untuk membuka kata berikutnya (*Next Word / Skip*) dan mengakhiri permainan (*Finish Game*).
- [x] **20 Questions (Cooperative Channel Mode):**
  - [x] `/radovid play 20-questions` langsung aktif seketika; AI Game Master langsung memilih entitas rahasia.
  - [x] Seluruh anggota server di channel tersebut berhak mengajukan pertanyaan via `/radovid ask` dan menebak via `/radovid guess` tanpa ditolak validasi isolasi lobi 2 orang.
  - [x] Kuota pertanyaan bersama (20 pertanyaan) berkurang secara global untuk channel tersebut.
- [x] **Pembersihan & Kebersihan Kode:**
  - [x] Teks bantuan `/radovid help` diperbarui mencerminkan format Party/Channel Game.
  - [x] Seluruh unit test Go lulus 100% (`go test -race ./...`) dengan memenuhi prinsip TDD.
  - [x] Mematuhi 100% Zero-Comment Policy (`coding.md` Seksi 1.1) pada seluruh berkas Go yang dimodifikasi.

## Target Lingkup File (Affected Files)
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go`
- `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`
- `../../../../radovid-bot-api/internal/delivery/discord/interaction_taboo_modal_test.go`
- `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go`

---

## AI Execution Log & Output
*Peringatan untuk AI Agent: Bagian ini WAJIB diisi SAAT dan SETELAH mengeksekusi tiket.*

- **Langkah Teknis Tereksekusi:**
  1. Mengubah `command_play_wyr.go` menjadi instant-start party mode:
     - Mengeliminasi waiting lobby pending 2 pemain.
     - Menyajikan deskripsi embed pertanyaan untuk diskusi publik terbuka ("Discuss with everyone in the channel!").
     - Mengganti komponen tombol A/B dengan `BuildWYRHostControlButtons` (Next: Round X dan Finish & View Summary).
  2. Mengubah `interaction_wyr_button.go` untuk menegakkan otorisasi Host (`isWYRHost` / `isSessionHost`) pada aksi tombol `next` dan `finish`. Interaksi dari pemain non-host ditolak dengan pesan ephemeral informatif.
  3. Mengubah `command_play_20q.go` menjadi instant-start party mode, langsung menginisialisasi sesi dan state entitas misterius AI Game Master.
  4. Mengubah `command_play_taboo.go` menjadi instant-start party mode dengan Host sebagai describer utama.
  5. Menambahkan validasi `isSessionHost` pada aksi `next` dan `finish` di `interaction_taboo_modal.go`.
  6. Memperbarui deskripsi bantuan pada `radovid_help.go` menjadi format party/channel multiplayer.
  7. Menyesuaikan dan menambahkan unit test suite pada `command_play_wyr_test.go`, `interaction_wyr_button_test.go`, `command_play_20q_test.go`, `command_play_taboo_test.go`, dan `interaction_taboo_modal_test.go`.
  8. Memverifikasi 100% kepatuhan Zero-Comment Policy di seluruh berkas Go yang dimodifikasi.
- **Ringkasan File Terpengaruh:**
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_taboo.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_20q.go`
  - `radovid-bot-api/internal/delivery/discord/command_play_20q_test.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_taboo_modal.go`
  - `radovid-bot-api/internal/delivery/discord/interaction_taboo_modal_test.go`
  - `radovid-bot-api/internal/delivery/discord/radovid_help.go`
- **Catatan & Keputusan Arsitektural:**
  - Wewenang Host diverifikasi melalui `session.RoleHost` pada koleksi partisipan sesi aktif.
  - Kuota dan pencegahan konkurensi antar-sesi tetap diisolasi per `channelID`, dilengkapi opsi pembatalan sesi macet (`Cancel Stuck Game`).

### Bukti Eksekusi & Pengujian (Evidence - O-03)
- **Perintah Pengujian:** `go test -race ./... && go vet ./... && go build ./cmd/bot`
- **Output Terminal:**
  ```text
  ok  	github.com/rakafebriansy/radovid-bot/config	1.627s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/discord	2.040s
  ok  	github.com/rakafebriansy/radovid-bot/internal/delivery/http	3.286s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/achievement	2.735s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/diary	4.877s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/feedback	4.479s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/game	3.656s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/gamemaster	5.378s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/notification	2.357s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/progress	4.069s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/session	4.381s
  ok  	github.com/rakafebriansy/radovid-bot/internal/domain/user	4.145s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/database	4.920s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/discord	4.352s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama	4.631s
  ok  	github.com/rakafebriansy/radovid-bot/internal/infrastructure/ollama/prompts	4.483s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/achievement	4.471s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/admin	4.478s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/feedback	4.580s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/game	5.465s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/notification	4.583s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/progress	4.559s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/session	4.086s
  ok  	github.com/rakafebriansy/radovid-bot/internal/usecase/user	4.239s
  ```

### Verifikasi Delivery Gate (Wajib Sebelum Done)
- **Pilar 1: Higienitas Kode & Data (CG-1 s/d CG-5):** PASSED (100% Zero-Comment Policy terpenuhi di seluruh berkas Go)
- **Pilar 2: Hard Gate Testing (HG-1 s/d HG-4):** PASSED (Seluruh test suite -race lulus 100%)
- **Pilar 3: Must Probe Status (MP-1 s/d MP-7):** PASSED
  - O-07 (Fail on Old Code): Lulus (tes lama yang mengasumsikan lobi 2 pemain gagal pada implementasi instan party mode dan telah dimigrasikan ke skenario party mode)
  - O-08 (Edge Case Atlas): Lulus (penolakan non-host pada tombol next/finish, deteksi sesi aktif per channel, pembatalan sesi)
  - O-09 (Damage Assessment): Lulus (tidak ada regresi pada game lain)
  - O-10 (Full Suite Metrics): Lulus (24 package test lolos)
- **Pilar 4: Tata Kelola Dokumen & Git (GG-1 s/d GG-5):** PASSED (Tiket, plan, changelog terkelola sinkron)

### Daftar Belum Diuji (Untested List - O-16)
- Interaksi riil gateway Discord menggunakan bot token aktif di server produksi (diuji melalui unit test mock framework DiscordGo).
- Inferensi LLM riil pada Ollama live instance port 11434 (diuji via mock LLM service dan fallback content generation).
