# Upgrade aman V4.2 → V4.2.1

Patch ini mempertahankan schema `c92448ce233b`, tabel sumber/snapshot/mapping, identitas, akun/password, ledger/ownership, tulisan terdahulu dan histori Run. **Tidak perlu refresh sumber atau membangun ulang mapping READY untuk memasang patch.** Jangan menjalankan `deploy/install.sh` pada VPS yang sudah berjalan. Akses VPS/Google asli tidak tersedia di lingkungan pengembangan; langkah systemd/HTTP/Google tetap harus diverifikasi di VPS Anda.

## Jalankan dari ZIP lengkap

Unduh `tutupkas-lion-group-v4.2.1.zip` dan checksum pasangannya dari GitHub, unggah melalui SFTP, lalu:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v4.2.1.zip.sha256
mkdir -p /tmp/tutupkas-staging-v421
unzip tutupkas-lion-group-v4.2.1.zip -d /tmp/tutupkas-staging-v421
cd /tmp/tutupkas-staging-v421/tutupkas-lion-group
less deploy/upgrade.sh
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh -p ActiveState
sudo bash deploy/upgrade.sh
```

Bila worker sedang berjalan, gunakan Stop aplikasi dan tunggu batas verifikasi aman terlebih dahulu bila memungkinkan; jangan kill -9 saat HTTP POST. Skrip juga meminta cancel melalui protokol database yang didukung worker V4.2 sebelum `systemctl stop`. Tidak ada financial replay otomatis oleh upgrade.

## Urutan skrip dan preservasi

1. Memeriksa instalasi, unit service user/working directory/environment, akses key lama, Nginx dan schema V4.2. Menyiapkan release baru dan dependensi dengan hash lock sebelum menghentikan layanan. Tidak menyalin unit systemd, environment, Nginx, sertifikat atau key.
2. Menghentikan API agar tidak ada job baru, membuat consistent backup `pre-stop.sqlite3`, lalu mengajukan cancel untuk job aktif melalui `request_*_cancel`, date lock dan audit maintenance. Worker lama menyelesaikan batas aman penulisan/verifikasi yang sudah dikirim.
3. Menghentikan dan men-disable worker saat boot; membuat consistent stopped backup `database.sqlite3` beserta aplikasi/frontend/environment lama. Backup stopped adalah pasangan rollback, agar hasil write sah yang selesai saat shutdown tetap tercatat. `pre-stop.sqlite3` dipertahankan untuk investigasi, bukan pengganti rollback otomatis.
4. `alembic current/upgrade head/check` tetap `c92448ce233b`; tidak ada migrasi baru. CLI lama `prepare-v42-upgrade --services-stopped` dipakai kembali untuk menahan antrean tersisa secara diaudit. Run/batch aktif menjadi CANCELED, item pending/retry aktif ditahan, keputusan konflik pending dapat dibuka untuk tinjauan. Run terminal/sukses tetap histori; PREPARED/VERIFIED ledger, snapshots, mapping dan financial results tidak dihapus/diakui ulang.
5. Verifikator read-only `scripts/verify-v421-upgrade.py BACKUP CURRENT` membandingkan integrity/FK, schema dan baris seluruh tabel terhadap stopped backup. Hanya perubahan field cancellation yang ditentukan untuk antrean aktif serta tambahan audit/reconciliation yang diizinkan. Protected rows dan seluruh histori lama wajib sama; kerusakan ledger menghentikan upgrade. Verifikator tidak menghubungi Google atau mengungkap isi/key.
6. Memindahkan symlink release dan frontend HTML setelah aset tersedia, memulai **API saja**, mengecek health, dan mencetak lokasi backup. **Worker tetap STOPPED dan disabled saat boot** sampai pemeriksaan manual selesai.

| Komponen dipertahankan | Lokasi |
|---|---|
| Active release | `/opt/tutupkas/current` |
| SQLite produksi | `/var/lib/tutupkas/tutupkas.sqlite3` (atau path lama dari app.env dalam direktori yang sama) |
| Environment | `/etc/tutupkas/app.env` |
| Key yang sama | `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json` |
| Backup baru | `/var/lib/tutupkas/backups/upgrade-v4.2.1-TIMESTAMP-PID/` |

MemoryMax/MemoryHigh, default cap respons 8 MiB, quota pacing, SSL/HTTPS dan akun server tidak diganti. Jika override VPS sudah `MemoryMax=1536M`, nilainya tetap. Skrip tidak menaikkannya. Backup environment bersifat privat.

## Verifikasi manual sebelum worker

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl status tutupkas-api --no-pager
sudo systemctl is-active tutupkas-worker
sudo systemctl is-enabled tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/alembic current; ../.venv/bin/alembic check'
```

Worker `inactive` dan `disabled` memang mengembalikan exit nonzero. Schema harus sama. Login memakai akun lama, cocokkan source settings/aktivasi/alias, periode, mapping READY beserta ID/revision/target count, Run history, ledger dan hasil lama. Jangan validasi ulang URL bulanan hanya untuk patch karena itu membuat revision mapping baru dan mengubah otorisasi Run lama.

Untuk Run #9 yang dilaporkan: jangan membuat ulang 8 snapshot/43 rencana atau menjalankan ulang seluruh sukses. Tinjau status tiap item, intent PREPARED dan isi sel Google saat ini. Jika hanya `expense_total` yang tertinggal dan input BIAYA sukses tetap valid, gunakan **retry item tersebut saja** pada mapping terpin yang sama. Tidak ada tombol replay otomatis dalam upgrade. Jika source settings atau workbook berubah, ikuti error/konflik dan otorisasi eksplisit; jangan menghapus ledger/mapping untuk meloloskannya.

Pada salinan tujuan yang diotorisasi, antrekan Preview yang telah ditinjau. Setelah memeriksa tidak ada job finansial lain yang akan berjalan tanpa tinjauan, mulai worker manual:

```bash
sudo systemctl start tutupkas-worker
sudo journalctl -u tutupkas-worker -n 150 --no-pager
```

Preview harus nol financial POST. Uji satu Run tulis terotorisasi; bandingkan sel/Note/fee/kiriman dan report aktual. Log baru mengklasifikasikan GET berdasarkan `read_kind` dan phase; lihat [ROOT_CAUSE_V421.md](ROOT_CAUSE_V421.md). Setelah Run SUCCESS, verifikator lama hanya GET:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; ../.venv/bin/python ../scripts/verify-real-run.py RUN_ID'
```

Ganti RUN_ID. KEEP memakai bukti konflik dan BALANCE aktual; KEEP tidak memaksa KLOP. Jika penerimaan selesai dan antrean sudah ditinjau, boleh aktifkan kembali boot dengan `sudo systemctl enable tutupkas-worker`. Target live <60 detik masih pending; pacing dan global guards dipertahankan.

## Rollback

Gunakan Stop dan tunggu batas aman, lalu jalankan dari paket V4.2.1 dengan folder backup tepat yang dicetak:

```bash
sudo bash /tmp/tutupkas-staging-v421/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v4.2.1-TIMESTAMP-PID
```

Skrip menghentikan API/worker, menyimpan database setelah upgrade sebagai `post-upgrade-*.sqlite3`, memulihkan **stopped database.sqlite3 dan release/frontend lama bersama**, menahan antrean restored, lalu memulai API saja. Worker tetap berhenti. app.env/key/Nginx tidak diganti; kedua database dan backup historis disimpan. Jangan menjalankan downgrade Alembic; schema patch sama.

Rollback SQLite **tidak membatalkan tulisan Google** yang terjadi setelah backup. Cocokkan target aktual, intent dan ownership sebelum retry/start worker. Bila upgrade terhenti sebelum symlink berpindah, API/worker bisa masih berhenti; baca lokasi backup/log dan pulihkan secara terarah, jangan menyalakan worker hanya agar service terlihat aktif. Skrip deployment diverifikasi sintaks dan fungsi preservasi database di fixture; eksekusi root/systemd/Nginx pada VPS asli belum diuji di sini.
