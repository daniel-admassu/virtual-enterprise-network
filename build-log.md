# Build Log

A running record of what I worked on, what went wrong, and how I fixed it. I add a new entry at the top each week. Hours are actual hours spent.

---

## Week 3 

### Done
- Finished and submitted the Design Specification Draft.
- Instructor meeting (Thu Oct 15): prepared questions, met, and followed up.
- Researched lab platforms and found that my laptop can't run the Cisco images (details below).
- Requested a loaner laptop from the CSS department. *(included above)*
- Set up the GitHub repository, project board, and this build log.

### Problem: My laptop can't run Cisco IOSv
- **What happened:** My laptop (Dell XPS 13, Snapdragon X Elite) has an **ARM** processor. Cisco IOSv and IOSvL2 only run on **x86** (Intel/AMD) processors. Emulating x86 on ARM is too slow, and the routers often don't finish booting.
- **Also:** The lab needs about 6–8 GB of RAM, and IOSv images have to be obtained legally (Cisco Modeling Labs or the department).
- **Options I compared:** borrow an x86 computer, rent a cloud VM, or switch to FRRouting with Containerlab, which runs on ARM.
- **Decision:** The department offered a loaner **Dell Latitude 5450** (Intel Core Ultra 5, 16 GB RAM). I emailed IT (Rene) to request it and asked whether the department has Cisco images. Backup plan: FRRouting with Containerlab on my own laptop.

### Blocked / waiting on
- Loaner laptop pickup (not this week).
- Answer about Cisco images or CML licenses.

### Next week
- Pick up the laptop, install GNS3, and test two routers.
- Get legal Cisco images.
- Update and submit the Final Design Specification.
- Start Module 1: VLANs on the HQ switch.

---

## Week 2

### Done
- Standup meeting 0: proposal, plus feedback to 2 classmates.
- Started the design specification.
- Researched how company networks are designed (VLANs, inter-VLAN routing, OSPF, DHCP relay, DNS).
- Sketched the high-level design: network diagram, VLAN plan, and IP plan.
- Found possible experts and stakeholders (5 experts, 7 stakeholder groups).
- Posted the weekly status report.

---

## Week 4 — Sun Oct 18 – Sat Oct 24, 2026

### Goal
Finish lab setup on the loaner laptop and start Module 1 (HQ VLANs).

### What I did
- Picked up the CSS loaner laptop (Dell Latitude 5450) from CSS IT.
- Met with my expert, Glenn (about 1 hour, in person). He suggested **EVE-NG instead of GNS3**, offered to provide the Cisco images (IOSv, IOSvL2, ASAv), and suggested security additions: ACLs, a branch firewall, routing branch internet through HQ, and site-to-site + remote-access VPN.
- Requested the images from Glenn and asked where they come from so I can document the source.
- Built **Module 1 in Cisco Packet Tracer** on my own laptop as practice, using my real VLAN and IP plan:
  - HQ-R1 (2911) + HQ-SW1 (2960), router-on-a-stick over an 802.1Q trunk, native VLAN 999
  - End devices in VLAN 10 (USERS), 20 (SERVERS), 40 (GUEST), 99 (MGMT)
  - Subinterfaces Gi0/1.10/.20/.30/.40/.99 with `.1` gateways; Loopback0 10.255.0.1
- Watched and took notes on three Jeremy's IT Lab CCNA videos (trunking, router-on-a-stick, Layer 3 switches/SVIs).

### Problems and fixes
| Problem | Fix / decision |
|---|---|
| Software downloads on the loaner only work on the UW campus network, and I couldn't get to campus. | Pivoted to Packet Tracer practice on my own laptop so I could still work on Module 1. |
| Not sure where to get Cisco images (university doesn't provide licenses). | Glenn offered to provide them from his Cisco Modeling Labs access. FRRouting stays the backup. |

### Verification
- `show vlan brief`: each access port in the correct VLAN; Gi0/1 left VLAN 1 once it became a trunk.
- `show ip interface brief`: all HQ-R1 subinterfaces up/up.
- Pings from PC-USER (VLAN 10) to the gateway, server (VLAN 20), guest PC (VLAN 40), and switch (VLAN 99) all succeeded. The first request to each new address timed out while ARP resolved.
- TTL values confirmed routing: 255 from the router itself, 127 from hosts in other VLANs (one routed hop).

### What I learned
- Creating a VLAN isn't enough: access ports, trunks, gateways, and addressing must all be configured, then verified.
- The native VLAN must match on both ends of a trunk.
- Router-on-a-stick suits a small office; a Layer 3 switch scales better (possible design trade-off for my docs).

### Files
- `labs/packet-tracer/module1-hq.pkt`
- `docs/screenshots/week4/` (topology, show vlan brief, show ip int brief, pings)

---

## Week 5 — Sun Oct 25 – Sat Oct 31, 2026

### Goal
Get the real lab platform running and pass a first connectivity test.

### What I did
- Received IOSv, IOSvL2, and ASAv images from Glenn.
- Moved the lab from the loaner to my MSI GS66 Stealth (Intel Core i9 10th gen, 32 GB RAM).
- Turned off Windows virtualization-based security (VBS) so VMware could pass VT-x through to EVE-NG. Set up a separate **"Lab mode"** boot entry that turns off the Windows hypervisor only when I'm working on the lab.
- Installed **VMware Workstation Pro** and **EVE-NG Community** from the ISO.
- Uploaded the three Cisco images and fixed permissions.
- Built **Checkpoint 0**: two IOSv routers pinging each other.
- Created stakeholder survey #1 (Google Forms) and drafted the email to Rene.

### Problems and fixes
| Problem | Fix / decision |
|---|---|
| Loaner laptop: VBS blocked Intel VT-x, so EVE-NG couldn't run nested VMs. | Switched to my MSI laptop. Disabled VBS and Memory Integrity, and added a "Lab mode" boot entry (`bcdedit` with `hypervisorlaunchtype off`). Confirmed in `msinfo32`: VBS = Not enabled. |
| EVE setup wizard never asked for an IP address. | The installer had already configured DHCP, so the wizard was skipped. EVE got 192.168.215.128 on the VMware NAT network. (To rerun the wizard: `rm -f /opt/ovf/.configured` then `reboot`.) |
| `ping www.google.com` from EVE: 100% loss, but DNS resolved. | The network blocks ping but allows web traffic. `apt update` worked, which is the real internet test. |
| Battery was dying during `apt upgrade`, and a config prompt appeared for `/etc/issue`. | Answered **N** (keep EVE's version, which shows the web UI address). Plan: run updates only when plugged in. Recovery if interrupted: `dpkg --configure -a`, `apt -f install`, `apt upgrade -y`. |
| `-bash: .../un1_wrapper: No such file or directory` | Typo: it's `unl_wrapper` (lowercase L for "UNetLab"), not the number 1. |
| Clicking a router opened a Windows popup ("Internet Shortcut Extension DLL") instead of a console. | EVE was in Native console mode and Windows has no telnet client. Logged out and chose **HTML5 console** on the login page. |

### Verification
- `egrep -c '(vmx|svm)' /proc/cpuinfo` → **16** (nested VT-x working inside EVE).
- `ls -lh /opt/unetlab/addons/qemu/*/` → each image folder has one `virtioa.qcow2` (ASAv 383 MB, IOSv 55 MB, IOSvL2 87 MB).
- IOSv routers show in blue in the node list and boot successfully.
- **Checkpoint 0:** R1 Gi0/0 10.0.0.1/30 ↔ R2 Gi0/0 10.0.0.2/30. `show ip interface brief` up/up; `ping 10.0.0.2` successful.

### What I learned
- Checking hardware and hypervisor compatibility first saves days. This blocker started in Week 2.
- Windows security features (VBS, Memory Integrity, Hyper-V) can silently block nested virtualization.
- A /30 subnet gives exactly two usable addresses, the standard choice for router-to-router links.
- Snapshot the VM before risky steps (updates, uploads) so mistakes can be undone.

### Still pending
- Finish EVE updates on a charger (`apt update && apt upgrade -y`, then `reboot`).
- Send survey #1 to Rene and North Seattle College IT.
- Rebuild Module 1 in EVE-NG with IOSvL2.

### Files
- `configs/checkpoint-0/R1.txt`, `configs/checkpoint-0/R2.txt`
- `docs/screenshots/week5/` (msinfo32 VBS, VMware settings, EVE login, image `ls`, topology, ping)


<!-- Template for new weeks: copy this block to the top

## Week N 

### Done
-

### Problems and fixes
- **What happened:**
- **What I tried:**
- **Fix / decision:**

### Blocked / waiting on
-

### Next week
-
-->
