# 001 — Keputusan hasil review spesifikasi

## Konteks

Versi 0.3 cara kerja dan versi 1.2 logika isolir memakai tiga default isolir, satuan uang yang berbeda, dan reaktivasi yang berhenti bila `isolir_enabled` false. Review 6 November 2026 mengunci pilihan di bawah.

## Keputusan

- Pelanggan baru: `isolir_enabled = true`, `mode_isolir = custom3`. Mode `default` dihapus.
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

## Konsekuensi

`docs/cara-kerja.md` versi 0.4 dan `docs/logika-isolir.md` versi 1.3 mengikuti keputusan ini. Perubahan berikutnya dicatat sebagai keputusan baru, bukan dengan menghidupkan kalimat lama.
