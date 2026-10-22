# Tutup Kas Lion Group V3

[Unduh ZIP V3 langsung](https://github.com/andisinga015/systemtutupkas/raw/main/tutupkas-lion-group-v3.zip) · [Checksum SHA256](tutupkas-lion-group-v3.zip.sha256)

Pembaruan korektif V2: pembacaan sumber DEPO/QRIS memakai kolom tetap yang dikonfirmasi, hari kosong terverifikasi, tanggal massal/individual terpisah, Stop/Batalkan Antrean persisten, retry item tersisa dan penanganan job gagal. Pemetaan tujuan, aturan fee/transfer dan perlindungan finansial dipertahankan.

[Ringkasan implementasi dan berkas](RELEASE_NOTES_V3.md) · [Panduan upgrade V2 ke V3](UPGRADE_V3.md) · [Laporan pengujian](TEST_REPORT.md)

137 tes backend lulus pada Python 3.10/3.12 dan 5 tes Chromium lulus. Pengembang belum menjalankan live Google Sheets crawl/write atau deployment VPS untuk V3.

Untuk VPS yang sudah berjalan: unggah ZIP melalui SFTP, ekstrak ke staging terpisah, ikuti panduan, lalu `sudo bash deploy/upgrade.sh`. Skrip membackup DB/aplikasi/frontend, mempertahankan akun dan konfigurasi, menjalankan migrasi aditif serta menahan antrean V2 lama sebelum worker V3 dimulai. Tidak mengganti Nginx, HTTPS, key atau app.env. Rollback disediakan.

Berkas ZIP V1 dan V2 tetap disimpan sebagai arsip; gunakan **tutupkas-lion-group-v3.zip** untuk koreksi terbaru. Paket tidak memuat DB operasional atau private key.
