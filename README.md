# Ansible Homelab & WSL Automation

Modular Ansible infrastructure-as-code setup for automating local Ubuntu WSL workstation configuration (`DESKTOP-84FQAHM`) and managing containerized services on a remote CasaOS homelab server (`imiucloud` / `192.168.1.126`) without disrupting existing CasaOS-managed services.

## Architecture & Nodes

| Host Alias | Target | OS / Environment | Purpose |
| :--- | :--- | :--- | :--- |
| `localhost` | `DESKTOP-84FQAHM` | Ubuntu on WSL2 | Ansible control node, local Docker Engine, CLI & network diagnostics |
| `homelab` | `192.168.1.126` (`imiucloud`) | Ubuntu Server + CasaOS | Remote homelab server running Portainer (`9443`), CasaOS (`8080`), and Nextcloud (`7580`) |

## Repository Structure

```text
ansible-local/
├── ansible.cfg          # Local inventory path, 30s timeout, and sudo privilege defaults
├── hosts.ini            # Inventory grouping local workstation and homelab nodes
├── setup.yml            # Main multi-node playbook orchestrating all roles
├── .gitignore           # Excludes retry logs, local secret files, and cache artifacts
└── roles/
    ├── docker/          # Installs Docker CE & Compose plugin on local WSL node only
    ├── configs/         # Applies baseline shell/environment configurations
    ├── devtools/        # Installs CLI & network diagnostic tools (nmap, tcpdump, iperf3, mtr-tiny, jq)
    └── stacks/          # Deploys /opt/homelab-stack/docker-compose.yml (Portainer CE on port 9443)
