# Design System Guide & Specification Template

> **[PRINSIP UTAMA]** Design System adalah acuan gaya visual (*styling ground truth*) untuk menjamin konsistensi antarmuka, aksesibilitas, dan identitas merek di seluruh aplikasi.

---

## 1. Arsitektur & Komponen Inti

### 1.1 Styling Ground Truth dalam AI Orchestration
Rujukan mutlak AI Agent saat menghasilkan kode UI (Flutter, CSS, Tailwind, SwiftUI, Web) guna mencegah pergeseran gaya visual (*design drift*) dan estetika generik.

### 1.2 Delapan Komponen Spesifikasi Wajib
1. **Identitas Merek (*Brand Identity*):** Filosofi desain, nuansa produk (*tone/vibe*), dan batasan aset merek.
2. **Palet Warna (*Color Palette*):** Peran fungsional (Primary, Secondary, Background, Surface, Error, Success, Warning), format Hex/RGB/HSL, serta kontras Dark & Light Mode.
3. **Tipografi (*Typography*):** Hierarki (H1-H6, Body, Subtitle, Caption, Button), font family, ukuran rem/pt, bobot, line-height, letter-spacing, dan `tabular-nums` untuk angka.
4. **Sistem Spacing & Tata Letak:** Skala kelipatan 4px/8px, grid/flexbox, breakpoint responsif (mobile, tablet, desktop), dan konsistensi border radius.
5. **Ikonografi & Aset:** Satu keluarga pustaka ikon (outlined/filled), skala dimensi standar, dan acuan ilustrasi.
6. **Pustaka Komponen UI:** Spesifikasi komponen reusable beserta kelengkapan 11 status fungsional sesuai [ui-and-assets.md](../global-guidelines/ui-and-assets.md).
7. **Animasi & Interaksi Mikro:** Durasi transisi, kurva easing, umpan balik visual, dan dukungan wajib `prefers-reduced-motion`.
8. **Standar Aksesibilitas (a11y):** Kontras WCAG AA (>= 4.5:1 teks normal, >= 3:1 teks besar), target sentuh (>= 24x24px web / >= 44x44pt mobile), dan focus visible ring.

### 1.3 Arsitektur Multi-Node
- **Global Tokens:** Aturan universal (warna brand, tipografi utama, logo) yang mengikat seluruh node.
- **Node-Specific Components:** Pemisahan spesifikasi per-node (misal: "Web-Admin" vs "Mobile-App") untuk mencegah kontaminasi rancangan antar-platform.

---

## 2. Alur Perumusan & Validasi Prototipe

### 2.1 Protokol Perumusan Design System
1. **Definisi Niat:** Gali preferensi estetika, target audiens, dan batas fungsi produk nyata.
2. **Pertanyaan Spesifik:** AI wajib mengonfirmasi palet warna, geometri sudut, mode gelap/terang, dan tipografi sebelum menyusun dokumen.
3. **Doktrin Anti-Slop & Keahlian Frontend:** Wajib mengintegrasikan keahlian resmi [frontend-design SKILL.md](../skills/frontend-design/SKILL.md) dan [ui-and-assets.md](../global-guidelines/ui-and-assets.md). Berakar pada domain produk (hindari mesh ungu/biru, dark navy neon, glassmorphism tebal, card soup), tetapkan 4–6 nilai hex spesifik, tentukan pasangan tipografi kontras, dan hapus dekorasi non-fungsional.
4. **Kebijakan Relative Path:** Seluruh tautan dokumen dan aset di dalam dokumen ini WAJIB menggunakan relative path. Dilarang menggunakan absolute path.

### 2.2 Validasi Prototipe Wajib (Pre-Coding Gate)
1. **[TERLARANG]** Menulis kode antarmuka pada framework produksi (Flutter, Next.js, SwiftUI) sebelum validasi visual.
2. **[WAJIB]** Rancang purwarupa interaktif (*Single-File HTML*) di `nodes/[nama-node]/prototypes/` sebelum koding produksi.


