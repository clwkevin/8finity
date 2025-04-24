# Tutorial Lengkap Infinity GPU Miner

Tutorial ini akan memandu Anda secara lengkap dan mudah dipahami untuk menginstal dan menggunakan Infinity GPU Miner, sebuah alat untuk mining token Infinity menggunakan GPU.

## Daftar Isi
1. [Pengenalan](#1-pengenalan)
2. [Persiapan Dependensi](#2-persiapan-dependensi)
3. [Instalasi](#3-instalasi)
4. [Konfigurasi](#4-konfigurasi)
5. [Memulai Mining](#5-memulai-mining)
6. [Pengaturan Lanjutan](#6-pengaturan-lanjutan)
7. [Pemecahan Masalah](#7-pemecahan-masalah)
8. [Tips dan Trik](#8-tips-dan-trik)

## 1. Pengenalan

Infinity GPU Miner adalah alat yang dioptimalkan untuk menyelesaikan masalah Proof-of-Work "Magic XOR" pada token Infinity. Miner ini menggunakan teknologi OpenCL untuk memanfaatkan kekuatan GPU Anda, menjadikannya jauh lebih efisien dibandingkan mining dengan CPU.

## 2. Persiapan Dependensi

Sebelum menginstal Infinity GPU Miner, pastikan sistem Anda memiliki semua dependensi yang diperlukan.

### Untuk Linux (Ubuntu/Debian)

```bash
# Update repository
sudo apt-get update

# Instal dependensi dasar
sudo apt-get install -y g++ make git curl python3 python3-pip nano

# Instal dependensi OpenCL
sudo apt-get install -y ocl-icd-opencl-dev libopencl-clang-dev clinfo

# Pastikan driver GPU Anda sudah terinstal
# Untuk NVIDIA:
sudo apt-get install -y nvidia-driver-XXX  # Ganti XXX dengan versi driver Anda

# Untuk AMD:
sudo apt-get install -y mesa-opencl-icd
```

### Untuk macOS

```bash
# Instal Xcode Command Line Tools (termasuk OpenCL)
xcode-select --install

# Instal Homebrew jika belum ada
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instal dependensi tambahan
brew install clinfo
```

### Instal Paket Python yang Diperlukan

```bash
# Untuk semua platform
pip3 install pybind11 safe-pysha3 ecdsa web3 coincurve websocket-client websockets python-dotenv
```

### Periksa Dukungan OpenCL

```bash
# Periksa apakah OpenCL berfungsi dengan benar
clinfo
```

Output `clinfo` harus menampilkan detail tentang platform dan device OpenCL Anda. Jika perintah ini mengembalikan error, maka OpenCL tidak dikonfigurasi dengan benar.

## 3. Instalasi

Ada tiga cara untuk menginstal Infinity GPU Miner:

### Metode 1: Docker (Direkomendasikan untuk Linux)

```bash
# Clone repository
git clone https://github.com/8finity-xyz/miner-gpu.git
cd miner-gpu

# Build Docker image
docker build -t infinity-miner .

# Jalankan container Docker dengan akses GPU
docker run --gpus all -it infinity-miner /bin/bash

# Di dalam container, navigasi ke direktori aplikasi
cd /app
```

### Metode 2: Menggunakan Container Pre-built

```bash
# Pull image dari Docker Hub
docker pull otonashilabs/infinity-miner:latest

# Jalankan container dengan akses GPU
docker run --gpus all -it otonashilabs/infinity-miner:latest /bin/bash

# Di dalam container, navigasi ke direktori aplikasi
cd /app
```

### Metode 3: Kompilasi Manual (Linux)

```bash
# Clone repository
git clone https://github.com/8finity-xyz/miner-gpu.git
cd miner-gpu

# Bersihkan dan build
make clean && make

# Jika mengalami masalah dengan NVIDIA dan OpenCL, coba:
mkdir -p /etc/OpenCL/vendors && echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd
```

### Metode 4: Kompilasi untuk macOS (Apple Silicon)

```bash
# Clone repository
git clone https://github.com/8finity-xyz/miner-gpu.git
cd miner-gpu

# Berikan izin eksekusi pada skrip build
chmod +x build_mac.sh

# Jalankan skrip build
./build_mac.sh
```

## 4. Konfigurasi

### Memeriksa Kompilasi OpenCL

Setelah instalasi, periksa apakah OpenCL berfungsi dengan benar:

```bash
python3 test_opencl_kernel.py
```

Jika tidak ada error, berarti OpenCL dikonfigurasi dengan benar.

### Mengatur Konfigurasi Wallet

1. Buat file konfigurasi:

```bash
# Edit file konfigurasi contoh
nano .env.example
```

2. Isi dengan detail wallet dan RPC:

```
# Valuables!
MASTER_ADDRESS = <ALAMAT_WALLET_MINING_ANDA>
MASTER_PKEY = <PRIVATE_KEY_WALLET_MINING_ANDA>
REWARDS_RECIPIENT_ADDRESS = <ALAMAT_WALLET_PENERIMA_REWARDS>

# RPCs
INFINITY_RPC = https://rpc.soniclabs.com
INFINITY_WS = wss://rpc.soniclabs.com
```

Keterangan:
- `MASTER_ADDRESS`: Alamat wallet yang digunakan untuk mining (perlu memiliki token Sonic untuk biaya gas)
- `MASTER_PKEY`: Private key dari wallet mining (ekspor dari Metamask/Zerion)
- `REWARDS_RECIPIENT_ADDRESS`: Alamat wallet yang akan menerima reward Infinity token

3. Simpan sebagai file `.env`:

```bash
# Mengubah nama file
mv .env.example .env
```

## 5. Memulai Mining

Setelah konfigurasi selesai, mulai mining dengan perintah:

```bash
python3 mine_infinity.py
```

Program akan memulai proses mining dan menampilkan statistik secara berkala.

### Memahami Output Mining

Saat mining berjalan, Anda akan melihat informasi seperti:
- Hashrate (berapa banyak hash per detik yang diproses)
- Jumlah nonce yang dicoba
- Blok yang sedang diproses
- Status koneksi ke jaringan
- Informasi tentang reward yang diterima (jika berhasil)

## 6. Pengaturan Lanjutan

Untuk performa optimal, Anda dapat menyesuaikan parameter di file `config.py`:

```bash
# Buka file konfigurasi
nano config.py
```

Parameter penting yang dapat disesuaikan:

```python
# Interval polling (dalam detik)
DEFAULT_POLL_INTERVAL_SECONDS = 0.3  # 300 ms

# Interval langkah loop utama (dalam detik)
DEFAULT_MAIN_LOOP_STEP_SECONDS = 0.005  # 5 ms

# Parameter kinerja OpenCL
WORKSIZE_LOCAL = 64
WORKSIZE_MAX = 0  # 0 berarti default
INVERSE_SIZE = 255
INVERSE_MULTIPLE = 1024  # 16384 untuk NVIDIA, 1024 untuk Apple Silicon
```

Tips Pengaturan:
- `INVERSE_MULTIPLE` harus berupa pangkat dari 2 (1024, 2048, 4096, dll.)
- Untuk GPU NVIDIA, nilai optimal `INVERSE_MULTIPLE` biasanya 16384
- Untuk Apple Silicon, nilai optimal `INVERSE_MULTIPLE` sekitar 1024

## 7. Pemecahan Masalah

### Masalah Umum dan Solusi

1. **OpenCL tidak terdeteksi**:
   - Pastikan driver GPU terinstal dengan benar
   - Periksa instalasi OpenCL dengan `clinfo`
   - Untuk NVIDIA, coba buat file ICD: `echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd`

2. **Error kompilasi**:
   - Pastikan semua dependensi terinstal
   - Periksa log error untuk detail masalah
   - Coba build ulang dengan `make clean && make`

3. **Kinerja rendah**:
   - Sesuaikan parameter `INVERSE_MULTIPLE` di `config.py`
   - Pastikan tidak ada aplikasi lain yang menggunakan GPU secara intensif

4. **Gagal terhubung ke RPC**:
   - Periksa koneksi internet
   - Verifikasi URL RPC di file `.env`
   - Coba gunakan RPC alternatif jika tersedia

## 8. Tips dan Trik

1. **Keamanan Wallet**:
   - Gunakan wallet terpisah dengan dana minimal untuk mining
   - Jangan pernah membagikan private key Anda
   - Pertimbangkan untuk menyimpan reward di wallet yang berbeda

2. **Optimalisasi Kinerja**:
   - Tutup aplikasi lain yang menggunakan GPU
   - Pertahankan suhu GPU yang optimal dengan pendinginan yang cukup
   - Pantau penggunaan daya dan suhu GPU

3. **Monitoring**:
   - Gunakan alat seperti `nvidia-smi` (NVIDIA) atau Activity Monitor (macOS) untuk memantau kinerja GPU
   - Periksa saldo Sonic secara berkala untuk memastikan cukup untuk biaya gas

4. **Strategi Mining**:
   - Mining dalam jangka panjang biasanya lebih menguntungkan daripada sesi pendek
   - Pertimbangkan biaya listrik dalam perhitungan profitabilitas

Selamat mining! Jika Anda menemukan perbaikan atau memiliki pertanyaan, jangan ragu untuk membuka issue atau pull request di GitHub.
