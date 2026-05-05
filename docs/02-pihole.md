# 02 Pi-hole Installation

## What is Pi-hole?

Pi-hole is a network-wide DNS-based ad and tracker blocker. Instead of installing an ad blocker in each browser on each device, Pi-hole sits between the network and the internet and blocks ads at the DNS level for every device at once phones, laptops, smart TVs, game consoles.

When a device wants to load `ads.doubleclick.net` it asks Pi-hole for the IP address. Pi-hole checks its blocklists sees it's a known ad domain and returns nothing so the ad never loads For legitimate domains Pi-hole forwards the query to an upstream DNS provider (Cloudflare in this setup).

Pi-hole also acts as the **DHCP server**, so every device that connects to the network automatically uses Pi-hole as its DNS server.

## Pre-installation

Before installing, temporarily set the DNS to Cloudflare. The Pi's DNS is currently pointing to itself but Pi-hole isn't installed yet so DNS resolution would fail during the install:

```bash
sudo nmcli con mod "preconfigured" ipv4.dns "1.1.1.1"
sudo nmcli con up "preconfigured"
```

## Installation

```bash
curl -sSL https://install.pi-hole.net | sudo bash
```

### Installer options

| Prompt                | Selection              |
| --------------------- | ---------------------- |
| Network interface     | `wlan0`                |
| Upstream DNS provider | `Cloudflare (1.1.1.1)` |
| Block lists           | Keep defaults          |
| Web admin interface   | On                     |
| lighttpd web server   | On                     |
| Log queries           | On                     |
| Privacy mode          | 0 — Show everything    |

At the end of the install, the admin password is displayed. Save it.

## After Installation

### Point DNS back to Pi-hole itself

```bash
sudo nmcli con mod "preconfigured" ipv4.dns "127.0.x.x"
sudo nmcli con up "preconfigured"
```

### Enable Pi-hole as DHCP server

The router does not allow changing the DNS server it distributes to clients. To make all devices use Pi-hole automatically, I enabled Pi-hole's built-in DHCP server and just disabled the router's DHCP.

**Step 1 — Enable DHCP in Pi-hole:**

1. Go to `http://192.168.x.x/admin`
2. Settings → DHCP
3. Enable DHCP server: On
4. Range: `192.168.x.x` → `192.168.x.x`
5. Gateway: `192.168.c.x`
6. Save

**Step 2 — Disable DHCP on the router:**

1. Log into `192.168.x.x` your default gateway
2. Modem → DHCP → Enable: **Off**
3. Save

From this point, every device that connects to the WiFi gets its IP from Pi-hole and automatically uses it as its DNS server.

## Web Interface

Accessible at: `http://192.168.x.x/admin`

The dashboard shows:

- Total DNS queries
- Queries blocked (with the percentage)
- Domains on blocklists
- Live query log per device

## Verification

From the laptop verify Pi-hole is responding to DNS queries:

```powershell
nslookup google.com 192.168.x.x
```

A response confirms Pi-hole is resolving DNS correctly.

Check Pi-hole service status on the Pi with :

```bash
pihole status
```
