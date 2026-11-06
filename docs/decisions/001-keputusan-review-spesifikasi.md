# 001 — Keputusan hasil review spesifikasi

## Konteks

Versi 0.3 cara kerja dan versi 1.2 logika isolir memakai tiga default isolir, satuan uang yang berbeda, dan reaktivasi yang berhenti bila `isolir_enabled` false. Review 6 November 2026 mengunci pilihan di bawah.

## Keputusan

- Pelanggan baru: `isolir_enabled = true`, `batas_tunggakan = 2`, `siklus_tagihan = tetap`.
- Isolir tidak punya empat mode. `batas_tunggakan` berisi `null`, `1`, atau `2`. `siklus_tagihan` berisi `tetap` atau `prepaid_30hari`. Kolom `masa_aktif_hingga` tetap untuk siklus prabayar.
- `isolir_enabled` hanya menahan scheduler. Reaktivasi tidak membaca flag itu.
- Isolir otomatis menunggu `tanggal_instalasi`. Biaya pasang adalah baris invoice, bukan tagihan kedua, dan tidak dihitung tunggakan.
- Tunggakan langganan meliputi `belum_bayar`, `sebagian`, dan `menunggu_verifikasi`.
- Uang disimpan sebagai integer rupiah. Harga paket sudah termasuk PPN. Prorata membagi harga dengan jumlah hari periode, lalu dibulatkan ke rupiah terdekat.
- Status isolir pelanggan final hanya setelah setiap Mikrotik yang dituju selesai. Antrian isolir memakai database queue.
- Admin Pusat pada dokumen lama adalah Admin Global. Wilayah admin disimpan di `user_wilayah`.
- Pesan pelanggan memakai tombol `wa.me`. Alert internal memakai Telegram.
- GPS dan cetak thermal dari browser hanya saat halaman terbuka. iOS mencetak PDF.
- Bonus teknisi tidak timbul dari tiket yang tertutup karena agenda massal.
- Failover server, replikasi database, dan restore SQL adalah prosedur infrastruktur, bukan modul aplikasi.
- Impor pelanggan lama memakai `isolir_mulai_evaluasi`. Mutasi uang yang dibatalkan menulis jurnal balik.
- Cuti tidak menagih tanggal cuti. Tagihan periode berjalan dipotong sampai sehari sebelum `cuti_mulai`. Aktif kembali di tengah bulan membuat tagihan prorata tersendiri sampai akhir bulan, dan tagihan itu masuk tunggakan.
- `diskon_selama_isolir` adalah centang pada siklus `tetap`. Diskon mengurangi tagihan atas hari terisolir, dan tanggal jatuh tempo tidak berubah. `prepaid_30hari` tidak memakai pilihan ini. Satu tagihan langganan yang sudah lewat jatuh tempo memutus pelanggan dengan `batas_tunggakan = 1`, termasuk tagihan bulan lain.
- Bulan tanpa hari layanan tidak punya tagihan dan tidak punya kuitansi. Aktivasi kembali di tengah bulan membuat tagihan prorata sampai akhir bulan. Tidak adanya tagihan tidak membuka isolir.
- Tombol simulasi admin menjumlahkan sisa tunggakan dan prorata aktivasi tanpa menyimpan apa pun.

## Konsekuensi

`docs/cara-kerja.md` versi 0.4 dan `docs/logika-isolir.md` versi 1.3 mengikuti keputusan ini. Perubahan berikutnya dicatat sebagai keputusan baru, bukan dengan menghidupkan kalimat lama.
