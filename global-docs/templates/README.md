# Templates Directory (`global-docs/templates/`)

> **[PRINSIP UTAMA]** Repositori terpusat seluruh cetak biru dokumen (*boilerplate*) standar ekosistem guna menjaga konsistensi format komunikasi dan pelaporan teknis.

---

## 1. Katalog Template Standar

| Berkas Template | Fungsi & Cakupan Penggunaan |
|---|---|
| **[`pull_request_template.md`](./pull_request_template.md)** | Deskripsi PR standar dengan sintaks penutup tiket wajib (`Resolves TICKET-XX`). |
| **[`commit_message_template.md`](./commit_message_template.md)** | Format Semantic Commit (`feat:`, `fix:`, dll.) dengan aturan pemisahan steril Project/Node vs Orchestrator. |
| **[`bug_report_template.md`](./bug_report_template.md)** | Format tiket pelaporan bug dengan analisis akar masalah, pengujian regresi O-07, dan damage assessment O-09. |
| **[`hunt-findings.md`](./hunt-findings.md)** | Format laporan sesi audit ketahanan sistem dan eksplorasi kasus ekstrem (Hunt Loop). |
| **[`api_documentation_template.md`](./api_documentation_template.md)** | Spesifikasi endpoint API, skema request/response payload, dan contoh JSON. |
| **[`routing_template.md`](./routing_template.md)** | Pemetaan rute halaman antarmuka web dan endpoint peladen. |
| **[`changelog_entry_template.md`](./changelog_entry_template.md)** | Format baku pencatatan log pembaruan riwayat `CHANGELOG.md` secara *reverse-chronological*. |

---

## 2. Kebijakan Penggunaan & Tata Kelola Git

### 2.1 Kewajiban Format Baku (SSoT)
AI Agent DILARANG mengarang struktur dokumen baru di luar format template resmi saat mengeksekusi penugasan terkait.

### 2.2 Aturan Istilah Orchestrator pada Aksi Git
- **Di Direktori Project/Node:** **[TERLARANG]** Menyisipkan istilah orchestrator atau ID tiket ke dalam pesan komit, branch, atau PR.
- **Di Repositori Orchestrator:** **[WAJIB]** Menggunakan cakupan orchestrator sebagai scope komit (contoh: `docs(orchestrator): update templates`).
