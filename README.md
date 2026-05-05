# PiSOC : Raspberry Pi Zero 2 W Home SOC Lab

Hey , I wanna to make a home Security Operations Center (SOC) lab built on a Raspberry Pi Zero 2 W I wanted to combines three security tools into a single always-on device that monitors the network, blocks malicious traffic, captures real attack data.

Built as a portfolio project during my final year of Applied IT at University of HOGENT Belgium with a focus on cybersecurity and netwerk and cloud.

## What This Project Does

### Pi-hole:

Pi-hole will act as a DNS server for the entire home network. Every IOT device sends its DNS queries through Pi-hole first. Pi-hole checks each domain against blocklists and drops requests to known ad servers, trackers, and malicious domains before the device even makes a connection. It also serves as the DHCP server so every device automatically uses it without manual configuration :)

### Netdata :

Netdata will provides a live dashboard showing CPU usage, memory, disk I/O, network traffic, and running processes on my Pi. It updates every second and gives visibility into the health of the SOC itself. And is also accessible from any browser on the network

### Cowrie:

Cowrie is there to simulates a fake SSH server on port 2222. Any attacker (or a scanner) that connects to it gets dropped into a fake Linux shell Every login attempt, command typed, and file downloaded is logged in structured JSON format. This produces real attack data that can be analyzed to understand attacker behavior and techniques.

## Architecture

```
Home Network (192.168.1.x)
        |
   [Router]
   192.168.1.x
        |
   [Raspberry Pi Zero 2 W]
   192.168.1.x (static)
   ├── Pi-hole     → port 53 (DNS) + port 80 (web UI)
   ├── Netdata     → port 19999
   └── Cowrie      → port 2222 (honeypot SSH)
```

All devices on the network use the Pi as their DNS server via Pi-hole's DHCP server.

## Hardware

| Component | Details                                 |
| --------- | --------------------------------------- |
| Board     | Raspberry Pi Zero 2 W                   |
| OS        | Raspberry Pi OS Lite (64-bit, Bookworm) |
| Storage   | MicroSD card (16GB+)                    |
| Network   | Built-in 2.4GHz WiFi (802.11n)          |

---

## Services Summary

| Service            | Port  | Auto-start |
| ------------------ | ----- | ---------- |
| SSH (admin access) | 22    | Yes        |
| Pi-hole DNS        | 53    | Yes        |
| Pi-hole Web UI     | 80    | Yes        |
| Netdata Dashboard  | 19999 | Yes        |
| Cowrie Honeypot    | 2222  | Yes        |

---

## Documentation

- [01 — Raspberry Pi Setup](docs/01-raspberry-pi-setup.md)
- [02 — Pi-hole Installation](docs/02-pihole.md)
- [03 — Netdata Installation](docs/03-netdata.md)
- [04 — Cowrie Honeypot Installation](docs/04-cowrie.md)
- [05 — Testing & Validation](docs/05-testing.md)

## Author

Christopher Ekoondo Student in Applied Computer Science
at
HOGENT University of Applied Sciences and Arts
