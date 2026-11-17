# Home Lab & Isolated Red Team Infrastructure

Complete operational reference, IP/port map, service directory, and recovery commands for the **Windows 11 WSL2 Desktop Host (DESKTOP-84FQAHM)**, **ZeroTier Overlay Network (10.241.54.218)**, and the **Isolated Red Team Lab (redteam-lan-imi)**.

---

## 1. Network Architecture & Access Strategy

| Segment / Interface | Subnet / IP | Access Model | Purpose |
| :--- | :--- | :--- | :--- |
| **Desktop Local Loopback (WSL2 Mirrored)** | localhost (127.0.0.1) | Direct on DESKTOP-84FQAHM | Primary local access on the desktop PC; keeps services isolated from home LAN (192.168.1.15) |
| **ZeroTier Overlay VPN (eth2)** | 10.241.0.0/16 (10.241.54.218) | Encrypted Mesh (Phone / Laptop / Outside) | Remote access to all homelab services anywhere over mobile data or external Wi-Fi (bypasses Maxis CGNAT) |
| **Libvirt Default NAT (virbr0)** | 192.168.122.0/24 (192.168.122.1) | Internal VM NAT | Upstream WAN uplink (vtnet0 / 192.168.122.132) for opnsense-fw |
| **Isolated Red Team LAN (virbr1 / redteam-lan-imi)** | 10.10.10.0/24 (10.10.10.254 host) | Air-Gapped Lab Segment | Isolated network for Kali Linux and target VMs behind OPNsense |

---

## 2. Service & Application Directory (DESKTOP-84FQAHM)

### Monitoring, Docker Management & LAN Discovery

| Application / Service | Desktop Local URL | ZeroTier Remote URL | Container / Stack | Function |
| :--- | :--- | :--- | :--- | :--- |
| **Portainer CE** | https://localhost:9443/ | https://10.241.54.218:9443/ | portainer | Web UI for managing Docker containers, inspecting logs, restarting stacks, and managing volumes |
| **Beszel Hub** | http://localhost:8090/ | http://10.241.54.218:8090/ | beszel | Real-time server telemetry dashboard (CPU, RAM, disk I/O, network throughput, and container metrics) |
| **Beszel Agent** | localhost:45876 (internal) | 10.241.54.218:45876 (internal) | beszel-agent | Host telemetry collector streaming hardware and Docker stats to Beszel Hub |
| **WatchYourLAN** | http://localhost:8840/ | http://10.241.54.218:8840/ | watchyourlan | Continuous 192.168.1.0/24 LAN ARP scanner tracking IP/MAC inventory and device online/offline history |

### Self-Hosted Apps, Automation & Red Team Targets

| Application / Service | Desktop Local URL | ZeroTier Remote URL | Container / Stack | Function |
| :--- | :--- | :--- | :--- | :--- |
| **Nextcloud** | http://localhost:7580/ | http://10.241.54.218:7580/ | nextcloud (+ db-nextcloud, redis-nextcloud, cron) | Self-hosted cloud file storage, sync, and personal document hub |
| **Blinko** | http://localhost:1111/ | http://10.241.54.218:1111/ | blinko (+ blinko-postgres) | Self-hosted quick-capture notes and personal knowledge base |
| **n8n Automation** | http://localhost:5678/ | http://10.241.54.218:5678/ | n8n | Workflow automation engine running locally in WSL2 (`N8N_SECURE_COOKIE=false` for HTTP) |
| **OWASP Juice Shop** | http://localhost:3000/ | http://10.241.54.218:3000/ | redteam-juice-shop | Intentionally vulnerable modern web application for Red Team / AppSec practice |
| **CyberChef** | http://localhost:8080/ | http://10.241.54.218:8080/ | big-bear-cyberchef | Data decoding, cryptography, encoding, and CTF analysis Swiss-army knife |
| **Cockpit Server GUI** | https://localhost:9090/ | https://10.241.54.218:9090/ | cockpit.socket (systemd) | Web-based Linux server administration, real-time telemetry, and browser terminal |
| **DVWA** | http://localhost:8081/ | http://10.241.54.218:8081/ | redteam-dvwa | Damn Vulnerable Web Application for practicing SQLi, XSS, CSRF, and command injection |

| **Hermes Agent** | http://localhost:9119/ | http://10.241.54.218:9119/ | `hermes-agent` | Autonomous AI agent & web dashboard powered by OpenRouter (Basic Auth enabled) |
### Hypervisor & Isolated Red Team Firewall

| Application / Service | Access URL / Endpoint | Host / Runtime | Function |
| :--- | :--- | :--- | :--- |
| **OPNsense 26.7 Firewall** | https://localhost:9443/ (via start-lab tunnel) | KVM VM (opnsense-fw / 10.10.10.1) | Perimeter firewall, DHCP/DNS (53), and RFC1918_Private isolation for 10.10.10.0/24 |

---

## 3. How to Run & Recover the Stack (DESKTOP-84FQAHM)

### A. Verify WSL2 Interfaces & Running Containers

    ip -4 addr show | grep -E "zt|10\.241\."
    sudo docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

### B. Start Docker Engine & All Containers / Compose Stacks

    sudo service docker start
    sudo docker start portainer beszel beszel-agent watchyourlan n8n redteam-juice-shop redteam-dvwa
    sudo docker compose -f /var/lib/casaos/apps/big-bear-nextcloud/docker-compose.yml up -d
    sudo docker compose -f /var/lib/casaos/apps/blinko/docker-compose.yml up -d
    sudo docker compose -f /var/lib/casaos/apps/big-bear-cyberchef/docker-compose.yml up -d
    sudo systemctl enable --now cockpit.socket

### C. Nextcloud Trusted Domains & n8n HTTP Cookie Configuration

If Nextcloud or n8n is ever recreated, ensure ZeroTier IP (`10.241.54.218`) is whitelisted and HTTP cookies are enabled:

    sudo docker exec -u www-data nextcloud php occ config:system:set trusted_domains 5 --value="10.241.54.218"

For `n8n` over HTTP (`http://localhost:5678` or `http://10.241.54.218:5678`), run with `-e N8N_SECURE_COOKIE=false`.

### D. Allow ZeroTier Inbound Ports in Windows Firewall (PowerShell Admin)

If ZeroTier remote access (`10.241.54.218`) is blocked after a Windows or ZeroTier update, run in PowerShell as Administrator:

    Get-NetConnectionProfile | Where-Object {$_.InterfaceAlias -like "*ZeroTier*"} | Set-NetConnectionProfile -NetworkCategory Private
    New-NetFirewallRule -DisplayName "ZeroTier Homelab Services" -Direction Inbound -Protocol TCP -LocalPort 3000,5678,7580,8080,8081,8090,8840,9443,1111 -Action Allow

### E. Bring Back OPNsense & Isolated Red Team Lab

    start-lab
