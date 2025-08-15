# minilab
minilab setup

skenario uji coba pentest berbasis **Podman** yang aman untuk dilakukan secara lokal atau di lab tertutup — supaya bisa simulasi serangan tanpa risiko ke sistem produksi.
Pendekatan ini mirip dengan membuat **mini lab pentest** di dalam container-container Podman.

---

## **Tujuan**

* Menguji tools pentest di lingkungan yang terisolasi.
* Memahami cara kerja serangan seperti brute force, SQL injection, XSS, dan eksploitasi service.
* Berlatih analisis log dan forensik pasca serangan.

---

## **Arsitektur Lab (Podman)**

```
+-------------------+
| Attacker Container| --> Kali Linux / Parrot OS
+-------------------+
        |
        v
+-------------------+    +-------------------+
| Web Server Target |    | Service Target    |
| DVWA / Juice Shop |    | SSH, FTP, SMB     |
+-------------------+    +-------------------+
```

Kita akan punya **tiga container**:

1. **Attacker** → Kali Linux/Parrot OS (isi Metasploit, Nmap, Hydra, Nikto, sqlmap, dll.)
2. **Web Target** → Aplikasi vulnerable seperti DVWA (*Damn Vulnerable Web App*) atau OWASP Juice Shop.
3. **Service Target** → Server yang menjalankan SSH/FTP/SMB dengan credential lemah.

---

## **1. Setup Jaringan di Podman**

Biar semua container bisa saling komunikasi, kita buat network khusus:

```bash
podman network create pentest-net
```

---

## **2. Buat Container Target Web (DVWA)**

```bash
podman run -d --name dvwa \
  --network pentest-net \
  -p 8080:80 \
  vulnerables/web-dvwa
```

* Akses dari host: `http://localhost:8080`
* Default user: `admin` / `password`

---

## **3. Buat Container Target Service**

Contoh target dengan SSH dan FTP:

```bash
podman run -d --name vuln-services \
  --network pentest-net \
  -p 2222:22 \
  -p 2121:21 \
  vulnerables/metasploitable
```

---

## **4. Buat Container Attacker (Kali Linux)**

```bash
podman run -it --name kali \
  --network pentest-net \
  kalilinux/kali-rolling bash
```

Di dalam container Kali:

```bash
apt update && apt install -y nmap hydra metasploit-framework sqlmap nikto
```

---

## **5. Skenario Uji Coba**

### **A. Reconnaissance (Scanning & Enumeration)**

* `nmap -sV dvwa`
* `nmap -sV vuln-services`

### **B. Web Exploitation**

* SQL Injection:
  `sqlmap -u "http://dvwa/login.php" --forms --batch --dbs`
* XSS:
  Uji input form di DVWA Level "Low".

### **C. Brute Force**

* SSH brute force:
  `hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://vuln-services`

### **D. Exploit Service**

* Metasploit:
  Jalankan `msfconsole` dan cari modul untuk service yang ditemukan.
* SMB enum dan exploit jika ada.

### **E. Post-Exploitation**

* Upload file berbahaya untuk reverse shell.
* Cek log di target (`/var/log/auth.log`) untuk analisis forensik.

---

## **6. Keamanan**

* Jangan jalankan di network publik.
* Gunakan Podman network khusus (isolasi).
* Pastikan container tidak menggunakan `--net=host`.

---

