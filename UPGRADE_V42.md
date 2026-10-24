# Upgrade V4.1 → V4.2 pada VPS yang sudah berjalan

Gunakan paket lengkap `tutupkas-lion-group-v4.2.zip`. Akses VPS/Google asli belum tersedia bagi pengembang; penerimaan live tetap dilakukan di VPS Anda. Jangan memakai `deploy/install.sh` pada instalasi yang sudah ada.

## Paket dan pemeriksaan awal

Unggah ZIP dan `.zip.sha256` melalui SFTP ke `/tmp/tutupkas-upload/`:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v4.2.zip.sha256
mkdir -p /tmp/tutupkas-staging-v42
unzip tutupkas-lion-group-v4.2.zip -d /tmp/tutupkas-staging-v42
cd /tmp/tutupkas-staging-v42/tutupkas-lion-group
less deploy/upgrade.sh
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh -p ActiveState
sudo bash deploy/upgrade.sh
```

Jika ada penarikan berjalan, gunakan Stop di aplikasi dan tunggu verifikasi tulisan sebelum upgrade. Hindari menghentikan paksa proses yang sedang POST. Tidak ada pekerjaan finansial yang dijalankan oleh skrip upgrade.

Skrip mencetak release terpasang, memeriksa user/unit/environment/key lama, menguji Nginx tanpa mengubah konfigurasinya, menyiapkan dependensi terkunci, menghentikan API dan worker, lalu membuat backup konsisten. Lokasi tetap:

| Komponen | Lokasi |
|---|---|
| Release aktif | `/opt/tutupkas/current/` |
| Database | `/var/lib/tutupkas/tutupkas.sqlite3` |
| Environment | `/etc/tutupkas/app.env` |
| Key Google yang sama | `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json` |
| Backup | `/var/lib/tutupkas/backups/upgrade-v4.2-.../` |

Backup SQLite memakai `sqlite3.Connection.backup`, lalu integrity check; aplikasi lama, frontend lama, environment privat, dan path pemulihan disimpan. Migrasi `b7c31ef05a92 → c92448ce233b` menambahkan **15 tabel** dan indeksnya. Tidak ada tabel lama yang dibangun ulang atau akun, password, URL, aktivasi, alias, hasil, ledger, ownership, maupun audit yang dihapus.

Setelah backup, `prepare-v42-upgrade --services-stopped` memfinalkan antrean lama yang belum selesai sebagai CANCELED secara diaudit. Run terminal tetap menjadi histori; intent PREPARED tetap tersimpan. API dimulai kembali; **worker tetap STOPPED dan dinonaktifkan saat boot** selama penerimaan. Nginx, HTTPS, akun server, app.env dan batas memori tidak diganti.

## Verifikasi API dan registrasi mapping

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl status tutupkas-api --no-pager
sudo systemctl is-active tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/alembic current; ../.venv/bin/alembic check'
```

`is-active` harus menunjukkan `inactive` (exit nonzero memang sesuai). Schema harus `c92448ce233b`; pertahankan `MemoryMax=1536M` yang sudah dikonfigurasi.

1. Masuk dengan akun lama; cocokkan daftar WEB, URL, alias dan histori. Hard refresh halaman.
2. Pada pengaturan sumber, validasi/refresh daftar sheet untuk merekam Google `sheetId`. Sheet aktif lama dipertahankan bila identitasnya sama; sheet baru/rename/recreated tidak otomatis diaktifkan. Tinjau lalu aktifkan yang diperlukan.
3. Pada Kas Kecil bulan Oktober 2026, validasi ulang URL bulanan yang benar. Ini membuat versi registry absolut baru melalui API meskipun worker berhenti. Tunggu READY dan tinjau daftar masalah mapping. READY berarti proses registrasi selesai; entri yang missing/ambiguous tetap ditolak pada modul terkait.
4. Cocokkan batas website/tanggal, kategori rekening, area fee dan KET. Robot tidak merombak workbook untuk menyesuaikan mapping. Jangan mengganti koordinat atau formula sebagai cara melewati error.
5. Indeks warm yang aman memakai metadata `version` Drive dengan Service Account yang sama dan scope `drive.metadata.readonly`. Jika Drive API/izin metadata tidak tersedia, robot memakai pembuktian bounded yang lebih lambat. Key JSON tidak diganti. Uji akses di VPS sebelum mengandalkan angka warm.

Mapping, throttling dan worker memakai direktori lock yang sama dari `TK_WORKER_LOCK`. User layanan API/worker yang sama harus mempunyai akses tulis pada direktori itu; file lock/kuota dibuat privat. Nilai default pacing tetap 1,2 detik lintas proses, bukan per instance.

## Penerimaan pada salinan aman

Worker belum berjalan: pastikan antrean lama sudah CANCELED dan tidak ada Run finansial yang akan dilanjutkan tanpa tinjauan. Pilih URL salinan kerja yang memang diotorisasi, lalu antrekan **Preview VELBETT 18 Oktober 2026**. Setelah itu:

```bash
sudo systemctl start tutupkas-worker
sudo journalctl -u tutupkas-worker -n 100 --no-pager
```

Preview harus tanpa POST finansial/ledger tulis; periksa sumber aktual, carry-forward, tanggal berikutnya, fixed columns, fee dan calon slot/Notes. Mapping bukan dry-run transaksi dan tidak menulis Google. Catatan waktu ada pada `config_snapshot.performance_attempts`, audit `run_performance_measured`, serta `worker-performance.log` di direktori lock; metrik mapping ada pada evidence versi registry.

Untuk mode tulis, gunakan salinan tujuan yang diotorisasi dan tinjau hasil. Jika data lama berbeda, pilih `Masukkan Data Baru` atau `Tetapkan Data Lama` untuk seluruh kelompok. Formula/merge/Note manual tetap dilindungi. Baca ulang kelompok bila stale. KEEP tidak memaksa BALANCE nol. Retry satu item membaca sumber terkini dan tidak mengulang sukses lainnya; perubahan BIAYA input dapat memerlukan retry total secara terpisah. Jika nama sumber berubah tanpa bukti, pilih sumber aktif secara eksplisit pada modal retry.

Histori V4.1 tanpa identitas tab tidak diadopsi berdasarkan nama saja. Bila error `LEGACY_LINEAGE_UNVERIFIED`, cocokkan workbook, histori, dan Google sheetId lalu konfirmasi hubungan lewat API admin `POST /api/source-identities/{id}/confirm-legacy` dengan `prior_run_item_id` dan `reviewed_history_and_google_identity:true`; gunakan sesi/CSRF admin. API menolak hubungan yang bertentangan dengan identitas yang sudah terbukti. Jangan menghapus ownership agar transaksi dapat lewat.

Setelah Run tulis SUCCESS:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/python ../scripts/verify-real-run.py RUN_ID'
```

Ganti `RUN_ID` dengan ID nyata. Verifikator hanya GET dan memeriksa semua sel/Notes dan laporan, bukan menulis atau otomatis Clear. Untuk keputusan KEEP, gunakan bukti konflik dan BALANCE aktual aplikasi; verifikator ini khusus ledger VERIFIED yang semuanya cocok dengan intent tulis.

Setelah penerimaan selesai dan antrean sudah ditinjau, aktifkan saat boot secara eksplisit:

```bash
sudo systemctl enable tutupkas-worker
```

Tidak ada klaim Preview live <60 detik. Pada fixture besar, steady warm Preview memakai 49 GET termasuk metadata, sehingga pacing saja sudah sekitar 57,6 detik; latensi menambah waktu. Cold, perubahan isi, dan fallback tanpa Drive memerlukan pembacaan lebih banyak. Lihat `PERFORMANCE_V42.md`.

## Rollback

Hentikan proses melalui Stop dan tunggu batas aman. Pulihkan **database backup beserta release/frontend lama**, bukan downgrade yang menghapus tabel finansial baru:

```bash
sudo bash /tmp/tutupkas-staging-v42/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v4.2-TIMESTAMP-PID
```

Gunakan folder backup tepat yang dicetak upgrade. Skrip menghentikan layanan, membackup database setelah upgrade, memulihkan SQLite konsisten dan symlink/frontend lama, menahan antrean lama lalu memulai API saja. Database setelah upgrade tetap tersimpan. Perubahan Google setelah backup **tidak dibatalkan oleh rollback database**; cocokkan sel aktual dan intent sebelum mengizinkan worker berjalan lagi. Backup schema V4.1 tidak boleh dijalankan dengan aplikasi V4.2 atau sebaliknya.

Downgrade Alembic ke V4.1 hanya diizinkan bila semua tabel V4.2 kosong. Bila ada snapshot, mapping, retry, keputusan atau audit V4.2, downgrade otomatis ditolak untuk menjaga histori; gunakan pemulihan backup. Simpan kedua database untuk pemeriksaan keuangan.
