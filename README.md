# Building and Automating a Virtual Enterprise Network

CSS 497 Capstone, UW Bothell. Daniel Admassu.

A virtual company network with a **main office (HQ)** and a **branch office** that you can build, test, and troubleshoot safely. It covers VLANs, routing between sites, DHCP, and DNS, plus **Python scripts** (Netmiko) that configure, back up, and check the devices, and a **monitoring dashboard**. The lab is split into step-by-step modules, each ending with a checkpoint you can verify.

## Project status

| Module | Topic | Status |
|---|---|---|
| 0 | Lab platform: EVE-NG + Cisco images, two-router test | ✅ Done (Week 5) |
| 1 | HQ network: VLANs, trunk, router-on-a-stick | 🔄 Practiced in Packet Tracer (Week 4); EVE-NG build next |
| 2 | Branch + simulated ISP, OSPF between offices | 📋 Planned |
| 3 | Services: DHCP and DNS server, DHCP relay | 📋 Planned |
| 4 | Automation: Netmiko show commands, config backups, check script | 📋 Planned |
| 5 | Monitoring dashboard (Prometheus + Grafana) | 📋 Planned (Quarter 2) |
| 6 | Security: ACLs, branch firewall (ASAv), site-to-site and remote-access VPN | 📋 Planned (Quarter 2) |

## Repository layout

```
configs/      Device configurations (per module / checkpoint)
scripts/      Python automation scripts
backups/      Config backups produced by the backup script
monitoring/   Prometheus / Grafana setup
diagrams/     Network diagrams (draw.io)
docs/         Step-by-step module guides and screenshots
labs/         Lab files (Packet Tracer practice, EVE-NG exports)
BUILD_LOG.md  What I did each week, problems hit, and how I fixed them
```

## Lab setup (Module 0)

### Host requirements
- **Intel CPU with VT-x** (EVE-NG Community does not support AMD or ARM). My host: MSI GS66 Stealth, Intel Core i9 10th gen, 32 GB RAM.
- 32 GB RAM recommended (16 GB works for small topologies).
- About 150 GB free disk.
- **Windows virtualization-based security (VBS) must be off.** Check with `msinfo32` → "Virtualization-based security: Not enabled". Also turn off Memory Integrity (Windows Security → Device security → Core isolation) and the Hyper-V, Virtual Machine Platform, and Windows Sandbox features.
  - Optional: create a separate "Lab mode" boot entry so the Windows hypervisor is off only when you're labbing:
    ```
    bcdedit /copy {current} /d "Lab mode"
    bcdedit /set {<GUID from previous command>} hypervisorlaunchtype off
    ```

### 1. Install VMware Workstation Pro
Free for personal use (requires a Broadcom account).

### 2. Create the EVE-NG VM
Download the **EVE-NG Community ISO** from eve-ng.net.

| Setting | Value |
|---|---|
| Guest OS | Linux → Ubuntu 64-bit |
| Disk | 150 GB, stored as a single file |
| Memory | 16 GB |
| Processors | 8 or more vCPUs |
| **Virtualize Intel VT-x/EPT** | ✅ Checked (required) |
| Network adapter | NAT (keeps EVE's IP stable when your Wi-Fi changes) |
| CD/DVD | EVE ISO, connect at power on |

### 3. Install EVE-NG
1. Boot, choose **Install EVE VM**, accept defaults (DHCP is fine).
2. At "Finish the installation," keep the ISO connected, then continue.
3. Log in at the console with username `root`, password `eve`, and change the password.
4. Note the IP shown on the login banner (`IPv4 address for pnet0`).
5. Update **while plugged in**:
   ```bash
   apt update && apt upgrade -y
   reboot
   ```
   If asked about modified config files (for example `/etc/issue`), answer **N** to keep EVE's version.
6. Verify nested virtualization works (result must be greater than 0):
   ```bash
   egrep -c '(vmx|svm)' /proc/cpuinfo
   ```
7. Take a VMware snapshot.

### 4. Add the Cisco images
Images are licensed Cisco software and are **not** included in this repo. Obtain them through a valid source (for example, Cisco Modeling Labs).

Connect with WinSCP (SFTP, port 22, user `root`) and create one folder per image under `/opt/unetlab/addons/qemu/`. The prefix before the first dash must match exactly:

| Image | Folder (example) | File name |
|---|---|---|
| IOSv router | `vios-adventerprisek9-m.SPA.159-3.M6` | `virtioa.qcow2` |
| IOSvL2 switch | `viosl2-adventerprisek9-m.SSA.high_iron_20200929` | `virtioa.qcow2` |
| ASAv firewall | `asav-9.20` | `virtioa.qcow2` |

Then fix permissions (that's a lowercase **L** in `unl`, not the number 1):
```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
ls -lh /opt/unetlab/addons/qemu/*/
```

### 5. Open the web UI
Browse to `http://<EVE IP>`, log in with `admin` / `eve` (change it), and choose **HTML5 console** on the login page. Native console needs a telnet client installed on Windows.

### 6. Checkpoint 0: two-router test
1. New lab → add 2 × **Cisco vIOS Router** (should appear in blue) → connect R1 Gi0/0 ↔ R2 Gi0/0 → start both.
2. Configure:
   ```
   ! R1
   enable
   configure terminal
   hostname R1
   interface GigabitEthernet0/0
    ip address 10.0.0.1 255.255.255.252
    no shutdown
   end
   write memory
   ```
   ```
   ! R2
   enable
   configure terminal
   hostname R2
   interface GigabitEthernet0/0
    ip address 10.0.0.2 255.255.255.252
    no shutdown
   end
   write memory
   ```
3. Verify on R1:
   ```
   show ip interface brief    ! Gi0/0 up/up
   ping 10.0.0.2              ! should succeed (first packet may time out during ARP)
   ```

✅ If the ping succeeds, the platform is ready for Module 1.

### Troubleshooting
| Symptom | Fix |
|---|---|
| VMware: "Virtualized Intel VT-x/EPT is not supported" | VBS or Hyper-V is still on. Recheck `msinfo32`, Memory Integrity, and Windows features. |
| EVE setup wizard didn't ask for an IP | Installer used DHCP. To rerun: `rm -f /opt/ovf/.configured` then `reboot`. |
| `ping` to the internet fails but DNS resolves | Some networks block ping. Test with `apt update` instead. |
| Node is grey in the "Add node" list | Wrong folder prefix or file name. Recheck, then rerun fixpermissions. |
| Clicking a node opens a Windows popup, no console | Switch to **HTML5 console** at login, or install the EVE Windows Client Integration pack. |
| Node shows a ⚠ and won't start | Right-click → **Wipe**, then start again. |
| `apt upgrade` interrupted | `dpkg --configure -a`, then `apt -f install`, then `apt upgrade -y`. |

## Network design (summary)

| VLAN | Name | HQ subnet | Branch subnet |
|---|---|---|---|
| 10 | USERS | 10.1.10.0/24 | 10.2.10.0/24 |
| 20 | SERVERS | 10.1.20.0/24 | — |
| 30 | VOICE | 10.1.30.0/24 | 10.2.30.0/24 |
| 40 | GUEST | 10.1.40.0/24 | 10.2.40.0/24 |
| 99 | MGMT | 10.1.99.0/24 | 10.2.99.0/24 |
| 999 | NATIVE | — | — |

WAN links: HQ-R1 ↔ ISP-R1 10.255.1.0/30, ISP-R1 ↔ BR-R1 10.255.1.4/30. Full diagram and IP plan in `diagrams/`.

## Tools
EVE-NG Community · VMware Workstation Pro · Cisco IOSv / IOSvL2 / ASAv · Ubuntu Server · Python + Netmiko · Prometheus + Grafana · Wireshark · Git/GitHub · draw.io · Cisco Packet Tracer (practice)
