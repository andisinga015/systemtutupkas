# TUTUP KAS · LION GROUP V4.2.1

Patch destination writer untuk snapshot/rencana dan monthly mapping READY yang sudah ada. Bounded shared validation/readback dan atomic batchUpdate mengurangi GET/POST tanpa mengubah koordinat, angka, deskripsi, Notes, konflik atau ownership V4.2. Fixture 43 item: **660 → 77 GET tujuan, 36 → 1 POST**, output 57 sel persis sama dengan V4.2 asli.

## Upgrade VPS yang sudah ada

Unduh ZIP lengkap beserta SHA256, unggah ke staging lalu ikuti [UPGRADE_V421.md](UPGRADE_V421.md). Script `deploy/upgrade.sh` mempertahankan schema `c92448ce233b`, membuat backup konsisten, menahan antrean lama, memverifikasi protected rows dan memulai **API saja**. Worker tetap berhenti/disabled sampai pemeriksaan manual. Akun, database, history, READY mappings, ledgers, app.env, key Google, Nginx/HTTPS dan MemoryMax dipertahankan. Jangan memakai install.sh pada instalasi lama.

```bash
sha256sum -c tutupkas-lion-group-v4.2.1.zip.sha256
unzip tutupkas-lion-group-v4.2.1.zip -d /tmp/tutupkas-staging-v421
cd /tmp/tutupkas-staging-v421/tutupkas-lion-group
sudo bash deploy/upgrade.sh
```

Untuk patch ini **tidak perlu registrasi ulang sumber atau rebuild mapping READY**. Tinjau data lama dan intent PREPARED sebelum menjalankan worker. Jika hanya expense_total tertinggal, otorisasi retry item itu saja. Sel lama berbeda tetap memerlukan keputusan admin. Rollback memulihkan release dan database backup bersama; tulisan Google tidak dibatalkan oleh rollback SQLite.

## Operasi

- Tab baru/rename perlu ditinjau dan diaktifkan; identitas recreated tidak diadopsi berdasarkan nama.
- Registry dibangun saat URL disimpan/divalidasi. Crawl/retry memakai koordinat yang dipin, tanpa remapping otomatis.
- Retry item gagal membaca raw source terkini; perubahan BIAYA input dapat memerlukan retry total terpisah.
- Konflik menampilkan nilai Google aktual dan hasil sumber. Formula/merge/Note manual tetap dilindungi; KEEP tidak memaksa KLOP.
- Preview dan Laporan deposit harian hanya baca. Selesai & Clear memerlukan seluruh WEB dan BALANCE konsolidasi sesuai aturan lama.

## Pengembangan dan verifikasi

Python 3.10+, Node 20+ dan dependensi yang terkunci:

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

ZIP menyertakan source, migrasi, test, dokumentasi dan frontend prebuilt `public/`. Key nyata, password, database, `.env`, cache, virtualenv dan node_modules dikecualikan. `scripts/verify-real-run.py RUN_ID` memverifikasi tulisan nyata hanya dengan GET dan registry yang dipin.

## Laporan rilis

- [Root cause dan jalur GET mahal](ROOT_CAUSE_V421.md)
- [Proof invariant dan fallback](INVARIANTS_V421.md)
- [Laporan pengujian](TEST_REPORT.md)
- [Benchmark sebelum/sesudah dan batas live](PERFORMANCE_V421.md)
- [Perubahan per file](CHANGED_FILES_V421.md)
- [Upgrade/rollback](UPGRADE_V421.md)
- [Catatan rilis](RELEASE_NOTES_V421.md)

Validasi Google/VPS asli dan target <60 detik masih pending. Pacing global default tetap 1,2 detik; 78 request tujuan saja memerlukan sekitar 92,4 detik jarak start sebelum latensi/sumber. Tanpa token, pada geometry sparse atau proof global mahal, fallback penuh tetap dipakai demi keselamatan. Source indexing/snapshots/registration/auth/UI/concurrency dan cap resource tidak berubah. Laporan V4.2/V4.1/V4/V3 dipertahankan di docs.
