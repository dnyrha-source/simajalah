# SiMajalah — Pencatatan Media & Tagihan Perpustakaan

Aplikasi pencatatan ceklist penerimaan media (koran/tabloid/majalah) dan tagihan
langganan, dengan database Firebase Firestore dan upload kuitansi ke Google Drive.

## Struktur

- `index.html` — seluruh aplikasi (HTML + CSS + JS dalam satu file, tidak perlu build step)
- `vercel.json` — konfigurasi deploy Vercel

## Sebelum deploy: isi konfigurasi

Buka `index.html`, cari tag `<script type="module">`, lalu isi 3 bagian berikut
dengan nilai milik Anda sendiri:

### 1. Firebase

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```
Didapat dari: Firebase Console → Project Settings → Your apps → SDK setup and configuration.

Pastikan juga:
- **Firestore Database** sudah dibuat (Firebase Console → Firestore Database → Create database)
- **Security Rules** Firestore mengizinkan akses (lihat bagian Keamanan di bawah)

### 2. Google OAuth Client ID (untuk login Google Drive)

```js
const GOOGLE_CLIENT_ID = "....apps.googleusercontent.com";
```
Didapat dari: Google Cloud Console → APIs & Services → Credentials → Create Credentials → OAuth Client ID → Web application.

**Penting:** domain Vercel Anda (mis. `simajalah.vercel.app`) harus didaftarkan
di kolom **Authorized JavaScript origins** pada OAuth Client tersebut, atau
proses Hubungkan Drive akan gagal/ditolak browser.

### 3. ID Folder Google Drive bersama

```js
const GOOGLE_DRIVE_FOLDER_ID = "...";
```
Ambil dari URL folder Drive: `drive.google.com/drive/folders/INI_ID_NYA`
Pastikan **Google Drive API** sudah diaktifkan di project Google Cloud yang sama
(APIs & Services → Library → cari "Google Drive API" → Enable).

## Login aplikasi (bukan login Google)

Login username/password tetap untuk masuk ke aplikasi (terpisah dari OAuth Google
yang hanya dipakai untuk upload kuitansi). Cari dan ubah sesuai kebutuhan:

```js
const LOGIN_USERNAME = "admin";
const LOGIN_PASSWORD = "perpustakaan123";
```

⚠️ Catatan keamanan: nilai ini terlihat siapa pun yang membuka "View Page Source",
karena ini aplikasi statis tanpa server. Cocok untuk mencegah akses orang awam,
bukan proteksi tingkat tinggi. Untuk keamanan lebih kuat, perlu migrasi ke
Firebase Authentication (bisa diminta dikembangkan lebih lanjut).

## Cara deploy ke Vercel

1. Push folder ini ke repository GitHub baru
2. Buka [vercel.com](https://vercel.com) → New Project → Import dari GitHub
3. Pilih repository ini, biarkan semua pengaturan default (tidak perlu build command,
   karena ini static HTML), klik Deploy
4. Setelah selesai, Vercel memberi URL seperti `https://nama-project.vercel.app`
5. Daftarkan URL tersebut ke **Authorized JavaScript origins** di Google Cloud Console
   (langkah 2 di atas), supaya Hubungkan Drive bisa berfungsi di domain itu

## Keamanan Firestore (rules sementara untuk testing)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

Rules ini terbuka untuk siapa saja yang tahu config Anda — cukup untuk uji coba,
**bukan untuk produksi jangka panjang**. Setelah aplikasi terbukti berjalan baik,
disarankan memperketat rules (misalnya membatasi field, ukuran data, atau
menambahkan Firebase Authentication agar hanya pengguna terverifikasi yang
bisa menulis data).
