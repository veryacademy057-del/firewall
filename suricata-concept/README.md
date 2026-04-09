# ⚡ Suricata — Engine IDS/IPS Open Source

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/_E3yVBDhxrc?si=-VyMEuiGeCZ-Ip0e)  
> **Kategori:** `Konsep Dasar, Network Security`

---

## 📋 Deskripsi

Materi ini membahas **Suricata** sebagai IDS/IPS engine yang digunakan di SOC Lab — mulai dari definisi, cara kerja, jenis rules yang digunakan, hingga integrasi dengan Splunk SIEM. Suricata adalah komponen inti dari layer deteksi di lab ini.

---

## 🎬 Video Tutorial

[![Suricata - Engine IDS/IPS Open Source](https://img.youtube.com/vi/_E3yVBDhxrc/maxresdefault.jpg)](https://youtu.be/_E3yVBDhxrc?si=-VyMEuiGeCZ-Ip0e)

> 📺 **[Tonton di YouTube → Suricata: Engine IDS/IPS Open Source Terkuat](https://youtu.be/_E3yVBDhxrc?si=-VyMEuiGeCZ-Ip0e)**

---

## 🔍 Apa itu Suricata?

**Suricata** adalah engine IDS/IPS open source yang dikembangkan oleh **OISF** (*Open Information Security Foundation*) sejak tahun 2009. Suricata mampu mendeteksi ancaman jaringan secara real-time dengan performa tinggi.

### Keunggulan Suricata

| Fitur | Penjelasan |
|-------|------------|
| **Open Source & Gratis** | Tidak ada biaya lisensi — aktif dikembangkan komunitas |
| **Multi-threading** | Memanfaatkan banyak CPU core untuk performa tinggi |
| **3 Mode Operasi** | IDS, IPS, dan Network Security Monitoring (NSM) |
| **EVE JSON Output** | Format log JSON yang mudah diintegrasikan ke SIEM |
| **Kompatibel Snort Rules** | Bisa menggunakan rules dari ekosistem Snort |

---

## ⚙️ Cara Kerja Suricata

```
[Traffic Jaringan]
        │
        ▼
┌───────────────────────────────────┐
│          Suricata Engine          │
│                                   │
│  Tangkap & analisa setiap paket   │
│               │                   │
│               ▼                   │
│  Cocokkan dengan Rules Database   │
│  (ET Open, abuse.ch, dll)         │
│               │                   │
│        ┌──────┴──────┐            │
│      Match         No Match       │
│        │               │          │
│   ┌────┴────┐          ▼          │
│  IDS      IPS       Lewat         │
│   │        │                      │
│ Alert  Alert +                    │
│        Block                      │
└───┬────────┬──────────────────────┘
    │        │
    └────┬───┘
         │ EVE JSON Log
         ▼
  [Splunk SIEM]
  Analisa & Alert
```

### Perbedaan Mode Operasi

| Mode | Cara Kerja | Aksi |
|------|------------|------|
| **IDS** | Pasif — membaca salinan traffic | Kirim alert, traffic tidak diblokir |
| **IPS** | Inline — berada di jalur traffic utama | Alert + langsung drop/block traffic |
| **NSM** | Pasif — logging lengkap semua traffic | Catat semua aktivitas jaringan untuk forensik |

---

## 📋 Jenis Rules Suricata

Rules adalah **database signature serangan** yang digunakan Suricata untuk mencocokkan traffic jaringan. Semakin lengkap rules, semakin banyak ancaman yang bisa dideteksi.

### Rules yang Digunakan di Lab

| Rules | Fungsi |
|-------|--------|
| **ET Open / emerging-scan** | Deteksi port scan & Nmap |
| **ET Open / emerging-exploit** | Deteksi exploit attempts |
| **ET Open / emerging-malware** | Deteksi malware traffic |
| **ET Open / emerging-dos** | Deteksi serangan DoS/DDoS |
| **ET Open / emerging-web_server** | Deteksi web application attack |
| **ET Open / emerging-shellcode** | Deteksi shellcode dalam traffic |
| **abuse.ch / Feodo Tracker** | Deteksi botnet C2 server |
| **abuse.ch / URLhaus** | Deteksi URL berbahaya |
| **abuse.ch / ThreatFox** | Deteksi IoC dari threat intelligence |

> 💡 **ET Open** (Emerging Threats Open) adalah ruleset gratis dengan 50.000+ signature yang diperbarui setiap hari — standar industri untuk IDS/IPS open source.

---

## 🏗️ Suricata di SOC Lab

### Topologi Integrasi

```
┌─────────────────────────────────────────────────────┐
│                  Internal Network                   │
│                                                     │
│   [Kali Linux]                                      │
│   10.200.200.10                                     │
│        │                                            │
│        │  Serangan (Nmap, Hydra, Exploit, dll)      │
│        ▼                                            │
│   [OPNsense Firewall]                               │
│   10.200.200.254                                    │
│        │                                            │
│        │  Semua traffic melewati sini               │
│        ▼                                            │
│   [Suricata IDS/IPS]                               │
│   (berjalan di OPNsense)                            │
│        │                                            │
│        │  Alert dalam format EVE JSON               │
│        ▼                                            │
│   [Syslog UDP Port 514]                             │
│        │                                            │
│        │  Kirim log ke SIEM                         │
│        ▼                                            │
│   [Splunk SIEM]                                     │
│   10.200.200.100                                    │
│   Analisa, Dashboard & Alert                        │
└─────────────────────────────────────────────────────┘
```

### Yang Bisa Dideteksi Suricata di Lab Ini

| Jenis Serangan | Rule yang Mendeteksi |
|----------------|----------------------|
| ✅ Nmap scan | ET SCAN Possible Nmap User-Agent |
| ✅ Port scan (SYN, FIN, XMAS scan) | ET SCAN rules |
| ✅ Brute force SSH | ET BRUTE rules |
| ✅ Web application attack | ET WEB_SERVER rules |
| ✅ Exploit attempts | ET EXPLOIT rules |
| ✅ Malware & botnet traffic | abuse.ch / Feodo Tracker |

---

## 📄 Format Log EVE JSON

Suricata menghasilkan log dalam format **EVE JSON** yang mudah dibaca oleh SIEM seperti Splunk:

```json
{
  "timestamp": "2024-01-15T10:23:45.123456+0000",
  "event_type": "alert",
  "src_ip": "10.200.200.10",
  "src_port": 54321,
  "dest_ip": "10.200.200.20",
  "dest_port": 22,
  "proto": "TCP",
  "alert": {
    "action": "allowed",
    "signature_id": 2001219,
    "signature": "ET SCAN Potential SSH Scan",
    "category": "Attempted Information Leak",
    "severity": 2
  }
}
```

**Field penting di EVE JSON:**

| Field | Keterangan |
|-------|------------|
| `timestamp` | Waktu kejadian |
| `src_ip` | IP sumber (attacker) |
| `dest_ip` | IP tujuan (target) |
| `alert.signature` | Nama rule yang match |
| `alert.severity` | Tingkat keparahan (1=kritis, 3=rendah) |
| `alert.action` | Aksi yang diambil (allowed/blocked) |

---

## 📌 Kesimpulan

| Konsep | Poin Penting |
|--------|-------------|
| **Suricata** | Engine IDS/IPS open source — gratis, cepat, multi-threaded |
| **Mode IDS** | Pasif — hanya alert, tidak memblokir |
| **Mode IPS** | Inline — alert + blokir secara real-time |
| **ET Open Rules** | 50.000+ signature, update harian — standar industri |
| **EVE JSON** | Format log JSON yang mudah diintegrasikan ke Splunk |
| **Di Lab Ini** | Suricata berjalan di OPNsense → log ke Splunk via Syslog |

---

## 📚 Referensi

- [Suricata Official Documentation](https://suricata.io/documentation/)
- [OISF — Open Information Security Foundation](https://oisf.net/)
- [Emerging Threats Open Ruleset](https://rules.emergingthreats.net/)
- [abuse.ch Rulesets](https://abuse.ch/)
- [Suricata EVE JSON Format](https://suricata.readthedocs.io/en/latest/output/eve/eve-json-format.html)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/_E3yVBDhxrc?si=-VyMEuiGeCZ-Ip0e) | ⭐ Star repo ini jika membantu!*
