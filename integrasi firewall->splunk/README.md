# 🔗 Integrasi Firewall ke SIEM — OPNsense → Splunk

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/sPRvSwbKl9E?si=fvW9wvCD6g0nAgSP)  
> **Kategori:** `SIEM Integration, Network Security`

---

## 📋 Deskripsi

Lab ini mengkonfigurasi **OPNsense** agar mengirimkan log firewall secara real-time ke **Splunk SIEM** melalui protokol **Syslog UDP port 514**. Setelah integrasi ini selesai, semua aktivitas jaringan yang melewati firewall — termasuk traffic yang diblokir dan diizinkan — dapat dipantau dan dianalisis langsung dari Splunk.

---

## 🎬 Video Tutorial

[![Integrasi OPNsense ke Splunk SIEM](https://img.youtube.com/vi/sPRvSwbKl9E/maxresdefault.jpg)](https://youtu.be/sPRvSwbKl9E?si=fvW9wvCD6g0nAgSP)

> 📺 **[Tonton di YouTube → Integrasi Firewall OPNsense ke Splunk SIEM](https://youtu.be/sPRvSwbKl9E?si=fvW9wvCD6g0nAgSP)**

---

## 🗺️ Topologi Integrasi

```
┌─────────────────────────────────────────────────────────┐
│                  Internal Network                       │
│                                                         │
│   [Semua VM]                                            │
│   (Kali, Win Server, Win 10, dll)                       │
│        │                                                │
│        │  Traffic jaringan                              │
│        ▼                                                │
│   [OPNsense Firewall]                                   │
│    10.200.200.254                                       │
│        │                                                │
│        │  Syslog UDP Port 514                           │
│        │  (Firewall log dikirim real-time)              │
│        ▼                                                │
│   [Splunk SIEM]                                         │
│    10.200.200.100                                       │
│    Index: opnsense                                      │
└─────────────────────────────────────────────────────────┘
```

---

## 📦 Yang Dibutuhkan

| Komponen | IP Address | Keterangan |
|----------|------------|------------|
| **OPNsense** | `10.200.200.254` | Sumber log (pengirim) |
| **Splunk** | `10.200.200.100` | Penerima log (SIEM) |
| **Protokol** | UDP Port 514 | Syslog standar |

---

## ⚙️ Bagian 1 — Konfigurasi Splunk (Penerima Log)

Splunk harus dikonfigurasi terlebih dahulu untuk siap menerima log dari OPNsense.

### Langkah 1 — Buat Index `opnsense`

Buka terminal di Ubuntu Server dan jalankan:

```bash
/opt/splunk/bin/splunk add index opnsense -auth admin:password
```

> 💡 Index adalah "tempat penyimpanan" data di Splunk. Kita buat index khusus `opnsense` agar log firewall terpisah dari log lainnya dan lebih mudah di-query.

---

### Langkah 2 — Buat File `inputs.conf`

File ini memberitahu Splunk untuk mendengarkan (*listen*) log yang masuk melalui UDP port 514:

```bash
sudo nano /opt/splunk/etc/system/local/inputs.conf
```

Isi file dengan konfigurasi berikut:

```ini
[udp://514]
connection_host = ip
sourcetype = syslog
index = opnsense
```

**Penjelasan setiap baris:**

| Parameter | Nilai | Keterangan |
|-----------|-------|------------|
| `[udp://514]` | — | Splunk mendengarkan di UDP port 514 |
| `connection_host` | `ip` | Gunakan IP pengirim sebagai nama host |
| `sourcetype` | `syslog` | Format data yang diterima adalah syslog |
| `index` | `opnsense` | Simpan ke index yang sudah dibuat |

---

### Langkah 3 — Restart Splunk

Terapkan perubahan konfigurasi dengan restart:

```bash
/opt/splunk/bin/splunk restart
```

> ⏳ Tunggu hingga Splunk selesai restart sebelum melanjutkan ke konfigurasi OPNsense.

---

## ⚙️ Bagian 2 — Konfigurasi OPNsense (Pengirim Log)

### Langkah 4 — Setup Remote Syslog

Login ke web interface OPNsense (`http://10.200.200.254`), lalu navigasi ke:

```
System → Settings → Logging / Targets → (+) Tambah baru
```

Isi form dengan konfigurasi berikut:

| Field | Nilai |
|-------|-------|
| **Enable** | ✅ Dicentang |
| **Transport** | `UDP(4)` |
| **Applications** | Select All |
| **Levels** | Select All |
| **Facilities** | Select All |
| **Hostname** | `10.200.200.100` |
| **Port** | `514` |
| **Description** | `Splunk-SIEM` |

Klik **Save** → **Apply**

> 💡 Dengan memilih **Select All** pada Applications, Levels, dan Facilities — semua jenis log dari OPNsense (firewall, DHCP, DNS, Suricata, dll) akan dikirimkan ke Splunk.

---

### Langkah 5 — Aktifkan Logging di Firewall Rules

Secara default, firewall rules tidak mencatat log traffic. Aktifkan logging pada rule utama:

```
Firewall → Rules → LAN
→ Edit rule "Default allow LAN to any"
```

Centang opsi:

```
☑ Log packets that are handled by this rule
```

Klik **Save** → **Apply Changes**

> 💡 Setelah ini, semua traffic yang melewati rule ini akan tercatat di log dan dikirim ke Splunk.

---

## ✅ Bagian 3 — Verifikasi Integrasi

### Langkah 6 — Cek Log Masuk dari OPNsense

Di Splunk Web (`http://10.200.200.100:8000`), buka Search & Reporting dan jalankan:

```splunk
index=opnsense host=10.200.200.254
```

> ✅ Jika log muncul, artinya OPNsense sudah berhasil mengirimkan data ke Splunk.

---

### Langkah 7 — Cek Firewall Log

```splunk
index=opnsense filterlog earliest=-5m
```

> 💡 `filterlog` adalah sourcetype khusus untuk log firewall OPNsense. `earliest=-5m` artinya tampilkan log 5 menit terakhir.

---

### Langkah 8 — Cek Traffic dari Kali Linux

```splunk
index=opnsense filterlog "10.200.200.10" earliest=-5m
```

> 💡 Query ini memfilter log yang melibatkan IP Kali Linux (`10.200.200.10`) — berguna untuk memantau atau menginvestigasi aktivitas dari mesin penyerang.

---

## 🔍 Query Tambahan yang Berguna

```splunk
# Lihat semua sumber log yang masuk
index=opnsense | stats count by host

# Lihat semua sourcetype yang tersedia
index=opnsense | stats count by sourcetype

# Cek log Suricata IDS (jika sudah dikonfigurasi)
index=opnsense sourcetype=suricata

# Lihat traffic berdasarkan IP tujuan
index=opnsense filterlog | stats count by dest_ip | sort -count

# Lihat traffic yang diblokir
index=opnsense filterlog action=block | stats count by src_ip | sort -count
```

---

## 📌 Kesimpulan

Setelah mengikuti lab ini, kamu telah berhasil:

- ✅ Membuat index khusus `opnsense` di Splunk
- ✅ Mengkonfigurasi Splunk untuk menerima log via UDP 514
- ✅ Mengkonfigurasi OPNsense untuk mengirimkan log ke Splunk
- ✅ Mengaktifkan logging pada firewall rules
- ✅ Memverifikasi integrasi berjalan dengan benar

Setelah integrasi ini, **semua aktivitas jaringan** yang melewati OPNsense dapat dipantau, dianalisis, dan dijadikan alert di Splunk secara real-time.

---

## 📚 Referensi

- [OPNsense Logging Documentation](https://docs.opnsense.org/manual/logging_settings.html)
- [Splunk inputs.conf Reference](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf)
- [Splunk Add UDP Input](https://docs.splunk.com/Documentation/Splunk/latest/Data/Monitornetworkports)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/sPRvSwbKl9E?si=fvW9wvCD6g0nAgSP) | ⭐ Star repo ini jika membantu!*
