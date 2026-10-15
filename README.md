# Praktikum DevOps

- Nama: I Made Dedy Wanditya
- NIM: 42430042

## Tahap 1 — Menjalankan Aplikasi Native

Backend dijalankan langsung pada macOS menggunakan Go, dengan
STORAGE_MODE=memory.

### Cara menjalankan

Dari folder backend:
1. Jalankan `go mod tidy` untuk menyiapkan dependensi.
2. Jalankan `go run main.go` untuk memulai server pada port 8080.

### Hasil pengujian

- GET `/healthz` menghasilkan HTTP 200, status `healthy`,
  nama I Made Dedy Wanditya, dan NIM 42430042.
- Nilai `container_or_host_id` adalah `MacBook-Pro-I.local`.
- GET `/api/services` menghasilkan HTTP 200 dan data
  `Standalone-API-Demo` dengan storage_type `in-memory`.
- Proses kedua pada port 8080 gagal dengan pesan
  `bind: address already in use`.
- Server pertama tetap merespons setelah proses kedua gagal.
- Percobaan tambahan dengan `APP_PORT=8081 go run main.go`
  berhasil. Backend pada port 8080 dan 8081 sama-sama merespons.

### Analisis

Pada konfigurasi ini, kedua proses tidak dapat mendengarkan
pada alamat dan port TCP yang sama secara bersamaan.
Mengganti port proses kedua menjadi 8081 menghindari benturan.
Variabel lingkungan APP_PORT memungkinkan perubahan konfigurasi
tanpa mengubah kode sumber.

### Bukti

![Server native berjalan](docs/screenshots/sc01-native-running.png)

![Benturan port dan respons server](docs/screenshots/sc01-native-port-conflict.png)

## Tahap 2 — Dockerfile Single-Stage

Image dibangun menggunakan Dockerfile.single dengan base image
golang:1.22-bookworm.

### Hasil pengujian

- Nama image: `backend-api:42430042-single`
- Image ID: `febd81bfce74`
- Disk Usage: `1.32 GB`
- Content Size: `311 MB`
- `kontainer-pertama` menggunakan pemetaan port `8081:8080`,
  dengan Container ID `d3371e2a0ae8`.
- `kontainer-kedua` menggunakan pemetaan port `8082:8080`,
  dengan Container ID `cac29d910eb3`.
- Kedua container berstatus Up.
- Kedua endpoint `/healthz` memberikan status `healthy`,
  nama I Made Dedy Wanditya, dan NIM 42430042.

### Analisis

Kedua container dapat menggunakan port internal 8080 karena
masing-masing memiliki lingkungan jaringan terpisah.
Akses dari Mac menggunakan port berbeda, yaitu 8081 dan 8082,
sehingga pemetaan port pada host tidak bentrok.

Image single-stage berukuran besar karena lingkungan Debian,
compiler Go, dependensi, dan berkas hasil build masih berada
dalam image akhir.

Docker menampilkan Disk Usage dan Content Size sebagai ukuran
yang berbeda. Disk Usage mencakup konten terkompresi dan hasil
ekstraksinya. Perbandingan dengan image multi-stage nantinya
akan menggunakan jenis ukuran yang sama.

### Bukti

![Bukti Tahap 2 Single-Stage](docs/screenshots/sc02-tahap2-single-stage.png)
