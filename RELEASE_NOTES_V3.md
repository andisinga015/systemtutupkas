# V3 — Perbaikan korektif V2

## Perilaku yang diperbaiki

Header sumber ganda/hilang tidak lagi memblokir metrik yang tata letaknya sudah dikonfirmasi. DEPO memakai J/K/L, H biaya dan I kiriman; QRIS memakai I/J/K, G biaya dan H fee/kiriman. Hari kosong yang terverifikasi menghasilkan total nol dengan saldo valid terakhir. Nilai rusak atau masalah tanggal tetap error. Pemetaan tujuan, aturan fee/kiriman, palet warna dan formula rekonsiliasi tetap dipertahankan.

Tanggal massal dan tiap WEB dipisahkan. Preview/Start individual membawa tanggal barisnya; batch membawa tanggal massal. Status/BALANCE mengambil record tanggal yang dipilih, dan respons usang dibuang.

Antrean dapat dibatalkan; Stop running memakai permintaan persisten dan batas aman. Tulisan yang sudah dikirim tetap diupayakan verifikasinya. Hasil sukses dan intent belum pasti disimpan, pembatalan tidak menjadi KLOP/Clear dan tidak auto-resume. Retry melanjutkan item gagal/dibatalkan tanpa replay hasil berhasil. Unexpected worker exception difinalkan dan dicatat.

## Berkas utama

| Berkas | Koreksi |
|---|---|
| `backend/app/engine/parser.py` | Tata letak sumber tetap, deteksi payload harian dan carry-forward tanpa header lookup |
| `backend/app/engine/runner.py` | Kolom/metrik/source_cell konsisten, cache source/error, pemeriksaan cancel dan fase diagnostik |
| `backend/app/engine/writer.py` | Cancel sebelum submit; setelah submit tetap write/read-back aman dan intent dipertahankan |
| `backend/app/sheets.py` | Retry GET dapat dihentikan; retry POST tetap bounded |
| `backend/app/job_control.py` | Claim, cancel, finalisasi, pembacaan status DB segar dan pencatatan fase |
| `backend/app/worker.py`, `batches.py` | Pemulihan cancellation, finalisasi error, batch stop/retry dan progress |
| `backend/app/operations.py`, `workflow.py` | Endpoint cancel, konflik CANCEL_REQUESTED, retry item tersisa |
| `backend/app/reconciliation.py` | Pemeriksaan dapat dibatalkan; run tetap aktif sampai tahap saldo selesai; aturan nol tidak berubah |
| `backend/app/models.py`, migrasi `b7c31ef05a92` | Kolom nullable cancellation/fase dan indeks aktif yang sesuai, tanpa rebuild tabel finansial |
| `backend/app/cli.py` | Pemeliharaan antrean V2 sebelum worker V3 dimulai |
| `frontend/src/Dashboard.tsx`, `api.ts` | Tanggal terpisah, status per tanggal, cancel/stop/retry, label fase Indonesia |
| `frontend/src/help.ts`, `App.tsx`, `styles.css` | Panduan pembatalan/tanggal/kolom sumber, versi 3, gaya kontrol yang mengikuti UI lama |
| `deploy/upgrade.sh`, `rollback.sh`, `scripts/cancel-legacy-queue.py` | Backup, penahanan antrean lama, rollback tanpa menghidupkan job salah tanggal |

## Verifikasi dan batas

137 tes backend lulus pada Python 3.10.22 dan 3.12.14; 5 tes Chromium lulus, termasuk tanggal terpisah dan cancel lewat UI/API. Contoh 18 Oktober diuji dengan closing row 309: DEPO 0, WD 0, SALDO 2.355.000. Semua jalur fee/kiriman diuji terhadap fixture tanpa header unik; tidak ada klaim live Google berhasil.

Pengukuran fixture: 3 pengambilan grid sumber untuk 17 item, tanpa refetch per metrik. V2 sudah memiliki cache tab; belum ada benchmark kecepatan live V3 terhadap V2. Integrasi Google Sheets, workbook, warna dan deployment VPS sebenarnya belum diuji dari lingkungan pengembang. Lihat TEST_REPORT.md.

## Upgrade V2

Unggah ZIP melalui SFTP, ekstrak ke staging terpisah, baca UPGRADE_V3.md, lalu jalankan `sudo bash deploy/upgrade.sh`. Skrip membackup DB/aplikasi/frontend, menjalankan migrasi aditif, memfinalkan antrean V2 belum selesai menjadi Dibatalkan sebelum worker baru dinyalakan, serta memperbarui aset hash. Akun, konfigurasi, URL, history, hasil dan ownership dipertahankan. Nginx/HTTPS/key/env tidak ditimpa.

Tinjau hasil V2 yang sudah terlanjur ditulis; upgrade tidak menghapus atau menulis ulang hasil lama secara otomatis. Pilih retry/Start yang benar secara eksplisit. Rollback memfinalkan antrean snapshot lama dan meninggalkan worker berhenti sampai tinjauan operator selesai. Panduan lengkap: UPGRADE_V3.md.
