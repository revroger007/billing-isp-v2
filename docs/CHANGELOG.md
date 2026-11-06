# Changelog

## 2026-11-06

Spesifikasi diselaraskan dengan review rancangan.

- Isolir default `toleransi_satu`, flag `isolir_enabled` tidak lagi menahan reaktivasi, dan status banyak router dicatat sampai semua node selesai.
- Nilai `mode_isolir` memakai bahasa Indonesia: `jatuh_tempo`, `tidak_isolir`, `toleransi_satu`, dan `masa_aktif`.
- Uang, PPN, prorata, baris tagihan, dan jurnal balik dikunci sebagai integer rupiah.
- Peran Admin Pusat disatukan ke Admin Global. WhatsApp pelanggan hanya lewat tombol. Failover database dikeluarkan dari modul aplikasi.

Belum dikerjakan: kode aplikasi. Tahap berikut adalah proyek Laravel, migrasi, serta autentikasi tujuh peran.
