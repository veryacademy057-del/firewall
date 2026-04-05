# 🛡️ Firewall — Konsep Dasar & Cara Kerja

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/KZbySud9L3c?si=71zLT5eOX4kll8S1)  
> **Kategori:** `Konsep Dasar, Network Security`

---

## 📋 Deskripsi

Materi ini membahas **konsep dasar Firewall** — mulai dari definisi, cara kerja, jenis-jenis firewall, hingga kombinasinya dengan IDS/IPS dalam sebuah arsitektur pertahanan berlapis. Pemahaman ini adalah fondasi sebelum mengkonfigurasi OPNsense dan Suricata di lab.

---

## 🎬 Video Tutorial

[![Firewall - Konsep Dasar & Cara Kerja](https://img.youtube.com/vi/KZbySud9L3c/maxresdefault.jpg)](https://youtu.be/KZbySud9L3c?si=71zLT5eOX4kll8S1)

> 📺 **[Tonton di YouTube → Firewall: Garis Pertahanan Pertama Jaringan Anda](https://youtu.be/KZbySud9L3c?si=71zLT5eOX4kll8S1)**

---

## 🔐 Apa itu Firewall?

Firewall adalah **sistem keamanan jaringan** yang memantau dan mengontrol lalu lintas data masuk dan keluar berdasarkan aturan keamanan yang telah ditentukan.

> 💡 **Analogi:** Seperti satpam gedung yang memeriksa setiap orang yang masuk dan keluar — firewall memeriksa setiap paket data yang melewati jaringan.

**Fungsi Utama Firewall:**

| Fungsi | Penjelasan |
|--------|------------|
| 🔍 Memfilter traffic jaringan | Memeriksa setiap paket data yang masuk dan keluar |
| 🚫 Mencegah akses tidak sah | Memblokir koneksi yang tidak memenuhi aturan keamanan |
| 🛡️ Melindungi sistem dari ancaman | Menjadi garis pertahanan pertama dari serangan luar |

---

## 🧱 Jenis-Jenis Firewall

### 1. Packet Filtering Firewall
Jenis paling sederhana. Memeriksa setiap paket data berdasarkan:
- IP Address (sumber dan tujuan)
- Port (sumber dan tujuan)
- Protokol (TCP, UDP, ICMP)

```
Keunggulan : Cepat dan ringan
Kelemahan  : Tidak bisa melihat isi paket atau status koneksi
```

---

### 2. Stateful Inspection Firewall
Lebih cerdas dari packet filtering — tidak hanya memeriksa paket individual, tetapi juga **memantau status koneksi** yang sedang aktif.

```
Keunggulan : Bisa membedakan koneksi baru vs koneksi yang sudah ada
Kelemahan  : Tidak bisa memeriksa isi aplikasi (HTTP, DNS, dll)
```

---

### 3. Application Layer Firewall (Layer 7)
Memeriksa traffic hingga level aplikasi. Dapat memahami isi dari protokol seperti HTTP, FTP, dan DNS.

```
Keunggulan : Deteksi lebih dalam, bisa filter konten spesifik
Kelemahan  : Lebih lambat karena pemrosesan lebih berat
```

---

### 4. Next Generation Firewall (NGFW)
Kombinasi semua kemampuan di atas, ditambah fitur-fitur modern:

| Fitur | Keterangan |
|-------|------------|
| IDS/IPS | Deteksi dan pencegahan intrusi |
| Deep Packet Inspection (DPI) | Analisis mendalam isi paket |
| Threat Intelligence | Database ancaman yang selalu diperbarui |
| Application Awareness | Identifikasi aplikasi berdasarkan perilaku |

---

## ⚙️ Cara Kerja Firewall

Setiap paket data yang melewati firewall akan diperiksa berdasarkan **ruleset** yang telah dikonfigurasi:

```
Internet
    │
    ▼
┌─────────────────────────────────┐
│            FIREWALL             │
│                                 │
│  Cek Rules:                     │
│  ├── IP Source / Destination    │
│  ├── Port                       │
│  ├── Protokol (TCP/UDP/ICMP)    │
│  └── Status Koneksi             │
│                                 │
│  Hasil:  ALLOW  atau  BLOCK     │
└─────────────────────────────────┘
    │
    ▼
Jaringan Internal
```

### Contoh Rules Firewall

| Rule | Aksi |
|------|------|
| Traffic HTTP/HTTPS dari internet → Web Server | ✅ ALLOW |
| Koneksi masuk ke port 22 (SSH) dari luar | ❌ BLOCK |
| Semua traffic dari jaringan internal → Internet | ✅ ALLOW |
| Traffic tidak dikenal / tidak sesuai rules | ❌ BLOCK |

---

## 🔍 Firewall + IDS/IPS

Firewall saja tidak cukup — kombinasi dengan IDS/IPS memberikan perlindungan yang jauh lebih kuat.

### Perbedaan Firewall, IDS, dan IPS

| Aspek | Firewall | IDS | IPS |
|-------|----------|-----|-----|
| **Fungsi** | Filter traffic | Deteksi serangan | Deteksi + Blokir |
| **Aksi** | Allow / Block | Alert saja | Alert + Block |
| **Posisi** | Gateway | Monitor (pasif) | Inline (aktif) |
| **Contoh Tool** | OPNsense | Suricata (IDS mode) | Suricata (IPS mode) |

---

### Cara Kerja IDS (Intrusion Detection System)

```
Traffic masuk
     │
     ▼
Suricata membaca traffic secara PASIF
     │
     ▼
Mencocokkan dengan rules ET Open
     │
     ├── Match → Kirim ALERT ke Splunk
     │            Traffic tetap JALAN
     │
     └── No Match → Traffic lewat normal
```

---

### Cara Kerja IPS (Intrusion Prevention System)

```
Traffic masuk
     │
     ▼
Suricata membaca traffic secara INLINE
     │
     ▼
Mencocokkan dengan rules ET Open
     │
     ├── Match → DROP / BLOCK traffic
     │            + Kirim ALERT ke Splunk
     │
     └── No Match → Traffic lewat normal
```

---

### Arsitektur Pertahanan Berlapis di SOC Lab

Kombinasi ideal yang digunakan dalam lab ini:

```
┌──────────────────────────────────────────────────────┐
│              DEFENSE IN DEPTH (Berlapis)             │
│                                                      │
│  [Internet]                                          │
│       │                                              │
│       ▼                                              │
│  ┌─────────────────┐                                 │
│  │   OPNsense      │ ← Layer 1: Filter IP, Port,    │
│  │   (Firewall)    │           Protokol              │
│  └────────┬────────┘                                 │
│           │                                          │
│           ▼                                          │
│  ┌─────────────────┐                                 │
│  │   Suricata      │ ← Layer 2: Deteksi/Blokir      │
│  │   (IDS/IPS)     │           Nmap, Exploit,        │
│  └────────┬────────┘           Brute Force           │
│           │                                          │
│           ▼                                          │
│  ┌─────────────────┐                                 │
│  │     Splunk      │ ← Layer 3: Analisis &           │
│  │     (SIEM)      │           Monitoring Log        │
│  └─────────────────┘                                 │
│                                                      │
└──────────────────────────────────────────────────────┘
```

| Layer | Tool | Peran |
|-------|------|-------|
| **Layer 1** | OPNsense | Filter traffic berdasarkan IP, port, protokol |
| **Layer 2** | Suricata IDS/IPS | Deteksi dan blokir serangan secara otomatis |
| **Layer 3** | Splunk SIEM | Analisis dan monitoring semua log secara terpusat |

---

## 📌 Kesimpulan

| Konsep | Poin Penting |
|--------|-------------|
| **Firewall** | Garis pertahanan pertama — filter traffic berdasarkan rules |
| **IDS** | Mendeteksi serangan secara pasif, tidak memblokir |
| **IPS** | Mendeteksi dan langsung memblokir serangan secara inline |
| **SIEM** | Mengumpulkan dan menganalisis semua log dari seluruh sistem |
| **Defense in Depth** | Kombinasi semua layer = perlindungan yang jauh lebih kuat |

---

## 📚 Referensi

- [OPNsense Documentation](https://docs.opnsense.org/)
- [Suricata IDS/IPS Documentation](https://suricata.io/documentation/)
- [Emerging Threats Open Ruleset](https://rules.emergingthreats.net/)
- [NIST — Firewall Guidelines](https://csrc.nist.gov/publications/detail/sp/800-41/rev-1/final)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/KZbySud9L3c?si=71zLT5eOX4kll8S1) | ⭐ Star repo ini jika membantu!*
