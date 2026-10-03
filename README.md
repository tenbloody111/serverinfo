# Bedrock Ping Checker

Alat untuk mengecek status server Minecraft Bedrock (online/offline, MOTD, versi, jumlah player, latency, packet loss) secara manual maupun real-time, lengkap dengan riwayat latency, pelacakan player masuk/keluar, dan log aktivitas backend. Backend jalan di Termux pakai Node.js (tanpa dependency eksternal), frontend berupa satu halaman HTML.

## Fitur

- **Cek Ping** — kirim 0–10 packet UDP asli ke server, dapat MOTD, versi, player, gamemode, world, packet loss, dan latency (min/avg/max).
- **Mode Realtime** — toggle kecil yang membuat status auto-update tiap 1 detik tanpa perlu klik apa pun.
- **Timeout / Latensi Jaringan** — pilihan 3 detik (cepat) / 30 detik / 1 menit / 2 menit, untuk server dengan koneksi lambat/jauh agar tidak salah dianggap offline.
- **Grafik Riwayat Latency** — card dengan grafik garis menampilkan 30 hasil ping terakhir; titik merah menandai packet yang gagal/timeout.
- **Log Pemain** — tab terpisah berisi:
  - Kotak "Total Pemain Saat Ini" dengan angka yang beranimasi rolling saat berubah, disertai badge `+1`/`-1`.
  - Riwayat perubahan jumlah player dengan jam (`[12:00] Player +1`), dicatat otomatis setiap kali jumlahnya naik/turun.
  - Catatan: hanya mendeteksi **jumlah**, bukan nama pemain (keterbatasan protokol, bukan bug).
- **Console Backend** — log teknis tiap langkah yang dilakukan backend (buat socket, kirim paket, tunggu balasan, parsing) untuk debugging.
- **Indikator koneksi backend** — titik hijau/merah di kolom "Alamat backend" menunjukkan apakah backend Termux sedang terhubung.
- **TPS** — ditampilkan sebagai "Tidak tersedia". Protokol ping Bedrock (RakNet) memang tidak membawa data TPS, dan untuk server orang lain data itu tidak bisa diakses tanpa izin admin (RCON/plugin).

## Cara Pakai di Termux

1. Install Node.js (sekali saja):
   ```
   pkg update
   pkg install nodejs
   ```

2. Pindahkan folder `bedrock-ping` ke Termux. Disarankan disalin ke home folder Termux sendiri (lebih stabil & cepat daripada dijalankan langsung dari folder Download):
   ```
   termux-setup-storage
   mkdir -p ~/ping
   cp -r ~/storage/downloads/1PINGBE ~/ping/
   ```

3. Jalankan server:
   ```
   cd ~/ping/1PINGBE
   node server.js
   ```
   Akan muncul pesan seperti:
   ```
   Bedrock Ping Checker jalan di http://0.0.0.0:3000
   ```

4. Cari IP HP kamu (masih di Termux):
   ```
   ip addr show wlan0 | grep inet
   ```
   atau buka Settings > Wi-Fi > info jaringan di HP.

5. Buka `public/index.html` di browser:
   - Di HP yang sama: buka file-nya langsung, atau akses `http://127.0.0.1:3000`
   - Dari device lain di WiFi yang sama: akses `http://IP-HP-TERMUX:3000`

6. Di kolom "Alamat backend", isi `http://127.0.0.1:3000` (kalau buka dari HP yang sama) atau `http://IP-HP-TERMUX:3000` (kalau dari device lain). Titik indikator di sebelahnya akan hijau kalau berhasil terhubung.

7. Isi host/IP server Minecraft Bedrock yang mau dicek + port (default 19132), atur jumlah packet & timeout sesuai kebutuhan, lalu tekan **Ping Server** — atau aktifkan **Mode Realtime** untuk pembaruan otomatis.

## Struktur Folder

```
bedrock-ping/
├── server.js          # Backend Node.js (HTTP + RakNet ping via UDP)
├── public/
│   └── index.html     # Frontend (UI, grafik, log pemain, console)
└── README.md
```

## Endpoint Backend

| Endpoint            | Fungsi                                                          |
|----------------------|-------------------------------------------------------------------|
| `GET /api/ping`      | Query `host`, `port`, `count` (0–10), `timeout` (3000/30000/60000/120000 ms) — mengecek status server |
| `GET /api/health`    | Cek apakah backend hidup (dipakai indikator koneksi)              |
| `GET /api/logs`      | Ambil log aktivitas backend (dipakai tab Console)                 |
| `GET /api/logs/clear`| Bersihkan log aktivitas backend                                   |

## Catatan Teknis

- Bedrock pakai protokol **RakNet** di atas UDP (beda dari Java Edition yang pakai TCP).
- Backend mengirim paket `Unconnected Ping` dan mem-parsing `Unconnected Pong` dari server untuk dapat MOTD, versi, dan jumlah player — ini cara standar/sah untuk query status server, sama seperti yang dipakai launcher resmi atau situs-situs status server publik.
- Jumlah packet UDP yang dikirim per pengecekan literal sesuai slider (0–10), bukan jumlah besar — supaya tetap aman dipakai bareng Mode Realtime tanpa berisiko membebani server orang lain.
- Tidak ada library eksternal yang dibutuhkan (hanya modul bawaan Node.js: `http`, `dgram`, `fs`, `path`), jadi tidak perlu `npm install`.
- Data "Log Pemain" dan grafik latency hanya tersimpan selama tab browser terbuka (tidak persisten setelah refresh), karena dihitung di sisi frontend dari hasil ping yang masuk.

## Agar Tetap Jalan di Background (opsional)

```
pkg install tmux
tmux new -s bedrockping
node server.js
# tekan Ctrl+B lalu D untuk detach
```
