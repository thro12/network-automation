# network-automation

Python network automation scripts and labs (Netmiko, SSH, GNS3).

# Python Network Automation - Complete Lab Guide

Documenting the network automation journey on my Mac (Apple M3) with GNS3, from the first
Netmiko script to a 3-router OSPF lab configured entirely over SSH - plus how to connect a
physical device from Mac Terminal (console cable and SSH).

**Full study guide:** `Python_Network_Automation_Complete_Guide.pdf` (19 pages: concepts,
every step we actually ran, errors we hit and fixed, interview Q&A, full command chart) -
upload pending (GitHub binary upload failing); currently delivered separately and backed up in Drive.

## The journey in six stages

| Stage | What it does | Connection | Result |
| --- | --- | --- | --- |
| 1. `collect.py` | Runs `show ip interface brief` on one router, saves `r1-status.txt` | GNS3 local console (port 5020) | Working |
| 2. `collect-full.py` | 55 read-only checks into an `r1-reports` folder | GNS3 local console | Working (33 OK / 17 NO-OUTPUT / 5 UNSUPPORTED) |
| 3. `auto-lab.py` + `reset-lab.py` | Blank lab (1 router, 2 switches, 4 PCs): give only a network range, script plans and assigns all IPs/gateways, runs 20 ping checks | GNS3 local console (Python sockets) | All 20 checks passed |
| 4. SSH enablement | Router SSH config + Mac-to-GNS3 path via macOS `feth` pair and a Cloud node | SSH from Mac Terminal | Ping 0% loss, SSH session established |
| 5. `ssh-collect.py` | Read-only collector over SSH | Netmiko + Paramiko (dedicated venv, Paramiko 4.0.0 pinned for the old IOS) | Report saved |
| 6. `lab3-all-v2.py` | 3 routers + 6 VPCS: configures IPs and OSPF over SSH, verifies all 30 PC-to-PC pings, OSPF neighbours/routes, traceroutes; saves only on success | SSH (management switch design) + console for PCs | 30/30 pings 5/5, OSPF verified |

## Setup (macOS)

```bash
# Console scripts
python3 -m venv ~/router-venv
~/router-venv/bin/python -m pip install netmiko

# SSH scripts (Paramiko pinned: v5 dropped legacy algorithms the old IOS needs)
~/router-venv/bin/python -m venv ~/router-ssh-venv
~/router-ssh-venv/bin/python -m pip install "netmiko==4.8.0" "paramiko==4.0.0"
```

The Mac-to-GNS3 network path uses a `feth` interface pair bound to a GNS3 Cloud node.
It is temporary: recreate it after every Mac restart (commands in the guide, Part 6).

## Physical device from Mac Terminal

- **Case 1 - console cable:** USB-to-serial adapter to the CONSOLE port, then
  `screen /dev/cu.usbserial-XXXX 9600` (Ctrl+A, K to exit). No IP needed - used for
  first-time setup.
- **Case 2 - SSH:** one-time router prep (hostname, domain, RSA keys, local user, vty
  `login local` + `transport input ssh`), then `ssh user@device-ip`. The same Netmiko
  scripts then work unchanged against the physical device.

## Safety rules followed

- No passwords in code: hidden local prompts, then the macOS Keychain for reuse.
- Read-only collectors before any configuration change.
- Scripts show the plan and wait for confirmation; devices are saved only after checks pass.
- Legacy SSH options (`-o KexAlgorithms=...`) and the Paramiko pin are scoped to the lab only.

## Status notes

- Lab 3 router config saves were not yet script-confirmed (`router-save-check.py` prepared).
- Physical-device steps are the standard macOS/Cisco procedure, not yet run against my
  physical device.
