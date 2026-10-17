# Zero Trust Network Architecture

> A network design and implementation project — identity-based access, segmentation, and automated threat detection

A Zero Trust network design that trusts no device or user by default. Access is granted only after authentication, the network is segmented so a breach can't spread, and all traffic is continuously monitored for anomalies with automated response.

---

## Overview

Traditional networks trust everything once it's inside the perimeter. This project implements a **Zero Trust** model built on three principles:

1. **Verify explicitly** — every device authenticates before it reaches the network (802.1X / NAC).
2. **Least-privilege access** — a device lands only in the VLAN-segmented zone it's authorized for.
3. **Assume breach** — all traffic is logged and analyzed for anomalies, with automated response.

## Architecture

```
                  ┌──────────────────────────┐
   Endpoint ────▶ │  802.1X / FreeRADIUS NAC  │
                  └────────────┬─────────────┘
                               │  authorized → assigned VLAN
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                     ▼
     ┌─────────┐          ┌─────────┐          ┌─────────┐
     │  Zone A │          │  Zone B │          │  Zone C │
     └────┬────┘          └────┬────┘          └────┬────┘
          └──────────── mirrored traffic ───────────┘
                               │
                   ┌───────────▼───────────┐
                   │  Python Behavioral IPS │
                   └───────────┬───────────┘
                               │
                   ┌───────────▼───────────┐
                   │       Splunk SIEM      │
                   └────────────────────────┘
```

*VLAN-segmented zones · behavioral IPS baselines traffic and flags anomalies · Splunk centralizes logs and alerts.*

## Key Components

| Component | Purpose |
|-----------|---------|
| **802.1X / FreeRADIUS NAC** | Authenticates every device before access; admits it only to its authorized VLAN zone |
| **VLAN Segmentation** | Isolates the network into zones so lateral movement is contained |
| **Python Behavioral IPS** | Baselines normal traffic and flags anomalies in real time for automated detection and response |
| **Splunk SIEM** | Centralizes logs with correlation rules and dashboards for continuous monitoring and alerting |

## Tech Stack (tools used)

`Cisco Modeling Labs (CML)` · `Cisco IOS` · `FreeRADIUS` · `Python` · `Splunk SIEM`

## Features Implemented

`Zero Trust model` · `802.1X / NAC` · `VLAN micro-segmentation` · `ACLs` · `behavioral IPS` · `least-privilege access`

## What I Learned

- Designing a network around identity instead of perimeter trust
- Automating threat detection with a behavioral (not signature-only) approach
- Correlating distributed logs into actionable alerts with a SIEM
- Enforcing least-privilege access at Layer 2 through segmentation and NAC

---

**Author:** Vraj Desai · [LinkedIn](https://linkedin.com/in/vraj-desai-03257323b)
