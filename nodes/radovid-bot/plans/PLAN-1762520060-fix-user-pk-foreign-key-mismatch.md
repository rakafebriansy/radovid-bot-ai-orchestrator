---
id: PLAN-1762520060
ticket_id: TICKET-1762520060
title: "Fix User Primary Key & Foreign Key Mismatch in Session and User Repositories"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Fix User Primary Key & Foreign Key Mismatch

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520060](../tickets/TICKET-1762520060-fix-user-pk-foreign-key-mismatch.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #2)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Database Guidelines](../../../global-guidelines/database.md) | [Coding Standards](../../../global-guidelines/coding.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menyelaraskan pembuatan dan resolusi primary key `users.id` antara `UserRepository` dan `SessionRepository` agar tidak pernah memicu pelanggaran `FOREIGN KEY constraint failed` pada tabel `session_participants`, `user_progress`, `user_xp_log`, dan `achievements`, serta menjaga `username` asli pengguna tidak tertimpa oleh Discord ID numerik.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `SessionRepository.CreateSession`, `SessionRepository.AddParticipant`, `UserRepository.GetOrCreateByDiscordID`, `UserRepository.UpdateDifficulty`.
  - Komponen Terdampak: `internal/infrastructure/database/session_repository.go` dan `internal/infrastructure/database/user_repository.go`.
  - Potensi Efek Samping: `SessionParticipant.UserID` di domain game digunakan untuk mengecek giliran pemain berdasarkan Discord User ID (`extractUserID(i)`). Oleh sebab itu, menstandarkan `users.id = cleanDiscordID` (atau me-resolve ID & memperbarui username jika `username == discord_id`) memastikan `users.id == users.discord_id` secara konsisten di seluruh tabel relasi.

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/user_repository.go` | Menstandarkan ID user baru ke `cleanDiscordID` dan memperbarui username jika sebelumnya masih berisi ID numerik placeholder |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/session_repository.go` | Memastikan `ensureUserExists` me-resolve `users.id` yang ada jika `discord_id` sudah terdaftar |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/session_repository_test.go` | Menambahkan unit test pembuatan sesi setelah user dibuat melalui `UserRepository` |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/database/user_repository_test.go` | Menambahkan unit test kompatibilitas `GetOrCreateByDiscordID` dengan user yang dibuat oleh `SessionRepository` |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Tambahkan test `TestSessionRepository_CreateSession_AfterUserRepositoryCreation` di `session_repository_test.go` yang memanggil `userRepo.GetOrCreateByDiscordID(ctx, "discord-123", "Alice")` lalu `sessionRepo.CreateSession` untuk `"discord-123"`.
   - Jalankan `go test` dan buktikan **GAGAL (RED)** dengan error `FOREIGN KEY constraint failed`.
2. **Langkah 2 (Implementasi Inti):**
   - Pada `UserRepository.GetOrCreateByDiscordID`: jika user sudah ada namun `existing.Username == existing.DiscordID` dan `cleanUsername != cleanDiscordID`, perbarui `username` di database. Saat membuat user baru, gunakan `ID: cleanDiscordID` (atau pastikan `SessionRepository` dan `UserRepository` memakai ID yang identik).
   - Pada `SessionRepository`: pastikan jika baris dengan `discord_id = p.UserID` sudah ada dengan `id` berbeda (data legacy `usr-...`), query tetap aman atau diselaraskan.
3. **Langkah 3 (Refactor & Integrasi):**
   - Pastikan seluruh test di `user_repository_test.go` dan `session_repository_test.go` hijau.
4. **Langkah 4 (Hunt Loop & Delivery Gate):**
   - Uji skenario urutan terbalik: (A) Settings -> Play Game -> Progress, dan (B) Play Game -> Progress -> Admin Dashboard Users.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
if existing != nil {
	if existing.Username == existing.DiscordID && cleanUsername != cleanDiscordID {
		existing.Username = cleanUsername
		_ = r.Update(ctx, existing)
	}
	return existing, nil
}

newUser := &user.User{
	ID:         cleanDiscordID,
	DiscordID:  cleanDiscordID,
	Username:   cleanUsername,
	Difficulty: user.DifficultyBeginner,
	CreatedAt:  now,
	UpdatedAt:  now,
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Kompatibilitas mundur jika di database sudah terlanjur ada record dengan `id = "usr-" + discordID`.
- **Strategi Rollback:** Revert file `user_repository.go` dan `session_repository.go`.

