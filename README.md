# Tutup Kas Lion Group V4

[Unduh ZIP V4 langsung](https://github.com/andisinga015/systemtutupkas/raw/main/tutupkas-lion-group-v4.zip) · [Checksum SHA256](tutupkas-lion-group-v4.zip.sha256)

Koreksi backend V3 untuk OOM: pembacaan sumber dalam chunk, konteks satu tanggal, format sel terpilih, batas byte respons streaming, log memori VPS, serta recovery yang memerlukan retry admin setelah crash. UI/aset, parser keuangan, pemetaan tujuan, Notes, perlindungan formula/ownership dan rekonsiliasi tetap identik dengan V3. Tidak ada worker pool atau kenaikan MemoryMax.

[Upgrade V3 → V4](UPGRADE_V4.md) · [Berkas/perubahan](RELEASE_NOTES_V4.md) · [Tes](TEST_REPORT.md) · [Audit/performa](PERFORMANCE_V4.md) · [Scope dan batas](SCOPE_V4.md)

163 tes backend lulus pada Python 3.10 dan 3.12; 5 tes Chromium lulus. Pada subprocess RLIMIT_AS 1536 MiB, probe sumber sintetis memakai peak sekitar 45 MiB untuk 4.000–40.000 baris histori, dan satu run tiga-tab sekitar 129 MiB. Ini bukan bukti hasil workbook/VPS produksi.

**Live Google/VPS masih pending.** Discovery tetap menelusuri nilai historis secara bertahap demi keunikan tanggal V3; larangan literal membaca seluruh nilai histori belum sepenuhnya terpenuhi pada discovery. Jumlah request total dapat naik. Optimisasi marker region yang memerlukan perubahan semantik dihentikan dan dijelaskan dalam laporan. Tidak ada klaim OOM produksi definitif hilang.

Untuk VPS yang sudah terpasang, unggah ZIP dengan SFTP, ekstrak ke staging terpisah, ikuti panduan dan `sudo bash deploy/upgrade.sh`. Backup DB/aplikasi/frontend dibuat lebih dahulu. Antrean lama ditahan secara diaudit; Run #5 tidak otomatis aktif lagi. **API dimulai, worker tetap berhenti** sampai probe baca saja, PREVIEW penuh dan tinjauan operator. Akun, histori, intent tulis, key, app.env, Nginx, HTTPS dan override memori dipertahankan. Rollback tersedia.

ZIP V1/V2/V3 tetap menjadi arsip. Source lengkap V4, frontend siap pakai, tests, benchmark dan panduan ada dalam ZIP V4. Paket tidak memuat DB operasional atau private key. Footer UI tetap v3.0 karena frontend sengaja tidak diubah.
