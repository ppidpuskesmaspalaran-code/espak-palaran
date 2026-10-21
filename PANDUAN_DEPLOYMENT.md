# Panduan Deployment Online E-SPAK
*(Elektronik Sistem Pemantauan Kinerja dan Analisis Kesehatan — UPTD Puskesmas Palaran)*

Aplikasi ini siap di-*deploy* secara online dan dapat diakses dari mana saja (HP, laptop, tablet) menggunakan layanan **Cloud Modern 100% GRATIS (Free Tier)**.

---

## Ringkasan Arsitektur Deployment

* **Database (MongoDB):** [MongoDB Atlas](https://www.mongodb.com/) (Free M0 Cluster — 512 MB gratis selamanya)
* **Backend API (FastAPI):** [Render.com](https://render.com/) (Web Service Python Gratis) atau [Google Cloud Run](https://cloud.google.com/run)
* **Frontend Web (React 19):** [Vercel](https://vercel.com/) (Platform React Gratis) atau [Firebase Hosting](https://firebase.google.com/)

---

## Tahap 1: Siapkan Database MongoDB Gratis (5 Menit)

1. Buka [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas/register) dan daftar/masuk akun gratis (bisa menggunakan *Sign in with Google*).
2. Buat Cluster baru:
   - Pilih opsi **M0 Free** (Gratis $0/bulan).
   - Pilih Provider: **AWS** atau **Google Cloud**, Region: **Singapore (ap-southeast-1)** atau yang terdekat.
3. Di menu **Security** $\rightarrow$ **Database Access**:
   - Klik **Add New Database User**.
   - Masukkan Username (misal: `espak_admin`) dan Password yang kuat. Simpan password ini.
4. Di menu **Security** $\rightarrow$ **Network Access**:
   - Klik **Add IP Address**.
   - Pilih **Allow Access From Anywhere** (`0.0.0.0/0`) agar backend cloud dapat terhubung.
5. Di menu **Database** $\rightarrow$ **Clusters**:
   - Klik tombol **Connect** $\rightarrow$ **Drivers** (Python).
   - Salin string koneksinya, contoh:
     ```
     mongodb+srv://espak_admin:<password>@cluster0.abcde.mongodb.net/?retryWrites=true&w=majority
     ```
   - Ganti `<password>` dengan password user database yang Anda buat tadi.

---

## Tahap 2: Unggah Kode ke GitHub (3 Menit)

Pastikan semua berkas proyek ini diunggah ke repositori GitHub Anda (bisa Private atau Public):
```bash
git init
git add .
git commit -m "feat: persiapan deployment E-SPAK"
git branch -M main
git remote add origin https://github.com/USERNAME-ANDA/espak.git
git push -u origin main
```

---

## Tahap 3: Deploy Backend di Render.com (10 Menit)

1. Buka [render.com](https://render.com/) dan login (bisa menggunakan akun GitHub).
2. Di Dashboard Render, klik **New +** $\rightarrow$ **Web Service**.
3. Pilih repositori GitHub `espak` yang sudah diunggah.
4. Atur konfigurasi berikut:
   - **Name:** `espak-backend` (atau nama lain yang Anda sukai)
   - **Region:** `Singapore`
   - **Root Directory:** `backend`
   - **Environment:** `Python 3`
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn server:app --host 0.0.0.0 --port $PORT`
   - **Instance Type:** `Free`
5. Gulir ke bawah ke bagian **Environment Variables**, klik **Add Environment Variable** dan masukkan:
   - `MONGO_URL` = *(isi dengan connection string MongoDB Atlas dari Tahap 1)*
   - `DB_NAME` = `espak`
   - `JWT_SECRET` = *(masukkan teks rahasia acak minimal 32 karakter)*
   - `CORS_ORIGINS` = `*`
6. Klik **Create Web Service**.
7. Tunggu sekitar 2-3 menit hingga status menjadi **Live**.
8. Salin URL backend Anda yang tertera di bagian atas dashboard Render, contoh:
   ```
   https://espak-backend.onrender.com
   ```
   *(Cek via browser pada `https://espak-backend.onrender.com/healthz`, jika muncul `{"status":"ok"}` maka backend sukses berjalan!)*

---

## Tahap 4: Deploy Frontend di Vercel (5 Menit)

1. Buka [vercel.com](https://vercel.com/) dan login menggunakan akun GitHub Anda.
2. Klik **Add New...** $\rightarrow$ **Project**.
3. Pilih repositori `espak` dari daftar GitHub Anda, lalu klik **Import**.
4. Di halaman konfigurasi:
   - **Project Name:** `espak-palaran` (atau sesuai keinginan)
   - **Root Directory:** Klik **Edit** dan pilih folder **`frontend`**
   - **Framework Preset:** Terdeteksi otomatis sebagai `Create React App`
5. Buka bagian **Environment Variables** dan tambahkan:
   - Key: `REACT_APP_BACKEND_URL`
   - Value: URL backend dari Tahap 3 (contoh: `https://espak-backend.onrender.com` — *tanpa tanda garis miring di akhir*).
6. Klik tombol **Deploy**.
7. Tunggu build selesai (~1-2 menit). Vercel akan memberikan domain publik gratis, contoh:
   ```
   https://espak-palaran.vercel.app
   ```

---

## Tahap 5: Akses & Uji Coba Webapp

Buka URL Vercel di browser (HP atau laptop). Sistem otomatis melakukan *seeding* akun awal:

| Role | Username / NIP | Password Bawaan | Hak Akses |
| :--- | :--- | :--- | :--- |
| **Admin** | `admin` | `admin123` | Akses penuh (Pengguna, Pengaturan, Verifikasi, SPM, Cuti, SIP, Backup) |
| **Kepala Puskesmas** | `kepala` | `kepala123` | Dashboard Eksekutif, Monitoring JPL/Pegawai/SIP, EWS, Policy Brief |
| **PJ KIA** | `pjkia` | `pj123` | Input data SPM KIA, Monitoring SPM, Laporan |
| **PJ TB / P2P** | `pjtb` | `pj123` | Input data SPM P2P, Monitoring SPM, Laporan |
| **Pegawai (Nakes)** | `pegawai1` | `pegawai123` | Upload Sertifikat, Permohonan Cuti, Input SIP |

*(Anda dapat mengganti password dan membuat akun baru di menu **Pengaturan** $\rightarrow$ **Manajemen Pengguna**)*.

---

## Opsi Alternatif: Google Cloud Run & Firebase Hosting

Jika instansi Anda mewajibkan penggunaan layanan Google Cloud (GCP):
1. **Backend ke Google Cloud Run:**
   - Gunakan berkas [`backend/Dockerfile`](backend/Dockerfile) yang telah kami buat.
   - Di terminal Google Cloud SDK:
     ```bash
     gcloud run deploy espak-backend --source backend --platform managed --region asia-southeast2 --allow-unauthenticated
     ```
2. **Frontend ke Firebase Hosting:**
   - Gunakan berkas [`frontend/firebase.json`](frontend/firebase.json) yang telah kami buat.
   - Jalankan `npm run build` di folder `frontend`.
   - Jalankan `firebase deploy --only hosting`.
