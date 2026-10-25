# TUTUP KAS · LION GROUP V4.2.2

Tiga dokumen sumber dan pengaturan sheet terpisah untuk setiap WEB/bulan:

| Dokumen | Tanggal |
|---|---|
| DOC A — DEPO/WD | 1–15 |
| DOC B — DEPO/WD | 16–akhir bulan |
| DOC QRIS | Seluruh bulan |

Run memilih satu DEPO yang sesuai tanggal sebelum membuat snapshot/item, ditambah QRIS bulanan bila berlaku. Tab bernama sama di A/B tetap berbeda identitas. Aktivasi/nonaktivasi, refresh, URL dan revision terisolasi per dokumen. Sheet baru tetap nonaktif; missing periode tidak memakai sumber lain sebagai fallback.

## Upgrade VPS yang sudah ada

Mendukung **V4.2 atau V4.2.1 → V4.2.2**. Ikuti [UPGRADE_V422.md](UPGRADE_V422.md), jangan gunakan install.sh pada instalasi lama.

```bash
sha256sum -c tutupkas-lion-group-v4.2.2.zip.sha256
unzip tutupkas-lion-group-v4.2.2.zip -d /tmp/tutupkas-staging-v422
cd /tmp/tutupkas-staging-v422/tutupkas-lion-group
sudo bash deploy/upgrade.sh
```

Skrip mendeteksi release/schema nyata, membuat backup konsisten dan menahan antrean secara diaudit, bermigrasi `c92448ce233b → d422a61e908f`, lalu memverifikasi protected rows dan memulai API saja. **Worker tetap STOPPED/disabled** untuk pemeriksaan manual. Akun/password, SourceDocument IDs/revisions/tab states, snapshots/indexes/history, ledgers/ownership, READY mappings, app.env/key Google, Nginx/HTTPS dan RAM settings dipertahankan.

DEPO lama berstatus **belum ditetapkan**: buka Admin pada bulan yang benar dan pilih **Tetapkan sebagai DOC A** atau **DOC B** secara eksplisit. Record dan status sheet lama mengikuti penetapan itu tanpa cloning. QRIS lama menjadi bulanan. Tidak ada penebakan dari nama file/tanggal Run lama, tidak ada pengaktifan otomatis, dan tidak ada Crawl otomatis.

Tinjau ketiga pilihan pada menu **Daftar Sheet**. Jika hanya satu item gagal, retry item itu saja dari periode/doc ID yang benar. Mapping Kas Kecil tidak perlu direbuild karena perubahan periode sumber. Histori lama tetap dapat dibaca; konteks sumber yang belum konsisten tidak dianggap current secara otomatis. Rollback harus memulihkan release dan database backup bersama; tulisan Google tidak dibatalkan oleh rollback SQLite.

## Finansial dan performa

Seluruh engine finansial dan destination writer/reader/registry V4.2.1 tetap byte-identik. Fixed columns, fee, BIAYA, KET, Notes/formula/merge/manual guards, explicit NEW/KEEP, ownership/idempotent recovery, read-only report dan true KLOP dipertahankan. Tidak ada concurrency, pacing, cap API atau MemoryMax yang dinaikkan.

Fixture 43 item/8 snapshot/57 sel dengan **10 tab DOC A aktif yang tidak dipilih** tetap memakai **77 destination GET, 1 POST, 117.986 byte**, output persis sama dengan V4.2.1 dan tidak membaca sumber A. Waktu fixture bukan waktu Google; penerimaan live/VPS dan target <60 detik belum terverifikasi. Pacing default tetap 1,2 detik.

## Pengembangan

Python 3.10+, Node 20+, dependency locks yang sama:

```bash
bash scripts/dev-install.sh
.venv/bin/python -m pytest backend/tests -q
cd frontend
npm ci
npm run build
cd ..
PLAYWRIGHT_CHROMIUM_EXECUTABLE=/usr/bin/chromium bash scripts/e2e.sh
python3 scripts/package.py
```

ZIP berisi source, migrasi, seluruh test/docs dan frontend prebuilt `public/`. Database/runtime state, key nyata, .env, caches, virtualenv dan node_modules dikecualikan.

## Bukti rilis

- [Architecture/routing/identity](ARCHITECTURE_V422.md)
- [Cakupan 79 persyaratan](COVERAGE_V422.md)
- [Hasil pengujian dan batas live](TEST_REPORT.md)
- [Benchmark V4.2.1 vs V4.2.2](PERFORMANCE_V422.md)
- [Upgrade dan rollback](UPGRADE_V422.md)
- [Perubahan per file](CHANGED_FILES_V422.md)
- [Baseline](BASELINE_V422.md) dan [catatan rilis](RELEASE_NOTES_V422.md)

Laporan dan arsip rilis sebelumnya tetap dipertahankan. Pengujian development menggunakan fixture Google terisolasi; tidak ada klaim deployment atau financial write produksi sudah dilakukan.
