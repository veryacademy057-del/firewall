# 🕷️ Suricata Detection — Mendeteksi Web Scanning dengan Nikto

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/x53LVeUtzdc?si=SN47s3gusKaPvO1c)  
> **Kategori:** `IDS Detection, Web Application Security`

---

## 📋 Deskripsi

Lab ini mendemonstrasikan bagaimana **Suricata IDS** mendeteksi serangan **web vulnerability scanning** menggunakan **Nikto**. Nikto secara otomatis mencoba berbagai teknik serangan ke web server — dan kita akan membuktikan bahwa setiap aktivitas tersebut terdeteksi oleh Suricata dan dapat dianalisa di Splunk.

**Yang disimulasikan Nikto:**
- Path traversal
- Command injection
- PHP code injection
- Directory bruteforce
- Akses ke halaman admin tersembunyi

---

## 🎬 Video Tutorial

[![Suricata Detection - Nikto Web Scanning](https://img.youtube.com/vi/x53LVeUtzdc/maxresdefault.jpg)](https://youtu.be/x53LVeUtzdc?si=SN47s3gusKaPvO1c)

> 📺 **[Tonton di YouTube → Suricata Detection: Web Scanning dengan Nikto](https://youtu.be/x53LVeUtzdc?si=SN47s3gusKaPvO1c)**

---

## 🗺️ Topologi Lab

```
┌──────────────────────────────────────────────────────────┐
│                   Internal Network                       │
│                                                          │
│   [Kali Linux]                                           │
│    10.200.200.10                                         │
│         │                                                │
│         │  Nikto Web Vulnerability Scan                  │
│         ▼                                                │
│   [OPNsense Firewall]                                    │
│    10.200.200.254                                        │
│         │                                                │
│         │  Suricata IDS mendeteksi request Nikto         │
│         │  → EVE JSON Alert (ET WEB_SERVER rules)        │
│         │                                                │
│         │  Syslog UDP Port 514                           │
│         ▼                                                │
│   [Splunk SIEM]                                          │
│    10.200.200.100                                        │
│    Analisa & Investigasi Alert                           │
└──────────────────────────────────────────────────────────┘
```

---

## ⚔️ Langkah 1 — Jalankan Nikto dari Kali Linux

Lakukan web vulnerability scanning ke OPNsense untuk mensimulasikan serangan web application:

```bash
nikto -h http://10.200.200.254
```

**Output yang diharapkan:**

```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.200.200.254
+ Target Hostname:    10.200.200.254
+ Target Port:        80
---------------------------------------------------------------------------
+ Server: nginx
+ Retrieved x-frame-options header: SAMEORIGIN
+ OSVDB-3092: /CFIDE/administrator/index.cfm: ColdFusion administrator access
...
```

> 💡 Nikto akan mencoba ratusan path dan teknik serangan secara otomatis. Setiap request yang dikirim Nikto akan dicatat oleh Suricata dan OPNsense audit log.

---

## 🔍 Langkah 2 — Monitor Suricata Secara Real-Time

Buka terminal baru dan SSH ke OPNsense untuk memantau alert Suricata secara langsung saat Nikto berjalan:

```bash
ssh root@10.200.200.254
# Pilih 8 (Shell)

# Monitor alert Suricata real-time
tail -f /var/log/suricata/eve.json | grep "alert"
```

**Contoh output EVE JSON saat Nikto berjalan:**

```json
{
  "timestamp": "2026-04-14T03:50:15",
  "event_type": "alert",
  "src_ip": "10.200.200.10",
  "dest_ip": "10.200.200.254",
  "dest_port": 80,
  "proto": "TCP",
  "alert": {
    "signature": "ET WEB_SERVER ColdFusion administrator access",
    "category": "Web Application Attack",
    "severity": 1
  },
  "http": {
    "url": "/cfide/Administrator/startstop.html",
    "http_user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
  }
}
```

---

## ✅ Langkah 3 — Verifikasi Alert di OPNsense Web UI

```
Services → Intrusion Detection → Alerts
```

**Alert yang seharusnya muncul:**

| Field | Nilai |
|-------|-------|
| **Signature** | `ET WEB_SERVER ColdFusion administrator access` |
| **Source** | `10.200.200.10` |
| **Destination** | `10.200.200.254:80` |
| **Category** | `Web Application Attack` |
| **Severity** | `1` (Paling tinggi) |

---

## 📊 Langkah 4 — Analisa Alert di Splunk

### Query 1 — Lihat Semua Alert Web Application Attack

```splunk
index=opnsense sourcetype=suricata
| rex field=_raw "suricata\[\d+\]:\s+(?P<json_raw>\{.+)"
| spath input=json_raw
| search "alert.category"="Web Application Attack"
| table _time, src_ip, dest_ip, "alert.signature", "http.url", "http.http_user_agent"
```

---

### Query 2 — Lihat URL yang Di-scan Nikto

```splunk
index=opnsense sourcetype=suricata
| rex field=_raw "suricata\[\d+\]:\s+(?P<json_raw>\{.+)"
| spath input=json_raw
| search src_ip="10.200.200.10"
| table _time, src_ip, "http.url", "alert.signature"
| sort _time
```

---

### Query 3 — Hitung Jumlah Serangan per Signature

```splunk
index=opnsense sourcetype=suricata
| rex field=_raw "suricata\[\d+\]:\s+(?P<json_raw>\{.+)"
| spath input=json_raw
| search alert.signature=*
| stats count by "alert.signature"
| sort -count
```

---

### Query 4 — Cek Audit Log OPNsense

```splunk
index=opnsense "Nikto" earliest=-30m
| table _time, host, _raw
```

---

### Hasil yang Diharapkan di Splunk

| `_time` | `src_ip` | `dest_ip` | `alert.signature` | `http.url` |
|---------|----------|-----------|-------------------|------------|
| 2026-04-14 03:50:15 | 10.200.200.10 | 10.200.200.254 | ET WEB_SERVER ColdFusion administrator access | `/cfide/Administrator/startstop.html` |
| 2026-04-14 03:50:21 | 10.200.200.10 | 10.200.200.254 | ET WEB_SERVER ColdFusion administrator access | `/CFIDE/administrator/index.cfm` |

---

## 🧠 Penjelasan Alert yang Muncul

### ET WEB_SERVER ColdFusion Administrator Access

| Field | Penjelasan |
|-------|------------|
| **Apa itu?** | Nikto mencoba akses halaman admin ColdFusion yang sering ada di web server lama |
| **URL yang dicoba** | `/CFIDE/administrator/index.cfm` dan `/cfide/Administrator/startstop.html` |
| **Bahaya** | Jika halaman ini bisa diakses, attacker bisa mengontrol penuh web server |
| **Category** | Web Application Attack |
| **Severity** | 1 (Paling tinggi) |

---

### Analisa Audit Log OPNsense

OPNsense juga mencatat request berbahaya dari Nikto di audit log:

```
no active session, user not found
(called "/forum/index.php?Nikto=forums&file=viewtopic
&rush=%6c%73%20%2d%61%6c" @ 10.200.200.10)
```

**Breakdown request Nikto:**

| Bagian | Penjelasan |
|--------|------------|
| `?Nikto=forums` | Parameter test bawaan Nikto |
| `rush=%6c%73%20%2d%61%6c` | URL encoded → **`ls -al`** → percobaan command injection |
| `highlight=%2527...` | Percobaan PHP code injection |
| `@ 10.200.200.10` | Request berasal dari Kali Linux |

> 💡 **Insight:** Nikto tidak hanya melakukan scanning pasif — ia secara aktif mencoba command injection dan PHP code injection. Inilah mengapa Suricata langsung menghasilkan alert severity 1.

---

## 📌 Kesimpulan

| Hasil | Status |
|-------|--------|
| Nikto berhasil melakukan web vulnerability scanning | ✅ |
| Suricata IDS mendeteksi scanning via rule ET WEB_SERVER | ✅ |
| OPNsense Audit Log mencatat semua request berbahaya | ✅ |
| Splunk menerima dan menampilkan alert dalam format terstruktur | ✅ |

---

## 📚 Referensi

- [Nikto Official Documentation](https://cirt.net/Nikto2)
- [Suricata ET WEB_SERVER Rules](https://rules.emergingthreats.net/)
- [OPNsense Intrusion Detection Docs](https://docs.opnsense.org/manual/ids.html)
- [MITRE ATT&CK — Web Application Exploitation (T1190)](https://attack.mitre.org/techniques/T1190/)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/x53LVeUtzdc?si=SN47s3gusKaPvO1c) | ⭐ Star repo ini jika membantu!*
