# Home Lab & Isolated Red Team Infrastructure

Unified operational guide, network map, and startup reference for the **Ubuntu KVM / CasaOS Home Server (`imiucloud`)** and the **Isolated Red Team Lab (`redteam-lan-imi`)**.

---

## 1. Network Architecture

| Segment / Interface | Subnet / IP | Gateway | Purpose |
| :--- | :--- | :--- | :--- |
| **Home LAN** (`enp*` / Wi-Fi) | `192.168.1.0/24` | `192.168.1.1` | Primary home network hosting the hypervisor (`192.168.1.126`) |
| **Libvirt Default NAT** (`virbr0`) | `192.168.122.0/24` | `192.168.122.1` | Upstream WAN interface (`vtnet0` / `192.168.122.132`) for `opnsense-fw` |
| **Isolated Red Team LAN** (`virbr1` / `redteam-lan-imi`) | `10.10.10.0/24` | `10.10.10.1` | Isolated lab network for Kali Linux and target VMs (Host bridge IP: `10.10.10.254`) |
| **ZeroTier Mesh** (`zt*`) | Overlay VPN | — | Remote encrypted access to `192.168.1.126` outside the home LAN |

---

## 2. Complete Service Directory (URLs & Ports)

| Service | Direct URL | Host / Container | Purpose & Day-to-Day Use |
| :--- | :--- | :--- | :--- |
| **Cockpit (KVM Manager)** | `https://192.168.1.126:9090/machines` | Systemd (`cockpit.socket`) | Manage `opnsense-fw`, attach Kali/target VMs to `redteam-lan-imi`, and open VM consoles |
| **Ubuntu KVM Host (SSH)** | `ssh imi@192.168.1.126` (Port `22`) | Systemd (`ssh`) | Hypervisor shell for `virsh`, Docker management, bridge networking, and SSH tunnels |
| **OPNsense 26.7 WebGUI** | `https://localhost:9443/` *(via SSH tunnel)* | KVM VM (`opnsense-fw`) | Firewall rules, `RFC1918_Private` isolation aliases, DHCP (`10.10.10.0/24`), and DNS (`53`) |
| **CasaOS** | `http://192.168.1.126:8080` *(or `http://192.168.1.126`)* | Systemd (`casaos`) | Main server dashboard, file manager, storage status, and CasaOS App Store |
| **Nextcloud** | `http://192.168.1.126:7580` | Docker (`nextcloud`) | Personal cloud storage, file sync, and document access |
| **Portainer CE** | `https://192.168.1.126:9443` | Docker (`portainer`) | Full Docker management UI (inspect container logs, restart containers, view volumes/networks) |
| **Beszel** | `http://192.168.1.126:8090` | Docker (`beszel` / `beszel-agent`) | Real-time hardware telemetry (CPU, RAM, disk, network throughput) and per-container usage |
| **WatchYourLAN** | `http://192.168.1.126:8840` | Docker (`watchyourlan`) | Live `192.168.1.0/24` LAN device scanner, MAC/IP inventory, and online/offline history |

> **Port Collision Note:** Portainer CE runs on `192.168.1.126:9443`, while the OPNsense SSH tunnel binds locally on your workstation at `localhost:9443`. Always use `192.168.1.126:9443` for Portainer and `localhost:9443` for OPNsense.

---

## 3. How to Start & Run Every Service

### A. Red Team Lab (`opnsense-fw` + `redteam-lan-imi`)
1. **Start the Libvirt Network & OPNsense VM** (on `192.168.1.126`):
   ```bash
   sudo virsh net-start redteam-lan-imi 2>/dev/null || true
   sudo virsh start opnsense-fw
