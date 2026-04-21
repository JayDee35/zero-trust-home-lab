# Zero Trust Home Lab

**Status:** 🔨 Active Build — Started April 2026  
**Goal:** Hands-on implementation of Zero Trust network architecture 
using commodity hardware, documented for learning and portfolio purposes.

---

## What This Is

A home network security lab built on a Raspberry Pi running 
Raspberry Pi OS Lite. This project documents my hands-on journey 
implementing Zero Trust principles from scratch — every config 
decision, mistake, and fix included.

This is not a tutorial. It's a real build log.

---

## Why Zero Trust

Traditional network security assumes everything inside the network 
perimeter is safe. Zero Trust assumes nothing is — every device, 
user, and connection must be verified before access is granted.

This is the model IAM operates on. Understanding it at the network 
level makes me a better IAM practitioner.

---

## Hardware

| Component | Details |
|-----------|---------|
| Board | Raspberry Pi (Pi OS Lite) |
| Storage | MicroSD |
| Network | Home LAN + planned VPN overlay |

---

## Planned Architecture
[Internet]
│
[Home Router]
│
[Raspberry Pi — Zero Trust Gateway]
├── Tailscale (mesh VPN / Zero Trust Network Access)
├── Pi-hole (DNS filtering / ad blocking)
└── Access logging + monitoring
---

## Build Phases

- [x] Hardware acquired — Raspberry Pi with Pi OS Lite
- [ ] Phase 1: Tailscale installation and mesh VPN configuration
- [ ] Phase 2: Pi-hole DNS filtering setup
- [ ] Phase 3: VPN kill switch configuration
- [ ] Phase 4: Access logging implementation
- [ ] Phase 5: Python script — automated access review report

---

## Skills Being Applied

- Linux command line (Raspberry Pi OS Lite)
- Network architecture and segmentation
- VPN configuration (Tailscale / WireGuard protocol)
- DNS security (Pi-hole)
- Zero Trust principles in practice
- Python scripting for IAM automation
- Technical documentation

---

## Background

I'm transitioning into Identity and Access Management (IAM) 
from a technical support background (Shopify). This lab exists 
because I believe in learning by building, not just studying.

Certifications in progress: Google Cybersecurity Professional 
Certificate, Okta Identity Foundations, CompTIA Security+.

---

*Updated regularly as the build progresses.*
