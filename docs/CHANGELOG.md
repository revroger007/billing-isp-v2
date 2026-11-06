# Changelog

## 2026-11-06

Spesifikasi diselaraskan dengan review rancangan.

- Isolir default Custom 3, flag `isolir_enabled` tidak lagi menahan reaktivasi, dan status banyak router dicatat sampai semua node selesai.
- Uang, PPN, prorata, baris tagihan, dan jurnal balik dikunci sebagai integer rupiah.
- Peran Admin Pusat disatukan ke Admin Global. WhatsApp pelanggan hanya lewat tombol. Failover database dikeluarkan dari modul aplikasi.

Belum dikerjakan: kode aplikasi. Tahap berikut adalah proyek Laravel, migrasi, serta autentikasi tujuh peran.
