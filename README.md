# DevOps — FiiPractic 2026

Notes and configurations from the FiiPractic 2026 DevOps course.

## Infrastructure

Two Rocky Linux 9 VMs managed with Vagrant:

| VM | Hostname | IP | Resources |
|---|---|---|---|
| App | app.fiipractic.lan | 192.168.100.10 | 2 CPU, 2 GB RAM |
| GitLab | gitlab.fiipractic.lan | 192.168.100.20 | 4 CPU, 4 GB RAM |

## Sessions

| # | Topic | Notes |
|---|---|---|
| 01 | Virtualization & Provisioning | [Session-01-Virtualization.md](Session-01-Virtualization.md) |
| 02 | Linux, Bash & Networking | [Session-02-Linux.md](Session-02-Linux.md) |
| 03 | Web Server, HTTPS & Reverse Proxy | [Session-03-WebServer.md](Session-03-WebServer.md) |

## Tools Used

- **VirtualBox** — hypervisor
- **Vagrant** — VM provisioning
- **MobaXterm** — SSH client (Windows)
- **Nginx** — web server / reverse proxy
- **Easy-RSA** — certificate authority
- **Netdata** — real-time monitoring
- **Wireshark** — network traffic analysis
- **OpenSSL** — encryption / decryption
