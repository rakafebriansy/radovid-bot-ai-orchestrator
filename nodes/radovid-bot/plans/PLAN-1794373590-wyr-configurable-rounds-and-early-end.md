---
id: PLAN-1794373590
ticket_id: TICKET-1794373590
title: Configurable Round Count and Early-End Support for Would You Rather Game
status: Completed
created_at: 2026-11-11
---

# Rencana Implementasi Teknis: Configurable Round Count and Early-End Support for Would You Rather Game

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1794373590](../tickets/TICKET-1794373590-wyr-configurable-rounds-and-early-end.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:**
  1. Memberikan kapabilitas bagi pengguna untuk memilih target jumlah ronde/soal permainan Would You Rather saat memanggil perintah `/radovid play would-you-rather rounds:[N]` (rentang 1-10, default 3).
  2. Menyimpan konfigurasi jumlah putaran target per sesi permainan di dalam state usecase (`WouldYouRatherUseCase`) dengan konkurensi aman (`sync.RWMutex`).
  3. Memperbarui antarmuka Discord Embed agar secara eksplisit menampilkan format progres `Round X/N` (contoh `Round 1/5: Would You Rather?`).
  4. Mengizinkan pemain untuk mengakhiri permainan lebih awal (*early end*) sejak ronde ke-1 selesai dengan selalu menyajikan tombol `Finish & View Summary` berdampingan dengan `Next: Round X`.
  5. Menghilangkan tombol `Next: Round X` ketika permainan telah mencapai target ronde yang dipilih (`roundNumber == targetRounds`), sehingga pemain diarahkan langsung untuk melihat rangkuman akhir dan klaim XP.
- **Hasil Pemetaan Ketergantungan (Dependency & Architecture Analysis):**
  - **Caller Modul:**
    - Perintah slash `/radovid play would-you-rather` diuraikan oleh `HandlePlayCommand` (`command_play_wyr.go`). Opsi baru `rounds` diekstraksi dari argumen interaksi.
    - Lobi sesi dibuat di `sessionManager.CreateSession`.
    - Host menetapkan `targetRounds` pada `wyrUseCase.SetSessionConfig(sessionID, config)`.
  - **Game Loop & Komponen Interaksi:**
    - Ketika Player 2 bergabung, `StartRound` dipanggil untuk Ronde 1. Embed soal menyertakan format judul `Round 1/N` di mana `N = targetRounds`.
    - Setelah kedua pemain memberikan suara di tombol `wyr:vote:...`, handler tombol `HandleButtonInteraction` (`interaction_wyr_button.go`) memeriksa apakah ronde saat ini sudah mencapai `targetRounds`:
      - Jika `roundNumber < targetRounds`: sajikan `BuildWYRFinishOrNextButtons(sessionID, nextRoundNumber)` sehingga pemain bebas memilih lanjut atau berhenti sekarang (*early end*).
      - Jika `roundNumber >= targetRounds`: HANYA sajikan tombol `Finish & View Summary` (`BuildWYRFinishOnlyButton(sessionID)`).
    - Ketika tombol `Finish & View Summary` ditekan:
      - `GetSessionSummary(ctx, sessionID)` memvalidasi kelayakan finish. Batas awal (`MinRoundsBeforeFinish = 3`) disesuaikan menjadi `totalRounds >= 1`.
      - Rangkuman sesi dibuat dan dicatat ke dalam XP progres pemain.
  - **Potensi Efek Samping:**
    - Jika sesi dibatalkan sebelum ronde 1 selesai, tidak ada XP yang diberikan (sesuai perilaku standar).
    - Nilai input `rounds` dari pengguna luar batas (misal <= 0 atau > 20) harus difilter / di-clamp secara fail-fast dengan nilai default 3.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/wyr_entity.go` | Penambahan konstanta `DefaultWYRRounds = 3`, `MinAllowedRounds = 1`, `MaxAllowedRounds = 10`, struct konfigurasi `WYRSessionConfig` |
| `MODIFY` | `../../../../radovid-bot-api/internal/domain/game/wyr_entity_test.go` | Unit test validasi rentang ronde dan batas entitas |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather.go` | Manajemen state target rounds (`sessionsMu sync.RWMutex`, `map[string]WYRSessionConfig`), method `SetTargetRounds`, `GetTargetRounds`, serta relaksasi `canFinish := totalRounds >= 1` di `GetSessionSummary` |
| `MODIFY` | `../../../../radovid-bot-api/internal/usecase/game/would_you_rather_test.go` | Unit test target rounds setting, retrieval, dan summary dengan 1 ronde |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_def.go` | Pendaftaran opsi integer `rounds` dengan pilihan (*choices*) atau batas 1-10 pada subcommand `would-you-rather` |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr.go` | Parsing opsi `rounds`, penetapan target rounds ke usecase, pembaharuan embed lobi dan embed pertanyaan (`Round X/N`) |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/command_play_wyr_test.go` | Unit test parsing opsi `rounds` dan formatting embed |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button.go` | Logika penyajian tombol pasca-voting: Finish-Only jika ronde terakhir, Finish & Next jika masih ada ronde berikutnya |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/interaction_wyr_button_test.go` | Unit test verifikasi tampilan tombol finish-only di ronde target dan finish-or-next di ronde awal |
| `MODIFY` | `../../../../radovid-bot-api/internal/delivery/discord/radovid_help.go` | Pembaruan teks panduan `/radovid help` terkait fleksibilitas jumlah ronde |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)

### Langkah 1: Persiapan & Red Test (TDD RED Phase)
1. Tulis unit test di `internal/usecase/game/would_you_rather_test.go` yang menguji `SetTargetRounds`, `GetTargetRounds`, dan `GetSessionSummary` dengan 1 ronde (memverifikasi bahwa `canFinish` bernilai true pada ronde 1).
2. Tulis unit test di `internal/delivery/discord/command_play_wyr_test.go` yang menguji pembacaan opsi `rounds` dan pembentukan embed `Round 1/5`.
3. Tulis unit test di `internal/delivery/discord/interaction_wyr_button_test.go` yang memvalidasi bahwa tombol di ronde 1 menyertakan opsi finish, dan ronde terakhir menyajikan finish-only.
4. Jalankan tes dan pastikan gagal (RED).

### Langkah 2: Implementasi Domain & Usecase Layer (GREEN Phase - Bagian 1)
1. Perbarui `internal/domain/game/wyr_entity.go`:
   - Definisikan `DefaultWYRRounds = 3`, `MinAllowedRounds = 1`, `MaxAllowedRounds = 10`.
   - Definisikan struct `WYRSessionConfig{ TargetRounds int }`.
2. Perbarui `internal/usecase/game/would_you_rather.go`:
   - Tambahkan `configsMu sync.RWMutex` dan `configs map[string]WYRSessionConfig` pada `WouldYouRatherUseCase`.
   - Tambahkan method `SetTargetRounds(sessionID string, targetRounds int)` dengan sanitasi (jika targetRounds < 1 atau > 10, gunakan DefaultWYRRounds).
   - Tambahkan method `GetTargetRounds(sessionID string) int` (mengembalikan default 3 jika belum diset).
   - Ubah `GetSessionSummary`: `canFinish := totalRounds >= 1`.

### Langkah 3: Implementasi Delivery Layer (GREEN Phase - Bagian 2)
1. Perbarui `internal/delivery/discord/command_def.go`:
   - Tambahkan opsi integer `rounds` pada subcommand `would-you-rather` dengan deskripsi dan pilihan/rentang yang jelas.
2. Perbarui `internal/delivery/discord/command_play_wyr.go`:
   - Baca `rounds` dari opsi command.
   - Tambahkan fungsi helper tombol `BuildWYRFinishOnlyButton(sessionID string) []discordgo.MessageComponent`.
   - Perbarui `BuildWYRQuestionEmbed(roundNumber, targetRounds int, question, optionA, optionB string)` agar menampilkan `Round X/N: Would You Rather?` jika `targetRounds > 0`.
   - Simpan `targetRounds` pada pembuatan sesi atau saat Player 1 membuat lobi.
3. Perbarui `internal/delivery/discord/interaction_wyr_button.go`:
   - Pada penanganan vote selesai (`finished`):
     - Ambil `targetRounds := h.wyrUseCase.GetTargetRounds(sessionID)`.
     - Jika `round.RoundNumber >= targetRounds`: gunakan `BuildWYRFinishOnlyButton(sessionID)`.
     - Jika `round.RoundNumber < targetRounds`: gunakan `BuildWYRFinishOrNextButtons(sessionID, nextRound)` (tersedia tombol next DAN finish).
   - Pastikan embed pertanyaan di ronde berikutnya juga menerima parameter `targetRounds`.
4. Perbarui `internal/delivery/discord/radovid_help.go` agar teks bantuan mencerminkan fleksibilitas jumlah ronde.

### Langkah 4: Pengujian Menyeluruh & Refactoring (REFACTOR & DELIVERY GATE)
1. Jalankan seluruh pengujian: `go test -race ./...`.
2. Verifikasi kepatuhan Zero-Comment Policy (`coding.md` Seksi 1.1) di seluruh berkas Go yang diubah.
3. Jalankan `go vet ./...` dan `go build ./cmd/bot`.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)

### Perubahan 1: Definisi Opsi Command pada `command_def.go`
```go
						{
							Type:        discordgo.ApplicationCommandOptionSubCommand,
							Name:        "would-you-rather",
							Description: "Play the Would You Rather dilemma game",
							Options: []*discordgo.ApplicationCommandOption{
								{
									Type:        discordgo.ApplicationCommandOptionString,
									Name:        "difficulty",
									Description: "Game difficulty level (easy, medium, hard)",
									Required:    false,
									Choices: []*discordgo.ApplicationCommandOptionChoice{
										{Name: "Easy", Value: "easy"},
										{Name: "Medium", Value: "medium"},
										{Name: "Hard", Value: "hard"},
									},
								},
+								{
+									Type:        discordgo.ApplicationCommandOptionInteger,
+									Name:        "rounds",
+									Description: "Number of dilemma rounds to play (1-10, default: 3)",
+									Required:    false,
+									Choices: []*discordgo.ApplicationCommandOptionChoice{
+										{Name: "1 Round (Quick)", Value: 1},
+										{Name: "3 Rounds (Standard)", Value: 3},
+										{Name: "5 Rounds (Extended)", Value: 5},
+										{Name: "10 Rounds (Marathon)", Value: 10},
+									},
+								},
							},
						},
```

### Perubahan 2: Pengelolaan Target Rounds pada `would_you_rather.go`
```go
+func (uc *WouldYouRatherUseCase) SetTargetRounds(sessionID string, targetRounds int) {
+	if strings.TrimSpace(sessionID) == "" {
+		return
+	}
+	if targetRounds < game.MinAllowedRounds || targetRounds > game.MaxAllowedRounds {
+		targetRounds = game.DefaultWYRRounds
+	}
+	uc.configsMu.Lock()
+	defer uc.configsMu.Unlock()
+	uc.configs[sessionID] = game.WYRSessionConfig{TargetRounds: targetRounds}
+}

+func (uc *WouldYouRatherUseCase) GetTargetRounds(sessionID string) int {
+	uc.configsMu.RLock()
+	defer uc.configsMu.RUnlock()
+	if cfg, ok := uc.configs[sessionID]; ok && cfg.TargetRounds > 0 {
+		return cfg.TargetRounds
+	}
+	return game.DefaultWYRRounds
+}
```

### Perubahan 3: Penyajian Tombol dan Early Finish pada `interaction_wyr_button.go`
```go
		if finished {
			var components []discordgo.MessageComponent
			nextRound := round.RoundNumber + 1
-			if round.RoundNumber < game.MinRoundsBeforeFinish {
-				components = BuildWYRNextRoundButton(sessionID, nextRound)
-			} else {
-				components = BuildWYRFinishOrNextButtons(sessionID, nextRound)
-			}
+			targetRounds := h.wyrUseCase.GetTargetRounds(sessionID)
+			if round.RoundNumber >= targetRounds {
+				components = BuildWYRFinishOnlyButton(sessionID)
+			} else {
+				components = BuildWYRFinishOrNextButtons(sessionID, nextRound)
+			}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
1. **Input Ronde di Luar Batas:**
   - Jika pengguna memanipulasi Discord API dengan mengirimkan nilai `rounds <= 0` atau `rounds > 100`, sanitasi di usecase akan secara otomatis menormalkannya ke nilai aman `DefaultWYRRounds = 3`.
2. **Penghentian di Ronde 1 (Early End Ekstrem):**
   - Rangkuman sesi (`GetSessionSummary`) mampu memproses 1 ronde secara elegan tanpa pembagian dengan nol atau array out-of-bounds.
3. **Penyelesaian Sesi Berulang Kali:**
   - Tombol `wyr:finish:...` hanya memicu transisi status dari `active` ke `completed` sekali. Jika tombol diklik berulang, operasi idempoten dan tidak menimbulkan duplikasi XP.
4. **Isolasi Konkurensi Lobi:**
   - Penggunaan `sync.RWMutex` pada map konfigurasi sesi di `WouldYouRatherUseCase` mencegah *data race* saat banyak channel memainkan permainan secara bersamaan.

