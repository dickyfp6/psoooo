# 🏨 CCWS Room Reservation

![CI/CD](https://github.com/AryasatyaWidyatna/roomreservation/actions/workflows/ci-cd.yaml/badge.svg)
[![Next.js](https://img.shields.io/badge/Next.js-13-blue)](https://nextjs.org/)
[![Deploy to Azure](https://img.shields.io/badge/Azure-App%20Service-blue)](https://azure.microsoft.com)

Sistem pemesanan ruang yang digunakan di lingkungan **CCWS** untuk mengatur jadwal dan ketersediaan ruangan. Aplikasi ini dibangun menggunakan **Next.js** dan di-*deploy* secara otomatis melalui pipeline **CI/CD GitHub Actions ke Azure App Service**.

🔗 **Live Demo**: [CCWS Room Reservation (Live Site)](https://ccwsreserve-ftcsf2fefghphxc2.indonesiacentral-01.azurewebsites.net)  
📦 **Repository**: [CCWS Room Reservation (GitHub Repo)](https://github.com/AryasatyaWidyatna/roomreservation)
[![Preview](https://raw.githubusercontent.com/USERNAME/REPO/main/path/to/screenshot.png)](https://ccwsreserve-ftcsf2fefghphxc2.indonesiacentral-01.azurewebsites.net)

---

## 📌 Scope & Objectives

- **Lingkup**:
  - Frontend menggunakan Next.js
  - Backend menggunakan Supabase
  - CI/CD menggunakan GitHub Actions
  - Deployment ke Azure App Service dengan Docker container
- **Tujuan**:
  - Otomatisasi build, test, dan deploy
  - Monitoring kesehatan deployment
  - Fleksibilitas untuk dikembangkan lebih lanjut (eks: login, integrasi SSO)

---

## 🛠️ CI/CD Pipeline Architecture

```mermaid
graph TD
    A[Developer Push Code] --> B[GitHub Repository]
    B --> C[CI/CD Pipeline]

    C --> D[Test Job]
    D --> D1[Checkout Repo]
    D1 --> D2[Setup Node.js v18]
    D2 --> D3[Install Dependencies]
    D3 --> D4[Run Tests]

    D --> E[Build Job]
    E --> E1[Checkout Repo]
    E1 --> E2[Setup Node.js v18]
    E2 --> E3[Install Dependencies]
    E3 --> E4[Build Next.js App]
    E4 --> E5[Login to ACR]
    E5 --> E6[Build & Push Docker Image]

    E --> F[Deploy Job]
    F --> F1[Checkout Repo]
    F1 --> F2[Deploy to Azure Web App]
````


## ⚙️ Alur Kerja Pipeline CI/CD

Pipeline ini dibagi menjadi **tiga tahap utama**: `test`, `build`, dan `deploy`. Masing-masing job memiliki tanggung jawab spesifik dan saling bergantung satu sama lain untuk memastikan integritas sistem sebelum live.

---

### 🧪 **Job: test** (Continuous Integration)

Job `test` bertanggung jawab untuk **memverifikasi kualitas dan fungsionalitas kode** sebelum masuk ke tahap build dan deploy.

1. **Checkout repository**
   Kode dari repositori diambil dan disiapkan untuk eksekusi.

2. **Setup Node.js**
   Menyiapkan environment Node.js versi **18.x**.

3. **Install dependencies**
   Menggunakan `npm ci` untuk instalasi yang bersih dan konsisten dengan `package-lock.json`.

4. **Run tests**
   Menjalankan pengujian menggunakan `npm run test` untuk memastikan fungsionalitas berjalan dengan benar.

---

### 🏗️ **Job: build** (Build Container Image)

Job `build` hanya dijalankan setelah `test` berhasil. Job ini membuat aplikasi production-ready dalam bentuk Docker image.

1. **Checkout repository**
   Kode diambil kembali untuk proses build.

2. **Setup Node.js**
   Menyiapkan environment Node.js versi 18.

3. **Install dependencies**
   Melakukan instalasi dependensi seperti pada tahap test.

4. **Build Next.js app**
   Menjalankan `npm run build` untuk membundel aplikasi dalam mode produksi.

5. **Check Next.js version**
   Memverifikasi versi framework yang digunakan.

6. **Login ke Azure Container Registry (ACR)**
   Autentikasi ke ACR menggunakan `AZURE_REGISTRY_USERNAME` dan `AZURE_REGISTRY_PASSWORD` dari GitHub Secrets.

7. **Build Docker image**
   Membuat image dengan tag `latest` yang ditujukan ke URL registri ACR.

8. **Push image ke ACR**
   Mendorong Docker image ke **Azure Container Registry** untuk siap di-deploy.

---

### 🚀 **Job: deploy** (Continuous Deployment)

Job `deploy` adalah tahap akhir yang dilakukan setelah build sukses. Proses ini akan menjadikan aplikasi live di Azure.

1. **Checkout repository**
   Mengambil ulang kode dari repository.

2. **Deploy to Azure Web App**
   Menggunakan GitHub Action `azure/webapps-deploy@v2`.

   * **app-name**: `CCWSRESERVE`
   * **slot-name**: `Production`
   * **publish-profile**: Menggunakan `AZURE_WEBAPP_PUBLISH_PROFILE` dari GitHub Secrets
   * **images**: `nextjs-app:latest` dari `AZURE_REGISTRY_URL`


## ⚙️ Implementasi Pipeline

### Struktur File

```bash
roomreservation/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── Dockerfile
├── next.config.js
├── package.json
└── ...
```

### Cuplikan `ci-cd.yml`

```yaml
name: Next.js CI

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm run test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Build Next.js app
        run: npm run build
      - name: Check Next.js version
        run: npx next --version
      - name: Log in to Azure Container Registry
        run: echo "${{ secrets.AZURE_REGISTRY_PASSWORD }}" | docker login ${{ secrets.AZURE_REGISTRY_URL }} -u ${{ secrets.AZURE_REGISTRY_USERNAME }} --password-stdin
      - name: Build Docker image
        run: docker build -t ${{ secrets.AZURE_REGISTRY_URL }}/nextjs-app:latest .
      - name: Push Docker image to ACR
        run: docker push ${{ secrets.AZURE_REGISTRY_URL }}/nextjs-app:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3
      - name: Deploy to Azure Web App
        uses: azure/webapps-deploy@v2
        with:
          app-name: CCWSRESERVE
          slot-name: Production
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          images: '${{ secrets.AZURE_REGISTRY_URL }}/nextjs-app:latest'
```

---

## 🧪 Error / Issue GitHub Actions

| Error                      | Penyebab Umum                 | Solusi                                  |
| -------------------------- | ----------------------------- | --------------------------------------- |
| `npm ci` failed            | Versi Node mismatch           | Pastikan `.nvmrc` dan `setup-node` sama |
| `docker push` unauthorized | Salah credential ACR          | Periksa secret `AZURE_ACR_*`            |
| `webapps-deploy` failed    | Publish profile tidak valid   | Re-download publish profile dari Azure  |
| `next build` error         | Config atau ENV tidak lengkap | Cek `.env` dan `next.config.js`         |

---

## 🧑‍💻 Local Development

```bash
git clone https://github.com/AryasatyaWidyatna/roomreservation
cd roomreservation
npm install
npm run dev
```

---

## 🙋‍♀️ Maintainers

* Arayzi Rayyansyah       (5026221194)
* Dicky Febri Primadhani  (5026221036)
* M. Rafi Novyansyah      (5026221171)
* Arya Satya Widyatna     (5026221207)

---

