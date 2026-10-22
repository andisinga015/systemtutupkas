# Upgrade korektif V2 → V3

Ini pembaruan aplikasi yang sudah ada, dengan struktur VPS tetap: backend `/opt/tutupkas/current/`, data `/var/lib/tutupkas/`, frontend `/var/www/html/`, environment `/etc/tutupkas/app.env`, dan Service Account `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json`. Jangan menghapus DB, membuat akun ulang, mengganti key, atau memasang ulang Nginx/HTTPS.

## Perubahan penting

V3 membaca kolom SUMBER tetap: DEPO J/K/L untuk total deposit/WD/saldo, H biaya, I kiriman; QRIS I/J/K untuk total deposit/WD/saldo, G biaya, H fee/kiriman. Deskripsi tetap D. Pemetaan TUJUAN tetap dinamis. Hari yang terbukti tanpa transaksi menghasilkan total nol dan membawa saldo valid terakhir. Formula/error/missing date tidak disamarkan menjadi nol.

Tanggal di atas panel hanya untuk penarikan massal; tiap WEB mempunyai tanggal sendiri dan status untuk tanggal itu. Job antre/berjalan, termasuk preview dan batch, dapat dibatalkan. Stop menunggu batas aman; tulisan yang sudah dikirim tetap diupayakan verifikasinya. Job Dibatalkan tidak otomatis dihidupkan lagi. Hasil berhasil dan ledger PREPARED/VERIFIED tetap tersimpan.

## 1. Unggah ke staging

Unduh `tutupkas-lion-group-v3.zip` dan `.zip.sha256`. Unggah keduanya dengan FileZilla/SFTP ke `/tmp/tutupkas-upload/`. Jangan mengekstrak menimpa `/opt/tutupkas/current`.

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v3.zip.sha256
mkdir -p /tmp/tutupkas-staging-v3
unzip tutupkas-lion-group-v3.zip -d /tmp/tutupkas-staging-v3
cd /tmp/tutupkas-staging-v3/tutupkas-lion-group
```

Paket sudah memuat `public/` hasil build; Node/npm tidak diperlukan di VPS. Python 3.10+, venv, rsync, tar dan curl harus tersedia.

## 2. Tinjau konfigurasi dan pekerjaan lama

```bash
sudo systemctl show tutupkas-api tutupkas-worker -p User -p WorkingDirectory -p EnvironmentFiles
sudo systemctl status tutupkas-api tutupkas-worker --no-pager
sudo nginx -t
less backend/migrations/versions/b7c31ef05a92_safe_cancellation.py
less deploy/upgrade.sh
```

Skrip mengharapkan user layanan `tutupkas`, WorkingDirectory `/opt/tutupkas/current/backend` dan environment file `/etc/tutupkas/app.env`. Jika berbeda, sesuaikan panduan/skrip dengan unit aktual; jangan menimpa unit yang berfungsi. DB diambil dari environment yang sudah ada dan harus berada di `/var/lib/tutupkas`.

Catat run yang masih QUEUED/RUNNING dan tanggalnya sebelum upgrade. V2 belum mempunyai Stop aman dari UI. Jadwalkan jeda operasional dan jangan menekan Start selama upgrade. Penghentian worker V2 dapat menunggu run yang sedang berlangsung selesai sampai batas timeout unit; jangan memakai kill -9. Jika penghentian gagal/timeout, periksa journal dan pastikan proses telah berhenti sebelum melanjutkan. Tulisan yang sudah diterima Google tidak dibatalkan atau dihapus oleh upgrade. Tinjau hasil pada tanggal yang salah bila penarikan itu sudah sempat berjalan.

## 3. Backup dan upgrade

```bash
sudo bash deploy/upgrade.sh
```

Skrip menyiapkan rilis terpisah di `/opt/tutupkas/releases/v3-...` dan dependensi Python terkunci. API dihentikan lebih dahulu, lalu worker. Setelah keduanya berhenti, dibuat folder `/var/lib/tutupkas/backups/upgrade-v3-...` berizin 0700, berisi backup SQLite dengan integrity check, seluruh aplikasi lama, frontend lama, environment privat dan path pemulihan. Backup SQLite tidak sekadar menyalin DB aktif tanpa WAL.

Migrasi `daf814b2a736 → b7c31ef05a92` hanya menambah kolom nullable pembatalan/fase dan memperbarui indeks pekerjaan aktif agar mencakup Pembatalan Diminta. Tidak membangun ulang/drop tabel finansial, menghapus history, atau mengganti akun. `alembic check` harus bersih.

**Sebelum worker V3 dinyalakan**, skrip menjalankan:

```bash
# Sudah dijalankan otomatis oleh upgrade.sh setelah backup dan migrasi.
# Jangan menjalankannya ketika layanan masih bekerja.
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/releases/NAMA-RILIS-V3/backend; exec ../.venv/bin/python -m app.cli prepare-v3-upgrade --services-stopped'
```

Perintah itu memfinalkan antrean/batch lama yang belum selesai menjadi Dibatalkan dan mencatat audit. Hasil SUCCESS/PREVIEWED, error, ledger, Note dan kepemilikan transfer tidak dihapus. Run yang sebelumnya memang Dibatalkan tetap tidak dijalankan. Ini mencegah tanggal salah atau pekerjaan V2 lama diteruskan diam-diam oleh engine yang sudah diperbaiki. Setelah upgrade, operator memeriksa intent yang belum pasti lalu memilih Ulangi Item Tersisa atau Start baru secara eksplisit.

Rilis kemudian diaktifkan, aset hash disalin ke `/var/www/html`, HTML diganti terakhir, dan API/worker dinyalakan. Nginx/Certbot/sertifikat/key/app.env/unit layanan tidak ditimpa. Bila `current` berupa direktori, isinya dipindah utuh ke rilis legacy setelah backup, lalu diganti symlink.

## 4. Health dan cache

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl is-active tutupkas-api tutupkas-worker
sudo journalctl -u tutupkas-api -u tutupkas-worker -n 80 --no-pager
curl -I https://tutupkasliongroup.com/
```

Bandingkan nama `/assets/index-HASH.js` dan CSS pada HTML domain dengan `public/index.html` paket. HTML/SPA harus menggunakan Cache-Control no-cache/no-store; aset hash boleh di-cache lama. Aset lama dipertahankan untuk halaman lama/rollback. Jika kebijakan Nginx yang ada sudah benar, tidak perlu mengubahnya. Bila perlu memperbaiki cache HTML, backup konfigurasi, ubah location yang ada tanpa membuat server SSL duplikat, jalankan `nginx -t`, baru reload Nginx. Lakukan Ctrl+Shift+R dan periksa cache CDN bila ada.

## 5. Smoke test yang harus dijalankan di VPS

1. Login dengan akun lama. Pastikan WEB, sumber/tujuan bulanan, revisi URL, sheet aktif/nonaktif, run dan error lama tetap tersedia. Footer menampilkan v3.0.
2. Periksa antrean V2 yang ditahan. Jangan langsung mengulangi semua tanggal; pilih hanya tanggal yang memang Anda otorisasi. Intent belum pasti tidak boleh dihapus agar retry tampak sukses.
3. Pilih tanggal massal, lalu ubah tanggal individual VELBETT. Pastikan tanggal FANTASI dan tanggal massal tidak berubah; status setiap baris mengikuti tanggalnya.
4. Pratinjau contoh VELBETT DOC DEPO, tab `WD BCA 1 TOKEN / IVAN RAKHMANDA ( PP )`, tanggal 18 Oktober 2026. Pada struktur yang diberikan, closing row 309, deposit 0, WD 0, saldo dari L sebesar 2.355.000. Header Total yang hilang/ganda tidak boleh menggagalkan pembacaan tetap tersebut.
5. Periksa hari bertransaksi: nilai J/K/L DEPO dan I/J/K QRIS harus sesuai sumber, bukan dipaksa nol. Missing date, saldo error dan masalah Google tetap harus muncul sebagai error yang akurat.
6. Bandingkan BIAYA H DEPO, biaya G QRIS, fee H QRIS, deskripsi asli/newline serta FEE WD AUTO sekali. Periksa transfer cokelat/biru, bank DEPO, urutan overflow QRIS, Note Shift+F2, TOTAL/formula dan isi manual.
7. Cancel satu antrean preview dan satu batch. Status harus Dibatalkan dan tidak bangkit setelah restart worker. Untuk Stop running, tunggu batas aman; tulisan yang sudah dikirim harus diverifikasi atau dicatat belum pasti. Setelah final, retry hanya item tersisa, tanpa menduplikasi hasil sukses.
8. Jalankan Start nyata hanya pada salinan kerja yang diotorisasi, lalu bandingkan hasil read-back. `Laporan deposit harian` harus tetap baca saja; saldo per WEB dan konsolidasi tepat nol sebelum Clear. Cancel tidak membuktikan KLOP.

Verifikasi tambahan yang hanya membaca:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec ../.venv/bin/python ../scripts/verify-real-run.py RUN_ID'
```

Ganti RUN_ID dengan hasil nyata terbaru yang telah Anda otorisasi. Pengembang tidak memiliki SSH/key VPS dan belum menguji penulisan Google asli untuk V3. Pengujian fixture bukan bukti live crawl/write.

## 6. Rollback ke V2

Rollback memulihkan backup DB sebelum upgrade; perubahan setelah backup tidak akan ada pada DB aktif hasil rollback. Helper juga menyimpan salinan DB setelah upgrade agar dapat ditinjau. Jangan kembali beroperasi sebelum hasil pemulihan diverifikasi.

```bash
sudo bash /tmp/tutupkas-staging-v3/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v3-TANGGAL-ID
```

Helper menghentikan layanan, menyimpan snapshot saat ini, memulihkan DB lama secara aman ketika layanan berhenti, mengembalikan symlink/frontend lama, lalu memfinalkan antrean snapshot lama melalui `scripts/cancel-legacy-queue.py`. Helper stdlib ini kompatibel dengan skema V2 dan mempertahankan hasil/error/ledger. Alasannya: backup V2 mungkin masih berisi job yang baru Anda batalkan sesudahnya pada V3; job itu tidak boleh otomatis hidup kembali saat rollback.

**Hanya API yang dinyalakan otomatis setelah rollback; worker tetap berhenti.** Tinjau akun, history, hasil Google dan intent PREPARED. Setelah yakin tidak ada pekerjaan salah tanggal yang akan dijalankan, barulah:

```bash
sudo systemctl start tutupkas-worker
sudo systemctl is-active tutupkas-api tutupkas-worker
```

Hard refresh browser. V2 tetap memiliki bug sumber yang diperbaiki V3; rollback hanya mengembalikan rilis, bukan memperbaiki V2. Tidak menggunakan downgrade destruktif Alembic. Jika rilis lama hilang di luar prosedur, ekstrak `application.tar.gz` ke rilis pemulihan terpisah dan sesuaikan `previous-release`; jangan menimpa rilis aktif.
