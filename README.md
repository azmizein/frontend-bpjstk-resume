# BPJS Frontend

Ini adalah frontend untuk aplikasi BPJS, dibangun menggunakan Vue 3 dan Vite.

## Prasyarat
- Node.js terinstal di sistem Anda

## Cara Menjalankan Project

1. **Buka terminal dan masuk ke direktori frontend:**
   ```bash
   cd frontend
   ```

2. **Instal dependensi yang dibutuhkan:**
   ```bash
   npm install
   ```

3. **Konfigurasi Environment Variable:**
   - Salin file `.env.example` dan ubah namanya menjadi `.env`:
     ```bash
     cp .env.example .env
     ```
   - Pastikan `VITE_API_URL` mengarah ke URL backend Anda (default: `http://localhost:8080`).

4. **Jalankan development server:**
   ```bash
   npm run dev
   ```
   Aplikasi Vue akan berjalan dan dapat diakses melalui browser (biasanya di `http://localhost:5173`).

## Cara Build untuk Production

Untuk melakukan build (menghasilkan file statis yang siap di-deploy):
```bash
npm run build
```
File hasil build akan berada di dalam direktori `dist`. Anda bisa mem-preview hasil build dengan menjalankan:
```bash
npm run preview
```
