# Upgrade Tutup Kas Lion Group ke v2

Panduan ini untuk VPS yang sudah beroperasi. Nginx, Certbot, sertifikat HTTPS, akun admin, `/etc/tutupkas/app.env`, palet, dan key Google yang sudah ada dipertahankan. Tidak perlu membuat akun admin baru. Pengembang tidak memiliki SSH ke VPS Anda.

## 1. Unggah dan ekstrak terpisah

Unduh `tutupkas-lion-group-v2.zip` dan berkas `.sha256` dari tautan rilis yang diberikan. Unggah keduanya melalui FileZilla/SFTP ke `/tmp/tutupkas-upload/`, bukan ke folder aplikasi aktif. Di terminal VPS:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v2.zip.sha256
mkdir -p /tmp/tutupkas-staging-v2
unzip tutupkas-lion-group-v2.zip -d /tmp/tutupkas-staging-v2
cd /tmp/tutupkas-staging-v2/tutupkas-lion-group
```

Paket sudah berisi hasil build `public/`; Node.js dan npm tidak diperlukan di VPS. Pastikan Python 3.10+ dan modul venv, rsync, tar, curl tersedia. Skrip upgrade tidak menginstal ulang Nginx atau Certbot.

## 2. Periksa layanan dan migrasi

```bash
sudo systemctl show tutupkas-api tutupkas-worker -p User -p WorkingDirectory -p EnvironmentFiles
sudo systemctl status tutupkas-api tutupkas-worker --no-pager
sudo nginx -t
less backend/migrations/versions/daf814b2a736_rekonsiliasi_tujuan_bulanan_dan_.py
less deploy/upgrade.sh
```

Skrip mengharapkan User `tutupkas`, WorkingDirectory `/opt/tutupkas/current/backend`, dan EnvironmentFile `/etc/tutupkas/app.env`. Jika instalasi memakai nama/path lain, sesuaikan skrip terhadap unit aktual sebelum menjalankannya. Jangan mengganti unit yang berfungsi hanya agar cocok dengan contoh. `/opt/tutupkas/current` boleh berupa direktori biasa atau symlink. Database harus berada di `/var/lib/tutupkas/`; lokasi diambil dari konfigurasi lama. Key tetap di `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json` dan tidak disalin ke rilis.

Migrasi `226dae92145d → daf814b2a736` menambah tabel tujuan bulanan, batch, rekonsiliasi dan Clear global, serta kolom nullable. Tabel finansial lama tidak dibangun ulang/dihapus. URL lama diadopsi hanya jika satu periode memiliki tepat satu URL. Dua konfigurasi lama dalam bulan yang sama memerlukan pilihan admin eksplisit. Clear lama tetap riwayat; tidak otomatis menjadi Clear seluruh WEB.

## 3. Backup, migrasi, dan aktivasi

Jadwalkan jeda operasional. Biarkan pekerjaan berjalan selesai dan jangan memulai penarikan dari browser selama upgrade. Jalankan:

```bash
sudo bash deploy/upgrade.sh
```

Skrip menyiapkan rilis baru di `/opt/tutupkas/releases/v2-...`, memasang dependensi Python terkunci, memeriksa akses key tanpa menampilkan isinya, lalu menghentikan API dan worker. Worker diberi kesempatan berhenti sesuai unit systemd. Skrip membuat folder backup berizin 0700 di `/var/lib/tutupkas/backups/upgrade-v2-...` berisi:

- `database.sqlite3`: backup SQLite melalui API backup, disertai integrity check.
- `application.tar.gz`: seluruh rilis aplikasi lama, termasuk virtualenv untuk pemulihan.
- `frontend.tar.gz`: isi `/var/www/html/` sebelum pembaruan.
- `app.env.backup`, `previous-release`, `database-path`: konfigurasi dan petunjuk pemulihan; jangan unggah ke GitHub.

Setelah backup, skrip menjalankan `alembic current`, `upgrade head`, dan `check`; mengganti symlink rilis secara atomik; menyalin aset baru ke `/var/www/html/` dan mengganti `index.html` terakhir; kemudian menyalakan API/worker dan memeriksa `/api/health`. Rilis lama dipertahankan. Direktori `current` lama yang bukan symlink dipindah utuh ke `releases/legacy-v2-...` setelah backup. Skrip tidak mengganti unit systemd atau mengubah konfigurasi Nginx. Jika gagal setelah penghentian layanan, ikuti rollback di bawah sebelum kembali bertransaksi.

## 4. Cache frontend

Nama JavaScript/CSS hasil build menggunakan hash. Aset baru dipasang sebelum HTML baru, dan aset lama dipertahankan sehingga halaman yang masih tersimpan tetap dapat memuat berkasnya. Periksa respons HTML dan aset pada domain Anda:

```bash
curl -I https://tutupkasliongroup.com/
curl -s https://tutupkasliongroup.com/ -o /tmp/tutupkas-served-index.html
```

Bandingkan nama `/assets/index-....js` dan `.css` pada HTML tersebut dengan `public/index.html` paket. HTML/route SPA harus memakai `Cache-Control: no-cache` (boleh ditambah `must-revalidate`) atau `no-store`; aset hash boleh `max-age=31536000` (opsional `public, immutable`). Jika header Nginx yang ada sudah sesuai, tidak perlu mengubahnya. Jika HTML di-cache lama, backup konfigurasi situs lalu tambahkan kebijakan pada location HTML/SPA yang sudah ada; jangan membuat location atau server HTTPS duplikat. Contoh kebijakan tersedia di `deploy/nginx-locations.conf`. Jalankan `sudo nginx -t` sebelum `sudo systemctl reload nginx` hanya bila konfigurasi memang diubah. Periksa cache CDN jika digunakan. Lakukan hard refresh browser (Ctrl+Shift+R).

## 5. Verifikasi dan smoke test

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl is-active tutupkas-api tutupkas-worker
sudo journalctl -u tutupkas-api -u tutupkas-worker -n 60 --no-pager
```

1. Masuk memakai akun lama. Pastikan WEB, sumber, sheet aktif, URL history dan run lama tetap tersedia.
2. Pastikan sidebar menampilkan **Tutup Kas All Web**, kartu Panduan dapat dicapai melalui scroll, dan `/panduan` dapat mencari `validasi`, `sheet aktif`, `crawl semua`, `balance tidak klop`, `url baru`.
3. Pada Kas Kecil, pilih bulan yang benar. Jika terdapat dua URL lama, simpan URL salinan kerja yang dipilih lalu validasi. Tidak ada pilihan TEST/produksi permanen.
4. Simpan/validasi tidak boleh membuat run baru. Periksa riwayat. Ganti URL hanya bila memang diperlukan; hasil lama harus tampil Perlu Sinkronisasi.
5. Pratinjau satu WEB di salinan kerja yang Anda izinkan. Tinjau pemetaan, angka, formula, Note, dan palet warna aktual sebelum Start nyata.
6. Jalankan satu tanggal yang Anda otorisasi, kemudian Semua WEB. Pastikan progress berasal dari backend dan WEB aktif tanpa sumber tercatat gagal. WEB nonaktif dikecualikan.
7. Bandingkan ketujuh modul dengan Google: TOTAL DEPO, TOTAL WD, SALDO BANK, BIAYA, kedua FEE QRIS, KET PENGIRIMAN. Formula TOTAL dan Note manual harus tetap utuh; Note QRIS terlihat melalui Shift+F2.
8. Pastikan Laporan deposit harian tidak berubah: formula, STATUS, KETERANGAN, format dan warna tetap sama. Penulisan berhasil dan KLOP ditampilkan terpisah. BALANCE nonnol menampilkan selisih aktual; Clear harus ditolak.
9. Setelah koreksi laporan/sumber yang sah, gunakan retry hanya pada item gagal atau **Periksa Ulang Rekonsiliasi** untuk membaca saldo tanpa replay tulisan. Konsolidasi menunggu seluruh WEB selesai; Clear memerlukan semua saldo WEB dan konsolidasi tepat nol.
10. Jika URL berubah, gunakan **Crawl Ulang ke URL Baru** dengan tanggal/rentang yang dipilih eksplisit. Jangan mengharapkan hasil lama membuktikan URL baru.

Verifikasi Google tambahan yang hanya membaca, dari folder staging di atas:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec /opt/tutupkas/current/.venv/bin/python ../scripts/verify-real-run.py RUN_ID'
```

Ganti RUN_ID dengan proses tulis nyata terbaru. Perintah tidak mengubah sel dan tidak otomatis Clear. Integrasi Google asli dan deployment VPS belum dijalankan oleh pengembang; smoke test di atas harus dilakukan pada konfigurasi Anda.

## 6. Rollback

Rollback mengembalikan aplikasi, frontend, dan database sebelum upgrade. Data/perubahan setelah backup tidak akan ada pada database aktif hasil rollback; salinan database setelah upgrade juga dibuat oleh helper agar dapat ditinjau. Lakukan sebelum kembali beroperasi jika smoke test gagal. Gunakan folder backup yang dicetak skrip, misalnya:

```bash
sudo bash /tmp/tutupkas-staging-v2/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v2-TANGGAL-ID
```

Helper menghentikan API/worker, menyimpan salinan DB saat ini, memeriksa backup, memulihkan DB dengan izin tutupkas, membuang sidecar SQLite yang tidak lagi cocok saat layanan sudah berhenti, mengembalikan symlink dan frontend, menyalakan layanan, serta mengecek health. Nginx/HTTPS/key tidak berubah. Hard refresh browser dan cek akun serta riwayat lama. Tidak menggunakan `alembic downgrade` karena penghapusan riwayat finansial otomatis sengaja ditolak.

Jika rilis lama telah dihapus di luar skrip, ekstrak `application.tar.gz` ke direktori rilis pemulihan dan sesuaikan `previous-release`; jangan ekstrak menimpa rilis aktif. Jangan menghapus backup atau rilis lama sampai pembaruan telah diterima dan masa retensi yang Anda tentukan selesai.
