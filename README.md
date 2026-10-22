# Tutup Kas Lion Group v2

[Unduh langsung ZIP v2](https://github.com/andisinga015/systemtutupkas/raw/main/tutupkas-lion-group-v2.zip) · [SHA256](tutupkas-lion-group-v2.zip.sha256)

Paket lengkap: source backend/frontend, frontend siap pakai, migrasi database, worker, rekonsiliasi, Crawl All, Help Center, tests, skrip upgrade/rollback dan panduan Indonesia. Tidak memuat database operasional atau key Google.

Untuk VPS yang sudah berjalan, baca [panduan upgrade v2](UPGRADE_V2.md), unggah ZIP melalui SFTP, ekstrak ke staging, lalu jalankan `sudo bash deploy/upgrade.sh`. Akun admin, database, Nginx, HTTPS dan key yang ada dipertahankan. Jangan membuat database atau admin ulang.

106 tes backend lulus pada Python 3.10 dan 3.12, serta 4 tes Chromium. Lihat [laporan pengujian](TEST_REPORT.md). Google Sheets dan deployment VPS sebenarnya belum dijalankan oleh pengembang untuk pembaruan ini.

Berkas `tutupkas-lion-group.zip` adalah arsip v1. Gunakan **tutupkas-lion-group-v2.zip** untuk pembaruan terbaru.
