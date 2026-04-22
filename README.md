# EdgeSecure Hybrid Lab

> “If you can’t fix it when it breaks, you don’t own the system.”

---

## Project Overview

EdgeSecure Hybrid Lab is a production-grade, hybrid infrastructure environment designed to simulate real-world enterprise systems under resource constraints.

This project combines:

* Local virtualization (VirtualBox)
* Heterogeneous Linux systems (Rocky Linux + Ubuntu)
* Cloud integration (AWS / OCI Free Tier)
* Infrastructure as Code (Ansible)
* Load balancing (HAProxy)
* Centralized logging (rsyslog)
* Active defense (Fail2Ban)
* Disaster Recovery (rsync + cron)

---

## Objectives

* Build system-level engineering intuition
* Design for failure containment and recovery
* Understand Linux internals beyond commands
* Simulate real-world production constraints
* Develop defensible engineering decisions

---

## Architecture

### Nodes

| Node      | OS            | Role                            |
| --------- | ------------- | ------------------------------- |
| edge01    | Rocky Linux   | Control Node, HAProxy, SIEM, DR |
| backend01 | Ubuntu Server | Local Web Server                |
| backend02 | Ubuntu Server | Cloud Web Server                |

---

### Network Design

* NAT (enp0s3) → Internet access (updates, package installs)
* Host-Only (enp0s8) → Internal communication (SSH, Ansible, HAProxy)

---

### Architecture Flow

```
            Internet
               ↑
        (NAT Interface)
            edge01
               ↓
     ---------------------
     |                   |
 backend01         backend02 (Cloud)
```

---

## Technologies Used

* Virtualization: Oracle VM VirtualBox
* Operating Systems: Rocky Linux, Ubuntu Server
* Storage: LVM (Logical Volume Management)
* Networking: NAT + Host-Only segmentation
* Automation: Ansible
* Load Balancer: HAProxy
* Security: SELinux, Firewalld, UFW
* SIEM: rsyslog
* Active Defense: Fail2Ban
* Backup: rsync + cron

---

## Phase Breakdown

### Phase 0 — Hardware Provisioning

* Created VMs with:

  * 1GB RAM each
  * 20GB disk
* Configured dual NICs:

  * NAT
  * Host-only

---

### Phase 1 — Storage Engineering (LVM)

#### Objective:

Isolate `/var` to prevent system-wide failure from log flooding.

#### Implementation:

* Added new disk (`/dev/sdb`)
* Created:

  * Physical Volume (PV)
  * Volume Group (VG)
  * Logical Volume (LV)
* Migrated `/var` using:

```bash
rsync -aAXv /var/ /mnt/newvar/
```

#### Critical Insight:

* `/var` contains logs, cache, runtime data
* Filling `/var` does not crash `/`

---

### Phase 2 — Networking & Access Control

* Static IP assignment
* SSH hardened with Ed25519 keys
* Firewalld (Rocky) default DROP
* UFW (Ubuntu) controlled access

---

### Phase 3 — Infrastructure as Code

* Ansible control node on `edge01`
* OS-aware playbooks using `ansible_os_family`
* Automated:

  * Apache deployment
  * Firewall rules

---

### Phase 4 — Load Balancing (HAProxy)

* Configured HAProxy on edge node
* Balanced traffic across:

  * Local backend
  * Cloud backend
* Enabled SELinux boolean:

```bash
setsebool -P haproxy_connect_any 1
```

---

### Phase 5 — SIEM + Active Defense + DR

#### Logging:

* rsyslog forwarding from backends to edge01

#### Defense:

* Fail2Ban monitors logs and bans brute-force attempts

#### Backup:

* Automated via cron:

```bash
rsync -avz user@backend:/etc/ /backup/
```

---

## Attack Simulations

### Log Flood Test

```bash
dd if=/dev/zero of=/var/log/fill.log bs=1M count=5000
```

Result:

* `/var` fills up
* `/` remains stable

---

### SSH Brute Force

```bash
for i in {1..10}; do ssh fake@server; done
```

Result:

* Fail2Ban bans attacker IP

---

## Screenshots

Include:

* lsblk, lvs, vgs
* LVM structure
* HAProxy config
* Ansible execution
* Fail2Ban status
* rsyslog logs
* SSH login failure
* Recovery via GRUB

---

## Key Troubleshooting Experience

### System Failure Scenario

After migrating `/var`:

* Login failed
* Services failed
* System partially booted

---

### Root Cause

* Incomplete data migration
* Missing SELinux contexts

---

### Recovery Process

* Booted into GRUB (`rd.break`)
* Mounted root filesystem manually
* Re-executed migration correctly
* Verified with:

```bash
mount -a
```

---

### Lesson

In production, systems are recovered, not reset.

---

## Lessons Learned

* LVM is not just storage—it is resilience engineering
* `/var` isolation prevents cascading failures
* SELinux context is critical to system integrity
* Testing before reboot is mandatory
* Failure is part of system design

---

## Future Improvements

* Full cloud automation (Terraform)
* CI/CD pipeline integration
* Monitoring stack (Prometheus + Grafana)
* IDS/IPS integration

---

## Interview Talking Points

* Designed hybrid infrastructure under hardware constraints
* Implemented LVM for failure isolation
* Built secure networking with zero-trust firewalling
* Automated deployments using Ansible
* Engineered recovery from real system failure

---

## Author

Edinen Udofia
Linux System Administration | Cybersecurity | Infrastructure Engineering

---

## Final Note

This project is not just about building systems.

It is about understanding them deeply enough to fix them when they break.
