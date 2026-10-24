# Tutup Kas Lion Group V4.2

[Unduh ZIP V4.2 langsung](https://github.com/andisinga015/systemtutupkas/raw/refs/heads/main/tutupkas-lion-group-v4.2.zip) · [SHA256](tutupkas-lion-group-v4.2.zip.sha256)

Paket berisi seluruh source, frontend prebuilt, migrasi, test dan dokumentasi. Unggah ZIP ke staging VPS, lalu ikuti [upgrade V4.1 → V4.2](UPGRADE_V42.md). Worker tetap berhenti untuk penerimaan manual setelah upgrade API.

283 tes backend lulus pada Python 3.10 dan 3.12; delapan tes Chromium lulus. [Laporan pengujian](TEST_REPORT.md), [benchmark](PERFORMANCE_V42.md), [perubahan per file](CHANGED_FILES_V42.md), dan [catatan rilis](RELEASE_NOTES_V42.md). Source serta tautan test lengkap pada laporan tersedia di dalam ZIP.

Validasi Google/VPS asli dan target Preview live <60 detik masih pending. Fast path memerlukan metadata versi Drive dengan Service Account yang sama; fallback tetap aman tetapi lebih lambat. Akun, app.env, key, Nginx/HTTPS dan MemoryMax lama dipertahankan. Simpan backup DB dan release sebelum upgrade.

Arsip V4.1/V4/V3/V2 tetap tersedia. Laporan V4.1 dipertahankan di TEST_REPORT_V41.md.
