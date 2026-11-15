# Ansible Homelab & WSL Automation

Modular Ansible infrastructure-as-code setup for automating local Ubuntu WSL workstation configuration (`DESKTOP-84FQAHM`) and managing containerized services on a remote CasaOS homelab server (`imiucloud` / `192.168.1.126`) without disrupting existing CasaOS-managed services.

## Architecture & Nodes

| Host Alias | Target | OS / Environment | Purpose |
| :--- | :--- | :--- | :--- |
| `localhost` | `DESKTOP-84FQAHM` | Ubuntu on WSL2 | Ansible control node, local Docker Engine, CLI & network diagnostics |
| `homelab` | `192.168.1.126` (`imiucloud`) | Ubuntu Server + CasaOS | Remote homelab server running CasaOS (`8080`), Nextcloud (`7580`), Portainer (`9443`), Beszel (`8090`), and WatchYourLAN (`8840`) |

## Active Services & Web UIs (`192.168.1.126`)

| Service | URL | Managed By | Purpose |
| :--- | :--- | :--- | :--- |
| **CasaOS** | `http://192.168.1.126:8080` | CasaOS Native | Main server dashboard, file manager, and App Store |
| **Nextcloud** | `http://192.168.1.126:7580` | CasaOS App Store | Personal cloud storage and file sync |
| **Portainer CE** | `https://192.168.1.126:9443` | Ansible (`stacks` role) | Full Docker environment, container logs, and volume UI |
| **Beszel Hub + Agent** | `http://192.168.1.126:8090` | Ansible (`stacks` role) | Hardware telemetry (CPU, RAM, disk, network) & Docker stats |
| **WatchYourLAN** | `http://192.168.1.126:8840` | Ansible (`stacks` role) | Real-time `enp2s0` (`192.168.1.0/24`) LAN device scanner |

## Repository Structure

```text
ansible-local/
├── ansible.cfg          # Local inventory path, timeout, sudo defaults, and vault_password_file
├── hosts.ini            # Inventory grouping local workstation and homelab
