# 🎉 Growth Ops Master Kit: Dokumen Master Infrastruktur & Otomasi

**Klasifikasi Dokumen**: Rahasia / Master Deliverable  
**Hak Akses**: Eksklusif Pembeli Terverifikasi  

Selamat datang di paket lengkap sistem pelacakan *Unit Economics* dan Otomasi Operasional Pemasaran Anda. Dokumen ini dirancang untuk menghilangkan pekerjaan manual (*copy-paste* data), menjaga kebersihan data dari prospek ganda/spam, dan memberikan kejelasan instan pada angka **Cost-Per-Lead (CPL)** serta kecepatan distribusi prospek ke tim sales Anda.

Ikuti 3 modul implementasi di bawah ini untuk memasang sistem pada bisnis Anda dalam waktu kurang dari 15 menit.

---

## 📊 Modul 1: Skema Database 10-Metrik Unit Economics

Untuk melacak secara tepat bagaimana anggaran iklan Anda di bagian atas corong (*top-of-funnel*) diubah menjadi penjualan nyata di tingkat ritel/dealer, siapkan database utama Anda (Google Sheets atau Feishu Base) dengan kolom-kolom wajib di bawah ini.

### 📑 Tab 1: `Ledger_Biaya_Kampanye`
Melacak tingkat pembakaran anggaran (*burn rate*) harian operasional Anda.
* `Tanggal` *(Format: YYYY-MM-DD)*
* `Nama_Inisiatif` *(Contoh: Iklan Meta Lookalike, Pameran Cabang Surabaya)*
* `Alokasi_Anggaran_Pusat` *(Rp - Dana yang disalurkan langsung dari kantor pusat)*
* `Biaya_Subsidi_Dealer` *(Rp - Subsidi langsung untuk mendukung promosi mitra)*
* `Pengeluaran_Lokal_Mandiri` *(Rp - Dana tambahan yang dikeluarkan sendiri oleh cabang/mitra)*
* `Total_Burn_Harian` *(Rumus: `=SUM(C2:E2)`)*
* `Total_Leads_Kotor` *(Angka mutlak jumlah entri form yang masuk)*
* `Leads_Unik_Terverifikasi` *(Jumlah prospek bersih setelah sistem menghapus duplikat)*
* `CPL_Harian_Mutlak` *(Rumus: `=F2/H2` — Mendeteksi lonjakan biaya akuisisi seketika)*
* `Durasi_Hari_Aktif` *(Angka kumulatif total hari kampanye berjalan)*

### 📑 Tab 2: `Ledger_Hasil_Penjualan`
Menghubungkan prospek iklan langsung ke performa penutupan penjualan (*closing*).
* `Lead_ID` *(Dibuat otomatis oleh sistem otomasi script)*
* `Data_Intensi_Klien` *(Data string JSON yang diekstrak dari jawaban kuesioner)*
* `Sales_Ditugaskan` *(Nama sales yang berhasil mengklaim prospek)*
* `Waktu_Klaim_Detik` *(Angka mutlak pelacakan kecepatan respons sales)*
* `Status_Penjualan_Akhir` *(Dropdown: `Menunggu_Kontak` / `Jadwal_Demo` / `Closing_Menang` / `Gagal_Tidak_Respon`)*
* `Nomor_Seri_Unit` *(Tautan pelacakan fisik perangkat/IoT jika ada)*

---

## ⚡ Modul 2: Script Otomasi Intake Leads & Filter Duplikat

Hentikan pembayaran langganan Zapier atau Make.com bulanan yang mahal hanya untuk memindahkan prospek dari Google Forms ke database Anda. Script mandiri ini mencegat data yang masuk, menyaring entri ganda secara otomatis, merapikan format data, dan mengirimkannya ke Webhook/WhatsApp Anda secara *real-time*.

### 🛠️ Petunjuk Pemasangan:
1. Buka Google Sheet yang terhubung dengan Google Form penerima prospek Anda.
2. Pada menu atas, klik **Extensions (Ekstensi) > Apps Script**.
3. Hapus semua kode bawaan dan tempel (*paste*) script produksi di bawah ini.
4. Ganti tautan pada `CONFIG.WEBHOOK_URL` dengan alamat Webhook tujuan Anda (Qontak/Make/CRM).
5. Pada menu kiri, klik ikon **Triggers (Pemicu / ⏰)** -> **Add Trigger (Tambahkan Pemicu)** -> Pilih fungsi `onFormSubmitTrigger` -> Sumber Acara: `Dari spreadsheet` -> Jenis Acara: `Saat mengirim formulir` -> Klik **Save (Simpan)**.

```javascript
/**
 * Script Master Intake Leads & Filter Duplikat
 * Mencegat pengiriman Google Form, menjaga kebersihan data CPL, dan mengirim Webhook tanpa biaya bulanan.
 */

const CONFIG = {
  // Ganti dengan URL endpoint Webhook penerima prospek bisnis Anda
  WEBHOOK_URL: "https://endpoint-webhook-sistem-routing-anda.com/api/v1/lead",
  AKTIFKAN_CEK_DUPLIKAT: true,
  INDEX_KOLOM_EMAIL: 2, // Asumsi Kolom B berisi Email Klien (Dimulai dari angka 1)
  INDEX_KOLOM_TELEPON: 3, // Asumsi Kolom C berisi Nomor WhatsApp Klien
  NAMA_SHEET_LOG: "Log_Sistem"
};

function onFormSubmitTrigger(e) {
  try {
    const sheet = e.range.getSheet();
    const responses = e.namedValues; 
    const rowIdx = e.range.getRow();
    const values = e.values;

    // 1. PENYARINGAN PROSPEK DUPLIKAT
    if (CONFIG.AKTIFKAN_CEK_DUPLIKAT) {
      if (cekEntriDuplikat(sheet, rowIdx, values)) {
        tulisLogSistem("PERINGATAN: Duplikat Diabaikan", `Baris ${rowIdx} diabaikan agar penghitungan CPL tetap akurat.`);
        // Memberikan warna latar kuning pada baris duplikat agar mudah dipantau admin
        sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#FFF3CD");
        return; 
      }
    }

    // 2. NORMALISASI FORMAT DATA
    const payload = {
      timestamp: new Date().toISOString(),
      lead_id: `LD-${Date.now()}`,
      sumber_form: sheet.getName(),
      data_intensi: {}
    };

    // Membersihkan karakter spasi dan menyusun properti database yang rapi
    for (let key in responses) {
      const kunciBersih = key.trim().replace(/\s+/g, "_").toLowerCase();
      payload.data_intensi[kunciBersih] = responses[key].join(", ").trim();
    }

    // 3. PENGIRIMAN WEBHOOK TANPA BIAYA
    const options = {
      method: "post",
      contentType: "application/json",
      payload: JSON.stringify(payload),
      muteHttpExceptions: true
    };

    const response = UrlFetchApp.fetch(CONFIG.WEBHOOK_URL, options);
    const responseCode = response.getResponseCode();

    if (responseCode >= 200 && responseCode < 300) {
      tulisLogSistem("SUKSES: Terkirim", `Lead ID ${payload.lead_id} berhasil diteruskan. Status: ${responseCode}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#D1E7DD"); // Warna hijau tanda sukses
    } else {
      tulisLogSistem("ERROR: Webhook Gagal", `Gagal meneruskan data prospek. Status: ${responseCode}`);
      sheet.getRange(rowIdx, 1, 1, sheet.getLastColumn()).setBackground("#F8D7DA"); // Warna merah tanda error
    }

  } catch (error) {
    tulisLogSistem("KRITIS: Sistem Error", error.toString());
  }
}

function cekEntriDuplikat(sheet, barisSaatIni, nilaiBaris) {
  const dataSebelumnya = sheet.getRange(2, 1, barisSaatIni - 2, sheet.getLastColumn()).getValues();
  const targetEmail = nilaiBaris[CONFIG.INDEX_KOLOM_EMAIL - 1]?.toString().toLowerCase().trim();
  const targetTelepon = nilaiBaris[CONFIG.INDEX_KOLOM_TELEPON - 1]?.toString().replace(/\D/g, "");

  for (let i = 0; i < dataSebelumnya.length; i++) {
    const emailLama = dataSebelumnya[i][CONFIG.INDEX_KOLOM_EMAIL - 1]?.toString().toLowerCase().trim();
    const teleponLama = dataSebelumnya[i][CONFIG.INDEX_KOLOM_TELEPON - 1]?.toString().replace(/\D/g, "");

    if ((targetEmail && targetEmail === emailLama) || (targetTelepon && targetTelepon === teleponLama)) {
      return true;
    }
  }
  return false;
}

function tulisLogSistem(jenisAudit, detailSistem) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sheetLog = ss.getSheetByName(CONFIG.NAMA_SHEET_LOG);
  
  if (!sheetLog) {
    sheetLog = ss.insertSheet(CONFIG.NAMA_SHEET_LOG);
    sheetLog.appendRow(["Waktu Sistem", "Status Audit", "Detail Operasional"]);
    sheetLog.getRange("A1:C1").setFontWeight("bold").setBackground("#2B2B2B").setFontColor("#FFFFFF");
  }
  
  sheetLog.appendRow([new Date().toLocaleString(), jenisAudit, detailSistem]);
}
```

---

## 🏆 Modul 3: Arsitektur Logika Distribusi WhatsApp "Competition Mode"

Sistem penugasan prospek tradisional (*round-robin*) sering gagal karena membiarkan tim sales menunda menghubungi prospek tanpa konsekuensi. Untuk mencapai kecepatan respons maksimal, terapkan skema **Competition Mode** ini pada sistem perutean Webhook Make.com atau API WhatsApp bisnis Anda.

### 🔄 Alur Logika Sistem:
1. **Pemberitahuan Serentak**: Webhook penerima langsung mengirimkan pesan notifikasi prospek baru ke 3 sales terbaik Anda secara bersamaan di WhatsApp menggunakan *Template Pesan Interaktif* yang memiliki tombol khusus `[ ⚡ Klaim Leads ]`.
2. **Verifikasi Siapa Cepat Dia Dapat**: Sistem *backend* pusat mencegat balasan saat tombol ditekan oleh sales.
3. **Sistem Penguncian Eksekusi**: Sistem memverifikasi apakah status prospek tersebut masih `BELUM_DIKLAIM`.
   * **Jika Valid**: Status prospek langsung diubah menjadi `DIKLAIM` atas nama sales tersebut. Sistem membalas dengan memberikan nomor WhatsApp dan email lengkap prospek untuk dihubungi.
   * **Jika Terlambat**: Sistem memblokir permintaan dan mengirimkan pesan langsung ke sales yang lambat: *"❌ TERLAMBAT: Prospek ini sudah diamankan oleh Sales [Nama] tepat [X.X] detik lebih cepat dari Anda."*

### 🗺️ Skema Topologi Alur Distribusi:
```text
[ Webhook Prospek Masuk ] ──► [ Ledger Pusat: Tandai BELUM_DIKLAIM ]
                                     │
       ┌─────────────────────────────┼─────────────────────────────┐
       ▼                             ▼                             ▼
[ Notifikasi: Sales 1 ]       [ Notifikasi: Sales 2 ]       [ Notifikasi: Sales 3 ]
       │                             │                             │
       └─────────────────────────────┼─────────────────────────────┘
                                     ▼
                     [ Tombol Pertama Ditekan Sales ]
                                     │
                                     ▼
                    [ Penguncian Status ke DIKLAIM ]
                                     │
                  ┌──────────────────┴──────────────────┐
                  ▼                                     ▼
   [ Buka Data Kontak ke Pemenang ]      [ Notifikasi Terlambat ke Sales Lain ]
```

---

## 📞 Bantuan & Pemasangan Khusus
Membutuhkan bantuan untuk mengintegrasikan arsitektur ini secara langsung ke ekosistem CRM perusahaan, infrastruktur Qontak, atau jaringan dealer regional Anda? Hubungi pembuat dokumen ini secara langsung untuk mendapatkan jadwal pendampingan teknis.
