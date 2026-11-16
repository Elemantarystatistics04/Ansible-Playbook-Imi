# Red Team Isolated Lab (OPNsense + KVM)

- **Hypervisor:** Ubuntu KVM (`192.168.1.126`)
- **Firewall Guest:** `opnsense-fw` (OPNsense 26.7)
- **WAN:** libvirt `default` (NAT)
- **Isolated Lab LAN:** `redteam-lan-imi` (`virbr1`, `10.10.10.0/24`, gateway `10.10.10.1`, host bridge `10.10.10.254`)
- **WebGUI Tunnel:** `ssh -N -L 0.0.0.0:9443:10.10.10.1:443 imi@192.168.1.126`
