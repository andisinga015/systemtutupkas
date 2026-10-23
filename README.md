# Tutup Kas Lion Group V4.1

[Unduh ZIP V4.1 langsung](https://github.com/andisinga015/systemtutupkas/raw/main/tutupkas-lion-group-v4.1.zip) · [Checksum SHA256](tutupkas-lion-group-v4.1.zip.sha256)

Patch akses tujuan Google Sheets yang melebar horizontal. V4 menolak alokasi besar melalui guard 250.000 sel dan masih menahan grid padat. V4.1 memakai indeks struktur temporer, target GET eksplisit, dan scan global terbatas untuk planning/pre-write/read-back/laporan. Tidak ada hardcode kolom website, kenaikan MemoryMax, atau perubahan aturan keuangan. Source reader V4, UI/aset, API/auth, skema DB dan worker tetap sama.

[Upgrade V4 → V4.1 dan rollback](UPGRADE_V41.md) · [Berkas/fungsi berubah](RELEASE_NOTES_V41.md) · [Tes](TEST_REPORT.md) · [Benchmark](PERFORMANCE_V41.md) · [Scope](SCOPE_V41.md)

**198 tes backend lulus pada Python 3.10 dan 3.12; 5 tes Chromium lulus.** Golden 22 output, dua layout kiriman, boundary website, far-right BBR75, Notes/manual data, ownership/PREPARED, merge, formula global dan laporan read-only diuji. Fixture 1.000 × 1.500 dengan seluruh blank terformat memakai peak proses 172,79 MiB di subprocess RLIMIT_AS 1536 MiB. Peak mencakup harness dan bukan bukti penggunaan RAM systemd VPS. Data mentah: [tujuan](PERFORMANCE_DESTINATION_V41.json) dan [run website](PERFORMANCE_WEBSITE_V41.json).

**Live Google/VPS masih pending.** Pemindaian global mempertahankan proteksi V4 dan tetap mahal: run fixture lengkap memakai 1.319 request gabungan, kira-kira 26,4 menit pada interval 1,2 detik sebelum latensi/retry. Tidak ada klaim OOM produksi pasti hilang atau run produksi selesai secepat benchmark offline. Vektor metadata merge yang sendiri melebihi cap byte gagal eksplisit.

Untuk VPS V4 yang sudah terpasang, unggah ZIP/checksum melalui SFTP, ekstrak ke staging terpisah, ikuti panduan dan jalankan `sudo bash deploy/upgrade.sh`. Backup dibuat sebelum antrean lama ditahan secara diaudit. **API dimulai, worker tetap berhenti; Run #7 tidak otomatis diulang.** PREVIEW VELBETT tanggal 18 Oktober 2026 dan pemeriksaan intent dilakukan sebelum izin tulis operator. Akun, history, ledger, key, app.env, Nginx, HTTPS dan override MemoryMax dipertahankan. Rollback tersedia.

Source lengkap, public frontend siap pakai, tests, benchmark, panduan dan laporan ada dalam ZIP. Tidak ada DB operasional, private key, atau environment privat. Footer tetap v3.0 sesuai batas tidak mengubah UI. ZIP V1/V2/V3/V4 tetap menjadi arsip; [laporan V4](TEST_REPORT_V4.md) dan [laporan V3](TEST_REPORT_V3.md) dipertahankan.
