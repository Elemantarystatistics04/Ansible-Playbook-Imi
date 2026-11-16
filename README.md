# Home Lab & Isolated Red Team Infrastructure

Complete operational reference, IP/port map, service functions, and recovery commands for the **Ubuntu KVM / CasaOS Home Server (imiucloud / 192.168.1.126)**, **Windows WSL2 Workstation (DESKTOP-84FQAHM)**, and the **Isolated Red Team Lab (redteam-lan-imi)**.

---

## 1. Network Architecture & IP Allocation

| Segment / Interface | Subnet / IP | Gateway | Purpose |
| :--- | :--- | :--- | :--- |
| **Home LAN** (enp* / Wi-Fi) | 192.168.1.0/24 (192.168.1.126) | 192.168.1.1 | Primary home network hosting imiucloud and all LAN-accessible web apps |
| **Libvirt Default NAT** (virbr0) | 192.168.122.0/24 (192.168.122.1) | 192.168.122.1 | Upstream WAN uplink (vtnet0 / 192.168.122.132) for opnsense-fw |
| **Isolated Red Team LAN** (virbr1 / redteam-lan-imi) | 10.10.10.0/24 (10.10.10.254 host) | 10.10.10.1 | Air-gapped lab segment for Kali Linux and target VMs behind OPNsense |
| **Tailscale / ZeroTier Mesh** | 100.99.61.58 / 10.241.211.197 | Overlay VPN | Encrypted remote access to imiucloud from outside the home LAN |

---

## 2. Service & Application Directory

### Hypervisor, Virtualization & Network Infrastructure (192.168.1.126)

| Application / Service | IP & URL | Host / Runtime | Function |
| :--- | :--- | :--- | :--- |
| **Cockpit (KVM Manager)** | https://192.168.1.126:9090/machines | 192.168.1.126 (cockpit.socket) | Manage KVM virtual machines (opnsense-fw, Kali, target VMs), virtual networks, and VM consoles |
| **OPNsense 26.7 Firewall** | https://localhost:9443/ (via start-lab tunnel) | KVM VM (opnsense-fw / 10.10.10.1) | Perimeter firewall, DHCP/DNS (53), and RFC1918_Private isolation for 10.10.10.0/24 |
| **Ubuntu KVM Host (SSH)** | ssh imi@192.168.1.126 (Port 22) | 192.168.1.126 (sshd) | Direct hypervisor shell for virsh, Docker containers, bridge interfaces, and SSH tunnels |
| **CasaOS Dashboard** | http://192.168.1.126:8080 | 192.168.1.126 (casaos-gateway) | Main server overview, file manager, storage monitoring, and CasaOS App Store |

### Monitoring, Docker Management & LAN Discovery (192.168.1.126)

| Application / Service | IP & URL | Container Name | Function |
| :--- | :--- | :--- | :--- |
| **Portainer CE** | https://192.168.1.126:9443/ | portainer | Web UI for managing Docker containers, inspecting logs, restarting stacks, and managing volumes |
| **Beszel Hub** | http://192.168.1.126:8090/ | beszel | Real-time server telemetry dashboard (CPU, RAM, disk I/O, network throughput, and container metrics) |
| **Beszel Agent** | 192.168.1.126:45876 (internal) | beszel-agent | Host telemetry collector streaming hardware and Docker stats to Beszel Hub |
| **WatchYourLAN** | http://192.168.1.126:8840/ | watchyourlan | Continuous 192.168.1.0/24 LAN ARP scanner tracking IP/MAC inventory and device online/offline history |

### Self-Hosted Apps, Red Team Targets & Local Workstation Tools

| Application / Service | IP & URL | Host / Container | Function |
| :--- | :--- | :--- | :--- |
| **Nextcloud** | http://192.168.1.126:7580/ | 192.168.1.126 (nextcloud) | Self-hosted cloud file storage, sync, and personal document hub |
| **Blinko** | http://192.168.1.126:1111/ | 192.168.1.126 (blinko) | Self-hosted quick-capture notes and personal knowledge base |
| **OWASP Juice Shop** | http://192.168.1.126:3000/ | 192.168.1.126 (redteam-juice-shop) | Intentionally vulnerable modern web application for Red Team / AppSec practice |
| **DVWA** | http://192.168.1.126:8081/ | 192.168.1.126 (redteam-dvwa) | Damn Vulnerable Web Application for practicing SQLi, XSS, CSRF, and command injection |
| **n8n Automation** | http://localhost:5678/ | WSL2 (DESKTOP-84FQAHM / n8n) | Workflow automation engine running locally on the Windows WSL2 workstation |

---

## 3. Recovery & Startup Playbook (If Any Service Is Down)

### A. Check Status of All Services on 192.168.1.126

    ssh -t imi@192.168.1.126 "sudo virsh list --all && sudo docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"

### B. Bring Back All Docker Containers on 192.168.1.126

All server containers (portainer, beszel, beszel-agent, watchyourlan, nextcloud, blinko, redteam-juice-shop, redteam-dvwa) use --restart=unless-stopped. If any container is Exited or stopped:

    ssh -t imi@192.168.1.126 "sudo docker start portainer beszel beszel-agent watchyourlan nextcloud blinko redteam-juice-shop redteam-dvwa"

### C. Bring Back OPNsense & Isolated Lab Network (https://localhost:9443/)

Run in WSL2 (DESKTOP-84FQAHM) after a Windows or Ubuntu reboot to bring up redteam-lan-imi (virbr1), boot opnsense-fw, and open the SSH port-forward:

    start-lab

Manual equivalent if start-lab is unavailable:

    ssh -t imi@192.168.1.126 "sudo virsh net-start redteam-lan-imi 2>/dev/null || true; sudo ip link set virbr1 up; sudo ip addr add 10.10.10.254/24 dev virbr1 2>/dev/null || true; sudo virsh start opnsense-fw 2>/dev/null || true"
    ssh -N -L 0.0.0.0:9443:10.10.10.1:443 imi@192.168.1.126

### D. Bring Back CasaOS or Cockpit If Web UIs Are Unreachable

    ssh -t imi@192.168.1.126 "sudo systemctl restart casaos-gateway cockpit.socket"

### E. Bring Back Local n8n on Windows WSL2 (DESKTOP-84FQAHM)

    docker start n8n
