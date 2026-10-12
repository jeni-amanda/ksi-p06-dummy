# P02 — Penilaian Risiko dan Risk Register

## Identitas
Nama: ____________________  
NIM: _____________________  
Kelas: ___________________

## Tujuan
Membangun lima demo web lokal untuk menunjukkan perbedaan perilaku antara aplikasi rentan dan aplikasi yang telah diberi kontrol keamanan. Seluruh pengujian dilakukan pada file lokal milik sendiri.

## T1 — Penyimpanan Sandi
**Risiko:** kebocoran basis data dapat mengekspos sandi pengguna jika plaintext disimpan.

**Uji rentan:** buat dua akun dengan sandi berbeda, lalu lihat Dump data anggota. Sandi asli terlihat.

**Patch:** versi aman menyimpan salt acak, jumlah iterasi, dan hash; plaintext tidak disimpan.

**Uji ulang:** buat akun pada versi aman. Dump hanya memperlihatkan metadata hash, sehingga payload dump tidak mengungkap sandi asli.

**Kesimpulan:** hashing dengan salt mengurangi dampak kebocoran dibanding plaintext. Dalam sistem produksi, gunakan KDF password khusus seperti Argon2id/bcrypt/scrypt/PBKDF2 sesuai kebijakan, bukan hash cepat biasa.

## T2 — SQL Injection
**Risiko:** input login yang dicampurkan ke perintah dapat mengubah logika query.

**Payload uji:** gunakan tautologi SQL pada field password, misalnya `' OR '1'='1' --`.

**Rentan:** simulator query memperlakukan input sebagai bagian dari ekspresi sehingga login dapat terbuka tanpa sandi benar.

**Patch:** versi aman memisahkan query dari parameter dan membandingkan nilai sebagai data.

**Kesimpulan:** parameter binding menghentikan perubahan struktur query. Blacklist satu payload bukan kontrol yang memadai.

## T3 — Stored XSS
**Risiko:** komentar tersimpan dapat dieksekusi saat dibaca pengguna lain jika dirender sebagai HTML.

**Payload 1:** `<img src=x onerror=alert('XSS-1')>`

**Payload 2:** `<svg onload=alert('XSS-2')></svg>`

**Rentan:** komentar dimasukkan melalui `innerHTML`, sehingga markup aktif diproses browser.

**Patch:** versi aman menggunakan `textContent` dan DOM API.

**Kesimpulan:** output harus diperlakukan sesuai konteks. Menghapus kata `script` saja tidak cukup karena event handler dan elemen lain dapat menjadi vektor.

## T4 — Brute Force
**Risiko:** login tanpa pembatasan memungkinkan banyak tebakan sandi.

**Uji:** lakukan sedikitnya 12 percobaan sandi salah berurutan.

**Rentan:** seluruh percobaan diproses.

**Patch:** setelah 5 kegagalan, demo aman melakukan blokir sementara 15 detik dan mencatat kejadian.

**Kesimpulan:** rate limiting menurunkan likelihood serangan online. Produksi sebaiknya menambahkan kontrol berbasis akun/IP/device, monitoring, MFA, dan notifikasi sesuai kebutuhan.

## T5 — IDOR / Broken Access Control
**Risiko:** pengguna dapat membaca objek pengguna lain hanya dengan mengetahui/mengubah ID.

**Uji:** sesi `andi`, minta `DOC-102` milik `bela`.

**Rentan:** objek dikembalikan berdasarkan ID tanpa pemeriksaan pemilik.

**Patch:** versi aman memeriksa `owner === sessionUser` sebelum mengembalikan isi objek.

**Kesimpulan:** autentikasi (siapa pengguna) berbeda dengan otorisasi (apa yang boleh diakses). ID yang sulit ditebak bukan pengganti pemeriksaan otorisasi.

## Ringkasan bukti sebelum/sesudah
| Kasus | Sebelum | Sesudah | Kontrol |
|---|---|---|---|
| T1 | Plaintext terlihat | Plaintext tidak ada | Salt + password hashing |
| T2 | Tautologi dapat mengubah logika | Input diperlakukan sebagai parameter | Parameterized query |
| T3 | Markup aktif dieksekusi | Payload tampil sebagai teks | textContent/output encoding |
| T4 | 12+ tebakan diproses | Diblokir setelah ambang | Rate limiting + temporary lock |
| T5 | Objek milik user lain terbaca | 403 | Object-level authorization |

## Checklist screenshot
Ambil screenshot yang memperlihatkan:
1. Nama file rentan/aman.
2. Payload yang digunakan.
3. Hasil serangan pada versi rentan.
4. Hasil payload identik pada versi aman.
5. Untuk T4, percobaan ke-12 atau kondisi blokir.
6. Untuk T5, sesi `andi` dan ID `DOC-102`.

## Catatan
Aplikasi ini adalah simulasi lokal untuk praktikum. Jangan menguji payload terhadap sistem pihak lain tanpa izin tertulis.
