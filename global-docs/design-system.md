# Design System

## Apa itu Design System
Design System adalah kumpulan terpusat yang berisi komponen visual, pedoman desain (seperti palet warna, tipografi, dan *spacing*), serta standar pola interaksi UI/UX yang dapat digunakan kembali (*reusable*). Dokumen ini bertujuan untuk memastikan konsistensi tampilan dan pengalaman pengguna di seluruh bagian aplikasi.

Dalam pendekatan *Vibe Coding* yang dikerjakan oleh AI Agent, Design System berfungsi sebagai pedoman gaya (*styling ground truth*). Saat AI meng-*generate* antarmuka (UI) atau komponen baru, AI akan merujuk secara ketat pada aturan-aturan di dokumen ini. Hal ini memastikan bahwa kode UI yang dihasilkan (misalnya kode Flutter atau CSS) akan secara otomatis selaras dengan identitas visual, *branding*, dan tema aplikasi, sehingga terhindar dari ketidakkonsistenan desain.

Secara rinci, sebuah dokumen Design System wajib memuat komponen-komponen berikut:
*   **Identitas Merek (*Brand Identity*):** Penjelasan mengenai filosofi desain, *vibe* atau nuansa aplikasi (contoh: *playful*, *minimalist*, korporat), serta *guideline* penggunaan aset merek.
*   **Palet Warna (*Color Palette*):** Daftar lengkap warna yang akan diaplikasikan, mencakup peran *Primary*, *Secondary*, *Background*, *Surface*, *Error*, *Success*, dan *Warning*. Wajib menyertakan kode warnanya (*Hex/RGB*) serta adaptasi untuk *Dark Mode* dan *Light Mode*.
*   **Tipografi (*Typography*):** Spesifikasi hierarki teks secara terstruktur, meliputi jenis huruf (*font family*), ukuran (*font sizes*), ketebalan (*font weights*), tingkat jarak huruf (*letter spacing*), dan tinggi baris (*line heights*) untuk elemen *Heading 1-6*, *Body*, *Subtitle*, *Caption*, dan teks pada tombol.
*   **Sistem *Spacing*, Tata Letak, & Breakpoints (*Spacing & Layout System*):** Skala kelipatan angka standar untuk penentuan jarak *margin*, *padding*, sistem *grid*, penentuan titik batas responsif (*responsive breakpoints* untuk *mobile*, *tablet*, *desktop*), serta spesifikasi sudut elemen (*border radius*).
*   **Aset & Ikonografi (*Iconography & Assets*):** Spesifikasi gaya visual ikon yang digunakan (seperti *outlined*, *filled*, atau *two-tone*), rasio ukuran standar ikon, dan acuan tata letak ilustrasi.
*   **Komponen UI Utama (*UI Components Library*):** Rincian dan spesifikasi desain untuk setiap elemen *reusable* (seperti *buttons*, *text fields*, *dropdowns*, *cards*, *modals*, *bottom sheets*, dll.) beserta semua kemungkinan statusnya (*default*, *hover*, *pressed*, *disabled*, *focus*, *error*).
*   **Animasi & Interaksi (*Micro-interactions/Animations*):** Standar perilaku gerakan antarmuka, mencakup durasi waktu (*duration*), kurva transisi (*easing curve*), dan *feedback* visual terhadap aksi dari pengguna.
*   **Standar Aksesibilitas (a11y):** Persyaratan minimum aksesibilitas yang harus ditaati (seperti *color contrast ratio*, *focus states* yang jelas, dan ketersediaan label pembaca layar/*screen reader*).

### Aturan Khusus Ekosistem Multi-Project (Multi-Node)
Jika ekosistem ini terdiri dari berbagai *platform* atau aplikasi yang memiliki interaksi (*Multi-Node*), strukturkan Design System ini menjadi dua tingkatan:
1. **Global Tokens:** Aturan fundamental (warna, tipografi utama, aset jenama/logo) yang wajib dipatuhi secara universal oleh seluruh *node* demi konsistensi *brand*.
2. **Node-Specific Components:** Bagilah daftar komponen UI spesifik ke dalam sub-bab masing-masing *node* (misalnya: sub-bab "Komponen Web-Admin" terpisah dari sub-bab "Komponen Mobile-App") guna mencegah kontaminasi rancangan dan kerancuan (*ambiguity*) ketika AI melakukan koding antarmuka di node tertentu.

## Cara Melakukan Generate Design System
Jelaskan "rasa" (*vibe*), tema, gaya visual (*modern*, minimalis, *playful*, dsb), serta preferensi estetika dari aplikasi Anda kepada AI Agent. Kemudian, mintalah AI Agent untuk mengajukan pertanyaan-pertanyaan spesifik terkait aset visual (seperti warna primer/sekunder, bentuk tombol, *dark/light mode*, tipografi) guna memperinci gambaran desain Anda. Berdasarkan interaksi tersebut, AI akan merumuskan dan menyusun pedoman komponen UI yang baku.

### Prinsip Anti-AI-Slop dalam Perancangan Design System
1. **Berakar pada Domain Produk (Product-Specific Intent):** Hindari estetika SaaS generik (mesh gradien ungu/biru, efek *glassmorphism* tebal, *glowing borders*, kartu berulang identik). Tentukan arah visual berbasis kebutuhan tugas pengguna nyata.
2. **Hierarki Grayscale First:** Pastikan struktur dan keterbacaan komponen berfungsi sempurna dalam monokrom/hitam-putih sebelum menetapkan palet warna dan dekorasi.
3. **Pencegahan Over-Decoration:** Komponen harus dirancang fungsional, memuat seluruh variasi status (*loading/empty/error*), memenuhi kontras WCAG AA, dan mematuhi seluruh hukum Anti-Slop di `global-guidelines/ui-and-assets.md`.

> **Kewajiban Uji Validasi HTML:** Desain yang dirumuskan di dokumen ini **TIDAK BOLEH** langsung dikoding ke dalam *framework* asli (Flutter, Next.js, SwiftUI, dll.). AI Agent **WAJIB** membuat sketsa purwarupanya terlebih dahulu dalam format HTML sederhana di direktori `nodes/[nama-node]/prototypes/` dari proyek yang bersangkutan.
## [GANTI_DENGAN_NAMA_PROYEK]

> ⚠️ AI Agent: Hapus placeholder judul di atas dan ganti dengan nama proyek atau ekosistem yang sebenarnya.

*(Detail dari komponen, warna, tipografi, dan panduan Design System spesifik untuk aplikasi atau ekosistem ini akan ditambahkan di bawah batas ini)*
