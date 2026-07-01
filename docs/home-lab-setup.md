# Home Lab Setup

## Host

Windows 11 + VirtualBox

## Virtual Machines

| VM | Role | Status |
|---|---|---|
| Kali Linux | attacking machine (primary) | ✅ |
| Parrot OS | attacking machine (secondary) | ✅ |
| Metasploitable2 | vulnerable target | ✅ in progress |
| Windows VM (HTB Academy course) | practice / future AD client | ✅ |
| Windows Server (DC) | Active Directory environment | 🔜 Phase 2 |
| ELK Stack / Splunk | SIEM monitoring | 🔜 Phase 4 |

## Networking

- Host-Only network: `192.168.56.0/24`
- HTB VPN: свързан от Kali Linux

## Tooling

- HTB Academy — Silver plan (Tier I + II)
- PortSwigger Web Academy — free web exploitation practice
- Windows VM license — `slmgr /rearm` + snapshot преди изтичане

See [network-diagram.md](network-diagram.md) for topology.
