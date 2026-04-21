# ═══════════════════════════════════════════════════════
#  SISTEM DETEKSI MENGANTUK - PANDUAN LENGKAP
# ═══════════════════════════════════════════════════════

## STRUKTUR FILE
```
project/
├── server/
│   ├── main.py           ← Server FastAPI (deploy ke cloud)
│   ├── requirements.txt  ← Dependencies server
│   └── dashboard.html    ← UI monitoring dosen (otomatis serve)
└── client/
    └── client_siswa.py   ← Dijalankan siswa di laptop masing-masing
```

═══════════════════════════════════════════════════════
## BAGIAN 1: DEPLOY SERVER (GRATIS)
═══════════════════════════════════════════════════════

### Opsi A — Railway.app (PALING MUDAH, GRATIS)

1. Daftar di https://railway.app (bisa login dengan GitHub)

2. Klik "New Project" → "Deploy from GitHub repo"
   ATAU upload folder server/ via CLI:
   ```
   npm install -g @railway/cli
   railway login
   cd server/
   railway init
   railway up
   ```

3. Setelah deploy, Railway memberi URL seperti:
   https://your-app-name.railway.app

4. Buka URL tersebut di browser — harus muncul:
   {"message": "Drowsiness Detection Server is running!"}

5. Dashboard dosen: https://your-app-name.railway.app/dashboard


### Opsi B — Render.com (Gratis, sedikit lebih lambat start)

1. Daftar di https://render.com
2. New → Web Service → Connect GitHub atau upload manual
3. Build Command: pip install -r requirements.txt
4. Start Command: uvicorn main:app --host 0.0.0.0 --port $PORT
5. URL otomatis diberikan setelah deploy


### Opsi C — Ngrok (Testing cepat dari laptop dosen)

Tidak perlu deploy! Jalankan server di laptop dosen:
```
pip install fastapi uvicorn
cd server/
uvicorn main:app --host 0.0.0.0 --port 8000

# Di terminal baru:
pip install ngrok
ngrok http 8000
```
Ngrok memberikan URL publik seperti: https://abc123.ngrok.io
Bagikan URL ini ke siswa!

CATATAN: Ngrok gratis = URL berubah tiap sesi. 
Untuk permanen pakai Railway/Render.


═══════════════════════════════════════════════════════
## BAGIAN 2: SETUP CLIENT SISWA
═══════════════════════════════════════════════════════

### Requirements siswa:
- Python 3.8 atau lebih baru
- Webcam (built-in laptop sudah cukup)
- File best.pt (model deteksi kamu)

### Langkah install:
```bash
pip install ultralytics opencv-python requests
```

### Edit 1 baris di client_siswa.py:
Buka file, cari baris:
SERVER_URL = "https://YOUR-SERVER-URL.railway.app"
Ganti dengan URL server yang sudah di-deploy.

### Jalankan:
```bash
# Letakkan best.pt di folder yang sama dengan client_siswa.py
python client_siswa.py
```

### Siswa akan diminta input:
- Nama lengkap
- NIM / ID
- Nama kelas

Setelah itu webcam aktif dan deteksi berjalan otomatis.
Tekan Q untuk keluar.


═══════════════════════════════════════════════════════
## BAGIAN 3: MONITORING DOSEN
═══════════════════════════════════════════════════════

Dosen buka browser dan akses:
https://your-server.railway.app/dashboard

ATAU buka file dashboard.html secara lokal di browser,
lalu masukkan URL server di kolom konfigurasi.

Fitur dashboard:
- ✓ Kartu setiap siswa dengan status real-time
- ✓ Filter: Semua / Awake / Drowsy / Sleeping
- ✓ Pencarian by nama atau NIM
- ✓ Alert popup + bunyi saat siswa tertidur
- ✓ Log tabel semua deteksi
- ✓ Statistik ringkasan (4 counter atas)
- ✓ Auto-reconnect jika koneksi putus


═══════════════════════════════════════════════════════
## BAGIAN 4: LABEL MODEL (best.pt)
═══════════════════════════════════════════════════════

Sistem menggunakan nama kelas dari model kamu.
Pastikan label di best.pt menggunakan salah satu dari:
- "awake" atau "open_eyes" atau "open"
- "drowsy" atau "microsleep"  
- "sleeping" atau "closed_eyes" atau "closed"

Jika nama labelnya berbeda, edit bagian ini di client_siswa.py:
```python
COLORS = {
    "awake":    (39, 174, 96),   # sesuaikan nama label kamu
    "drowsy":   (230, 126, 34),
    "sleeping": (231, 76, 60),
}
```


═══════════════════════════════════════════════════════
## TROUBLESHOOTING
═══════════════════════════════════════════════════════

❌ "Kamera tidak bisa dibuka"
→ Ganti CAMERA_INDEX = 1 atau 2 di client_siswa.py

❌ "Server tidak terhubung"
→ Pastikan SERVER_URL sudah diubah dan server running

❌ "Model tidak ditemukan"
→ Letakkan best.pt di folder yang SAMA dengan client_siswa.py

❌ Model tidak mendeteksi apapun
→ Pastikan nama label model sesuai (awake/drowsy/sleeping)
→ Coba turunkan confidence threshold: model(frame, conf=0.25)

❌ Dashboard tidak update
→ Refresh halaman, pastikan URL server benar di config bar


═══════════════════════════════════════════════════════
## API ENDPOINTS (untuk pengembangan lanjut)
═══════════════════════════════════════════════════════

GET  /              → Status server
POST /api/detection → Kirim data deteksi
GET  /api/students  → Daftar semua siswa + status
GET  /api/logs      → Log deteksi (query: ?limit=100&student_id=xxx)
GET  /api/stats     → Statistik ringkasan
DELETE /api/reset   → Reset semua data
WS   /ws/dashboard  → WebSocket real-time untuk dashboard
GET  /dashboard     → Buka UI dashboard di browser
