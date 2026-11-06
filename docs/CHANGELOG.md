# Changelog

## 2026-11-06

Spesifikasi diselaraskan dengan review rancangan.

- Isolir memakai `batas_tunggakan` (`null`, `1`, `2`) dan `siklus_tagihan` (`tetap`, `prepaid_30hari`). Flag `isolir_enabled` tidak menahan reaktivasi.
- Cuti memotong tagihan bulan berjalan secara prorata sampai sehari sebelum cuti. Aktif kembali di tengah bulan menerbitkan prorata baru sampai akhir bulan. Keduanya tetap invoice tunggakan.
- `diskon_selama_isolir` hanya centang pada siklus `tetap`. Tanggal jatuh tempo tidak pindah. `prepaid_30hari` memindahkan isolir berikutnya dari tanggal bayar.
- Bulan yang terisolir penuh tidak diterbitkan, sehingga tidak ada kuitansi bulan itu. Aktivasi di tengah bulan berikutnya menerbitkan prorata seperti pasang baru.
- Admin punya tombol simulasi biaya aktivasi dan tunggakan. Hasilnya pratinjau, tidak menulis tagihan atau isolir.
- Uang, PPN, prorata, baris tagihan, dan jurnal balik dikunci sebagai integer rupiah.
- Peran Admin Pusat disatukan ke Admin Global. WhatsApp pelanggan hanya lewat tombol. Failover database dikeluarkan dari modul aplikasi.

Belum dikerjakan: kode aplikasi. Tahap berikut adalah proyek Laravel, migrasi, serta autentikasi tujuh peran.
