# 🎯 Suricata Detection — Mendeteksi Nmap Scan dengan IDS

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/5hg_m_DpZfY?si=uflRMcEhzMfRRvei)  
> **Kategori:** `IDS Configuration, Threat Detection`

---

## 📋 Deskripsi

Lab ini mengkonfigurasi **Suricata** sebagai IDS di OPNsense, mendownload rules serangan, lalu memverifikasi deteksi dengan melakukan **Nmap scan** dari Kali Linux. Alert dari Suricata dikirim ke **Splunk** via Syslog dalam format **EVE JSON** untuk dianalisa.

---

## 🎬 Video Tutorial

[![Suricata Detection - Nmap Scan](https://img.youtube.com/vi/5hg_m_DpZfY/maxresdefault.jpg)](https://youtu.be/5hg_m_DpZfY?si=uflRMcEhzMfRRvei)

> 📺 **[Tonton di YouTube → Suricata Detection: Mendeteksi Nmap Scan](https://youtu.be/5hg_m_DpZfY?si=uflRMcEhzMfRRvei)**

---

## 🗺️ Topologi Lab

```
┌──────────────────────────────────────────────────────────┐
│                   Internal Network                       │
│                                                          │
│   [Kali Linux]                                           │
│    10.200.200.10                                         │
│         │                                                │
│         │  Nmap scan / serangan                         │
│         ▼                                                │
│   [OPNsense Firewall]                                    │
│    10.200.200.254                                        │
│         │                                                │
│         │  Suricata IDS memantau traffic                 │
│         │  → Match rules → EVE JSON Alert               │
│         │                                                │
│         │  Syslog UDP Port 514                           │
│         ▼                                                │
│   [Splunk SIEM]                                          │
│    10.200.200.100                                        │
│    Analisa & Investigasi Alert                           │
└──────────────────────────────────────────────────────────┘
```

---

## ⚙️ Bagian 1 — Konfigurasi Suricata di OPNsense

Mengaktifkan dan mengkonfigurasi Suricata sebagai IDS di OPNsense agar bisa mendeteksi serangan yang melewati firewall.

### Langkah 1 — Buka Intrusion Detection

```
Services → Intrusion Detection → Administration
```

---

### Langkah 2 — Konfigurasi General Settings

| Field | Nilai | Penjelasan |
|-------|-------|------------|
| **Enabled** | ✅ | Aktifkan Suricata |
| **IPS Mode** | ❌ | Pakai mode IDS — traffic tidak diblokir |
| **Promiscuous mode** | ✅ | Suricata bisa membaca semua traffic di interface |
| **Interfaces** | `LAN, WAN` | Monitor traffic di kedua interface |
| **Pattern matcher** | `Hyperscan` | Engine pencocokan rules tercepat |

> 💡 **Kenapa IPS Mode dimatikan?** Karena ini lab belajar — kita tidak ingin traffic terganggu. Mode IDS hanya mendeteksi dan menghasilkan alert tanpa memblokir apapun. Di environment production, IPS Mode bisa diaktifkan untuk proteksi aktif.

---

### Konfigurasi Logging

| Field | Nilai | Penjelasan |
|-------|-------|------------|
| **Enable syslog alerts** | ✅ | Kirim alert Suricata ke syslog OPNsense |
| **Enable eve syslog output** | ✅ | Kirim log EVE JSON ke syslog |
| **Rotate log** | `Daily` | Log dirotasi setiap hari agar tidak memenuhi storage |
| **Save logs** | `4` | Simpan log 4 hari terakhir |

> 💡 **Kenapa `Enable eve syslog output` penting?** EVE JSON adalah format log Suricata yang sangat detail — mencakup `src_ip`, `dest_ip`, `protocol`, `user_agent`, `alert.signature`, dan lainnya. Tanpa ini, alert Suricata tidak akan terkirim ke Splunk dalam format yang bisa dianalisa per field.

Klik **Save** → **Apply**

---

## 📦 Bagian 2 — Download Rules

Rules adalah **otak dari Suricata**. Tanpa rules, Suricata tidak tahu traffic mana yang berbahaya. Semakin lengkap rules, semakin banyak jenis serangan yang bisa dideteksi.

### Langkah 3 — Buka Tab Download

```
Services → Intrusion Detection → Download
```

### Langkah 4 — Download & Update Rules

Centang semua rules lalu klik **Download & Update Rules**. Tunggu hingga semua rules berhasil terdownload dan muncul tanggal update di kolom **Last Updated**.

**Rules penting yang didownload:**

| Rules | Fungsi |
|-------|--------|
| **ET Open / emerging-scan** | Deteksi port scan & Nmap |
| **ET Open / emerging-exploit** | Deteksi exploit attempts |
| **ET Open / emerging-malware** | Deteksi malware traffic |
| **ET Open / emerging-dos** | Deteksi serangan DoS/DDoS |
| **ET Open / emerging-shellcode** | Deteksi shellcode injection |
| **ET Open / emerging-web_server** | Deteksi web application attack |
| **ET Open / emerging-user_agents** | Deteksi user agent berbahaya (Nmap, sqlmap) |
| **abuse.ch / Feodo Tracker** | Deteksi botnet Command & Control |
| **abuse.ch / URLhaus** | Deteksi URL berbahaya |
| **abuse.ch / ThreatFox** | Deteksi threat intelligence terbaru |

> 💡 Untuk lab belajar, download semua rules agar bisa melihat berbagai jenis deteksi. Di production environment, pilih rules sesuai kebutuhan agar tidak membebani performa sistem.

---

## ▶️ Bagian 3 — Start Suricata

### Langkah 5 — Jalankan Suricata

Klik tombol **▶️ Play** di pojok kanan atas halaman Administration.

Jika tidak bisa via Web UI, start via SSH:

```bash
ssh root@10.200.200.254
# Pilih 8 (Shell)

# Start Suricata
configctl ids start

# Cek status
configctl ids status
```

**Output yang diharapkan:**

```
suricata is running as pid XXXXX
```

Cek log Suricata secara real-time:

```bash
# Lihat semua EVE JSON log
tail -f /var/log/suricata/eve.json

# Filter hanya alert saja
tail -f /var/log/suricata/eve.json | grep "alert"
```

---

## ⚙️ Bagian 4 — Konfigurasi Splunk untuk Suricata (EVE JSON)

Log Suricata dikirim ke Splunk berbentuk JSON. Tanpa konfigurasi tambahan, Splunk hanya membacanya sebagai teks biasa sehingga field penting seperti `src_ip`, `dest_ip`, `alert.signature` tidak bisa di-search secara terpisah.

Dengan `props.conf` dan `transforms.conf`, Splunk akan otomatis mengenali dan memisahkan field-field tersebut.

> 💡 **Analogi:** Seperti menerima surat dalam bahasa asing. Tanpa kamus (`props.conf` + `transforms.conf`) kamu tidak bisa membacanya. Dengan kamus, kamu bisa membaca dan memahami isinya secara detail.

---

### Langkah 6 — Buat `props.conf`

`props.conf` mengatur cara Splunk membaca dan memproses data yang masuk:

```bash
sudo nano /opt/splunk/etc/system/local/props.conf
```

Isi file:

```ini
[syslog]
KV_MODE = none
TRANSFORMS-suricata = suricata_json
```

**Penjelasan:**

| Baris | Fungsi |
|-------|--------|
| `[syslog]` | Berlaku untuk semua data dengan sourcetype syslog |
| `KV_MODE = none` | Matikan key-value extraction otomatis karena kita pakai JSON |
| `TRANSFORMS-suricata = suricata_json` | Panggil transforms bernama `suricata_json` |

---

### Langkah 7 — Buat `transforms.conf`

`transforms.conf` mengubah atau mengklasifikasikan data sebelum diindeks. Bekerja bersama `props.conf`:

```bash
sudo nano /opt/splunk/etc/system/local/transforms.conf
```

Isi file:

```ini
[suricata_json]
REGEX = \{\"timestamp\"
DEST_KEY = MetaData:Sourcetype
FORMAT = sourcetype::suricata
```

**Penjelasan:**

| Baris | Fungsi |
|-------|--------|
| `[suricata_json]` | Nama transform yang dipanggil dari `props.conf` |
| `REGEX = \{\"timestamp\"` | Cari log yang mengandung `{"timestamp"` — ciri khas EVE JSON Suricata |
| `DEST_KEY = MetaData:Sourcetype` | Ubah metadata sourcetype data tersebut |
| `FORMAT = sourcetype::suricata` | Ganti sourcetype menjadi `suricata` |

---

### Langkah 8 — Restart Splunk

```bash
/opt/splunk/bin/splunk restart
```

> 💡 Splunk hanya membaca konfigurasi saat pertama kali start. Setiap ada perubahan di `props.conf` atau `transforms.conf`, Splunk harus direstart agar perubahan berlaku.

---

## 🧪 Bagian 5 — Verifikasi: Test Serangan & Cek Alert

### Langkah 9 — Test Serangan dari Kali Linux

Jalankan Nmap scan dari Kali Linux ke OPNsense:

```bash
# Nmap service detection scan
nmap -sV 10.200.200.254

# Nmap aggressive scan
nmap -A 10.200.200.254
```

> 💡 **Kenapa pakai `-sV` dan `-A`?** Nmap dengan flag ini menggunakan **Nmap Scripting Engine (NSE)** yang mengirim HTTP request dengan **User-Agent khas Nmap**. Inilah yang dideteksi oleh rule `ET SCAN Possible Nmap User-Agent Observed` di Suricata.

---

### Langkah 10 — Cek Alert di OPNsense

```
Services → Intrusion Detection → Alerts
```

Pastikan muncul alert:

```
Signature : ET SCAN Possible Nmap User-Agent Observed
Source    : 10.200.200.10
Dest      : 10.200.200.254:80
```

---

### Langkah 11 — Cek Alert di Splunk

```splunk
# Cari semua alert ET SCAN
index=opnsense "ET SCAN" earliest=-5m

# Cari khusus Nmap
index=opnsense "Nmap" earliest=-5m

# Cari semua log Suricata
index=opnsense sourcetype=suricata earliest=-5m

# Lihat detail alert: src_ip, dest_ip, signature
index=opnsense sourcetype=suricata
| table _time, src_ip, dest_ip, alert.signature, alert.severity
```

> ✅ Jika alert `ET SCAN Possible Nmap User-Agent Observed` muncul di Splunk dengan `src_ip: 10.200.200.10`, maka seluruh pipeline **Suricata → Syslog → Splunk** berjalan dengan benar.

---

## 📌 Kesimpulan

Setelah mengikuti lab ini, kamu telah berhasil:

- ✅ Mengkonfigurasi Suricata sebagai IDS di OPNsense
- ✅ Mendownload dan mengaktifkan rules ET Open + abuse.ch
- ✅ Mengkonfigurasi Splunk untuk membaca EVE JSON dari Suricata
- ✅ Melakukan simulasi Nmap scan dan mendeteksinya di Suricata
- ✅ Memverifikasi alert masuk ke Splunk dengan query

---

## 📚 Referensi

- [Suricata Official Documentation](https://suricata.io/documentation/)
- [OPNsense Intrusion Detection Docs](https://docs.opnsense.org/manual/ids.html)
- [Emerging Threats Open Ruleset](https://rules.emergingthreats.net/)
- [Splunk props.conf Reference](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Propsconf)
- [Splunk transforms.conf Reference](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Transformsconf)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/5hg_m_DpZfY?si=uflRMcEhzMfRRvei) | ⭐ Star repo ini jika membantu!*
