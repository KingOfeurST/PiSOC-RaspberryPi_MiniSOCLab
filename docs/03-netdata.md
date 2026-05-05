# 03  Netdata Installation

## What is Netdata?

Netdata is a real-time performance monitoring tool It provides a live web dashboard showing CPU, memory, disk, network, and running processes updated every second. For a SOC lab, it gives visibility into the health of the monitoring device itself and helps detect unusual activity like a spike in CPU (like possible crypto mining) or network traffic (like possible exfiltration).


## Installation

```bash
wget -O /tmp/netdata.sh https://get.netdata.cloud/kickstart.sh && sudo bash /tmp/netdata.sh
```

Accept all defaults during the install. The process takes time on the Pi Zero 2 W.


## Access

Once installed, the dashboard is available at:

(The RasPI IP address)
```
http://192.168.x.x:19999
```

No login required on the local network.



## What the dashboard shows

| Panel | What it monitors |
|---|---|
| CPU | Overall usage and per-core breakdown |
| Memory | RAM and swap usage |
| Disk I/O | Read/write activity on the SD card |
| Network | Traffic in/out on wlan0 |
| Processes | Running services and their resource usage |
| System load | 1/5/15 minute averages |

---

## Verification

```bash
sudo systemctl status netdata
```

Expected: `active (running)`

Access `http://192.168.x.x:19999` from any browser on the network. Live charts should be visible immediately :)