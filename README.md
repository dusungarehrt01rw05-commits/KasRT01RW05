# 📊 Buku Kas RT 01 / RW 05 - Dusun Gareh

Aplikasi web interaktif untuk mengelola keuangan **Buku Kas RT** dengan fitur tracking transaksi, data warga, laporan cetak, dan **integrasi otomatis ke Google Sheets**.

![Status](https://img.shields.io/badge/status-active-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.0.0-blue)

---

## 📋 Daftar Isi

- [Fitur Utama](#fitur-utama)
- [Tech Stack](#tech-stack)
- [Setup & Instalasi](#setup--instalasi)
- [Penggunaan](#penggunaan)
- [Google Sheets Integration](#-google-sheets-integration)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Lisensi](#lisensi)

---

## ✨ Fitur Utama

### 1. 📝 Manajemen Transaksi
- ✅ Tambah transaksi (Masuk/Keluar)
- ✅ Edit dan hapus transaksi
- ✅ Kategori transaksi yang dapat dikustomisasi
- ✅ Tracking saldo berjalan otomatis
- ✅ Filter transaksi berdasarkan tanggal & kategori

### 2. 👥 Manajemen Data Warga
- ✅ Database warga dengan nama, alamat, HP
- ✅ Import warga dari text area
- ✅ Edit dan hapus data warga
- ✅ Export data warga ke CSV

### 3. 📊 Laporan & Analitik
- ✅ Ringkasan keuangan (Saldo Awal, Masuk, Keluar, Akhir)
- ✅ Grafik perbandingan pemasukan vs pengeluaran
- ✅ Laporan periode dengan filter tanggal
- ✅ Print-friendly report
- ✅ Export ke CSV

### 4. 🌐 Google Sheets Integration (NEW!)
- ✅ **Auto-sync transaksi** ke Google Sheets
- ✅ **Auto-sync warga** ke Google Sheets
- ✅ **Auto-update ringkasan** keuangan
- ✅ **Export semua data** sekaligus
- ✅ Real-time synchronization

### 5. 💾 Penyimpanan Data
- ✅ LocalStorage untuk data offline
- ✅ Google Sheets untuk cloud backup
- ✅ Dapat di-export/import dengan CSV

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | React + Tailwind CSS |
| **State Management** | React Hooks (useState, useEffect) |
| **Storage** | Browser LocalStorage |
| **Cloud Sync** | Google Apps Script + Google Sheets API |
| **Export** | CSV, Print HTML |

---

## 🚀 Setup & Instalasi

### Prerequisites
- Web browser modern (Chrome, Firefox, Safari, Edge)
- Google Account (untuk Google Sheets integration)
- Internet connection (untuk sync ke Sheet)

### Instalasi Lokal

```bash
# Clone repository
git clone https://github.com/dusungarehrt01rw05-commits/KasRT01RW05.git
cd KasRT01RW05

# Switch ke branch dengan integrasi Google Sheets
git checkout google-sheets-integration

# Buka file di browser
open index.html
```

### Setup Google Sheets Integration

**Google Sheet yang digunakan:**
```
https://docs.google.com/spreadsheets/d/1sLpY4ZuvlNcuQ-GtuL9-qgp3Lhcde7pQMvgRLudxUUk/
```

**Google Apps Script (sudah aktif):**
```
https://script.google.com/macros/s/AKfycbyWWGcvvPe_OGaJ5d_lLtmT1XjF99M38tEGdPqj5-0gh2CiRAok5jW5GddzN4InUtn1/exec
```

✅ Sudah siap digunakan!

---

## 📖 Penggunaan

### Workflow Dasar

#### 1. Menambah Transaksi
```
Klik "Kelola Transaksi" 
  → Isi form: Tanggal, Kategori, Keterangan, Jenis, Nominal
  → Klik "Simpan"
  → ✓ Data tersimpan lokal & otomatis ke Google Sheets
```

#### 2. Menambah Warga
```
Klik "Kelola Warga"
  → Isi form: Nama, Alamat, Nomor HP
  → Klik "Tambah Warga"
  → ✓ Data tersimpan & otomatis ke Google Sheets
```

#### 3. Lihat Laporan
```
Klik "Laporan"
  → Pilih tanggal mulai & akhir (optional)
  → Lihat ringkasan & tabel transaksi
  → Klik "Print" atau "Export CSV"
```

---

## 🌐 Google Sheets Integration

### 4 Fungsi Utama

#### ✅ 1. sendTransaksiToSheet(transaksi)
Kirim transaksi baru ke Google Sheet

```javascript
await window.sendTransaksiToSheet({
  tgl: '2024-08-30',
  kategori: 'Iuran RT',
  ket: 'Iuran Agustus',
  jenis: 'Masuk',
  nominal: 50000,
  saldoBerjalan: 500000
});
```

#### ✅ 2. sendWargaToSheet(warga)
Kirim warga baru ke Google Sheet

```javascript
await window.sendWargaToSheet({
  nama: 'Ahmad Rizki',
  alamat: 'Jl. Merdeka No.5',
  hp: '081234567890'
});
```

#### ✅ 3. updateRingkasanSheet(ringkasan)
Update ringkasan keuangan di Google Sheet

```javascript
await window.updateRingkasanSheet({
  saldoAwalPeriode: 100000,
  totalMasuk: 500000,
  totalKeluar: 200000,
  saldoAkhir: 400000
});
```

#### ✅ 4. exportAllToSheet(transaksi, warga, ringkasan)
Export semua data sekaligus

```javascript
await window.exportAllToSheet(
  allTransaksi,
  allWarga,
  ringkasanData
);
```

---

## 📡 API Reference

### window.sendTransaksiToSheet(data)

```javascript
// Parameter
{
  tgl: string,              // "YYYY-MM-DD"
  kategori: string,
  ket: string,
  jenis: string,            // "Masuk" | "Keluar"
  nominal: number,
  saldoBerjalan: number
}

// Return
{ success: boolean, message: string }
```

### window.sendWargaToSheet(data)

```javascript
// Parameter
{
  nama: string,
  alamat: string,
  hp: string
}

// Return
{ success: boolean, message: string }
```

### window.updateRingkasanSheet(data)

```javascript
// Parameter
{
  saldoAwalPeriode: number,
  totalMasuk: number,
  totalKeluar: number,
  saldoAkhir: number
}

// Return
{ success: boolean, message: string }
```

### window.exportAllToSheet(transaksi, warga, ringkasan)

```javascript
// Parameter
transaksi: Array,    // Array of objects
warga: Array,        // Array of objects
ringkasan: Object    // Object

// Return
{ success: boolean, message: string }
```

---

## 🐛 Troubleshooting

### Sheet tidak terupdate
1. Pastikan koneksi internet stabil
2. Buka Console Browser (F12) untuk melihat error
3. Refresh Google Sheet
4. Coba manual klik "Update ke Sheet"

### CORS atau Network Error
1. Reload halaman aplikasi
2. Clear browser cache
3. Pastikan Google Apps Script masih aktif

### Test Integrasi di Console Browser

```javascript
// Cek apakah fungsi tersedia
console.log(typeof window.updateRingkasanSheet); // "function"

// Test dengan data dummy
await window.updateRingkasanSheet({
  saldoAwalPeriode: 100000,
  totalMasuk: 500000,
  totalKeluar: 200000,
  saldoAkhir: 400000
});
```

---

## 📚 Dokumentasi Lengkap

Baca file `GOOGLE_SHEETS_INTEGRATION.md` untuk:
- Setup step-by-step
- 4 opsi implementasi integrasi
- Contoh kode React lengkap
- Troubleshooting guide detail

---

## 📝 Lisensi

Proyek ini dilisensikan di bawah **MIT License**

---

## 🎯 Quick Links

- 📊 [Buka Google Sheet](https://docs.google.com/spreadsheets/d/1sLpY4ZuvlNcuQ-GtuL9-qgp3Lhcde7pQMvgRLudxUUk/)
- 📖 [Baca Google Sheets Integration Guide](GOOGLE_SHEETS_INTEGRATION.md)
- 🐛 [Report Issues](https://github.com/dusungarehrt01rw05-commits/KasRT01RW05/issues)
- 🌳 [GitHub Repository](https://github.com/dusungarehrt01rw05-commits/KasRT01RW05)

---

## ✨ Features Highlights

| Fitur | Deskripsi |
|-------|-----------|
| 📝 Transaksi | Tambah, edit, hapus transaksi dengan kategori |
| 👥 Warga | Kelola data warga (nama, alamat, HP) |
| 📊 Laporan | Ringkasan keuangan & laporan periode |
| 🌐 Google Sheets | Auto-sync ke cloud secara real-time |
| 💾 Offline Ready | Bekerja offline, sync saat online |
| 📄 Export | CSV, Print, dan Sheet integration |

---

**Status:** ✅ Production Ready  
**Version:** 1.0.0  
**Last Updated:** 2024-08-30
