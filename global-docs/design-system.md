# Design System — Radovid Bot

**Versi:** 0.1 (Draft)
**Terakhir diperbarui:** 2026-11-03
**Berlaku untuk:** Discord Bot Interface + Dashboard Admin HTMX Internal

---

## Daftar Isi

1. [Brand Identity](#1-brand-identity)
2. [Discord UI Guidelines](#2-discord-ui-guidelines)
3. [Dashboard Admin (HTMX)](#3-dashboard-admin-htmx)
4. [Konvensi Pesan Bot](#4-konvensi-pesan-bot)
5. [Accessibility](#5-accessibility)
6. [Komponen UI — Embed Templates](#6-komponen-ui--embed-templates)

---

## 1. Brand Identity

### Nama & Positioning

**Radovid Bot** — companion belajar bahasa Inggris. Bukan guru, bukan hakim. Posisinya adalah teman belajar yang memberi feedback jujur tanpa membuat pengguna merasa bodoh.

### Tone of Voice

| Konteks | Tone |
|---|---|
| Instruksi aktivitas | Jelas, ringkas, tidak bertele-tele |
| Koreksi grammar | Netral dan informatif — bukan "salah!", tapi "versi yang lebih natural adalah..." |
| Feedback positif | Tulus tapi tidak berlebihan — satu kalimat sudah cukup |
| Error / masalah teknis | Langsung, jujur, tidak panik |
| Achievement / milestone | Sedikit lebih warm — boleh ada satu emoji yang relevan |

**Yang dihindari:**
- Nada patronizing: "Good job, buddy! You're doing SO great!" (berlebihan)
- Nada dingin robotik: "ERROR: Input tidak valid." (tanpa konteks)
- Terlalu banyak emoji dalam satu pesan
- Exclamation marks berlebihan

### Personality

Radovid Bot adalah fasilitator, bukan performer. Tugasnya membuat pengguna nyaman berbicara bahasa Inggris, bukan membuat dirinya terlihat menarik. Respons bot harus terasa seperti datang dari seseorang yang sabar dan kompeten, bukan dari marketing copy.

---

## 2. Discord UI Guidelines

### 2.1 Warna Embed

Discord embed memiliki satu color bar di sisi kiri. Gunakan warna ini secara konsisten berdasarkan konteks:

| Konteks | Warna | Hex | Penggunaan |
|---|---|---|---|
| **Informasi / Instruksi** | Biru bersih | `#3B82F6` | Instruksi aktivitas, prompts, informasi sesi |
| **Sukses / Selesai** | Hijau netral | `#22C55E` | Sesi selesai, jawaban benar, achievement |
| **Koreksi / Perhatian** | Kuning amber | `#F59E0B` | Feedback grammar, koreksi bahasa |
| **Error / Gagal** | Merah | `#EF4444` | Error Ollama, command invalid, sesi gagal |
| **Netral / Progress** | Abu-abu gelap | `#6B7280` | Informasi statistik, leaderboard, history |

> **Catatan:** `#5865F2` (Discord Blurple) tidak digunakan sebagai warna aksen Radovid Bot untuk menghindari kebingungan dengan UI Discord sendiri. Warna utama Radovid Bot adalah `#3B82F6`.

### 2.2 Struktur Embed

Setiap embed Radovid Bot mengikuti hierarki ini:

```
[Color bar — sesuai konteks]
[Title] — singkat, deskriptif (max ~60 karakter)
[Description] — konten utama (max ~200 karakter; gunakan field untuk detail lebih)
[Fields] — key-value pairs untuk data terstruktur
  - Field inline: untuk data berpasangan (nama / nilai, skor / XP)
  - Field non-inline: untuk teks yang lebih panjang (feedback, koreksi)
[Footer] — metadata ringan: nama aktivitas, atau hint command
[Timestamp] — gunakan jika relevan (waktu sesi selesai, dll.)
```

**Yang dihindari dalam embed:**
- Thumbnail atau image yang tidak relevan (jangan tambahkan gambar hanya untuk dekorasi)
- Author field kecuali benar-benar dibutuhkan
- Embed di dalam embed (Discord tidak support ini, jangan coba)
- Title yang berisi emoji banyak

### 2.3 Button Styles

Discord menyediakan empat style button. Gunakan dengan konsisten:

| Style | Kode | Warna | Penggunaan di Radovid Bot |
|---|---|---|---|
| **Primary** | `ButtonStylePrimary` | Biru (Discord default) | Aksi utama: "Mulai Aktivitas", "Lanjutkan", "Pilih Ini" |
| **Secondary** | `ButtonStyleSecondary` | Abu-abu | Aksi sekunder: "Lihat Detail", "Kembali", "Skip" |
| **Success** | `ButtonStyleSuccess` | Hijau | Konfirmasi positif: "Selesai", "Submit Jawaban" |
| **Danger** | `ButtonStyleDanger` | Merah | Aksi destruktif: "Akhiri Sesi", "Reset Progress" |

**Konvensi label button:**
- Gunakan kata kerja aktif: "Mulai", "Pilih", "Lihat", bukan "OK" atau "Submit" yang generik.
- Maksimal 2–3 kata per label button.
- Jangan pakai capslock untuk penekanan.

### 2.4 Message Formatting

**Markdown yang diizinkan di Discord dan penggunaannya:**

| Format | Syntax | Penggunaan |
|---|---|---|
| Bold | `**teks**` | Kata kunci penting, nama aktivitas, jawaban yang benar |
| Italic | `*teks*` | Contoh kalimat, judul buku/film dalam konteks review |
| Code inline | `` `teks` `` | Kata yang salah (dalam koreksi), command Discord |
| Code block | ` ```teks``` ` | Contoh kalimat panjang atau paragraf yang ditulis pengguna |
| Blockquote | `> teks` | Mengutip kalimat pengguna sebelum memberikan koreksi |

**Yang tidak digunakan:**
- Heading (`# ##`) di dalam pesan Discord — Discord tidak render heading, hasilnya terlihat aneh.
- Strikethrough berlebihan.

---

## 3. Dashboard Admin (HTMX)

Dashboard ini adalah tool internal untuk admin/operator. Bukan untuk pengguna Discord. Desainnya mengutamakan keterbacaan dan kecepatan akses informasi, bukan estetika.

### 3.1 Palet Warna Dashboard

| Token | Hex | Penggunaan |
|---|---|---|
| `--color-bg` | `#F9FAFB` | Background halaman |
| `--color-surface` | `#FFFFFF` | Card, panel, tabel |
| `--color-border` | `#E5E7EB` | Border card, divider tabel |
| `--color-text-primary` | `#111827` | Teks utama, heading |
| `--color-text-secondary` | `#6B7280` | Label, metadata, placeholder |
| `--color-accent` | `#3B82F6` | Link aktif, badge status UP, tombol aksi |
| `--color-accent-hover` | `#2563EB` | Hover state tombol aksi |
| `--color-success` | `#22C55E` | Badge "UP", status positif |
| `--color-warning` | `#F59E0B` | Badge "DEGRADED", peringatan |
| `--color-error` | `#EF4444` | Badge "DOWN", error state |

Tidak ada dark mode. Tidak ada gradient. Tidak ada shadow tebal. Surface putih di atas background abu-abu terang sudah cukup untuk menciptakan hierarki visual yang jelas.

### 3.2 Tipografi

```css
font-family: Inter, system-ui, -apple-system, sans-serif;
```

- Inter digunakan jika tersedia; sistem fallback ke font sistem native.
- Tidak ada web font eksternal yang di-load (tidak ada Google Fonts import) — dashboard harus bisa diakses offline jika Inter sudah di-install, atau fallback ke system-ui.

| Elemen | Size | Weight |
|---|---|---|
| Page heading (h1) | `1.5rem` (24px) | `600` |
| Section heading (h2) | `1.125rem` (18px) | `600` |
| Body text | `0.875rem` (14px) | `400` |
| Label / metadata | `0.75rem` (12px) | `400` |
| Tombol | `0.875rem` (14px) | `500` |
| Tabel header | `0.75rem` (12px) | `600` (uppercase, letter-spacing sedikit) |

### 3.3 Layout

- Sidebar navigasi di kiri (lebar fixed: `240px`).
- Konten utama di kanan, lebar maksimal `1200px`, padding `24px`.
- Tidak ada multi-column layout yang kompleks. Satu kolom untuk konten utama, grid 2–3 kolom hanya untuk stat cards di halaman overview.

### 3.4 Komponen Dashboard

**Stat Card (Overview):**
```
┌─────────────────────┐
│  Label              │
│  Nilai Besar        │
│  Sub-teks kecil     │
└─────────────────────┘
```
- Border: `1px solid var(--color-border)`
- Background: `var(--color-surface)`
- Border-radius: `8px`
- Padding: `16px`
- Tidak ada shadow, tidak ada gradient.

**Tabel:**
- Header row: background `var(--color-bg)`, teks uppercase `12px`, color `var(--color-text-secondary)`.
- Row: border-bottom `1px solid var(--color-border)`, hover background `#F3F4F6`.
- Tidak ada zebra stripe — border sudah cukup untuk separasi visual.

**Status Badge:**
```
[ UP ]     → background #DCFCE7, text #15803D
[ DOWN ]   → background #FEE2E2, text #DC2626
[ DEGRADED ] → background #FEF3C7, text #D97706
```
- Border-radius: `9999px` (pill shape)
- Padding: `2px 8px`
- Font size: `12px`, weight `500`

**Tombol Aksi:**
- Primary: `background: #3B82F6`, `color: #FFFFFF`, hover `#2563EB`.
- Border-radius: `6px`, padding `8px 16px`.
- Tidak ada outline atau ghost button untuk aksi utama di dashboard.

---

## 4. Konvensi Pesan Bot

### 4.1 Format Respons Teks Biasa

Pesan bot yang tidak menggunakan embed (misalnya konfirmasi singkat atau pertanyaan sederhana):
- Mulai dengan konteks langsung: "Sesi **Story Relay** dimulai." bukan "Hey! I'm so excited to tell you that your Story Relay session has begun!"
- Satu paragraf, maksimal 3–4 baris.
- Jika ada instruksi lanjutan, pisahkan dengan baris kosong.

### 4.2 Embed Title Convention

Format: `[Nama Aktivitas] — [Fase/Konteks]`

Contoh:
- `Grammar Boss Fight — Ronde 3`
- `Daily Diary — Koreksi`
- `Sesi Selesai — Ringkasan`
- `Job Interview — Pertanyaan 5/10`

Tidak ada emoji di title kecuali satu emoji yang sangat relevan di awal, dan hanya untuk embed achievement atau milestone.

### 4.3 Embed Description

- Kalimat pertama adalah isi utama atau instruksi.
- Kalimat kedua (opsional) adalah konteks tambahan atau hint.
- Hindari pengulangan informasi yang sudah ada di title.

### 4.4 Field Naming Convention

Field name harus konsisten di seluruh aktivitas:

| Data | Field Name |
|---|---|
| Skor sesi | `Skor` |
| XP yang didapat | `XP Didapat` |
| Total XP pengguna | `Total XP` |
| Streak hari ini | `Streak` |
| Durasi sesi | `Durasi` |
| Kalimat asli pengguna | `Kalimat Kamu` |
| Versi yang dikoreksi | `Versi yang Lebih Baik` |
| Penjelasan koreksi | `Kenapa` |
| Saran tambahan | `Tips` |
| Level kesulitan | `Level` |
| Peserta sesi | `Pemain` |
| Aktivitas | `Aktivitas` |

---

## 5. Accessibility

### 5.1 Dashboard Admin

Karena dashboard adalah tool internal (bukan publik), standar accessibility yang berlaku adalah **WCAG 2.1 Level AA** untuk kontras warna — cukup untuk memastikan teks terbaca dengan baik.

**Kontras minimum (teks normal ≥ 4.5:1, teks besar ≥ 3:1):**

| Elemen | Foreground | Background | Rasio (approx.) |
|---|---|---|---|
| Body text | `#111827` | `#FFFFFF` | ~18:1 ✓ |
| Secondary text | `#6B7280` | `#FFFFFF` | ~4.6:1 ✓ |
| Tombol primary teks | `#FFFFFF` | `#3B82F6` | ~4.5:1 ✓ |
| Badge "UP" teks | `#15803D` | `#DCFCE7` | ~5.1:1 ✓ |
| Badge "DOWN" teks | `#DC2626` | `#FEE2E2` | ~4.5:1 ✓ |
| Tabel header teks | `#6B7280` | `#F9FAFB` | ~3.5:1 ✓ (teks uppercase/bold — memenuhi threshold teks besar) |

Semua elemen interaktif (button, link) harus memiliki `:focus` style yang visible — gunakan `outline: 2px solid #3B82F6`.

### 5.2 Discord Bot

Discord mengontrol sebagian besar aspek accessibility (screen reader support, contrast). Tanggung jawab Radovid Bot:
- Jangan gunakan emoji sebagai satu-satunya cara menyampaikan informasi penting (selalu sertakan teks).
- Jangan bergantung pada warna embed saja untuk membedakan jenis pesan — gunakan juga teks di title atau description.

---

## 6. Komponen UI — Embed Templates

### 6.1 Embed: Instruksi Aktivitas

```
Color: #3B82F6 (biru)
Title: [Nama Aktivitas]
Description: [Deskripsi singkat apa yang harus dilakukan]
Fields:
  - "Level" (inline): Beginner / Intermediate / Advanced
  - "Pemain" (inline): @user1, @user2
  - "Cara Main" (non-inline): [instruksi langkah demi langkah, 2–4 poin]
Footer: Ketik /radovid stop untuk mengakhiri sesi kapan saja
```

### 6.2 Embed: Koreksi Grammar

```
Color: #F59E0B (amber)
Title: English Correction
Description: (kosong atau satu kalimat konteks singkat)
Fields:
  - "Kalimat Kamu" (non-inline): > [kalimat asli pengguna dalam blockquote]
  - "Versi yang Lebih Baik" (non-inline): [kalimat yang sudah dikoreksi, bold pada bagian yang berubah]
  - "Kenapa" (non-inline): [penjelasan singkat, Bahasa Indonesia, max 2 kalimat]
  - "Tips" (non-inline, opsional): [saran tambahan jika ada]
Footer: (nama aktivitas yang sedang berjalan)
```

### 6.3 Embed: Ringkasan Sesi

```
Color: #22C55E (hijau)
Title: Sesi Selesai — [Nama Aktivitas]
Description: [Satu kalimat ringkasan opsional]
Fields:
  - "Durasi" (inline): [menit:detik]
  - "Aktivitas" (inline): [nama aktivitas]
  - "XP Didapat" (inline): +[jumlah] XP
  - "Total XP" (inline): [total] XP
  - "Streak" (inline): [N] hari 🔥
  - "Feedback" (non-inline, opsional): [1–2 kalimat feedback umum dari AI]
Buttons: [Mainkan Lagi] [Pilih Aktivitas Lain]
```

### 6.4 Embed: Scoreboard (Aktivitas Kompetitif)

```
Color: #6B7280 (abu-abu)
Title: Skor — [Nama Aktivitas]
Description: Ronde [N] dari [Total]
Fields (inline, per pemain):
  - "@user1": [skor]
  - "@user2": [skor]
Footer: (opsional: siapa giliran berikutnya)
```

### 6.5 Embed: Daily Challenge

```
Color: #3B82F6 (biru)
Title: Daily Challenge — [tanggal]
Description: [instruksi challenge hari ini]
Fields:
  - "Level" (inline): [level]
  - "Batas Waktu" (inline): Sebelum 23:59 WIB
Footer: Jawab langsung di channel ini | /radovid progress untuk cek streak
```

### 6.6 Embed: Achievement

```
Color: #22C55E (hijau)
Title: 🏅 Achievement Didapat!
Description: **[Nama Achievement]**
             [Deskripsi achievement dalam 1 kalimat]
Fields:
  - "Diraih oleh" (inline): @username
  - "XP Bonus" (inline): +[jumlah] XP
Footer: (total achievement yang sudah dikumpulkan pengguna ini)
```

---

*Design system ini adalah dokumen referensi teknis, bukan style guide marketing. Perubahan pada warna atau komponen harus mempertimbangkan dampak pada konsistensi di seluruh aktivitas dan dashboard.*
