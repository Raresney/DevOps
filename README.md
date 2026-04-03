# Sesiunea 03 — Web Server, HTTPS & Reverse Proxy

Rezumat al sesiunii FiiPractic 2026 — configurare nginx cu HTTPS, certificate SSL, reverse proxy și monitorizare cu Netdata.

## Infrastructură

| VM | Hostname | IP |
|---|---|---|
| app | app.fiipractic.lan | 192.168.100.10 |
| gitlab | gitlab.fiipractic.lan | 192.168.100.20 |

## 1. Certificate Authority (CA) cu Easy-RSA

### Instalare Easy-RSA

```bash
yum install -y epel-release && yum install -y easy-rsa
```

### Inițializare PKI și creare CA

```bash
cd /usr/share/easy-rsa/3
./easyrsa init-pki
./easyrsa build-ca
```

> La `build-ca` se setează o passphrase pentru cheia CA — va fi necesară la fiecare semnare de certificat.

### Generare certificate server

```bash
# Certificat pentru app
./easyrsa --subject-alt-name=DNS:app.fiipractic.lan build-server-full app nopass

# Certificat pentru gitlab
./easyrsa --subject-alt-name=DNS:gitlab.fiipractic.lan build-server-full gitlab nopass

# Certificat pentru netdata
./easyrsa --subject-alt-name=DNS:netdata.fiipractic.lan build-server-full netdata nopass
```

### Locații fișiere

| Fișier | Path |
|---|---|
| CA cert | `/usr/share/easy-rsa/3/pki/ca.crt` |
| Certificat server | `/usr/share/easy-rsa/3/pki/issued/<name>.crt` |
| Cheie privată | `/usr/share/easy-rsa/3/pki/private/<name>.key` |

### Instalare CA pe Windows

1. Copiază conținutul `ca.crt` pe Windows
2. Dublu-click → Install Certificate → **Current User** → Trusted Root Certification Authorities
3. Repetă pentru **Local Machine**

## 2. Configurare Nginx cu HTTPS

### Server block HTTPS

```nginx
# /etc/nginx/conf.d/app-ssl.conf
server {
    listen 443 ssl;
    server_name app.fiipractic.lan;
    root /var/www/example;
    index index.html;

    ssl_certificate /usr/share/easy-rsa/3/pki/issued/app.crt;
    ssl_certificate_key /usr/share/easy-rsa/3/pki/private/app.key;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### SELinux — comenzi necesare

```bash
# Permite nginx să servească fișiere din /var/www/example
chcon -R -t httpd_sys_content_t /var/www/example/

# Permite nginx să facă conexiuni de rețea (proxy_pass)
setsebool -P httpd_can_network_connect 1
```

## 3. Redirect HTTP → HTTPS

```nginx
# /etc/nginx/conf.d/app-http.conf
server {
    listen 80;
    server_name app.fiipractic.lan;
    return 301 https://$host$request_uri;
}
```

## 4. Wireshark — HTTP vs HTTPS

- **HTTP (port 80):** traficul e în clar — headere, HTML, totul vizibil
- **HTTPS (port 443):** traficul e criptat (TLS) — nimic lizibil

Filtru Wireshark:
```
ip.addr == 192.168.100.10 && tcp.port == 80
ip.addr == 192.168.100.10 && tcp.port == 443
```

## 5. Netdata — Reverse Proxy

### Instalare

```bash
yum install -y netdata
systemctl start netdata
systemctl enable netdata
ss -tlnp | grep netdata   # ascultă pe portul 19999
```

### Reverse proxy pe subpath `/netdata/`

```nginx
# În /etc/nginx/conf.d/app-ssl.conf
location /netdata/ {
    proxy_pass http://127.0.0.1:19999/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_intercept_errors on;
    error_page 502 503 504 =503 /maintenance.html;
}
```

### Pagina de mentenanță (503)

```bash
cat > /var/www/example/maintenance.html << 'EOF'
<html><body><h1>Service temporarily unavailable - Maintenance in progress</h1></body></html>
EOF
chcon -t httpd_sys_content_t /var/www/example/maintenance.html
```

Test: `systemctl stop netdata` → accesezi `/netdata/` → pagina de mentenanță.

### Reverse proxy pe subdomain separat

```nginx
# /etc/nginx/conf.d/netdata-ssl.conf
server {
    listen 443 ssl;
    server_name netdata.fiipractic.lan;

    ssl_certificate /usr/share/easy-rsa/3/pki/issued/netdata.crt;
    ssl_certificate_key /usr/share/easy-rsa/3/pki/private/netdata.key;

    location / {
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://127.0.0.1:19999/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> DNS: adaugă `192.168.100.10 netdata.fiipractic.lan` în `C:\Windows\System32\drivers\etc\hosts`

## 6. Basic Authentication

```bash
yum install -y httpd-tools
htpasswd -c /etc/nginx/.htpasswd admin
```

Apoi adaugă în location-ul nginx:

```nginx
auth_basic "Restricted Access";
auth_basic_user_file /etc/nginx/.htpasswd;
```

## 7. Criptare / Decriptare cu OpenSSL

### Decriptare fișier cu cheie privată

```bash
openssl pkeyutl -decrypt -inkey myfiipracticapp.com.key -in plaintext.encrypt -out plaintext.decrypt
cat plaintext.decrypt
```

### Criptare fișier cu cheie publică din certificat

```bash
# Extrage cheia publică
openssl x509 -pubkey -noout -in /usr/share/easy-rsa/3/pki/issued/app.crt > pubkey.pem

# Criptează
openssl pkeyutl -encrypt -pubin -inkey pubkey.pem -in secret.txt -out secret.encrypt

# Decriptează
openssl pkeyutl -decrypt -inkey /usr/share/easy-rsa/3/pki/private/app.key -in secret.encrypt -out secret.decrypt
```

## Troubleshooting

| Problemă | Soluție |
|---|---|
| 403 Forbidden | `chcon -R -t httpd_sys_content_t /var/www/example/` |
| 502 Bad Gateway la proxy | `setsebool -P httpd_can_network_connect 1` |
| `maybe wrong password` la easy-rsa | Resetează PKI: `rm -rf pki && ./easyrsa init-pki && ./easyrsa build-ca` |
| DNS_PROBE_FINISHED_NXDOMAIN | Adaugă hostname în `C:\Windows\System32\drivers\etc\hosts` |
| `nginx -t` failed | Verifică sintaxa config-ului cu `cat /etc/nginx/conf.d/*.conf` |
