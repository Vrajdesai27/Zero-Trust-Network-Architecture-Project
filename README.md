# Zero Trust Network Architecture

> Prevention, detection, and automated response

A Zero Trust network that removes implicit trust based on location. Every cross-segment flow must pass through a firewall policy enforcement point, devices are admitted only after certificate-based authentication, a behavior-based IPS automatically quarantines hosts that probe restricted zones, and Splunk centralizes telemetry for detection and alerting. Design and testing are mapped to **NIST SP 800-207**.

---

## Architecture

```
          INTERNET
             │
         ┌────────┐
         │FW-EDGE │   perimeter + screened DMZ
         └───┬────┘
             │
        DMZ (VLAN 50) ── Ubuntu-5 web server
             │
         ┌────────┐
         │FW-CORE │   internal segmentation
         └───┬────┘
             │ deny syslog → IPS
   ┌─────────┼──────────┬──────────────┐
   │         │          │              │
 USER      ADMIN    DC-APP/DB/       QUARANTINE
(VLAN 20) (VLAN 30)  STORAGE          (VLAN 99)
  PC-LAN   JUMP-VM  (VLAN 40-42)    probed hosts
            + IPS    APP/DB/STOR     land here
```

Traffic between zones is forced through **FW-CORE** (Cisco ASAv) with explicit service-based permits followed by logged default-deny; **FW-EDGE** protects the Internet boundary and a screened DMZ. IOSvL2 switches (`SW-CORE`, `SW-ACCESS`, `SW-DMZ`) carry the tagged VLAN segments.

## How It Works

| Pillar | Implementation |
|--------|----------------|
| **Segmentation & least privilege** | 8 VLAN zones (USER, ADMIN, DC-APP/DB/STORAGE, DMZ, QUARANTINE, EDGE-TRANSIT); FW-CORE enforces service-specific permits + default-deny between them |
| **Device verification (NAC)** | 802.1X + FreeRADIUS (EAP-TLS). A valid certificate admits the device to its corporate VLAN; a missing/invalid certificate is placed in the QUARANTINE VLAN |
| **Automated response (IPS)** | Python service listens for FW-CORE ASA deny syslog (msg 106100) on UDP 514. **≥5 denied probes to restricted zones within 15 s** → resolves IP → MAC → switch port and moves that port into VLAN 99 (quarantine). Behavior-driven, not signature-based |
| **Detection & monitoring (SIEM)** | Splunk Universal Forwarder on the DMZ server ships Nginx access/error, auth, and system logs into a dedicated `ztna` index; scheduled searches flag repeated DMZ enumeration (404 sweeps) and repeated auth failures |

## VLAN Segmentation

| VLAN | Zone | Purpose |
|------|------|---------|
| 20 | USER | End-user workstations |
| 30 | ADMIN | Management / jump host / IPS |
| 40 | DC-APP | Application servers |
| 41 | DC-DB | Database servers |
| 42 | DC-STORAGE | Storage servers |
| 50 | DMZ | Public-facing web server |
| 99 | QUARANTINE | Isolated / contained hosts |
| 200 | EDGE-TRANSIT | FW-EDGE ↔ FW-CORE link |

## Tested & Verified

- ✅ **Permitted vs. denied traffic** — users reached approved web services but could **not** directly reach the database, storage, or admin tiers; the DMZ server could not initiate connections inward
- ✅ **Automated quarantine** — repeated denied probes from both a dedicated attacker and a normal host triggered dynamic port-to-VLAN-99 containment
- ✅ **DMZ enumeration detection** — a 15-path `/admin_N.php` sweep was correlated by source IP in Splunk
- ✅ **Auth-failure detection** — repeated authentication failures grouped and counted
- ✅ **Persistence** — a reboot test (`ZTNA_REBOOT_TEST`) confirmed the forwarder and end-to-end logging recover automatically

## Tech Stack (tools used)

`Cisco Modeling Labs (CML)` · `Cisco ASAv` · `Cisco IOSvL2` · `FreeRADIUS` · `Python` · `Splunk Enterprise` · `Nginx / UFW (Ubuntu)` · `Proxmox`

## Framework

Designed and evaluated against **NIST SP 800-207 (Zero Trust Architecture)**.

---

**Author:** Vraj Desai · [LinkedIn](https://linkedin.com/in/vraj-desai-03257323b)
