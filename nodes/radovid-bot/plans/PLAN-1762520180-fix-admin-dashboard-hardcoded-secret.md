---
id: PLAN-1762520180
ticket_id: TICKET-1762520180
title: "Remove Hardcoded Admin Secret Fallback & Enforce Config Validation"
status: Completed
created_at: 2026-11-08
---

# Rencana Implementasi Teknis: Remove Hardcoded Admin Secret Fallback

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762520180](../tickets/TICKET-1762520180-fix-admin-dashboard-hardcoded-secret.md)
- **Laporan Audit:** [hunt-001-2026-11-08.md](../hunt/hunt-001-2026-11-08.md) (Temuan #4)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Security Guidelines](../../../global-guidelines/security.md) | [Coding Standards](../../../global-guidelines/coding.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Menghapus kredensial fallback `"admin:admin"` pada `cmd/bot/main.go` dan menambahkan validasi *fail-fast* pada `config/config.go` untuk memastikan `ADMIN_DASHBOARD_SECRET` (ketika disetel) wajib mengikuti format `username:password` yang sah, serta menolak akses admin dashboard jika secret tidak dikonfigurasi.
- **Hasil Pemetaan Ketergantungan:**
  - Modul / Fungsi Pemanggil: `main()` di `cmd/bot/main.go` dan `config.LoadConfig()` di `config/config.go`.
  - Komponen Terdampak: `cmd/bot/main.go`, `config/config.go`, `config/config_test.go`.
  - Potensi Efek Samping: Pengujian yang sudah ada di `config_test.go` perlu diperiksa agar tidak rusak jika `ADMIN_DASHBOARD_SECRET` bersifat opsional saat startup (di mana `BasicAuthMiddleware` sudah menolak request dengan 500 jika secret kosong) namun fail-fast jika diisi dengan format yang salah (tanpa `:`).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Menghapus blok `if adminSecret == "" { adminSecret = "admin:admin" }` |
| `MODIFY` | `../../../../radovid-bot-api/config/config.go` | Menambahkan validasi fail-fast untuk format `ADMIN_DASHBOARD_SECRET` (`user:pass`) dan nilai numerik env var |
| `MODIFY` | `../../../../radovid-bot-api/config/config_test.go` | Menambahkan unit test penolakan `ADMIN_DASHBOARD_SECRET` dengan format tidak valid |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test — O-07):**
   - Tambahkan unit test di `config/config_test.go` yang menguji `ADMIN_DASHBOARD_SECRET="invalid_without_colon"` atau `":emptyuser"` dan mengharapkan error fail-fast dari `LoadConfig()`.
   - Jalankan `go test ./config/...` dan buktikan tes **GAGAL (RED)** pada kode lama.
2. **Langkah 2 (Implementasi Inti):**
   - Di `config/config.go`, validasi bahwa jika `ADMIN_DASHBOARD_SECRET` diisi, nilainya wajib mengandung `:` dengan username dan password tidak kosong.
   - Di `cmd/bot/main.go`, hapus fallback `adminSecret = "admin:admin"` dan teruskan `cfg.AdminDashboardSecret` secara langsung ke `httpDelivery.NewRouter`.
3. **Langkah 3 (Refactor & Integrasi):**
   - Jalankan `go test ./config/... ./internal/delivery/http/...` dan pastikan seluruh tes lulus.

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
adminSecret := strings.TrimSpace(getEnv("ADMIN_DASHBOARD_SECRET"))
if adminSecret != "" {
	parts := strings.SplitN(adminSecret, ":", 2)
	if len(parts) != 2 || strings.TrimSpace(parts[0]) == "" || strings.TrimSpace(parts[1]) == "" {
		return nil, errors.New("fail-fast: invalid ADMIN_DASHBOARD_SECRET format, expected username:password")
	}
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Password mengandung karakter `:` tambahan (misal `admin:p@ss:word`) tetap didukung dengan aman karena menggunakan `strings.SplitN(..., ":", 2)`.
- **Strategi Rollback:** Revert perubahan pada `config.go` dan `main.go`.

