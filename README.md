# Ansible Homelab & Red Team Lab Automation

Modular Ansible infrastructure-as-code setup for automating local Ubuntu WSL workstation configuration (`DESKTOP-84FQAHM`), managing production monitoring services, and deploying on-demand Red Team vulnerable targets on a remote CasaOS homelab server (`imiucloud` / `192.168.1.126`).

## Architecture & Nodes

| Host Alias | Target | OS / Environment | Purpose |
| :--- | :--- | :--- | :--- |
| `localhost` | `DESKTOP-84FQAHM` | Ubuntu on WSL2 | Ansible control node, local Docker Engine, Red Team recon & fuzzing toolkit |
| `homelab` | `192.168.1.126` (`imiucloud`) | Ubuntu Server + CasaOS | Remote server hosting CasaOS, monitoring tools, and isolated Red Team lab targets |

## Active Services & Web UIs (`192.168.1.126`)

| Category | Service | URL | Managed By | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Core** | **CasaOS** | `http://192.168.1.126:8080` | CasaOS Native | Main server dashboard, file manager, and App Store |
| **Core** | **Nextcloud** | `http://192.168.1.126:7580` | CasaOS App Store | Personal cloud storage and file sync |
| **Monitoring** | **Portainer CE** | `https://192.168.1.126:9443` | Ansible (`stacks` role) | Full Docker environment, container logs, and volume UI |
| **Monitoring** | **Beszel Hub + Agent** | `http://192.168.1.126:8090` | Ansible (`stacks` role) | Hardware telemetry (CPU, RAM, disk, network) & Docker stats |
| **Monitoring** | **WatchYourLAN** | `http://192.168.1.126:8840` | Ansible (`stacks` role) | Real-time `enp2s0` (`192.168.1.0/24`) LAN device scanner |
| **Red Team Lab** | **OWASP Juice Shop** | `http://192.168.1.126:3000` | Ansible (`redteam_lab` role) | Vulnerable Node.js/Angular app for OWASP Top 10 practice |
| **Red Team Lab** | **DVWA** | `http://192.168.1.126:8081` | Ansible (`redteam_lab` role) | Damn Vulnerable Web App (`admin` / `password`) for SQLi/XSS/fuzzing |

## Local Network IP & Device Map (`192.168.1.0/24` via `enp2s0`)

| IP Address | MAC Address | Hardware / Vendor | Role / Identification |
| :--- | :--- | :--- | :--- |
| `192.168.1.254` | `64:5e:10:89:17:8e` | Huawei Technologies | Main LAN Gateway / Router (`_gateway.`) |
| `192.168.1.126` | *Host (`enp2s0`)* | `imiucloud` Server | Homelab Docker Host (CasaOS, Monitoring, Red Team Targets) |
| `192.168.1.28` | `64:5e:10:89:17:17` | Huawei Technologies | Secondary Huawei Network Node / AP |
| `192.168.1.15` | `c8:7f:54:56:7e:e7` | ASUSTek Computer | ASUS Workstation / Laptop |
| `192.168.1.54` | `80:54:e3:76:cd:e8` | Apple, Inc. | Apple Client Device |
| `192.168.1.146` | `ac:16:15:16:46:b1` | Apple, Inc. | Apple Client Device |
| `192.168.1.125` | `6c:e8:c6:97:76:08` | Earda Technologies | Smart Home / IoT Device |
| `192.168.1.134` | `7e:9f:b8:39:e0:a0` | Locally Administered | Mobile Device (Private / Randomized Wi-Fi MAC) |
| `172.25.77.124` | `00:15:5d:ff:31:f2` | Microsoft Hyper-V (`eth0`) | Local WSL2 Ansible Controller (`DESKTOP-84FQAHM`) |

## Repository Structure

```text
ansible-local/
├── ansible.cfg          # Local inventory path, timeout, sudo defaults, and vault_password_file
├── hosts.ini            # Inventory grouping local workstation and homelab nodes
├── setup.yml            # Main multi-node playbook with base, monitoring, and redteam tags
├── .gitignore           # Excludes retry logs, local secret files, and cache artifacts
├── group_vars/
│   └── homelab/
│       └── vault.yml    # AES-256 Ansible Vault encrypted Beszel agent keys and tokens
└── roles/
    ├── docker/          # Installs Docker CE & Compose plugin on local WSL node only
    ├── configs/         # Applies baseline shell/environment configurations
    ├── devtools/        # Installs recon & fuzzing CLI tools (nmap, ffuf, gobuster, sqlmap, hydra, dirb)
    ├── stacks/          # Deploys /opt/homelab-stack/docker-compose.yml (Portainer, Beszel, WatchYourLAN)
    └── redteam_lab/     # Deploys /opt/redteam-lab/docker-compose.yml (OWASP Juice Shop & DVWA)
# Deploy or update core monitoring stack only
ansible-playbook -i hosts.ini setup.yml --tags monitoring

# Deploy Red Team recon CLI tools and vulnerable web targets
ansible-playbook -i hosts.ini setup.yml --tags redteam

# Tear down Red Team vulnerable targets after a lab session
ssh imi@192.168.1.126 "docker compose -f /opt/redteam-lab/docker-compose.yml down"
