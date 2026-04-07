# 🔎 IDS & IPS — Sistem Deteksi dan Pencegahan Intrusi

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/a7dwX8LnINw?si=FGCeoFQSXSgX_Yyd)  
> **Kategori:** `Konsep Dasar, Network Security`

---

## 📋 Deskripsi

Materi ini membahas **konsep dasar IDS dan IPS** — dua komponen penting dalam arsitektur keamanan jaringan modern. Pemahaman ini adalah fondasi sebelum mengkonfigurasi **Suricata** sebagai IDS/IPS engine di lab.

---

## 🎬 Video Tutorial

[![IDS & IPS - Sistem Deteksi dan Pencegahan Intrusi](https://img.youtube.com/vi/a7dwX8LnINw/maxresdefault.jpg)](https://youtu.be/a7dwX8LnINw?si=FGCeoFQSXSgX_Yyd)

> 📺 **[Tonton di YouTube → IDS & IPS: Sistem Deteksi dan Pencegahan Intrusi](https://youtu.be/a7dwX8LnINw?si=FGCeoFQSXSgX_Yyd)**

---

## 🔍 Apa itu IDS?

**IDS (Intrusion Detection System)** adalah sistem yang memantau lalu lintas jaringan secara **pasif** untuk mendeteksi aktivitas mencurigakan dan memberikan peringatan (*alert*).

> 💡 **Analogi:** Seperti CCTV di gedung — IDS merekam dan memberikan peringatan jika ada aktivitas mencurigakan, tetapi **tidak mengambil tindakan** untuk menghentikannya.

### Cara Kerja IDS

```
Traffic Jaringan
      │
      ▼
┌─────────────────────────────┐
│         IDS Engine          │
│                             │
│  Baca traffic secara PASIF  │
│  (salinan traffic / mirror) │
│           │                 │
│           ▼                 │
│  Cocokkan dengan Rules /    │
│  Signature Database         │
│           │                 │
│    ┌──────┴──────┐          │
│  Match?        No Match     │
│    │               │        │
│    ▼               ▼        │
│  Kirim ALERT    Lewat saja  │
│  ke SIEM                    │
└─────────────────────────────┘
      │
      ▼
Traffic tetap BERJALAN (tidak diblokir)
```

**Kelebihan IDS:**
- Tidak mengganggu traffic normal sama sekali
- Cocok untuk monitoring, analisa, dan forensik
- Aman digunakan di lingkungan lab karena tidak memblokir apapun

---

## 🛡️ Apa itu IPS?

**IPS (Intrusion Prevention System)** adalah pengembangan dari IDS yang tidak hanya mendeteksi tetapi juga secara aktif **memblokir traffic berbahaya** secara real-time.

> 💡 **Analogi:** Seperti satpam gedung — IPS tidak hanya merekam, tetapi langsung **menghalangi dan mengusir** orang yang mencurigakan sebelum masuk ke dalam gedung.

### Cara Kerja IPS

```
Traffic Jaringan
      │
      ▼
┌─────────────────────────────┐
│         IPS Engine          │
│                             │
│  Baca traffic secara INLINE │
│  (di jalur utama traffic)   │
│           │                 │
│           ▼                 │
│  Cocokkan dengan Rules /    │
│  Signature Database         │
│           │                 │
│    ┌──────┴──────┐          │
│  Match?        No Match     │
│    │               │        │
│    ▼               ▼        │
│  DROP / BLOCK   Teruskan    │
│  Traffic        Traffic     │
│  + Kirim ALERT              │
└─────────────────────────────┘
```

**Kelebihan IPS:**
- Proteksi real-time otomatis tanpa intervensi manual
- Mencegah serangan sebelum masuk ke jaringan internal

---

## ⚖️ Perbandingan IDS vs IPS

| Aspek | IDS | IPS |
|-------|-----|-----|
| **Kepanjangan** | Intrusion Detection System | Intrusion Prevention System |
| **Fungsi** | Deteksi saja | Deteksi + Blokir |
| **Posisi di Jaringan** | Out-of-band (monitor) | Inline (di jalur traffic) |
| **Aksi saat Ancaman** | Kirim alert | Block + Kirim alert |
| **Risiko False Positive** | Rendah | Tinggi (bisa blokir traffic normal) |
| **Dampak ke Performa** | Minimal | Sedikit lebih berat |
| **Cocok untuk** | Monitoring & analisa | Proteksi aktif real-time |
| **Contoh Tool** | Suricata (IDS mode) | Suricata (IPS mode) |

> 💡 **Mana yang lebih baik?** Tergantung kebutuhan:
> - **Lab / Belajar** → IDS lebih aman karena tidak mengganggu traffic
> - **Production / Enterprise** → IPS lebih disarankan untuk proteksi aktif

---

## 🧠 Jenis-Jenis Metode Deteksi

### 1. Signature-Based Detection

Mencocokkan traffic dengan **database signature** serangan yang sudah dikenal sebelumnya.

```
Traffic → Cocokkan dengan 50.000+ signature → Match → Alert/Block
```

| Kelebihan | Kekurangan |
|-----------|------------|
| Cepat dan akurat untuk serangan yang sudah dikenal | Tidak bisa deteksi serangan baru (zero-day) |
| False positive rendah | Perlu update rules secara rutin |

**Contoh ruleset yang digunakan:**
- **ET Open** (Emerging Threats Open) — ruleset gratis yang diperbarui setiap hari
- **abuse.ch** — rules khusus malware dan botnet
- **Feodo Tracker** — rules untuk tracking Command & Control server

---

### 2. Anomaly-Based Detection

Membuat **baseline traffic normal**, kemudian memberikan alert jika ada penyimpangan dari baseline tersebut.

```
Pelajari traffic normal (baseline)
→ Pantau traffic real-time
→ Jika ada penyimpangan signifikan → Alert
```

| Kelebihan | Kekurangan |
|-----------|------------|
| Bisa deteksi serangan baru / zero-day | False positive lebih tinggi |
| Tidak bergantung pada signature database | Butuh waktu untuk membentuk baseline |

---

### 3. Hybrid Detection

Kombinasi signature-based + anomaly-based untuk hasil yang paling lengkap dan akurat.

```
Traffic
  │
  ├──► Signature Check  → Alert jika match signature
  │
  └──► Anomaly Check    → Alert jika menyimpang dari baseline
```

> 💡 Pendekatan hybrid adalah standar yang digunakan di lingkungan SOC modern.

---

## 🛠️ Tools yang Digunakan di Lab

| Tool | Peran |
|------|-------|
| **Suricata** | IDS/IPS engine — mendeteksi dan/atau memblokir serangan |
| **ET Open Rules** | Database signature serangan (50.000+ rules, update harian) |
| **Splunk** | Mengumpulkan, menganalisa, dan memvisualisasikan alert dari Suricata |

### Arsitektur di SOC Lab

```
[Traffic Jaringan]
        │
        ▼
┌───────────────┐
│   Suricata    │ ← IDS: Monitor pasif
│   (IDS/IPS)  │   IPS: Inline blocking
└───────┬───────┘
        │ Alert / Log
        ▼
┌───────────────┐
│    Splunk     │ ← Analisa, dashboard,
│    (SIEM)     │   dan alert otomatis
└───────────────┘
```

---

## 📌 Kesimpulan

| Konsep | Poin Penting |
|--------|-------------|
| **IDS** | Mendeteksi secara pasif — tidak memblokir, hanya alert |
| **IPS** | Mendeteksi dan memblokir secara inline — proteksi aktif |
| **Signature-Based** | Akurat untuk serangan dikenal, tidak efektif untuk zero-day |
| **Anomaly-Based** | Bisa deteksi zero-day, tapi lebih banyak false positive |
| **Hybrid** | Kombinasi terbaik untuk lingkungan produksi |
| **Suricata + Splunk** | Kombinasi IDS/IPS + SIEM yang digunakan di lab ini |

---

## 📚 Referensi

- [Suricata Official Documentation](https://suricata.io/documentation/)
- [Emerging Threats Open Ruleset](https://rules.emergingthreats.net/)
- [abuse.ch Ruleset](https://abuse.ch/)
- [MITRE ATT&CK — Network Intrusion Detection](https://attack.mitre.org/)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/a7dwX8LnINw?si=FGCeoFQSXSgX_Yyd) | ⭐ Star repo ini jika membantu!*
