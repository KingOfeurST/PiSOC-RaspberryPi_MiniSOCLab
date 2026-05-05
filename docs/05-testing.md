# 05  Testing & Validation

This records the validation tests run after the full project was deployed.


## 1. Pi-hole DNS Blocking

**Test:** Query a known ad domain directly against Pi-hole.

```powershell
nslookup doubleclick.net 192.168.1.13
```

**Expected result:** Pi-hole returns `0.0.0.0` (blocked) instead of a real IP.

**Verify in dashboard:** Go to `http://192.168.x.x/admin` → Query Log. The blocked query should appear with status `Blocked`.


## 2. Pi-hole DHCP

**Test:** Disconnect and reconnect a phone to the WiFi.

**Expected result:** The phone receives an IP in the range `192.168.1.2–192.168.1.63` and its DNS is set to `192.168.1.13`.

**Verify:** On the phone, check the network details DNS server should show `192.168.1.13`.

Also visible in Pi-hole dashboard under Settings → DHCP → Active leases.


## 3. Netdata Monitoring

**Test:** Access the Netdata dashboard.

```
http://192.168.x.x:19999
```

**Expected result:** Live charts visible for CPU, memory, disk, and network. Charts update every second.

**Service check on Pi:**

```bash
sudo systemctl status netdata
```

Expected: `active (running)`

---

## 4. Cowrie Honeypot

**Test:** Connect to the honeypot from a laptop.

```powershell
ssh -p 2222 root@192.168.1.13
```

Use any password (like `123`) an you should land in a fake shell with hostname `svr04`.

Run a few commands inside the fake shell:
```bash
whoami
cat /etc/passwd
ls /home
```

Type `exit` to disconnect.

**Verify logs on Pi:**

```bash
sudo tail -20 /home/cowrie/cowrie/var/log/cowrie/cowrie.json
```

Expected entries:
- `cowrie.session.connect`: connection logged with source IP
- `cowrie.login.success` : username and password captured
- `cowrie.command.input` : every command recorded

---

## 5. Reboot Persistence

**Test:** Reboot the Pi and verify all services come back up automatically.

```bash
sudo reboot
```

After 5 minutes, reconnect and check:

```bash
sudo systemctl status cowrie
pihole status
sudo systemctl status netdata
```
All three services should be `active (running)` without any manual intervention.

## Summary

| Test | Result |
|---|---|
| Pi-hole blocking ad domains | Pass |
| Pi-hole DHCP serving all devices | Pass |
| Netdata dashboard accessible | Pass |
| Cowrie capturing SSH sessions | Pass |
| All services survive reboot | Pass |


