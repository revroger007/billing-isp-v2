# 002 — Biaya pasang dan prorata sebagai piutang, dusun wajib sebelum ditagih, dying gasp tanpa tabel

## Konteks

Review 7 November 2026 atas keputusan 001 menemukan tiga hal:

- Biaya pasang dan prorata dicatat sebagai hutang mengendap, sehingga tidak pernah menjadi piutang dan tidak ditagih kolektor. Pendapatan pemasangan bocor.
- Dusun opsional, sedangkan cakupan Admin Wilayah, kolektor, dan teknisi dibatasi per dusun di `user_wilayah`. Pelanggan aktif tanpa dusun tidak terlihat oleh petugas wilayah mana pun.
- Spesifikasi membaca dying gasp dari ACS. ONT mengirim dying gasp ke OLT lewat jaringan PON. Sesi TR-069 ke ACS tidak sempat terjadi saat listrik padam.

## Keputusan

Keputusan ini menggantikan kalimat 001 yang menyebut biaya pasang, prorata aktivasi, prorata cuti, dan prorata aktivasi prabayar sebagai hutang mengendap.

- Biaya pasang dan semua prorata (aktivasi, cuti, aktivasi prabayar, nyala kembali di tengah periode) diterbitkan sebagai `TAGIHAN`. Tipe `biaya_pasang` dan `prorata`, `periode_penuh = false`. Tagihan itu piutang: masuk `sisa`, ditagih kolektor, dicetak, dan tampil di portal.
- Tagihan `biaya_pasang` dan `prorata` tidak dihitung `jumlah` tunggakan isolir, tidak menambah `N`, dan tidak mengubah `next_isolir`.
- `biaya_pasang` terbit saat `tanggal_instalasi` diisi, bila nominalnya lebih dari 0. Prorata aktivasi terbit pada saat yang sama, untuk sisa hari periode berjalan. Prorata cuti terbit oleh generate bulanan bila periode itu terpotong cuti. Prorata nyala kembali terbit saat layanan menyala pada periode yang belum punya tagihan. Jatuh tempo semuanya adalah tanggal terbit.
- `alokasi_pembayaran` melunasi tagihan `langganan` terlama lebih dulu, lalu tagihan lain yang masih bersisa, urut `jatuh_tempo`.
- Biaya aktivasi dan diskon tetap di `HUTANG_MENGENDAP` sampai ada keputusan lain.
- Pelanggan berstatus `menunggu_verifikasi_pusat` boleh tanpa dusun. Persetujuan Admin Global ditolak bila dusun kosong. Tagihan hanya terbit untuk pelanggan yang dusunnya terisi. Database menolak `dusun_id` kosong pada status selain `menunggu_verifikasi_pusat`.
- Pelanggan tanpa dusun hanya tampil di antrean verifikasi Admin Global dan bagi user yang menginputnya.
- Dying gasp dibaca Zabbix dari trap atau syslog OLT. Korelasi mati lampu per kluster adalah trigger Zabbix. Database aplikasi tidak punya tabel atau kolom untuk dying gasp maupun status mati lampu. Kejadian itu tercatat di log alert Zabbix, dan di log tiket atau agenda bila webhook-nya menempel ke tiket yang sedang terbuka.

## Konsekuensi

- Kendala unik `(pelanggan_id, tipe_tagihan, layanan_mulai)` juga berlaku untuk `prorata` dan `biaya_pasang`. `layanan_mulai` pada `biaya_pasang` adalah tanggal instalasi, sehingga satu pelanggan hanya punya satu tagihan biaya pasang per instalasi.
- Laporan piutang memuat `biaya_pasang` dan `prorata`. Laporan tunggakan isolir tidak memuatnya.
- Tanda `tidak_tertib` dihitung dari sisa tagihan yang tidak dihitung isolir. Tagihan biaya pasang yang baru terbit sudah ikut menyalakan tanda itu. Perlu keputusan apakah tanda hanya dihitung setelah jatuh tempo lewat.
- Impor pelanggan aktif dan tagihan lama menolak baris tanpa dusun.
- Peta membaca problem mati lampu yang sedang terbuka di Zabbix saat lapisan dibuka. Badge tidak disimpan.
