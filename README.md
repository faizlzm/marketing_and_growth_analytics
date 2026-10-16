# Marketing & Growth Analytics: Paid Ads vs Channel Gratis

Analisis efektivitas Paid Ads dibandingkan channel gratis pada 800 pengguna (Januari–Maret 2025) dengan Python dan Tableau. Fokusnya adalah menilai apakah biaya iklan sepadan dengan hasilnya, lewat metrik ROAS, CAC, *break-even margin*, dan pendapatan bersih per pengguna.
---

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Problem Statement](#problem-statement)
- [Struktur Repositori](#struktur-repositori)
- [Dataset](#dataset)
- [Metodologi](#metodologi)
- [Temuan Utama](#temuan-utama)
- [Kesimpulan dan Rekomendasi](#kesimpulan-dan-rekomendasi)
- [Keterbatasan](#keterbatasan)
- [Cara Menjalankan](#cara-menjalankan)
- [Penulis](#penulis)

---

## Latar Belakang

Perusahaan memperoleh pengguna dari dua jenis saluran: **Paid Ads**, yang memerlukan anggaran, dan **channel gratis** (Email, Organic Search, Referral, Social Media). Sebelum anggaran iklan dilanjutkan atau ditambah, perlu dipastikan bahwa iklan berbayar memberikan hasil yang sepadan dengan biayanya.

## Problem Statement

> Apakah Paid Ads lebih efektif dibandingkan channel gratis, dan apakah anggaran iklan layak dilanjutkan, ditingkatkan, atau dioptimasi?

## Struktur Repositori

```text
marketing_and_growth_analytics/
├── analysis_process/
│   └── main.ipynb                       # Data preparation, EDA, dan analisis Paid vs Gratis
├── dataset/
│   ├── data_marketing_growth_analytics.csv   # Dataset mentah (800 baris, 16 kolom)
│   └── data_clean.csv                        # Dataset hasil cleaning (800 baris, 23 kolom)
├── dashboard/
│   └── Analisis Efektivitas Paid Ads vs Organik Ads pada Dataset Marketing Growth.twbx   # Dashboard Tableau
├── report/
│   └── Analisis Efektivitas Paid Ads vs Organik Gratis pada Dataset Marketing Growth.pdf # Laporan presentasi
└── README.md
```

## Dataset

- **Jumlah baris:** 800 pengguna (satu baris per `user_id`)
- **Kolom mentah:** 16, menjadi 23 setelah feature engineering
- **Periode kunjungan pertama:** 1 Januari – 1 Maret 2025
- **Satuan uang:** Rupiah (mengikuti laporan; dataset sendiri tidak mencantumkan mata uang)

| Kelompok | Kolom |
| --- | --- |
| Identitas dan atribusi | `user_id`, `first_visit_date`, `channel`, `source`, `device` |
| Perilaku | `sessions`, `page_views`, `bounce` |
| Funnel | `impressions`, `clicks`, `add_to_cart`, `purchase` |
| Nilai | `revenue`, `ad_cost` |
| Retensi | `repeat_purchase`, `retention_month` |

Komposisi channel: Paid Ads 251, Organic Search 198, Social Media 151, Email 125, Referral 75. Perangkat: Mobile 504, Desktop 262, Tablet 34.

## Metodologi

### 1. Data Preparation

| Pemeriksaan | Hasil | Tindakan |
| --- | --- | --- |
| Missing value (eksplisit dan tersembunyi) | Tidak ada | Tidak perlu penanganan |
| Duplikasi | 0 baris; `user_id` unik | Tidak perlu penanganan |
| Konsistensi kategori | Spasi berlebih dirapikan | `strip`, lalu tipe `category` |
| Tipe data | `first_visit_date` berupa teks | Dikonversi ke datetime |
| Validasi rentang | Tidak ada nilai negatif; kolom biner hanya 0/1 | Tidak perlu penanganan |
| Outlier (IQR) | Terdeteksi pada beberapa kolom numerik | **Dipertahankan** agar pembeli bernilai tinggi tidak hilang |

### 2. Feature Engineering

| Kolom | Rumus / Keterangan |
| --- | --- |
| `tipe_channel` | `Paid` jika `ad_cost > 0`, selain itu `Gratis` |
| `revenue_bersih` | `revenue - ad_cost` |
| `ctr` | `clicks / impressions` |
| `pages_per_session` | `page_views / sessions` |
| `roas_user` | `revenue / ad_cost` (kosong untuk pengguna gratis) |
| `status_pembeli` | `Pembeli` atau `Bukan Pembeli` |
| `status_repeat` | `Repeat`, `Sekali Beli`, atau `Bukan Pembeli` |

Semua pengguna `Paid Ads` memiliki `ad_cost > 0`, dan tidak ada pengguna channel lain yang memiliki biaya iklan, sehingga `tipe_channel` konsisten dengan kolom `channel`.

### 3. Exploratory Data Analysis

Univariat (kategorikal, numerik, zero-inflated, waktu), bivariat (cohort retention bulanan), dan multivariat (korelasi Spearman).

### 4. Analisis Paid vs Gratis

| Indikator | Rumus |
| --- | --- |
| ROAS | pendapatan ÷ biaya iklan |
| CAC | biaya iklan ÷ jumlah pembeli |
| *Break-even margin* | 1 ÷ ROAS (margin kotor minimum agar iklan tidak rugi) |
| Pendapatan bersih per pengguna | (pendapatan − biaya iklan) ÷ jumlah pengguna |

Paid Ads kemudian diurai berdasarkan `source` dan `device`, dan funnel Impressions → Clicks → Add to Cart → Purchase dibandingkan antara Paid dan Gratis.

## Temuan Utama

> Angka diambil dari laporan dan notebook, lalu dicocokkan ulang dengan `dataset/data_marketing_growth_analytics.csv`.

### 1. Paid Ads menghasilkan pendapatan kotor terbesar, tetapi tidak setelah biaya

| | Paid | Gratis |
| --- | ---: | ---: |
| Pengguna | 251 (31%) | 549 (69%) |
| Pembeli | 51 | 99 |
| Conversion rate | 20,3% | 18,0% |
| Pendapatan kotor | Rp13,18 juta | Rp27,83 juta |
| Biaya iklan | Rp10,95 juta | Rp0 |
| Pendapatan bersih | Rp2,23 juta | Rp27,83 juta |
| Pendapatan per pengguna | Rp52,5 ribu | Rp50,7 ribu |
| Pendapatan bersih per pengguna | Rp8,9 ribu | Rp50,7 ribu |

Paid Ads adalah channel tunggal dengan pendapatan kotor terbesar, tetapi pendapatan per penggunanya hampir sama dengan channel gratis. Setelah biaya iklan dikurangkan, selisihnya besar.

### 2. ROAS 1,20 dan break-even margin 83%

- **ROAS:** 1,20 (Rp13,18 juta pendapatan dari Rp10,95 juta biaya).
- **CAC:** sekitar Rp214 ribu per pembeli, sedangkan rata-rata belanja pembeli Paid sekitar Rp258 ribu.
- **Break-even margin:** sekitar 83%. Pada margin kotor yang lazim (20–60%), Paid Ads merugi sekitar Rp3–8 juta.

### 3. Kinerja Paid Ads sangat berbeda antar source

| Source | Pengguna | Conversion | ROAS | CAC |
| --- | ---: | ---: | ---: | ---: |
| Google | 42 | 21% | 1,52 | Rp205.885 |
| Facebook | 40 | 22% | 1,40 | Rp186.959 |
| TikTok | 40 | 30% | 1,37 | Rp153.731 |
| Email | 43 | 19% | 1,25 | Rp212.920 |
| Instagram | 41 | 17% | 1,03 | Rp273.258 |
| Direct | 45 | 13% | 0,70 | Rp325.395 |

Direct merugi (ROAS di bawah 1), Instagram nyaris impas, dan TikTok punya conversion tertinggi dengan CAC terendah.

### 4. Kinerja per device

| Device | Pengguna | ROAS |
| --- | ---: | ---: |
| Desktop | 87 | 1,25 |
| Mobile | 155 | 1,21 |
| Tablet | 9 | 0,75 |

### 5. Retensi turun tajam di bulan pertama

Pada cohort Januari dan Februari 2025, retensi turun dari 100% (M0) ke sekitar 18–19% di M1, lalu terus menyusut hingga 2–4% di M5.

## Kesimpulan dan Rekomendasi

**Apakah Paid Ads lebih efektif daripada channel gratis?** Berdasarkan data ini, **tidak terbukti**. Conversion Paid sedikit lebih tinggi (20,3% vs 18,0%), tetapi biaya iklan menghabiskan sebagian besar pendapatannya.

1. **Jangan menaikkan anggaran Paid Ads pada skema saat ini.** ROAS hanya 1,20, dan setiap tambahan biaya perlu diuji lebih dulu (misalnya A/B test per source).
2. **Optimalkan Paid Ads.** Pertahankan Google, Facebook, dan TikTok; hentikan atau evaluasi ulang source Direct dan Instagram.
3. **Prioritaskan channel gratis,** terutama Social Media dan Referral.
4. **Perbaiki onboarding dan aktivasi di 30 hari pertama** untuk menekan churn di M0 → M1.

## Penulis

**Ahmad Faiz Ali Azmi**
Lulusan Teknik Informatika, Universitas Brawijaya. Peserta Offline Bootcamp Data Analyst Batch 4 di dibimbing.id.

GitHub: [@faizlzm](https://github.com/faizlzm)
