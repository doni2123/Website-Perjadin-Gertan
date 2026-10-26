# WebGIS Perjalanan Dinas

Starter frontend statis berbahasa Indonesia: peta Leaflet, filter tahun/jenis/pencarian, statistik, detail kegiatan, login serta CRUD kegiatan dan penugasan. Anggap seluruh record di tiga tabel public sebagai informasi yang telah disetujui untuk dipublikasikan. Jangan masukkan nomor surat internal, NIK, kontak pribadi, rincian keamanan, atau data yang belum lolos pemeriksaan publikasi.

## Berkas

- index.html, style.css, app.js: halaman peta dan formulir anggota.
- supabase/schema.sql: tabel, grants, helper keanggotaan dan RLS.
- Salin config.example.js sebagai config.js, lalu isi URL dan publishable key. Commit/deploy config.js karena publishable key memang public; jangan pernah letakkan service_role/secret key di sana.

## Implementasi berurutan

1. Buat proyek Supabase Free; ambil Project URL dan publishable key.
2. Jalankan semua SQL dari supabase/schema.sql di SQL Editor. Periksa RLS aktif untuk activities, team_members, assignments.
3. Tambahkan roster di public.team_members; display_name/position adalah data yang akan dilihat pengunjung. Authentication → Providers → Email: nonaktifkan pendaftaran publik. Undang anggota melalui Dashboard.
4. Di Authentication → Users, ambil UUID akun yang diundang. Hubungkan akun dengan record roster melalui SQL Editor (administrator saja): insert into private.team_access(user_id,team_member_id) values ('AUTH-USER-UUID','TEAM-MEMBER-UUID'); Untuk mencabut akses, set active=false atau hapus baris. Jangan pernah memberi anggota umum akses admin ke Supabase Dashboard.
5. Salin config.example.js ke config.js dan isi Project URL + publishable key (atau legacy anon key). Sertakan config.js di file deploy/repository: publishable key memang boleh tampak di browser. Keamanan akses berasal dari grants dan RLS; jangan pernah memasukkan service_role/secret key.
6. Deploy ke Cloudflare Pages Free: unggah folder ini sebagai proyek static site atau hubungkan repo GitHub/GitLab; build command kosong dan output folder root. Pages mengizinkan repository public/private dan menyediakan deploy otomatis dari Git. GitHub Pages secara eksplisit menyatakan situsnya tidak seharusnya dipakai untuk transaksi sensitif seperti pengiriman password; untuk aplikasi login anggota gunakan Cloudflare Pages atau host setara. GitHub Pages cocok untuk versi read-only tanpa form login. Set alamat site dan redirect URL di Supabase Authentication → URL Configuration. Path relatif mendukung alamat GitHub Pages di subfolder.
7. Setelah deploy, buka sebagai publik, akun login yang bukan anggota, dan anggota. Publik harus berhasil membaca tetapi ditolak saat menulis; nonanggota terautentikasi juga ditolak saat menulis; anggota aktif dapat CRUD. Periksa Network/berkas hasil deploy agar tak ada secret.

## Model

- activities: satu kegiatan dan koordinat WGS84, tanggal, jenis, ringkasan, pembuat.
- team_members: nama publik yang sudah disetujui.
- assignments: relasi banyak-ke-banyak kegiatan dan personel.
- private.team_access: pemetaan auth UUID ke roster. Skema private tidak terbuka pada API.

## Batas dan operasi

Form memvalidasi teks, jenis, rentang koordinat dan urutan tanggal; CHECK database menerapkan validasi yang sama pada request API. Anggota yang sudah terdaftar dapat mengelola semua record kegiatan. Penulisan kegiatan dan penugasan merupakan dua transaksi request terpisah; jika request kedua gagal, periksa ulang assignment di halaman dan simpan kembali.

Supabase Free cocok untuk prototipe, dengan kuota dan ketentuan yang dapat berubah. Saat diperiksa, halaman resmi menampilkan dua proyek aktif, 500 MB database, 5 GB egress, 5 GB cached egress, 1 GB file storage, 50.000 MAU; proyek dapat dijeda setelah sepekan tidak aktif. Pantau penggunaan, batas email Auth dan rencanakan backup/ekspor. GitHub Pages Free tersedia untuk repository publik; batas lunak bandwidth 100 GB/bulan, dan website terpublikasi maksimum 1 GB menurut dokumentasi. Tile OpenStreetMap wajib atribusi dan layanan tile standar bukan hosting tile dengan SLA; untuk penggunaan berat, gunakan penyedia tile yang sesuai kebijakannya.

Tile OSM standar harus menggunakan URL HTTPS tile.openstreetmap.org, attribution yang terlihat, cache browser normal, dan tidak boleh bulk-download/prefetch. Layanan ini best-effort tanpa SLA; untuk pemakaian besar pilih provider tile lain. Lihat https://operations.osmfoundation.org/policies/tiles/.
