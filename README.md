# Lion Parcel - ETL Pipeline

## Overview

Project ini dibuat untuk technical test Lion Parcel. Tujuannya adalah mengubah data pengiriman mentah menjadi dataset yang siap untuk analisis dan dashboard.

Saya menggunakan Python dengan pandas untuk proses cleaning dan agregasi data. Hasilnya adalah dua file CSV: satu untuk data yang sudah dibersihkan, dan satu lagi untuk ringkasan performa per customer.

## Files

Ada 2 file input yang dibutuhkan:
- `shipments_raw.csv` - data transaksi pengiriman
- `customers_raw.csv` - master data customer

Output yang dihasilkan:
- `shipment_transformed.csv` - data setelah cleaning
- `shipment_performance.csv` - agregasi per customer per bulan

## Prerequisites

Python 3.x dengan library berikut:
```bash
pip install pandas numpy
```

## Cara Menjalankan

1. Pastikan kedua file CSV input sudah ada di directory yang sama dengan script
2. Jalankan script:
   ```bash
   python lion_parcel_transform.py
   ```
3. Script akan otomatis generate 2 file output

## Data Transformation Process

### Part 1: Data Cleaning (Silver Layer)

Beberapa hal yang saya lakukan di tahap ini:

**1. Duplicate Removal**
- Drop duplicate berdasarkan `shipment_id`
- Keep first occurrence

**2. Status Normalization**
- Standardisasi berbagai variasi status (case-insensitive)
- Mapping: `in-transit/DELIVERED/pending/cancelled` → format yang konsisten

**3. Date Standardization**
- Handle multiple date formats (YYYY-MM-DD, DD/MM/YYYY, YYYY/MM/DD)
- Convert semua ke format YYYY-MM-DD
- Parse manual karena formatnya mixed di source data

**4. Calculated Fields**
- `delivery_duration_days`: selisih antara delivered_date dan booked_date
- `delivery_delay_days`: selisih antara delivered_date dan estimated_delivery_date (hanya untuk status Delivered)
- `is_delayed`: boolean flag untuk identifikasi keterlambatan

**5. Data Validation**
- Handle negative duration (set to null)
- Check missing values untuk critical fields

### Part 2: Data Aggregation (Gold Layer)

Di tahap ini saya aggregate data per customer per month untuk keperluan reporting.

**Metrics yang dihitung:**
- `total_shipments`: jumlah total pengiriman
- `delivered_shipments`: jumlah yang statusnya Delivered
- `on_process_shipments`: jumlah yang statusnya In Transit atau Pending
- `cancelled_shipments`: jumlah yang statusnya Cancelled
- `avg_delivery_days`: rata-rata waktu pengiriman (dari delivered shipments only)
- `delayed_shipments`: jumlah pengiriman yang telat
- `delayed_rate`: persentase keterlambatan (delayed_shipments / delivered_shipments)

## Output Schema

### shipment_transformed.csv
| Column | Type | Description |
|--------|------|-------------|
| shipment_id | string | Unique identifier untuk setiap pengiriman |
| customer_id | int | Foreign key ke customer master |
| origin_city | string | Kota asal pengiriman |
| destination_city | string | Kota tujuan pengiriman |
| status | string | Status pengiriman (Delivered/In Transit/Pending/Cancelled) |
| booked_date | date | Tanggal booking dalam format YYYY-MM-DD |
| estimated_delivery_date | date | Target tanggal pengiriman |
| delivered_date | date | Actual tanggal pengiriman (null jika belum delivered) |
| delivery_duration_days | int | Durasi pengiriman dalam hari |
| delivery_delay_days | int | Selisih dari estimasi (positif = telat, negatif = cepat) |
| is_delayed | boolean | Flag keterlambatan |
| chargeable_weight_kg | float | Berat untuk charging |
| total_amount | float | Total biaya pengiriman |

### shipment_performance.csv
| Column | Type | Description |
|--------|------|-------------|
| customer_id | int | ID customer |
| customer_name | string | Nama customer |
| month_year | string | Period dalam format YYYY-MM |
| total_shipments | int | Total shipments di period tersebut |
| delivered_shipments | int | Jumlah yang berhasil delivered |
| on_process_shipments | int | Jumlah yang masih on process |
| cancelled_shipments | int | Jumlah yang cancelled |
| avg_delivery_days | float | Average delivery time |
| delayed_shipments | int | Jumlah yang telat |
| delayed_rate | float | Delayed rate (0-1) |

## Notes & Assumptions

Beberapa catatan selama develop:

1. **Date Parsing**: Source data punya mixed date formats. Saya handle dengan try-except multiple formats. Kalau semua gagal, return None.

2. **Negative Duration**: Ada beberapa record dengan delivered_date < booked_date. Kemungkinan data entry error. Saya set ke None untuk avoid skewing metrics.

3. **Delayed Calculation**: Hanya applicable untuk shipments dengan status "Delivered". Yang lain (In Transit, Pending, Cancelled) tidak dihitung sebagai delayed.

4. **On Process Definition**: Saya gabungkan "In Transit" dan "Pending" sebagai satu kategori "on process" karena keduanya represent shipments yang belum selesai.

5. **Average Calculation**: Average delivery days dihitung hanya dari delivered shipments untuk avoid bias dari shipments yang masih on going.

## Known Issues & Future Improvements

Issues yang saya notice:
- Ada ~70 duplicate records di raw data
- Ada 14 records dengan negative duration
- Beberapa delivered shipments missing delivered_date (tapi statusnya sudah Delivered)

Potential improvements:
- Add data quality report/logging
- Handle outliers di delivery duration
- Add visualization untuk hasil agregasi
- Optimize untuk dataset yang lebih besar (currently works fine untuk ~1000 rows)

## Troubleshooting

**File not found error:**
Make sure file CSV ada di directory yang sama dengan script.

**Module not found:**
Install dependencies dulu: `pip install pandas numpy`

**Date parsing issues:**
Cek format date di source file. Kalau ada format baru, perlu ditambahkan di function `perbaiki_tanggal()`.

---

**Author**: Rika Elisabeth
**Date**: November 2024  
**Purpose**: Technical Test - Lion Parcel Data Analytics
