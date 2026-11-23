---
id: PLAN-1795426945
ticket_id: TICKET-1795426945
title: Transform 2-Player Locked Games (WYR, Taboo, 20Q) into Channel-Wide Party Games
status: Completed
created_at: 2026-11-23
---

# Rencana Implementasi Teknis: Transform 2-Player Locked Games into Channel-Wide Party Games

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1795426945](../tickets/TICKET-1795426945-party-mode-multiplayer-redesign.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:**
  1. Mengeliminasi *bottleneck* lobi pending (`StatusPending`) pada 3 game multiplayer (`would-you-rather`, `taboo`, dan `20-questions`), sehingga sesi langsung berstatus aktif (`StatusActive`) segera setelah host menjalankan perintah slash.
  2. Mengubah mekanisme Would You Rather menjadi **Open Dilemma Topic**:
     - Bot tidak lagi menampilkan tombol voting `Option A` dan `Option B`.
     - Bot menyajikan embed dilema (Option A vs Option B) sebagai teks terbuka untuk bahan diskusi seluruh anggota channel/voice.
     - Bot menyajikan tombol navigasi `Next: Round X` dan `Finish & View Summary` yang **hanya dapat dikendalikan oleh Host pembuat sesi**.
  3. Mengubah Taboo menjadi **Party Describer Game**:
     - Sesi langsung aktif; Host bertindak sebagai describer yang melihat kata target & kata terlarang via pesan rahasia (ephemeral).
     - Seluruh anggota channel dapat menebak secara bebas, dan Host mengontrol tombol `Next Word` / `Finish Game`.
  4. Mengubah 20 Questions menjadi **Cooperative Channel Deduction**:
     - Sesi langsung aktif dengan AI Game Master memilih entitas misterius.
     - Menghapus validasi kaku `isParticipant` pada `/radovid ask` dan `/radovid guess` sehingga **seluruh anggota server di channel tersebut** dapat berpartisipasi mengajukan pertanyaan dan menebak jawaban.
- **Hasil Pemetaan Ketergantungan (Dependency Analysis):**
  - **Caller Modul:**
    - `command_play_wyr.go`: Mengubah alur dari `CreateSession` (pending) $\to$ langsung `CreateSession` + `StartSession` + `StartRound` via deferral.
    - `command_play_taboo.go`: Mengubah alur menjadi instan start dengan kartu rahasia untuk Host.
    - `command_play_20q.go`, `command_ask_20q.go`, `command_guess_20q.go`: Menghapus pemeriksaan partisipan lobi khusus, mengizinkan interaksi dari siapa saja di channel aktif.
  - **Komponen Terdampak:**
    - `interaction_wyr_button.go`: Menghapus kebutuhan voting selesai (`finished` via AddVote); tombol navigasi diikat validasi otorisasi Host (`userID == hostID`).
    - `would_you_rather.go`: Menambahkan kemampuan menandai ronde selesai / merekam topik tanpa mengharuskan input 2 vote manual.
  - **Potensi Efek Samping:**
    - Sesi aktif ganda di channel yang sama tetap dicegah oleh pengecekan sesi aktif per `channelID` (tetap menyediakan tombol *Cancel Stuck Game* jika sesi macet).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Instan start sesi WYR tanpa lobi pending, pembuatan embed topik tanpa tombol A/B, penyajian tombol navigasi Host |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go` | Validasi wewenang Host untuk aksi `next` dan `finish`, penyajian ronde baru tanpa tombol A/B |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go` | Unit test alur instan start WYR dan penyajian embed topik diskusi |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go` | Unit test otorisasi Host pada tombol Next/Finish WYR dan penolakan non-host |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo.go` | Instan start sesi Taboo tanpa lobi pending, kartu rahasia Host dan kontrol navigasi |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_taboo_test.go` | Unit test alur instan start Taboo |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q.go` | Instan start sesi 20 Questions tanpa lobi pending |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_ask_20q.go` | Mengizinkan pertanyaan dari siapa saja di channel aktif |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_guess_20q.go` | Mengizinkan tebakan dari siapa saja di channel aktif |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_20q_test.go` | Unit test alur instan start 20 Questions dan partisipasi terbuka channel |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go` | Pembaruan teks bantuan deskripsi game menjadi party/channel mode |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go` | Penyesuaian usecase summary dan pencatatan ronde tanpa voting wajib |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go` | Unit test usecase untuk penyelesaian ronde mode party |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)

### Fase 1: Would You Rather (Discussion Party Mode)
1. **RED Test:** Perbarui unit test pada `command_play_wyr_test.go` dan `interaction_wyr_button_test.go`:
   - Host menjalankan command langsung mendapatkan soal Ronde 1 dengan tombol navigasi Host (tanpa tombol voting A/B).
   - Pemain non-host yang mengklik tombol Next/Finish mendapatkan penolakan ephemeral.
2. **GREEN Implementation:**
   - Ubah `HandlePlayCommand` pada `command_play_wyr.go`:
     - Panggil `CreateSession` $\to$ `StartSession` $\to$ `StartRound` dengan deferral interaksi.
     - Kirim embed soal topik dengan tombol `BuildWYRHostControlButtons(sessionID, nextRound, targetRounds)`.
   - Perbarui `interaction_wyr_button.go`:
     - Pada `case "next"` dan `case "finish"`: verifikasi bahwa `userID == hostID` (cek dari sesi).
     - Sajikan embed ronde berikutnya tanpa tombol A/B.
   - Perbarui `would_you_rather.go` untuk menandai ronde selesai secara otomatis saat host menekan next/finish.

### Fase 2: 20 Questions (Cooperative Channel Mode)
1. **RED Test:** Perbarui `command_play_20q_test.go` untuk menguji bahwa sesi langsung dimulai dan pemain non-host dapat memanggil `/radovid ask` dan `/radovid guess`.
2. **GREEN Implementation:**
   - Ubah `command_play_20q.go`: instan start tanpa lobi pending.
   - Ubah `command_ask_20q.go` & `command_guess_20q.go`: hapus pengecekan keanggotaan partisipan kaku; asalkan ada sesi aktif di channel, siapa pun dapat bertanya/menebak.

### Fase 3: Taboo (Party Describer Mode)
1. **RED Test:** Perbarui `command_play_taboo_test.go` untuk memverifikasi instan start.
2. **GREEN Implementation:**
   - Ubah `command_play_taboo.go`: instan start tanpa lobi pending. Host mendapatkan kata rahasia via kartu ephemeral.

### Fase 4: Refactor, Help Text, & Delivery Gate
1. Perbarui teks panduan di `radovid_help.go`.
2. Jalankan `go test -race ./...` di `radovid-bot-api`.
3. Verifikasi kepatuhan Zero-Comment Policy di seluruh berkas yang disentuh.
4. Pastikan `go vet ./...` dan `go build ./cmd/bot` bebas galat/peringatan.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

### Would You Rather: Embed Tanpa Tombol Voting A/B
```go
// Helper komponen kontrol Host WYR
func BuildWYRHostControlButtons(sessionID string, currentRound, targetRounds int) []discordgo.MessageComponent {
	var buttons []discordgo.MessageComponent
	if currentRound < targetRounds {
		buttons = append(buttons, discordgo.Button{
			Label:    fmt.Sprintf("Next: Round %d", currentRound+1),
			Style:    discordgo.PrimaryButton,
			CustomID: fmt.Sprintf("wyr:next:%s:%d", sessionID, currentRound+1),
		})
	}
	buttons = append(buttons, discordgo.Button{
		Label:    "Finish & View Summary",
		Style:    discordgo.SecondaryButton,
		CustomID: fmt.Sprintf("wyr:finish:%s", sessionID),
	})
	return []discordgo.MessageComponent{discordgo.ActionsRow{Components: buttons}}
}
```

### Otorisasi Host pada Handler Tombol
```go
	case "next", "finish":
		sess, err := h.sessionManager.GetSession(ctx, sessionID)
		if err != nil || sess == nil {
			return RespondEphemeralError(s, i, "Game session not found.")
		}
		if sess.HostUserID != userID { // atau pengecekan Host role
			return RespondEphemeralError(s, i, fmt.Sprintf("Only the game host (<@%s>) can control this game.", sess.GetHostID()))
		}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
1. **Non-Host Menekan Tombol:** Pesan ephemeral langsung memberitahu bahwa hanya Host yang dapat melanjutkan ronde atau menyelesaikan permainan.
2. **Sesi Macet:** Tombol `Cancel Stuck Game & Reset` tetap disediakan jika ada perintah baru yang dipanggil di channel saat sesi lama belum selesai.
3. **Penyelesaian Lebih Awal (Early Finish):** Host dapat menekan `Finish & View Summary` kapan saja sejak Ronde 1.
4. **Isolasi Konkurensi Channel:** Menggunakan RWMutex di layer usecase untuk mencegah bentrok data antar-channel Discord.
