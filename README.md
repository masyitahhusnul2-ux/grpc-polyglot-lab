# gRPC Polyglot Lab

Server gRPC (Go) yang dipanggil oleh dua client berbeda bahasa (Python dan Node.js) menggunakan satu kontrak `user.proto`.

## Analisis Pendekatan Kompilasi: Static Code Generation vs Dynamic Runtime Generation

Pada pendekatan *static code generation* (Go dan Python), file `user.proto` diproses lebih dulu oleh compiler `protoc` pada tahap pengembangan. Hasilnya adalah file kode sumber nyata (`user.pb.go` dan `user_grpc.pb.go` untuk Go, serta `user_pb2.py` dan `user_pb2_grpc.py` untuk Python) yang disimpan di proyek dan ikut di-commit. Kode tersebut memberi tipe data yang jelas, sehingga kesalahan seperti salah nama field bisa terdeteksi lebih awal dan editor dapat memberi autocompletion. Konsekuensinya, setiap kali `user.proto` berubah (misalnya penambahan field `phone`), developer wajib menjalankan ulang `protoc` agar kode hasil generate sinkron dengan kontrak. Sebaliknya, pada pendekatan *dynamic runtime generation* (Node.js), tidak ada tahap kompilasi dan tidak ada file kode hasil generate. Library `@grpc/proto-loader` membaca file `user.proto` langsung saat program dijalankan, lalu membangun definisi service dan message di memori. Pendekatan ini lebih praktis dan fleksibel karena perubahan skema cukup disimpan di `user.proto` dan program dijalankan ulang, tetapi tidak ada pengecekan tipe saat penulisan kode, kesalahan baru muncul saat runtime, dan ada sedikit beban tambahan karena file `.proto` harus diparsing setiap kali aplikasi dimulai serta harus tersedia di lokasi runtime.

## Menjalankan Proyek dengan Docker

### 1. Persyaratan

Pastikan perangkat sudah memiliki:
- Docker Desktop
- Git

Pastikan Docker Desktop sudah berjalan sebelum menjalankan perintah berikut.

### 2. Mengunduh Proyek

Jika proyek belum tersedia di perangkat, jalankan:

```bash
git clone https://github.com/masyitahhusnul2-ux/grpc-polyglot-lab.git
cd grpc-polyglot-lab
```

Jika proyek sudah tersedia, langsung masuk ke folder `grpc-polyglot-lab`.

### 3. Membuat Docker Image

Jalankan perintah berikut dari folder utama proyek:

```bash
docker build -f Dockerfile.server -t grpc-go-server .
docker build -f Dockerfile.node -t grpc-node-client .
docker build -f Dockerfile.python -t grpc-python-client .
```

Periksa image yang berhasil dibuat:

```bash
docker images
```

Pastikan tersedia image `grpc-go-server`, `grpc-node-client`, dan `grpc-python-client`.

### 4. Membuat Jaringan Docker

```bash
docker network create grpc-network
```

Jika jaringan sudah tersedia, langkah ini tidak perlu diulangi.

### 5. Menjalankan Go Server

Jalankan server menggunakan perintah:

```bash
docker run -d --name grpc-go-server --network grpc-network grpc-go-server
```

Periksa log server:

```bash
docker logs grpc-go-server
```

Server akan menerima komunikasi gRPC pada port `50051` di jaringan Docker.

Jika container server sudah ada tetapi berhenti, jalankan kembali dengan:

```bash
docker start grpc-go-server
```

### 6. Menjalankan Client Node.js

```bash
docker run --rm --network grpc-network -e GRPC_SERVER=grpc-go-server:50051 grpc-node-client
```

Client Node.js akan mengirim permintaan ke Go Server dan menampilkan data pengguna yang diterima.

### 7. Menjalankan Client Python

```bash
docker run --rm --network grpc-network -e GRPC_SERVER=grpc-go-server:50051 grpc-python-client
```

Client Python akan mengirim permintaan ke Go Server dan menampilkan data pengguna yang diterima.

### 8. Hasil Pengujian

Jika komunikasi berhasil, kedua client akan menerima respons pengguna, antara lain:

- Nama: Budi Santoso
- Email: budi@universitas.ac.id
- Telepon: 081234567890

Hasil tersebut menunjukkan bahwa Go Server dapat melayani permintaan gRPC dari client Node.js dan Python melalui jaringan Docker.

### 9. Menghentikan Server

Untuk menghentikan Go Server:

```bash
docker stop grpc-go-server
```

Container server tetap tersimpan dan dapat dijalankan kembali menggunakan `docker start grpc-go-server`.

### 10. Catatan

Docker image dibuat secara terpisah untuk Go Server, client Node.js, dan client Python. Ketiga komponen menggunakan file protokol bersama, yaitu `user.proto`, untuk mendefinisikan komunikasi gRPC.
