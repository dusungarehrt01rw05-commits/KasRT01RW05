# 📊 Panduan Integrasi Google Sheets

Dokumentasi lengkap untuk mengintegrasikan aplikasi Buku Kas RT dengan Google Sheets secara otomatis.

---

## 📌 Daftar Isi

1. [Setup Awal](#setup-awal)
2. [Fungsi yang Tersedia](#fungsi-yang-tersedia)
3. [Cara Integrasi](#cara-integrasi)
4. [Contoh Implementasi](#contoh-implementasi)
5. [Troubleshooting](#troubleshooting)

---

## 🔧 Setup Awal

### Google Apps Script URL
```
https://script.google.com/macros/s/AKfycbyWWGcvvPe_OGaJ5d_lLtmT1XjF99M38tEGdPqj5-0gh2CiRAok5jW5GddzN4InUtn1/exec
```

Google Sheet yang terhubung:
```
https://docs.google.com/spreadsheets/d/1sLpY4ZuvlNcuQ-GtuL9-qgp3Lhcde7pQMvgRLudxUUk/
```

---

## 📋 Fungsi yang Tersedia

### 1. `sendTransaksiToSheet(transaksi)`
**Deskripsi:** Mengirim satu transaksi baru ke Google Sheet

**Parameter:**
```javascript
{
  tgl: string,              // Format: "2024-08-30"
  kategori: string,         // Misal: "Iuran RT"
  ket: string,             // Keterangan transaksi
  jenis: string,           // "Masuk" atau "Keluar"
  nominal: number,         // Nominal uang
  saldoBerjalan: number    // Saldo setelah transaksi
}
```

**Contoh Penggunaan:**
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

---

### 2. `sendWargaToSheet(warga)`
**Deskripsi:** Mengirim data warga baru ke Google Sheet

**Parameter:**
```javascript
{
  nama: string,      // Nama warga
  alamat: string,    // Alamat warga
  hp: string        // Nomor HP warga
}
```

**Contoh Penggunaan:**
```javascript
await window.sendWargaToSheet({
  nama: 'Ahmad Rizki',
  alamat: 'Jl. Merdeka No.5',
  hp: '081234567890'
});
```

---

### 3. `updateRingkasanSheet(ringkasan)` ⭐ PENTING
**Deskripsi:** Update ringkasan keuangan (Saldo Awal, Total Masuk, Total Keluar, Saldo Akhir)

**Parameter:**
```javascript
{
  saldoAwalPeriode: number,  // Saldo di awal periode
  totalMasuk: number,        // Total uang masuk
  totalKeluar: number,       // Total uang keluar
  saldoAkhir: number         // Saldo di akhir periode
}
```

**Contoh Penggunaan:**
```javascript
await window.updateRingkasanSheet({
  saldoAwalPeriode: 100000,
  totalMasuk: 500000,
  totalKeluar: 200000,
  saldoAkhir: 400000
});
```

**⚠️ Penting:** Ringkasan akan di-update di baris:
- B2: Saldo Awal
- B3: Total Masuk
- B4: Total Keluar
- B5: Saldo Akhir

---

### 4. `exportAllToSheet(transaksi, warga, ringkasan)`
**Deskripsi:** Export semua data sekaligus ke Google Sheet

**Parameter:**
```javascript
{
  transaksi: Array,    // Array dari seluruh transaksi
  warga: Array,        // Array dari seluruh warga
  ringkasan: Object    // Object ringkasan keuangan
}
```

**Contoh Penggunaan:**
```javascript
await window.exportAllToSheet(
  allTransaksi,  // Array transaksi
  allWarga,      // Array warga
  ringkasanData  // Object ringkasan
);
```

---

## 🔌 Cara Integrasi

### Opsi 1: Trigger Saat Halaman Load (Recommended)

Tambahkan di `useEffect` saat komponen pertama kali render:

```javascript
useEffect(() => {
  // Panggil fungsi untuk update ringkasan ke Sheet saat aplikasi load
  const updateSheet = async () => {
    try {
      const result = await window.updateRingkasanSheet({
        saldoAwalPeriode: ringkasan.saldoAwalPeriode,
        totalMasuk: ringkasan.totalMasuk,
        totalKeluar: ringkasan.totalKeluar,
        saldoAkhir: ringkasan.saldoAkhir
      });
      
      if (result.success) {
        console.log('✓ Ringkasan berhasil disinkronkan dengan Google Sheets');
      } else {
        console.error('✗ Gagal mensinkronkan:', result.message);
      }
    } catch (error) {
      console.error('Error:', error);
    }
  };

  updateSheet();
}, []); // Dependency array kosong = hanya jalan sekali saat load
```

---

### Opsi 2: Trigger Setiap Kali Ada Transaksi Baru

Tambahkan setelah transaksi ditambahkan ke state:

```javascript
// Fungsi untuk tambah transaksi
const tambahTransaksi = async (transaksiData) => {
  // 1. Hitung saldo berjalan
  const saldoBerjalan = calculateSaldoBerjalan(transaksiData);
  
  // 2. Simpan ke state lokal
  setTransaksi(prev => [...prev, {
    ...transaksiData,
    saldoBerjalan
  }]);
  
  // 3. Kirim transaksi ke Google Sheets
  try {
    const result = await window.sendTransaksiToSheet({
      tgl: transaksiData.tgl,
      kategori: transaksiData.kategori,
      ket: transaksiData.ket,
      jenis: transaksiData.jenis,
      nominal: transaksiData.nominal,
      saldoBerjalan: saldoBerjalan
    });
    
    if (result.success) {
      console.log('✓ Transaksi berhasil dikirim ke Google Sheets');
    } else {
      alert('⚠️ ' + result.message);
    }
  } catch (error) {
    console.error('Error:', error);
  }
  
  // 4. Hitung ulang ringkasan
  const newRingkasan = calculateRingkasan();
  setRingkasan(newRingkasan);
  
  // 5. Update ringkasan di Google Sheets
  try {
    await window.updateRingkasanSheet({
      saldoAwalPeriode: newRingkasan.saldoAwalPeriode,
      totalMasuk: newRingkasan.totalMasuk,
      totalKeluar: newRingkasan.totalKeluar,
      saldoAkhir: newRingkasan.saldoAkhir
    });
    
    console.log('✓ Ringkasan berhasil diupdate di Google Sheets');
  } catch (error) {
    console.error('Error:', error);
  }
};
```

---

### Opsi 3: Trigger dengan Tombol "Update ke Sheet"

Tambahkan tombol khusus di UI:

```javascript
// Komponen Button
<button 
  onClick={async () => {
    try {
      const result = await window.updateRingkasanSheet({
        saldoAwalPeriode: ringkasan.saldoAwalPeriode,
        totalMasuk: ringkasan.totalMasuk,
        totalKeluar: ringkasan.totalKeluar,
        saldoAkhir: ringkasan.saldoAkhir
      });
      
      if (result.success) {
        alert('✓ Ringkasan berhasil diupdate di Google Sheets!');
      } else {
        alert('✗ ' + result.message);
      }
    } catch (error) {
      alert('Error: ' + error.message);
    }
  }}
  className="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
>
  📊 Update ke Google Sheets
</button>
```

---

### Opsi 4: Trigger Setiap Kali Ringkasan Berubah

Gunakan `useEffect` dengan dependency array yang berisi ringkasan:

```javascript
useEffect(() => {
  const updateRingkasanAuto = async () => {
    try {
      const result = await window.updateRingkasanSheet({
        saldoAwalPeriode: ringkasan.saldoAwalPeriode,
        totalMasuk: ringkasan.totalMasuk,
        totalKeluar: ringkasan.totalKeluar,
        saldoAkhir: ringkasan.saldoAkhir
      });
      
      if (result.success) {
        console.log('✓ Ringkasan auto-update ke Google Sheets');
      }
    } catch (error) {
      console.error('Error:', error);
    }
  };

  // Jangan trigger saat pertama kali load (gunakan flag jika diperlukan)
  updateRingkasanAuto();
}, [ringkasan]); // Update setiap kali ringkasan berubah
```

---

## 💡 Contoh Implementasi Lengkap

Berikut adalah contoh implementasi **lengkap** dengan React:

```javascript
import { useState, useEffect } from 'react';

export default function BukuKasApp() {
  const [transaksi, setTransaksi] = useState([]);
  const [warga, setWarga] = useState([]);
  const [ringkasan, setRingkasan] = useState({
    saldoAwalPeriode: 0,
    totalMasuk: 0,
    totalKeluar: 0,
    saldoAkhir: 0
  });
  const [isLoading, setIsLoading] = useState(false);

  // 1️⃣ Trigger saat aplikasi pertama kali load
  useEffect(() => {
    const syncOnLoad = async () => {
      console.log('📊 Menyinkronkan data ke Google Sheets...');
      
      try {
        const result = await window.updateRingkasanSheet(ringkasan);
        if (result.success) {
          console.log('✓ Sinkronisasi berhasil');
        }
      } catch (error) {
        console.error('Sinkronisasi gagal:', error);
      }
    };

    syncOnLoad();
  }, []); // Hanya jalan sekali saat load

  // Helper function untuk hitung saldo berjalan
  const calculateSaldoBerjalan = (newTransaksi) => {
    const totalSebelumnya = transaksi.reduce((sum, t) => {
      if (t.jenis === 'Masuk') return sum + t.nominal;
      else return sum - t.nominal;
    }, ringkasan.saldoAwalPeriode);

    if (newTransaksi.jenis === 'Masuk') {
      return totalSebelumnya + newTransaksi.nominal;
    } else {
      return totalSebelumnya - newTransaksi.nominal;
    }
  };

  // Helper function untuk hitung ringkasan
  const calculateRingkasan = (allTransaksi = transaksi) => {
    const totalMasuk = allTransaksi
      .filter(t => t.jenis === 'Masuk')
      .reduce((sum, t) => sum + t.nominal, 0);

    const totalKeluar = allTransaksi
      .filter(t => t.jenis === 'Keluar')
      .reduce((sum, t) => sum + t.nominal, 0);

    const saldoAkhir = ringkasan.saldoAwalPeriode + totalMasuk - totalKeluar;

    return {
      saldoAwalPeriode: ringkasan.saldoAwalPeriode,
      totalMasuk,
      totalKeluar,
      saldoAkhir
    };
  };

  // 2️⃣ Tambah Transaksi (dengan auto-update ke Sheet)
  const handleTambahTransaksi = async (formData) => {
    setIsLoading(true);

    try {
      // Hitung saldo berjalan
      const saldoBerjalan = calculateSaldoBerjalan(formData);

      // Simpan ke state lokal
      const newTransaksi = {
        id: Date.now(),
        ...formData,
        saldoBerjalan
      };
      setTransaksi(prev => [...prev, newTransaksi]);

      // Kirim ke Google Sheets
      const result = await window.sendTransaksiToSheet({
        tgl: formData.tgl,
        kategori: formData.kategori,
        ket: formData.ket,
        jenis: formData.jenis,
        nominal: formData.nominal,
        saldoBerjalan
      });

      if (!result.success) {
        throw new Error(result.message);
      }

      console.log('✓ Transaksi berhasil disimpan');

      // Update ringkasan
      const newRingkasan = calculateRingkasan([...transaksi, newTransaksi]);
      setRingkasan(newRingkasan);

      // Update ringkasan di Google Sheets
      const ringkasanResult = await window.updateRingkasanSheet(newRingkasan);
      
      if (ringkasanResult.success) {
        console.log('✓ Ringkasan berhasil diupdate');
      }

    } catch (error) {
      console.error('Error:', error);
      alert('❌ Gagal menyimpan transaksi: ' + error.message);
    } finally {
      setIsLoading(false);
    }
  };

  // 3️⃣ Tambah Warga (dengan auto-save ke Sheet)
  const handleTambahWarga = async (formData) => {
    setIsLoading(true);

    try {
      // Simpan ke state lokal
      const newWarga = {
        id: Date.now(),
        ...formData
      };
      setWarga(prev => [...prev, newWarga]);

      // Kirim ke Google Sheets
      const result = await window.sendWargaToSheet(formData);

      if (!result.success) {
        throw new Error(result.message);
      }

      console.log('✓ Warga berhasil disimpan');
      alert('✓ Warga berhasil ditambahkan');

    } catch (error) {
      console.error('Error:', error);
      alert('❌ Gagal menambah warga: ' + error.message);
    } finally {
      setIsLoading(false);
    }
  };

  // 4️⃣ Manual Update Ringkasan ke Sheet
  const handleUpdateManual = async () => {
    setIsLoading(true);

    try {
      const result = await window.updateRingkasanSheet(ringkasan);

      if (result.success) {
        alert('✓ Ringkasan berhasil diupdate ke Google Sheets!');
      } else {
        throw new Error(result.message);
      }

    } catch (error) {
      alert('❌ Error: ' + error.message);
    } finally {
      setIsLoading(false);
    }
  };

  // 5️⃣ Export Semua Data
  const handleExportAll = async () => {
    setIsLoading(true);

    try {
      const result = await window.exportAllToSheet(transaksi, warga, ringkasan);

      if (result.success) {
        alert('✓ Semua data berhasil di-export ke Google Sheets!');
      } else {
        throw new Error(result.message);
      }

    } catch (error) {
      alert('❌ Error: ' + error.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="p-6">
      <h1 className="text-3xl font-bold mb-6">📊 Buku Kas RT</h1>

      {/* Ringkasan */}
      <div className="grid grid-cols-4 gap-4 mb-6">
        <div className="bg-blue-100 p-4 rounded">
          <p className="text-sm text-gray-600">Saldo Awal</p>
          <p className="text-2xl font-bold">Rp {ringkasan.saldoAwalPeriode.toLocaleString('id-ID')}</p>
        </div>
        <div className="bg-green-100 p-4 rounded">
          <p className="text-sm text-gray-600">Pemasukan</p>
          <p className="text-2xl font-bold">Rp {ringkasan.totalMasuk.toLocaleString('id-ID')}</p>
        </div>
        <div className="bg-red-100 p-4 rounded">
          <p className="text-sm text-gray-600">Pengeluaran</p>
          <p className="text-2xl font-bold">Rp {ringkasan.totalKeluar.toLocaleString('id-ID')}</p>
        </div>
        <div className="bg-purple-100 p-4 rounded">
          <p className="text-sm text-gray-600">Saldo Akhir</p>
          <p className="text-2xl font-bold">Rp {ringkasan.saldoAkhir.toLocaleString('id-ID')}</p>
        </div>
      </div>

      {/* Tombol-tombol */}
      <div className="flex gap-2 mb-6">
        <button
          onClick={handleUpdateManual}
          disabled={isLoading}
          className="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600 disabled:bg-gray-400"
        >
          {isLoading ? '⏳ Mengirim...' : '📤 Update Ringkasan ke Sheet'}
        </button>
        <button
          onClick={handleExportAll}
          disabled={isLoading}
          className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:bg-gray-400"
        >
          {isLoading ? '⏳ Exporting...' : '📊 Export Semua Data'}
        </button>
      </div>

      {/* Daftar Transaksi */}
      <div className="mb-6">
        <h2 className="text-xl font-bold mb-4">Transaksi</h2>
        {/* Form tambah transaksi dan list transaksi di sini */}
      </div>

      {/* Daftar Warga */}
      <div>
        <h2 className="text-xl font-bold mb-4">Data Warga</h2>
        {/* Form tambah warga dan list warga di sini */}
      </div>
    </div>
  );
}
```

---

## 🚨 Troubleshooting

### ❌ Error: "GOOGLE_APPS_SCRIPT_URL is not defined"
**Solusi:** Pastikan Anda sudah berada di halaman HTML yang sudah di-update dengan Google Sheets integration

### ❌ Error: "Action tidak dikenali"
**Solusi:** Periksa parameter yang dikirim sudah sesuai dengan format

### ❌ Sheet tidak ter-update
**Solusi:**
1. Pastikan nama sheet sudah benar (Transaksi, Warga, Ringkasan)
2. Cek console browser untuk error message lebih detail
3. Buka Google Sheet, refresh halaman

### ❌ CORS Error
**Solusi:** Google Apps Script sudah dikonfigurasi untuk menerima request dari mana saja

### ✅ Test Integrasi
Buka Console Browser (F12) dan jalankan:
```javascript
// Test fungsi tersedia
console.log(typeof window.updateRingkasanSheet); // Seharusnya "function"

// Test dengan data dummy
await window.updateRingkasanSheet({
  saldoAwalPeriode: 100000,
  totalMasuk: 500000,
  totalKeluar: 200000,
  saldoAkhir: 400000
});
```

---

## 📚 Referensi

- **Google Apps Script Docs:** https://developers.google.com/apps-script
- **Google Sheets API:** https://developers.google.com/sheets/api
- **React Hooks Guide:** https://react.dev/reference/react

---

**Terakhir diupdate:** 2024-08-30
**Status:** ✅ Production Ready
