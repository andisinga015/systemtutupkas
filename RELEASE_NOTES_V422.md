# V4.2.2 — 25 Oktober 2026

Admin sekarang mempunyai tiga URL dan tiga pilihan sheet aktif/nonaktif terpisah: DOC A untuk 1–15, DOC B untuk 16–akhir bulan, dan QRIS bulanan. Dokumen/revision aktual menjadi batas setiap fetch dan activation. Respons lama tidak dapat mengganti daftar dokumen yang sedang dipilih; nama tab sama antar A/B tetap berbeda identitas.

Run memilih sumber berdasarkan tanggal sebelum snapshot/item dibuat. Retry satu item tetap memakai record sumber yang sama pada periode yang benar. Perubahan periode lain tidak membuat Run stale. Mapping destination dan writer/perhitungan V4.2.1 tetap dipertahankan byte-identik.

Migration `c92448ce233b → d422a61e908f` mempertahankan ID/FK/history. DEPO lama menjadi UNASSIGNED dan harus ditetapkan Admin secara eksplisit ke A/B untuk digunakan; QRIS lama menjadi MONTHLY. Tidak ada cloning tab atau pengaktifan otomatis.

Upgrade dari V4.2 maupun V4.2.1 disertakan: backup konsisten, audited queue hold, protected-row verifier, preservation environment/SSL/key/RAM, API health/schema check, worker tetap STOPPED/disabled. Rollback memakai database dan release lama bersama, tanpa menghapus arsip finansial.

Benchmark 43 item dengan 10 tab A aktif yang tidak terpilih tetap **77 GET, 1 POST, 117.986 byte**, output persis sama dengan V4.2.1. Pengujian live Google/VPS masih pending. Ikuti [UPGRADE_V422.md](UPGRADE_V422.md) dan baca [TEST_REPORT.md](TEST_REPORT.md).
