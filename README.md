# Red Team Isolated Lab & Home Server Directory

| Service | Direct URL | Purpose & Day-to-Day Use |
| :--- | :--- | :--- |
| **Cockpit (KVM Manager)** | `https://192.168.1.126:9090/machines` | Web hypervisor UI to manage `opnsense-fw`, attach Kali/target VMs to `redteam-lan-imi`, and open the VM console |
| **Ubuntu KVM Host (SSH)** | `ssh imi@192.168.1.126` (Port `22`) | Host terminal access for `virsh` commands, managing bridge `virbr1` (`10.10.10.254/24`), and creating SSH tunnels |
| **OPNsense 26.7 WebGUI** | `https://localhost:9443/` *(via SSH tunnel to `10.10.10.1:443`)* | Firewall rule management, DHCP leases (`10.10.10.0/24`), `RFC1918_Private` isolation aliases, and DNS (`53`) |
| **CasaOS** | `http://192.168.1.126:8080` *(or `http://192.168.1.126`)* | Main server dashboard, file manager, storage status, and CasaOS App Store |
| **Nextcloud** | `http://192.168.1.126:7580` | Personal cloud storage, file sync, and document access |
| **Portainer CE** | `https://192.168.1.126:9443` | Full Docker management UI (inspect container logs, restart containers, view volumes/networks) |
| **Beszel** | `http://192.168.1.126:8090` | Real-time hardware telemetry (CPU, RAM, disk, network throughput) and per-container resource usage |
| **WatchYourLAN** | `http://192.168.1.126:8840` | Live `192.168.1.0/24` LAN device scanner, MAC/IP inventory, and online/offline history |

## How to Open OPNsense WebGUI
1. Ensure `virbr1` holds `10.10.10.254/24` on the KVM host and open the SSH tunnel from WSL2:
   ```bash
   ssh -t imi@192.168.1.126 'sudo ip addr del 10.10.10.1/24 dev virbr1 2>/dev/null; sudo ip addr add 10.10.10.254/24 dev virbr1 2>/dev/null'
   ssh -N -L 0.0.0.0:9443:10.10.10.1:443 imi@192.168.1.126


Open https://localhost:9443/ in Brave and log in as root.
