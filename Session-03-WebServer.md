# Session 03 — Web Server, HTTPS & Reverse Proxy

Summary of FiiPractic 2026 — configuring nginx with HTTPS, SSL certificates, reverse proxy and monitoring with Netdata.

## Infrastructure

| VM | Hostname | IP |
|---|---|---|
| app | app.fiipractic.lan | 192.168.100.10 |
| gitlab | gitlab.fiipractic.lan | 192.168.100.20 |

## 1. Certificate Authority (CA) with Easy-RSA

### Install Easy-RSA

```bash
yum install -y epel-release && yum install -y easy-rsa
```

### Initialize PKI and Create CA

```bash
cd /usr/share/easy-rsa/3
./easyrsa init-pki
./easyrsa build-ca
```

> At `build-ca` you set a passphrase for the CA key — it will be required every time you sign a certificate.

### Generate Server Certificates

```bash
# Certificate for app
./easyrsa --subject-alt-name=DNS:app.fiipractic.lan build-server-full app nopass

# Certificate for gitlab
./easyrsa --subject-alt-name=DNS:gitlab.fiipractic.lan build-server-full gitlab nopass

# Certificate for netdata
./easyrsa --subject-alt-name=DNS:netdata.fiipractic.lan build-server-full netdata nopass
```

### File Locations

| File | Path |
|---|---|
| CA cert | `/usr/share/easy-rsa/3/pki/ca.crt` |
| Server cert | `/usr/share/easy-rsa/3/pki/issued/<name>.crt` |
| Private key | `/usr/share/easy-rsa/3/pki/private/<name>.key` |

### Install CA on Windows

1. Copy the contents of `ca.crt` to Windows
2. Double-click → Install Certificate → **Current User** → Trusted Root Certification Authorities
3. Repeat for **Local Machine**

## 2. Nginx with HTTPS

### HTTPS Server Block

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

### SELinux — Required Commands

```bash
# Allow nginx to serve files from /var/www/example
chcon -R -t httpd_sys_content_t /var/www/example/

# Allow nginx to make network connections (proxy_pass)
setsebool -P httpd_can_network_connect 1
```

## 3. HTTP → HTTPS Redirect

```nginx
# /etc/nginx/conf.d/app-http.conf
server {
    listen 80;
    server_name app.fiipractic.lan;
    return 301 https://$host$request_uri;
}
```

## 4. Wireshark — HTTP vs HTTPS

- **HTTP (port 80):** traffic is in cleartext — headers, HTML, everything visible
- **HTTPS (port 443):** traffic is encrypted (TLS) — nothing readable

Wireshark filters:
```
ip.addr == 192.168.100.10 && tcp.port == 80
ip.addr == 192.168.100.10 && tcp.port == 443
```

## 5. Netdata — Reverse Proxy

### Installation

```bash
yum install -y netdata
systemctl start netdata
systemctl enable netdata
ss -tlnp | grep netdata   # listens on port 19999
```

### Reverse Proxy on Subpath `/netdata/`

```nginx
# In /etc/nginx/conf.d/app-ssl.conf
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

### Maintenance Page (503)

```bash
cat > /var/www/example/maintenance.html << 'EOF'
<html><body><h1>Service temporarily unavailable - Maintenance in progress</h1></body></html>
EOF
chcon -t httpd_sys_content_t /var/www/example/maintenance.html
```

Test: `systemctl stop netdata` → access `/netdata/` → maintenance page.

### Reverse Proxy on Separate Subdomain

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

> DNS: add `192.168.100.10 netdata.fiipractic.lan` to `C:\Windows\System32\drivers\etc\hosts`

## 6. Basic Authentication

```bash
yum install -y httpd-tools
htpasswd -c /etc/nginx/.htpasswd admin
```

Then add to the nginx location block:

```nginx
auth_basic "Restricted Access";
auth_basic_user_file /etc/nginx/.htpasswd;
```

## 7. Encryption / Decryption with OpenSSL

### Decrypt a File with Private Key

```bash
openssl pkeyutl -decrypt -inkey myfiipracticapp.com.key -in plaintext.encrypt -out plaintext.decrypt
cat plaintext.decrypt
```

### Encrypt a File with Public Key from Certificate

```bash
# Extract public key
openssl x509 -pubkey -noout -in /usr/share/easy-rsa/3/pki/issued/app.crt > pubkey.pem

# Encrypt
openssl pkeyutl -encrypt -pubin -inkey pubkey.pem -in secret.txt -out secret.encrypt

# Decrypt
openssl pkeyutl -decrypt -inkey /usr/share/easy-rsa/3/pki/private/app.key -in secret.encrypt -out secret.decrypt
```

## Troubleshooting

| Problem | Solution |
|---|---|
| 403 Forbidden | `chcon -R -t httpd_sys_content_t /var/www/example/` |
| 502 Bad Gateway on proxy | `setsebool -P httpd_can_network_connect 1` |
| `maybe wrong password` in easy-rsa | Reset PKI: `rm -rf pki && ./easyrsa init-pki && ./easyrsa build-ca` |
| DNS_PROBE_FINISHED_NXDOMAIN | Add hostname to `C:\Windows\System32\drivers\etc\hosts` |
| `nginx -t` failed | Check config syntax: `cat /etc/nginx/conf.d/*.conf` |
