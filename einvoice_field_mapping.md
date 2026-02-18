# E-Invoice Field Mapping & Calculation Report

## Contoh Data (dari Payment ff8081819c699d51019c69a2fc14056d)

| # | Description | Qty | Unit Price | Discount | Subtotal |
|---|---|---|---|---|---|
| 1 | 初诊 First Consultation(专家 Senior) | 1 | 40.50 | 4.50 | 36.00 |
| 2 | 复诊 Subsequent(专家 Senior) | 1 | 29.10 | 0.90 | 28.20 |
| 3 | 膏方加工费 Linctus Preparation Cost | 1 | 300.00 | 0.00 | 300.00 |
| 4 | 单部位少针 Single Area Less Needles | 1 | 24.00 | 6.00 | 18.00 |
| 5 | 单部位多针 Single Area More Needles | 1 | 40.00 | 0.00 | 40.00 |
| | **TOTAL** | | **433.60** | **11.40** | **422.20** |

---

## Line Item Fields (per baris)

Contoh: Item 1 — 初诊 First Consultation(专家 Senior), Harga 40.50, Diskon 4.50, Subtotal 36.00

### 1. ✅ Classification Codes = 022

| | |
|---|---|
| **UBL** | `CommodityClassification > ItemClassificationCode` |
| **Sumber** | Di-set langsung ke `022` (kode untuk "Others - Healthcare") di kode. Semua item klinik pakai kode yang sama |

### 2. ✅ Description = 初诊 First Consultation(专家 Senior)

| | |
|---|---|
| **UBL** | `Item > Description` |
| **Sumber** | Diambil dari nama/deskripsi fee di halaman Visit → Patient Fee, kolom "Description" |

### 3. ✅ Quantity = 1.00

| | |
|---|---|
| **UBL** | `InvoicedQuantity` |
| **Sumber** | Diambil dari kolom "Quantity" di halaman Visit → Patient Fee. Kalau tidak diisi, otomatis default ke 1 |

### 4. ✅ Measurement = Unit

| | |
|---|---|
| **UBL** | `InvoicedQuantity > unitCode` attribute |
| **Sumber** | Di-set langsung ke `XUN` (kode standar untuk "Unit"). Semua item pakai satuan yang sama |

### 5. ✅ Unit Price = RM 40.50

| | |
|---|---|
| **UBL** | `Price > PriceAmount` |
| **Sumber** | Diambil dari kolom "Fee" di halaman Visit → Patient Fee. Ini harga asli **sebelum diskon** |

### 6. ✅ Subtotal = RM 36.00

| | |
|---|---|
| **UBL** | `LineExtensionAmount` |
| **Sumber** | Dihitung otomatis dari harga asli dikali jumlah, lalu dikurangi diskon |
| **Rumus** | (Unit Price × Quantity) - Discount = (40.50 × 1) - 4.50 = 36.00 |

### 7. ✅ Total Tax Amount = RM 0.00

| | |
|---|---|
| **UBL** | `TaxTotal > TaxAmount` |
| **Sumber** | Di-set langsung ke `0.00` karena semua layanan klinik bebas pajak (tax-exempt) |

### 8. ✅ Total Excluding Tax = RM 36.00

| | |
|---|---|
| **UBL** | Sama dengan Subtotal |
| **Sumber** | Nilainya sama persis dengan Subtotal karena pajak = 0. Tidak ada perhitungan tambahan |

### 9. ✅ Total Discount Amount = RM 4.50

| | |
|---|---|
| **UBL** | `AllowanceCharge > Amount` (dengan `ChargeIndicator = false`) |
| **Sumber** | Diambil dari kolom "Discount Amount" di halaman Visit → Patient Fee. Hanya muncul kalau diskon lebih dari 0 |

### 10. ❌ Total Fees and Charges = RM 0.00

| | |
|---|---|
| **UBL** | `AllowanceCharge > Amount` (dengan `ChargeIndicator = true`) |
| **Sumber** | Tidak ada data di ITCM. Fitur biaya tambahan per item (contoh: surcharge, pengemasan) belum tersedia di aplikasi |

### 11. ✅ Amount Exempted From Tax = RM 36.00

| | |
|---|---|
| **UBL** | `TaxSubtotal (TaxCategory ID=E) > TaxableAmount` |
| **Sumber** | Nilainya sama dengan Subtotal item. Ditampilkan karena semua layanan klinik masuk kategori tax-exempt (kode E) |

### 12. ❌ Product Tariff Code = (Kosong)

| | |
|---|---|
| **UBL** | `ItemClassificationCode (listID=PTC)` |
| **Sumber** | Tidak ada data tariff code di ITCM. Field ini opsional dan tidak wajib diisi untuk layanan healthcare |

### 13. ✅ Country of Origin = MYS

| | |
|---|---|
| **UBL** | `OriginCountry > IdentificationCode` |
| **Sumber** | Di-set langsung ke `MYS` (kode negara Malaysia). Semua item pakai negara asal yang sama |

---

## Summary Fields (Total Invoice)

### 1. ✅ Total Tax Amount = RM 0.00

| | |
|---|---|
| **UBL** | `TaxTotal > TaxAmount` |
| **Sumber** | Di-set langsung ke `0.00` di kode karena semua layanan klinik saat ini bebas pajak |
| **Kenapa 0?** | Layanan healthcare di Malaysia **tax-exempt** (0% tax) |
| **Rumus** | Σ (line tax) = 0 + 0 + 0 + 0 + 0 = 0.00 |

### 2. ✅ Total Sales Amount = RM 422.20

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > LineExtensionAmount` |
| **Sumber** | Dijumlahkan dari subtotal setiap item di halaman Visit → Patient Fee. Subtotal tiap item = (harga asli × jumlah) dikurangi diskon |
| **Rumus** | 36.00 + 28.20 + 300.00 + 18.00 + 40.00 = 422.20 |

### 3. ✅ Total Discount Amount = RM 11.40

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > AllowanceTotalAmount` |
| **Sumber** | Dijumlahkan dari kolom "Discount Amount" setiap item di halaman Visit → Patient Fee |
| **Rumus** | 4.50 + 0.90 + 0.00 + 6.00 + 0.00 = 11.40 |

### 4. ❌ Total Fees and Charges = RM 0.00

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > ChargeTotalAmount` |
| **Sumber** | Tidak ada data di ITCM. Fitur surcharge / biaya tambahan per item belum tersedia di aplikasi |
| **Kenapa 0?** | ITCM gak punya fitur surcharge/biaya tambahan per item |

### 5. ❌ Invoice Additional Discount Amount = RM 0.00

| | |
|---|---|
| **UBL** | Invoice-level `AllowanceCharge` ( `ChargeIndicator = false` ) |
| **Sumber** | Tidak di-implement. Diskon di ITCM hanya ada di level per item (kolom Discount Amount di tabel fee), bukan di level keseluruhan invoice |
| **Kenapa 0?** | Diskon udah masuk di tiap line item. Kalau ditambahin lagi = **double counting** |

### 6. ❌ Invoice Additional Fee / Charge Amount = RM 0.00

| | |
|---|---|
| **UBL** | Invoice-level `AllowanceCharge` ( `ChargeIndicator = true` ) |
| **Sumber** | Tidak ada data di ITCM. Fitur biaya tambahan di level keseluruhan invoice (misal: admin fee, delivery fee) belum tersedia di aplikasi |
| **Kenapa 0?** | ITCM gak punya fitur biaya tambahan di level invoice |

### 7. ✅ Total Excluding Tax = RM 422.20

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > TaxExclusiveAmount` |
| **Sumber** | Dihitung otomatis dari Total Sales dikurangi Invoice Additional Discount ditambah Invoice Additional Fee |
| **Rumus** | Total Sales (422.20) - Invoice Discount (0) + Invoice Fee (0) = 422.20 |

### 8. ✅ Total Including Tax = RM 422.20

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > TaxInclusiveAmount` |
| **Sumber** | Dihitung otomatis dari Total Excluding Tax ditambah Total Tax Amount |
| **Rumus** | Total Excluding Tax (422.20) + Total Tax (0.00) = 422.20 |

### 9. ✅ Prepayment Amount = RM 0.00

| | |
|---|---|
| **UBL** | `PrepaidPayment > PaidAmount` |
| **Sumber** | Diambil dari field "Deposit Fee" di data Payment. Nilainya dihitung otomatis dari fee-fee yang ditandai checkbox "Is Deposit Fee" di halaman setting Tenant → Fee |
| **Kenapa 0?** | Tidak ada fee yang ditandai sebagai deposit di klinik ini |

### 10. ✅ Total Payable Amount = RM 422.20

| | |
|---|---|
| **UBL** | `LegalMonetaryTotal > PayableAmount` |
| **Sumber** | Dihitung otomatis dari Total Including Tax dikurangi Prepayment Amount (uang muka / deposit) |
| **Rumus** | Total Including Tax (422.20) - Prepayment (0.00) = 422.20 |


---

## Ringkasan Status

| Status | Jumlah | Field |
|---|---|---|
| ✅ Sudah benar | **7 field** | Tax, Sales, Discount, Excl Tax, Incl Tax, Prepayment, Payable |
| ❌ Tidak ada fiturnya | **3 field** | Fees & Charges, Invoice Discount, Invoice Fee |

---

## Alur Kalkulasi (Step by Step)

```
STEP 1: Hitung per item
   Subtotal = (Harga Asli × Jumlah) - Diskon
   Contoh item 1: (40.50 × 1) - 4.50 = 36.00

STEP 2: Jumlahkan semua item
   Total Sales     = 36.00 + 28.20 + 300.00 + 18.00 + 40.00 = 422.20
   Total Discount  = 4.50 + 0.90 + 0 + 6.00 + 0             = 11.40
   Total Tax       = 0 + 0 + 0 + 0 + 0                       = 0.00

STEP 3: Hitung total invoice
   Total Excluding Tax  = Total Sales                         = 422.20
   Total Including Tax  = Total Excluding Tax + Total Tax     = 422.20
   Prepayment           = Deposit Fee dari Payment            = 0.00
   Total Payable        = Total Including Tax - Prepayment    = 422.20
```

---

## Sumber Data di ITCM

| Data | Asal | Keterangan |
|---|---|---|
| Daftar item & harga | Tabel fee di halaman Visit | Setiap visit punya daftar fee (konsultasi, akupunktur, dll) |
| Diskon per item | Kolom "Discount Amount" di fee | Bisa diisi manual atau otomatis dari Discount Rate % |
| Deposit / Uang Muka | Field "Deposit Fee" di Payment | Otomatis dari fee yang ditandai "Is Deposit Fee" di setting klinik |
| Data pasien (pembeli) | Halaman Patient | Nama, IC/Passport, alamat, telepon, email |
| Data klinik (penjual) | Setting Tenant/Clinic | Nama klinik, alamat, telepon, email, registration no |
| TIN, BRN, sertifikat | Template XML + konfigurasi | Di-set saat setup awal e-invoice |

