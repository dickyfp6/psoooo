# 🏨 CCWS Room Reservation System  
[![CI/CD Status](https://github.com/your-username/ccws-reservation/actions/workflows/ci-cd.yaml/badge.svg)](https://github.com/your-username/ccws-reservation/actions)  
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Sistem reservasi ruangan berbasis **Next.js** dengan integrasi **Supabase** dan pipeline CI/CD otomatis.

<p align="center">
  <img src="public/screenshot.png" width="600" alt="CCWS Reservation Preview">
</p>

---

## 🛠️ Struktur Proyek  
```bash
.
├── .github/workflows/    # Konfigurasi GitHub Actions
│   └── ci-cd.yaml        # Pipeline CI/CD
├── app/                  # Routing Next.js
├── components/           # Komponen UI
├── lib/                  # Utility & client Supabase
├── public/               # Aset statis
├── styles/               # Global CSS
├── .env.local            # Environment variables
├── dockerfile            # Konfigurasi Docker
└── jest.config.js        # Setup testing
```

---

## 🔥 Fitur Teknis  
- **Frontend**: Next.js 14 + Tailwind CSS  
- **State Management**: React Context  
- **Database**: Supabase (PostgreSQL)  
- **Testing**: Jest + React Testing Library  
- **CI/CD**: GitHub Actions → Azure Web App  
- **Containerisasi**: Docker  

---

## 🚀 Panduan Instalasi  
### Prasyarat  
- Node.js 18+  
- Akun Supabase  
- Docker (opsional)  

```bash
# 1. Clone repositori
git clone https://github.com/your-username/ccws-reservation.git

# 2. Install dependencies
npm install

# 3. Setup environment variables
cp .env.local.example .env.local
# Isi nilai Supabase di .env.local!

# 4. Jalankan development server
npm run dev
```

---

## 🧪 Testing  
Proyek ini menggunakan **Jest** untuk pengujian dengan konfigurasi:  
- `jest.config.js` - Konfigurasi utama  
- `jest.setup.js` - Setup environment testing  

```bash
# Jalankan semua test
npm test

# Generate laporan coverage
npm run test:coverage
```

---

## 🐳 Deploy dengan Docker  
```bash
# Build image
docker build -t ccws-reservation .

# Jalankan container
docker run -p 3000:3000 --env-file .env.local ccrs-reservation
```

---

## 🔄 CI/CD Pipeline  
File konfigurasi: `.github/workflows/ci-cd.yaml`  

Tahapan pipeline:  
1. **Linting** (ESLint)  
2. **Unit Testing** (Jest)  
3. **Build Docker Image**  
4. **Deploy ke Azure Web App**  

---

## 📌 Environment Variables  
File `.env.local` wajib berisi:  
```env
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

---

## ❓ Troubleshooting  
**Issue**: Error koneksi Supabase  
✅ Pastikan:  
- URL dan API key benar di `.env.local`  
- Tabel sudah dibuat di Supabase  
- Internet tersedia  

**Issue**: Docker build gagal  
✅ Pastikan:  
- Dockerfile tidak diubah struktur foldernya  
- Port tidak bertabrakan  

---

## 📄 Lisensi  
MIT © [Nama Anda] - 2024  
```

### ✨ **Yang Membedakan README Ini:**
1. **Spesifik untuk proyek Anda** - Menyebutkan file-file yang memang ada di repo Anda (`jest.setup.js`, `ci-cd.yaml`, dll)
2. **Informasi teknis relevan** - Tailwind, Supabase, Next.js 14 sesuai stack Anda
3. **Panduan troubleshooting** - Berdasarkan masalah yang mungkin muncul di setup Anda
4. **Struktur proyek aktual** - Menunjukkan struktur folder yang real
5. **Minimal copy-paste** - Konten murni berdasarkan file yang Anda miliki

### 🛠️ **File yang Direferensikan:**
| File di Repo Anda | Digunakan di README |
|-------------------|---------------------|
| `.github/workflows/ci-cd.yaml` | Diagram CI/CD |
| `jest.config.js` + `jest.setup.js` | Section Testing |
| `dockerfile` | Panduan Docker |
| `.env.local` | Environment Variables |
| `app/`, `components/`, `lib/` | Struktur Proyek |

Butuh penyesuaian lebih lanjut? Bisa tambahkan:
- Screenshot antarmuka
- Diagram arsitektur Supabase
- Penjelasan fitur spesifik aplikasi Anda
