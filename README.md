# Building and Automating a Virtual Enterprise Network

**CSS 497 Capstone, UW Bothell** · Daniel Admassu · Fall 2026 – Winter 2027

A virtual network for a small business with a **main office (HQ)** and a **branch office**. You can build, test, and troubleshoot it safely. It covers VLANs, routing, DHCP, and DNS, uses Python scripts to configure and check the network devices, and includes a dashboard that shows network health.

> **Status:** Week 3. Design is drafted; lab platform is being set up. See [`build-log.md`](build-log.md) for progress.

## What's in the lab

| Part | What it does | Tool |
|---|---|---|
| HQ router, branch router, ISP router | Connect the two offices and share routes using OSPF | Cisco IOSv in GNS3 |
| HQ switch, branch switch | Separate devices into VLANs | Cisco IOSvL2 |
| HQ server | Hands out IP addresses (DHCP) and resolves names (DNS) for both offices | Ubuntu Server |
| Automation PC | Runs the Python scripts and the monitoring dashboard | Ubuntu, Python, Netmiko, Prometheus, Grafana |

## Deliverables

- [ ] Virtual network with a main office and a branch office
- [ ] VLANs at each office and routing between the offices
- [ ] DHCP and DNS server
- [ ] Python scripts that push device configurations and back them up
- [ ] Python scripts that check whether the network is working
- [ ] Monitoring dashboard showing which devices and links are up or down
- [ ] Network diagram and setup guide (this README plus `docs/`)
- [ ] Short demo video

## Repository layout

```
configs/      Device configurations (hq/, branch/, isp/)
scripts/      Python automation: push configs, back up, check the network
backups/      Config backups saved by the backup script
monitoring/   Prometheus and Grafana setup
diagrams/     Network diagram (draw.io) and exported images
docs/         Module guides, design notes, and decisions
build-log.md  Week-by-week log of what was done, problems, and fixes
```

## Build plan (modules)

1. **Module 1: HQ network.** VLANs, trunk, router-on-a-stick
2. **Module 2: Branch and WAN.** Branch network, ISP router, OSPF between offices
3. **Module 3: Services.** DHCP and DNS server, DHCP relay
4. **Automation.** Netmiko scripts for show commands, backups, and checks
5. **Monitoring.** Prometheus and Grafana dashboard (Winter quarter)

## Setup

Step-by-step setup instructions will be added here as each module is built.

## Security note

Device passwords and other secrets are **never** committed. They go in a local `.env` file, which `.gitignore` excludes.
