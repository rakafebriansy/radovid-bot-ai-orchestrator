<type>(<scope>): <subject>

<body>

<footer>

---

# CONTOH PENULISAN (DILARANG DIMASUKKAN KE COMMIT MESSAGE):
# Type Options:
# - feat: Menambah fitur baru
# - fix: Memperbaiki bug
# - docs: Perubahan terkait dokumentasi (README, JSDoc, dll)
# - style: Perubahan formatting, tidak mengubah logika (spasi, titik koma)
# - refactor: Refactoring kode tanpa mengubah fungsionalitas
# - test: Menambahkan/mengubah skrip testing
# - chore: Perbaikan build process, dependencies, tooling
# 
# ATURAN ISTILAH ORCHESTRATOR:
# Istilah-istilah orchestrator meliputi: ticket, TICKET-XX, prd, design system, guideline, node, retrospective, orchestrator, template, dll.
#
# 1. CONTOH UNTUK REPOSITORI PROJECT / NODE (WAJIB 100% BERSIH DARI ISTILAH ORCHESTRATOR):
# DILARANG memuat kata 'orchestrator', 'ticket', 'TICKET-XX', 'Resolves TICKET-XX', 'prd', 'node', dll.
#
# feat(auth): add google oauth integration
# 
# Mengimplementasikan alur masuk menggunakan kredensial Google, 
# menyimpan token ke secure storage lokal.
#
# 2. CONTOH UNTUK REPOSITORI ORCHESTRATOR:
# WAJIB menggunakan istilah orchestrator sebagai scope:
#
# docs(orchestrator): update version control guideline
# 
# Memperbarui aturan commit message agar bersih dari istilah orchestrator pada node.
