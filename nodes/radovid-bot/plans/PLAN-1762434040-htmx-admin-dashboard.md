---
id: PLAN-1762434040
ticket_id: TICKET-1762434040
title: HTMX Admin Dashboard
status: Completed
created_at: 2026-11-04
---

# Rencana Implementasi Teknis: HTMX Admin Dashboard

## 1. Referensi Dokumen & Kaitan Tugas
- **Tiket Terkait:** [TICKET-1762434040](../tickets/TICKET-1762434040-htmx-admin-dashboard.md)
- **Dokumen Arsitektur:** [System Design](../docs/system-design.md)
- **Pedoman Terkait:** [Coding Standards](../../../global-guidelines/coding.md) | [Safe File Operations](../../../global-guidelines/safe-file-operations.md)

---

## 2. Ringkasan Eksekutif & Analisis Arsitektur
- **Tujuan Teknis:** Membangun server web internal menggunakan `net/http` Go standar dengan template server-side dan library ringan HTMX, diproteksi oleh middleware HTTP Basic Authentication, untuk observabilitas sesi Discord, data pemain, dan kesehatan infrastruktur bot.
- **Hasil Pemetaan Ketergantungan (Graphify Query):**
  - Modul / Fungsi Pemanggil: Goroutine listener di `cmd/bot/main.go` yang dijalankan paralel bersama koneksi Discord gateway.
  - Komponen Terdampak: `internal/delivery/http`, `internal/usecase/admin`, template files di `web/templates`, dan static assets di `web/static`.
  - Potensi Efek Samping: Server HTTP harus menangani graceful shutdown bersamaan dengan bot Discord saat menerima sinyal SIGINT/SIGTERM tanpa kebocoran resource port (`TIME_WAIT`).

---

## 3. Ruang Lingkup Mutasi Berkas (Target Files)
| Tipe Aksi | Path Berkas (Relative) | Deskripsi Perubahan |
| :--- | :--- | :--- |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/middleware.go` | Middleware Basic Auth dengan constant-time comparison |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/middleware_test.go` | Unit test keamanan Basic Auth |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/admin_handler.go` | HTTP handler endpoints `/admin`, `/sessions`, `/users`, `/health` |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/admin_handler_test.go` | Unit test HTTP route responses dan status codes |
| `CREATE` | `../../../../radovid-bot-api/internal/delivery/http/router.go` | Setup http.ServeMux dan binding embed.FS templates |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/admin/dashboard_stats.go` | Use case kalkulasi ringkasan metrik dashboard |
| `CREATE` | `../../../../radovid-bot-api/internal/usecase/admin/dashboard_stats_test.go` | Unit test use case dashboard |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/admin_repository.go` | Repositori SQLite untuk statistik admin |
| `CREATE` | `../../../../radovid-bot-api/internal/infrastructure/database/admin_repository_test.go` | Unit test repositori admin SQLite |
| `CREATE` | `../../../../radovid-bot-api/web/embed.go` | Embedding aset templates dan static ke binary Go |
| `CREATE` | `../../../../radovid-bot-api/web/templates/base.html` | Layout dasar HTML5 dengan CDN/lokal HTMX import |
| `CREATE` | `../../../../radovid-bot-api/web/templates/dashboard.html` | View panel ringkasan metrik bot |
| `CREATE` | `../../../../radovid-bot-api/web/templates/sessions.html` | View tabel sesi dengan atribut `hx-get` filter |
| `CREATE` | `../../../../radovid-bot-api/web/templates/users.html` | View tabel pengguna dan level progress |
| `CREATE` | `../../../../radovid-bot-api/web/templates/health.html` | View partial status kesehatan koneksi bot & Ollama |
| `CREATE` | `../../../../radovid-bot-api/web/static/style.css` | Stylesheet CSS minimalis bertema gelap (dark mode) |
| `CREATE` | `../../../../radovid-bot-api/web/static/htmx.min.js` | Berkas pustaka JavaScript HTMX versi terkompresi |
| `MODIFY` | `../../../../radovid-bot-api/internal/infrastructure/ollama/client.go` | Penambahan method `CheckHealth` untuk pemeriksaan ketersediaan Ollama |
| `MODIFY` | `../../../../radovid-bot-api/cmd/bot/main.go` | Menyalakan HTTP server pada goroutine dengan konteks shutdown |

---

## 4. Langkah Eksekusi Sekuensial (Step-by-Step Procedure)
1. **Langkah 1 (Persiapan & Red Test):**
   - Buat file test `internal/delivery/http/middleware_test.go`.
   - Uji verifikasi kredensial: header kosong -> 401, password salah -> 401, kredensial tepat -> status 200 (next handler dipanggil).
   - Pastikan tes gagal (RED).
2. **Langkah 2 (Implementasi Middleware & Router):**
   - Terapkan `BasicAuthMiddleware(secret string, next http.Handler) http.Handler` dengan pemisahan string `user:password` menggunakan `crypto/subtle.ConstantTimeCompare` guna mencegah serangan *timing attack*.
   - Siapkan embedding aset menggunakan `//go:embed web/templates/* web/static/*` di `router.go`.
3. **Langkah 3 (HTMX Views & Partial Endpoints):**
   - Susun template `base.html` dan partial views (`dashboard.html`, `sessions.html`, `users.html`, `health.html`).
   - Buat handler di `admin_handler.go` yang mendeteksi header `HX-Request`: jika bernilai `true`, render hanya potongan template partial yang relevan; jika request browser biasa, render layout `base.html` lengkap.
   - Buat use case `GetDashboardStats` yang membaca agregat data dari repositori.
4. **Langkah 4 (Integrasi & Delivery Gate):**
   - Hubungkan HTTP server ke `main.go` menggunakan `http.Server{Addr: fmt.Sprintf(":%s", port)}`.
   - Tambahkan registrasi cleanup graceful shutdown menggunakan `server.Shutdown(ctx)`.
   - Verifikasi tidak ada file static eksternal yang diunduh saat runtime (100% offline-first / embedded).

---

## 5. Pratinjau Kode / Diff Preview (Focus Diff)
```go
// internal/delivery/http/middleware.go
package http

import (
	"crypto/subtle"
	"net/http"
	"strings"
)

func BasicAuthMiddleware(expectedSecret string, next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		user, pass, ok := r.BasicAuth()
		if !ok {
			w.Header().Set("WWW-Authenticate", `Basic realm="Radovid Admin"`)
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		parts := strings.SplitN(expectedSecret, ":", 2)
		if len(parts) != 2 {
			http.Error(w, "Server configuration error", http.StatusInternalServerError)
			return
		}

		userMatch := subtle.ConstantTimeCompare([]byte(user), []byte(parts[0])) == 1
		passMatch := subtle.ConstantTimeCompare([]byte(pass), []byte(parts[1])) == 1

		if !userMatch || !passMatch {
			w.Header().Set("WWW-Authenticate", `Basic realm="Radovid Admin"`)
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		next(w, r)
	}
}
```

---

## 6. Pertimbangan Kasus Batas (Edge Cases & Safety)
- **Penanganan Input Ekstrem:** Parameter rahasia `ADMIN_DASHBOARD_SECRET` yang tidak memiliki pemisah titik dua (`:`) ditolak saat startup via validasi konfigurasi *fail-fast*.
- **Strategi Rollback:** Apabila server HTTP gagal melakukan binding port (misal: port sudah terpakai), catat log fatal dan hentikan proses secara aman agar tidak meninggalkan bot dalam status setengah berjalan (*half-initialized*).
