# Home Lab & Isolated Red Team Infrastructure

Unified operational guide, network map, and startup reference for the **Ubuntu KVM / CasaOS Home Server (`imiucloud` / `192.168.1.126`)**, **Windows WSL2 Workstation (`DESKTOP-84FQAHM`)**, and the **Isolated Red Team Lab (`redteam-lan-imi`)**.

---

## 1. Network Architecture

| Segment / Interface | Subnet / IP | Gateway | Purpose |
| :--- | :--- | :--- | :--- |
| **Home LAN** (`enp*` / Wi-Fi) | `192.168.1.0/24` | `192.168.1.1` | Primary home network hosting the hypervisor (`192.168.1.126`) |
| **Libvirt Default NAT** (`virbr0`) | `192.168.122.0/24` | `192.168.122.1` | Upstream WAN interface (`vtnet0` / `192.168.122.132`) for `opnsense-fw` |
| **Isolated Red Team LAN** (`virbr1` / `redteam-lan-imi`) | `10.10.10.0/24` | `10.10.10.1` | Isolated lab network for Kali Linux and target VMs (Host bridge IP: `10.10.10.254`) |
| **Tailscale / ZeroTier Mesh** | `100.99.61.58` / `10.241.211.197` | Overlay VPN | Encrypted remote access to `192.168.1.126` from outside the home LAN |

---

## 2. Complete Service Directory (URLs & Ports)

| Service | Direct URL | Host / Container | Purpose & Day-to-Day Use |
| :--- | :--- | :--- | :--- |
| **Cockpit (KVM Manager)** | `https://192.168.1.126:9090/machines` | `192.168.1.126` (`cockpit.socket`) | Manage `opnsense-fw`, attach Kali/target VMs to `redteam-lan-imi`, and open VM consoles |
| **Ubuntu KVM Host (SSH)** | `ssh imi@192.168.1.126` (Port `22`) | `192.168.1.126` (`sshd`) | Hypervisor shell for `virsh`, Docker management, bridge networking, and SSH tunnels |
| **OPNsense 26.7 WebGUI** | `https://localhost:9443/` *(via `start-lab` tunnel)* | KVM VM (`opnsense-fw`) | Firewall rules, `RFC1918_Private` isolation aliases, DHCP (`10.10.10.0/24`), and DNS (`53`) |
| **CasaOS Dashboard** | `http://192.168.1.126:8080` | `192.168.1.126` (`casaos-gateway`) | Main server dashboard, file manager, storage status, and CasaOS App Store |
| **Nextcloud** | `http://192.168.1.126:7580` | `192.168.1.126` (`nextcloud`) | Personal cloud storage, file sync, and document access |
| **Portainer CE** | `https://192.168.1.126:9443` | `192.168.1.126` (`portainer`) | Full Docker management UI (inspect container logs, restart containers, view volumes/networks) |
| **Beszel** | `http://192.168.1.126:8090` | `192.168.1.126` (`beszel`) | Real-time hardware telemetry (CPU, RAM, disk, network throughput) and per-container usage |
| **WatchYourLAN** | `http://192.168.1.126:8840` | `192.168.1.126` (`watchyourlan`) | Live `192.168.1.0/24` LAN device scanner, MAC/IP inventory, and online/offline history |
| **OWASP Juice Shop** | `http://192.168.1.126:3000` | `192.168.1.126` (`redteam-juice-shop`) | Vulnerable web application target for Red Team practice |
| **DVWA** | `http://192.168.1.126:8081` | `192.168.1.126` (`redteam-dvwa`) | Damn Vulnerable Web Application target container |
| **Blinko** | `http://192.168.1.126:1111` | `192.168.1.126` (`blinko`) | Self-hosted notes and quick-capture service |
| **n8n Automation** | `http://localhost:5678` | Local WSL2 (`n8n`) | Workflow automation engine running locally on `DESKTOP-84FQAHM` |

---

## 3. Daily Startup & Recovery Commands

### Open OPNsense WebGUI After PC Reboot (in WSL2)
```bash
start-lab
# Then open https://localhost:9443/ in Brave (login: root)
