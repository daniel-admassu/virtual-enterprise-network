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
