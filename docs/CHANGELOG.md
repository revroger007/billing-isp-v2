# Changelog

## 2026-11-06

Spesifikasi diselaraskan dengan review rancangan.

- Isolir default `dua_tunggakan`, flag `isolir_enabled` tidak lagi menahan reaktivasi, dan status banyak router dicatat sampai semua node selesai.
- Nilai `mode_isolir`: `jatuh_tempo`, `tidak_isolir`, `dua_tunggakan`, dan `prepaid_30hari`. Kolom tanggal `masa_aktif_hingga` tidak berganti nama.
- Cuti memotong tagihan bulan berjalan secara prorata sampai sehari sebelum cuti. Aktif kembali di tengah bulan menerbitkan prorata baru sampai akhir bulan. Keduanya tetap invoice tunggakan.
- Mode `jatuh_tempo` punya pilihan `diskon_selama_isolir`. Tanggal isolir tidak pindah. Mode `prepaid_30hari` tetap memindahkan tanggal isolir dari tanggal bayar.
- Uang, PPN, prorata, baris tagihan, dan jurnal balik dikunci sebagai integer rupiah.
- Peran Admin Pusat disatukan ke Admin Global. WhatsApp pelanggan hanya lewat tombol. Failover database dikeluarkan dari modul aplikasi.

Belum dikerjakan: kode aplikasi. Tahap berikut adalah proyek Laravel, migrasi, serta autentikasi tujuh peran.
