# Upgrade V3 → V4 pada VPS yang sudah berjalan

V4 memperbaiki pembacaan dan batas memori backend. UI, aturan keuangan, pemetaan tujuan, akun, URL, key Google, Nginx, HTTPS, dan override `MemoryMax=1536M` yang ada dipertahankan. Jangan gunakan `deploy/install.sh` untuk upgrade ini. Worker tetap satu proses. Footer UI tetap v3.0 karena frontend sengaja tidak diubah.

Pengembang belum mengakses VPS/key/workbook asli. Uji live berikut **masih menunggu operator**, diawali PREVIEW baca saja. Tidak ada klaim OOM produksi sudah pasti selesai.

## 1. Unggah dan periksa paket

Unduh ZIP V4 dan checksum dari GitHub. Unggah lewat FileZilla/SFTP ke `/tmp/tutupkas-upload/`, lalu:

```bash
cd /tmp/tutupkas-upload
sha256sum -c tutupkas-lion-group-v4.zip.sha256
mkdir -p /tmp/tutupkas-staging-v4
unzip tutupkas-lion-group-v4.zip -d /tmp/tutupkas-staging-v4
cd /tmp/tutupkas-staging-v4/tutupkas-lion-group
less deploy/upgrade.sh
sudo systemctl is-active tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryMax -p MemoryHigh -p User -p WorkingDirectory
sudo systemctl cat tutupkas-worker
```

Worker setelah insiden seharusnya `inactive`/`failed`. Jangan menjalankan worker V3 untuk mencoba lagi. Jika masih aktif, gunakan Stop dari aplikasi, lalu `sudo systemctl stop tutupkas-worker`; tunggu penghentian aman. Jangan menggunakan kill -9. Tidak perlu mengubah MemoryMax, swap, OOM killer, atau key.

## 2. Backup, tahan antrean lama, dan upgrade

```bash
sudo bash deploy/upgrade.sh
```

Skrip menyiapkan `/opt/tutupkas/releases/v4-...` beserta dependensi terkunci, menghentikan API/worker, dan menonaktifkan auto-start worker saat boot selama penerimaan. Backup disimpan di `/var/lib/tutupkas/backups/upgrade-v4-...` (0700): database melalui SQLite backup API dengan integrity check, aplikasi lama, frontend lama, environment privat, dan path pemulihan. Cadangkan folder ini di lokasi privat sesuai kebijakan Anda.

Setelah backup, Alembic memeriksa head yang sama dengan V3 (`b7c31ef05a92`); V4 tidak menambah/menghapus tabel atau kolom. Langkah `prepare-v4-upgrade --services-stopped` memfinalkan pekerjaan lama yang masih QUEUED/RUNNING/CANCEL_REQUESTED sebagai CANCELED dan mencatat audit. **Run #5 tidak otomatis aktif kembali.** Cancellation lama, hasil sukses, ledger PREPARED/VERIFIED, ownership, akun, dan histori tetap disimpan. Ini adalah perubahan status pemeliharaan yang disengaja, bukan penghapusan histori.

Skrip mengganti symlink rilis, memublikasikan aset frontend V3 yang sama, lalu memulai **API saja**. Key `/home/aksesfinancebot/spreadsheet-robot/credentials/robot-key.json`, `/etc/tutupkas/app.env`, konfigurasi systemd/override memori, Nginx dan sertifikat tidak ditulis ulang. Default pembacaan V4 berlaku tanpa mengedit app.env.

```bash
curl --fail http://127.0.0.1:8000/api/health
sudo systemctl status tutupkas-api --no-pager
sudo systemctl is-active tutupkas-worker
readlink -f /opt/tutupkas/current
```

API harus sehat; worker harus tetap berhenti. Login memakai akun lama dan periksa WEB, URL bulanan, sheet aktif/nonaktif, histori, serta Run #5. Jika skrip gagal, jangan memulai worker; simpan pesan error dan gunakan backup/rollback.

## 3. Tinjau penulisan yang mungkin terputus

Tinjauan ini hanya membaca DB. Ganti path DB bila konfigurasi lama memakai nama berbeda:

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec ../.venv/bin/python -' <<'PY'
from sqlalchemy import select, func
from app.db import SessionLocal
from app.models import Run, RunItem, WriteLedger
with SessionLocal() as db:
    run=db.get(Run,5)
    print('Run 5:',run.status if run else 'tidak ditemukan')
    for status,count in db.execute(select(WriteLedger.verification_status,func.count()).group_by(WriteLedger.verification_status)):
        print('Ledger:',status,count)
    for row in db.scalars(select(WriteLedger).join(RunItem,WriteLedger.run_item_id==RunItem.id).where(RunItem.run_id==5)):
        print(row.destination_tab,row.target_range,row.verification_status)
PY
```

PREPARED berarti hasil tulis belum pasti. Bandingkan sel/Note/formula tujuan dengan intent tersimpan sebelum mengizinkan retry. Jangan menghapus ledger atau mengubah DB menjadi SUCCESS/QUEUED secara manual. Retry yang diotorisasi lewat aplikasi tetap membaca tujuan: nilai harus sesuai intent lama atau nilai sebelum tulis yang dimiliki; perubahan manual dan sumber berubah pada uncertain write tetap konflik. Hasil VERIFIED tidak dihapus atau diulang sebagai item sukses.

## 4. Uji sumber referensi secara baca saja, worker tetap berhenti

```bash
sudo -u tutupkas bash -c 'set -a; source /etc/tutupkas/app.env; set +a; cd /opt/tutupkas/current/backend; exec ../.venv/bin/python ../scripts/preview-source-v4.py 5'
```

Probe tidak menjalankan POST Google atau commit DB. Ia memakai snapshot Run #5 untuk membaca VELBETT, tanggal `2026-10-18`, tab `WD BCA 1 TOKEN / IVAN RAKHMANDA ( PP )`. Target: TOTAL DEPO **0**, TOTAL WD **0**, SALDO **2.355.000**, `expected_match=true`, tanpa full source grid. Hasil aktual, RSS, peak RSS, durasi dan ukuran respons tampil hanya di terminal/log VPS. Ini pemeriksaan sumber, belum membuktikan seluruh website.

Jika run referensi mempunyai ID lain, gunakan ID yang sesuai. Bila sumber aktual sudah berubah atau probe gagal, hentikan penerimaan dan tinjau sumber; jangan memaksa angka target.

## 5. PREVIEW penuh, kemudian start worker secara manual

Pastikan tidak ada antrean finansial tulis lain yang belum ditinjau. Dalam UI lama, pilih tanggal individual **18 Oktober 2026** pada VELBETT dan buat **Pratinjau**. Pastikan run baru `preview=true`. Jangan menekan Mulai/Crawl Ulang tulis pada tahap ini.

```bash
sudo systemctl start tutupkas-worker
sudo systemctl show tutupkas-worker -p MemoryCurrent -p MemoryPeak -p MemoryMax -p NRestarts
sudo journalctl -u tutupkas-worker -f -n 50
```

Di terminal terpisah, pengamatan berkala:

```bash
watch -n 2 'systemctl show tutupkas-worker -p MemoryCurrent -p MemoryPeak -p MemoryMax -p NRestarts'
```

`MemoryPeak` mungkin tidak tersedia pada systemd lama; gunakan `peak_rss_mb` dari log. Log JSON juga disimpan di `/var/lib/tutupkas/worker-performance.log`, rotasi 5 MiB × 4 berkas (maksimum sekitar 20 MiB), dengan izin privat. Log tidak berisi body transaksi, header autentikasi, token, atau key. Untuk journal gunakan `journalctl --disk-usage` dan kebijakan rotasi journald VPS yang ada; skrip tidak mengganti konfigurasi journal global.

PREVIEW penuh harus selesai tanpa POST, tanpa ledger finansial baru, dengan rencana nilai/sel/Note yang benar untuk seluruh tab aktif. Pastikan sumber referensi memberikan 0/0/2.355.000; fee, biaya dan kiriman tab lain juga diperiksa. `Laporan deposit harian` tetap baca saja. Jika ada SOURCE_RESOURCE_LIMIT, SHEET_TOO_LARGE atau RESPONSE_TOO_LARGE, transaksi tidak dipotong; simpan diagnostik, tinjau ukuran sumber/tujuan, lalu hentikan worker. Jangan menaikkan MemoryMax atau menghapus data keuangan sebagai solusi.

**Sesudah PREVIEW diperiksa dan operator secara eksplisit mengizinkan penulisan**, gunakan Ulangi Item Tersisa pada run terputus yang konfigurasinya masih cocok, atau alur sinkronisasi yang sudah ada jika URL/revisi berubah. Retry merupakan otorisasi baru; item sukses, ownership dan proteksi uncertain write tetap berlaku. Jangan mengganti URL untuk menghindari konflik kepemilikan.

Untuk penulisan yang sudah benar-benar selesai, skrip lama `scripts/verify-real-run.py RUN_ID` tetap dapat dipakai baca saja. Verifikasi saldo WEB/konsolidasi dan Clear tetap melalui aturan aplikasi yang sama.

Setelah penerimaan operasional selesai, pilih secara sadar apakah worker boleh aktif saat boot:

```bash
sudo systemctl enable tutupkas-worker
```

Pada startup berikutnya, V4 menandai job RUNNING terputus sebagai FAILED/PARTIAL (`WORKER_INTERRUPTED`), bukan mengantrekan ulang otomatis. CANCEL_REQUESTED tetap diselesaikan CANCELED. Antrean baru yang sengaja dibuat admin tetap berjalan berurutan; tidak ada worker pool.

## 6. Batas pembacaan

| Pengaturan | Default |
|---|---:|
| `TK_SOURCE_CHUNK_ROWS` | 200 (1–250) |
| `TK_SOURCE_SCAN_COLUMNS` | 16 (1–32 rentang kolom sempit per batch) |
| `TK_MAX_SOURCE_SCAN_CELLS` | 2.000.000 sel area metadata |
| `TK_MAX_SOURCE_DAY_CELLS` | 100.000 sel konteks satu hari |
| `TK_MAX_DESTINATION_CELLS` | 250.000 sel per snapshot tujuan/laporan |
| `TK_GRID_CHUNK_CELLS` | 8.000 sel per permintaan tujuan |
| `TK_MAX_API_RESPONSE_BYTES` | 8 MiB, dihitung saat streaming setelah dekompresi |

Penelusuran global tanggal menggunakan values-only, tanpa menyimpan histori. Seluruh area ditelusuri secara bertahap untuk mempertahankan penolakan tanggal duplikat di mana pun; bukan server-side search. Tidak ada cache posisi lintas run yang bisa menjadi stale. Jeda Google 1,2 detik dan retry maksimal 4 tetap berlaku. Banyaknya chunk dapat membuat workbook besar memerlukan beberapa menit; benchmark lokal tidak memasukkan throttle/latensi Google. Lihat [PERFORMANCE_V4.md](PERFORMANCE_V4.md).

## 7. Rollback

Rollback mengembalikan DB sebelum upgrade; perubahan setelah backup tidak masuk DB aktif hasil pemulihan. Skrip menyimpan DB sesudah upgrade secara terpisah agar tidak hilang. Tulisan Google yang sudah terjadi tidak dibatalkan oleh rollback DB; tinjau intent dan sel tujuan sebelum kembali beroperasi.

```bash
sudo bash /tmp/tutupkas-staging-v4/tutupkas-lion-group/deploy/rollback.sh /var/lib/tutupkas/backups/upgrade-v4-TANGGAL-ID
```

API lama kembali; worker tetap dihentikan dan antrean snapshot lama ditahan oleh helper offline. Jangan memulai worker V3 yang sebelumnya OOM; lakukan investigasi/pemulihan terkontrol. Konfigurasi Nginx/HTTPS/key tetap dipertahankan. Backup memuat environment privat; jangan unggah folder backup ke GitHub.
