# 🏨 CCWS Room Reservation

[![CI/CD](https://github.com/AryasatyaWidyatna/roomreservation/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/AryasatyaWidyatna/roomreservation/actions)
[![Next.js](https://img.shields.io/badge/Next.js-13-blue)](https://nextjs.org/)
[![Deploy to Azure](https://img.shields.io/badge/Azure-App%20Service-blue)](https://azure.microsoft.com)

Sistem pemesanan ruang yang digunakan di lingkungan **CCWS** untuk mengatur jadwal dan ketersediaan ruangan. Aplikasi ini dibangun menggunakan **Next.js** dan di-*deploy* secara otomatis melalui pipeline **CI/CD GitHub Actions ke Azure App Service**.

🔗 **Live Demo**: [ccwsreserve-ftcsf2fefghphxc2.indonesiacentral-01.azurewebsites.net](https://ccwsreserve-ftcsf2fefghphxc2.indonesiacentral-01.azurewebsites.net)  
📦 **Repository**: [github.com/AryasatyaWidyatna/roomreservation](https://github.com/AryasatyaWidyatna/roomreservation)

---

## 📌 Scope & Objectives

- **Lingkup**:
  - Frontend menggunakan Next.js
  - CI/CD menggunakan GitHub Actions
  - Deployment ke Azure App Service dengan Docker container
- **Tujuan**:
  - Otomatisasi build, test, dan deploy
  - Monitoring kesehatan deployment
  - Fleksibilitas untuk dikembangkan lebih lanjut (eks: login, integrasi SSO)

---

## 🛠️ CI/CD Pipeline Architecture

```mermaid
flowchart LR
    A[Developer Push Code] --> B[GitHub Repo]
    B --> C[CI/CD Pipeline (GitHub Actions)]
    C --> D[Build Docker Image]
    D --> E[Push to Azure Container Registry]
    E --> F[Deploy to Azure App Service]
    F --> G[User Access Web App]
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

👉 Hasil dari tahap ini adalah **aplikasi dapat diakses secara publik** melalui Azure Web App yang telah dikonfigurasi.
---

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
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - name: Build & push Docker image
        run: |
          docker build -t <your-acr>.azurecr.io/ccwsapp:latest .
          echo "${{ secrets.AZURE_ACR_PASSWORD }}" | docker login <your-acr>.azurecr.io -u ${{ secrets.AZURE_ACR_USERNAME }} --password-stdin
          docker push <your-acr>.azurecr.io/ccwsapp:latest
      - name: Deploy to Azure
        uses: azure/webapps-deploy@v2
        with:
          app-name: ccwsreserve
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          images: '<your-acr>.azurecr.io/ccwsapp:latest'
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

## 📜 License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.

---

## 🙋‍♀️ Maintainers

* Arya Satya Widyatna
* Dicky Febri (DevOps Engineer)

---

Kalau kamu ingin aku bantu generate `README.md`-nya langsung dalam bentuk file siap pakai atau mau ditambahin bagian *env setup*, tinggal bilang aja ya!
