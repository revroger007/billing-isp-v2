# 001 — Keputusan hasil review spesifikasi

## Konteks

Versi 0.3 cara kerja dan versi 1.2 logika isolir memakai tiga default isolir, satuan uang yang berbeda, dan reaktivasi yang berhenti bila `isolir_enabled` false. Review 6 November 2026 mengunci pilihan di bawah.

## Keputusan

- Pelanggan baru: `batas_tunggakan = 2`, `siklus_tagihan = tanggal_tetap`.
- Isolir tidak punya empat mode dan tidak punya `isolir_enabled`. `batas_tunggakan` berisi `null`, `1`, atau `2`. `siklus_tagihan` berisi `tanggal_tetap` atau `prabayar`.
- Scheduler hanya membaca `isolir_berikutnya`. Penundaan hanya lewat `isolir_ditunda_hingga`.
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
- Tagihan langganan hanya menghitung hari layanan. Hari cuti dan hari `putus` tidak ditagih, sehingga bulan kosong tidak punya kuitansi.
- `isolir_ditunda_hingga` menggantikan `isolir_sementara` dan `bypass_until`. Alasan berisi `toleransi` atau `bypass`. Pembayaran satu bulan saat toleransi mengembalikan jadwal ke `hari_isolir` pada semua siklus.
- Satu tagihan langganan yang sudah lewat jatuh tempo memutus pelanggan dengan `batas_tunggakan = 1`, termasuk tagihan bulan lain.
- Bulan tanpa hari layanan tidak punya tagihan dan tidak punya kuitansi. Aktivasi kembali di tengah bulan membuat tagihan prorata sampai akhir bulan. Tidak adanya tagihan tidak membuka isolir.
- Tombol simulasi admin menjumlahkan sisa tunggakan dan prorata aktivasi tanpa menyimpan apa pun.
- Generate tagihan dan autopay tanggal 1 berjalan pukul 00:01, isolir pukul 00:05. `alokasi_pembayaran` melunasi tagihan langganan terlama lebih dulu. `N` adalah jumlah tagihan yang menjadi lunas.
- Nominal tagihan dipotong hanya saat `status_layanan` menjadi `putus`, dalam transaksi yang sama. Nyala layanan memakai antrean satu sesi per router dengan prioritas tinggi.
- `riwayat_isolir` menyimpan periode putus sampai nyala per pelanggan. `log_api_mikrotik` tetap menyimpan percobaan API per router.
- Komentar address-list isolir adalah nomor pelanggan lalu nama. Nomor pelanggan tidak dipotong. Nama yang tidak muat dipotong pada batas 128 karakter.
- Seribu isolir pada hari yang sama tidak membuka seribu koneksi. Satu Mikrotik satu sesi API, 50 entri, jeda 1 detik. Nyala layanan dari pembayaran didahulukan.
- Perintah yang gagal karena Mikrotik mati dicoba 10 kali, jeda 5 menit. Setelah batas itu, satu percobaan ulang dijalankan 5 menit setelah router terpantau nyala, hanya untuk baris yang gagal.
- Menyalakan layanan membatalkan baris putus yang belum dijalankan. `hari_isolir` yang melewati akhir bulan menjadi hari terakhir bulan itu. Entri address-list tidak punya timeout.
- `status_layanan` memakai `tidak_serempak` untuk router yang tidak selesai bersama. `sebagian` tetap status pembayaran.

## Konsekuensi

`docs/cara-kerja.md` dan `docs/logika-isolir.md` versi 2.10 mengikuti keputusan ini. Perubahan berikutnya dicatat sebagai keputusan baru, bukan dengan menghidupkan kalimat lama.
