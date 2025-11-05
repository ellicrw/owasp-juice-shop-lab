# OWASP Juice Shop Pentest Lab — Windows 11 (Docker) + Kali (VMware Host-Only / VMnet1)

## Overview
This lab runs OWASP Juice Shop on a Windows 11 host (Docker) and uses a Kali Linux VM (VMware) in Host-Only (VMnet1) mode as the attacker machine. From Kali you should be able to access the Juice Shop served from the Windows host at port 3000:

- Host: Windows 11 — Juice Shop served from Docker on port 3000.
- Attacker: Kali Linux VM (VMware) — Host-Only (VMnet1).
- Goal: From Kali, open Firefox / curl / nmap → http://<vmnet1-ip>:3000 to practice OWASP Top 10.

---

## Requirements
- Windows 11 (admin access)
- Docker Desktop for Windows (or Docker Engine)
- VMware Workstation Player / Pro (or VMware Fusion)
- Official Kali Linux VMware image
- Basic familiarity with PowerShell and VMware UI

---

## 1) Run OWASP Juice Shop on Windows 11 (Docker — recommended)

Open PowerShell as Administrator and run:

```powershell
# pull latest image
docker pull bkimminich/juice-shop

# run Juice Shop mapped to all interfaces on port 3000
docker run --rm -d -p 0.0.0.0:3000:3000 bkimminich/juice-shop
```

Verify on the Windows host by opening:
```
http://localhost:3000
```

Add a Windows Firewall rule (PowerShell admin) to allow inbound TCP on port 3000:

```powershell
New-NetFirewallRule -DisplayName "JuiceShop 3000" -Direction Inbound -LocalPort 3000 -Protocol TCP -Action Allow
```

Important: make sure the container was started with `0.0.0.0:3000:3000`. If it's bound only to `127.0.0.1`, the Kali VM won't reach it.

---

## 2) Configure VMware — Host-Only (VMnet1)

We use Host-Only (VMnet1) so the Kali VM and the host can communicate while keeping the lab isolated from your other networks.

1. Power off the Kali VM.
2. In VMware: Select the Kali VM → Settings → Network Adapter.
3. Choose **Host-Only (VMnet1)**. Apply and close.

On Windows, the adapter is usually named: `VMware Network Adapter VMnet1`. We'll use that adapter's IPv4 address as the host IP that Kali will reach.

---

## 3) PowerShell — find the VMnet1 (host) IP

Open PowerShell (Admin) and run:

```powershell
# show Windows adapter list to find the VMware Host-Only (VMnet1) entry
ipconfig /all
```

Find the adapter called `VMware Network Adapter VMnet1` and copy its IPv4 Address (example: `192.168.56.1`). That is the host-side VMnet1 IP — Kali can reach services bound to the host via that IP.

---

## 4) Start Kali (Host-Only) and open Firefox → http://<vmnet1-ip>:3000

1. Start the Kali VM (network set to Host-Only / VMnet1).
2. In Kali, confirm networking:

```bash
# show network interfaces
ip addr show

# or test connectivity to host
ping -c 3 <VMnet1-host-ip>
```

3. Open Firefox in Kali and visit:

```
http://<VMnet1-host-ip>:3000
```

Example:
```
http://192.168.56.1:3000
```

If the Juice Shop page loads in Kali’s Firefox, the Kali VM can reach the host Juice Shop.

---

## 5) Verify from Kali — quick testing commands

From the Kali terminal:

```bash
# simple HTTP check (response headers)
curl -I http://<VMnet1-IP>:3000

# full HTML output (first 20 lines)
curl -s http://<VMnet1-IP>:3000 | head -n 20

# nmap port check
nmap -Pn -p 3000 <VMnet1-IP>
```

Basic lab tooling (for practice only): Burp Suite, nmap, gobuster, sqlmap, wfuzz, etc.

---

## Snapshots, safety & troubleshooting

Snapshots
- Take a Windows snapshot before major changes (Docker install, network config).
- Take a Kali snapshot after configuring it as your attacker baseline.
- Revert snapshots between exercises to reset state.

Troubleshooting checklist
- Host loads http://localhost:3000 but Kali cannot:
  - Ensure Docker used: `-p 0.0.0.0:3000:3000` (not `127.0.0.1`). Re-run if needed.
  - Confirm the VM network is Host-Only (VMnet1) in VM settings.
  - Confirm the host VMnet1 adapter IP from `ipconfig` and use that IP in Kali.
  - Confirm Windows Firewall allows inbound TCP on port 3000 (see firewall rule above).
  - If `ipconfig` shows `VMnet8` or `VMnet1` mismatch: double-check the VM adapter type in VMware (Host-Only = VMnet1).
  - If `curl` from Kali returns HTML but Firefox times out: check Kali’s browser proxy settings (is Burp running?).

---

## Safety & ethics
This lab is for education on machines you own or are authorized to test. Do not attack systems you don’t own or have explicit permission to test. Follow local laws and institutional policies.
