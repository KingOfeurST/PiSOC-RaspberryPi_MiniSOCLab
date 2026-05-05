# 01 Raspberry Pi Setup


## OS selection

I chose **Raspberry Pi OS Lite (64-bit, Bookworm)** the headless version with no desktop because this will keep my resource usage low on the Pi Zero 2 W's limited hardware.

> I initially tried Ubuntu Server for Raspberry Pi which uses cloud-init for first-boot configuration (network-config, user-data files). The WiFi configuration kept failing silently due to an invalid regulatory-domain field and a pre-hashed password that didn't match. I switched to Raspberry Pi OS for better hardware support


## Flashing the SDcard

1. Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Open Imager → Choose Device: `Raspberry Pi Zero 2 W` (your raspberry)
3. → Choose OS: `Raspberry Pi OS (other)` → `Raspberry Pi OS Lite (64-bit)`
4. → Choose Storage: your SD card
5. → Click Next → when asked about customisation → click Edit Settings

### Customisation settings

| Field | Value |
|---|---|
| Hostname | `PISOC` |
| Username | `Name of your Pi` |
| Password | (your choice) |
| WiFi SSID | `Your network` |
| WiFi Password | (your WiFi password) |
| WiFi Country | `BE` |
| Enable SSH | Yes — password authentication |

1. Click **Save** → **Yes** → **Write**

to be sure look if the `firstrun.sh` exists on the boot partition with 

```powershell
Get-ChildItem D:\ | Select-Object Name | findstr "firstrun"
```

If `firstrun.sh` is present the settings were applied correctly



## WiFi troubleshooting

Getting the Pi to connect to WiFi was the most difficult part of this project. Here is what I ran into and how I fixed it.

### Problem 1 : Ubuntu cloud-init was ignoring the WIFI config
The `network-config` file had an invalid `regulatory-domain` field that caused the entire config to fail silently. Switching to Raspberry Pi OS solved this

### Problem 2 : Imager customisation not applied
After flashing, `firstrun.sh` was missing from the SD card. This meant the Imager wrote the image but skipped the customisation step no idea why . Solution: re-flash and click **Edit Settings** before writing.

### Problem 3 : WiFi connected but Pi not reachable
After fixing the flash, the Pi booted correctly (steady green LED) but never appeared on the router's device list. My router does not expose DNS or detailed connection logs, so I couldn't see what was happening on the router side.

**Solution: USB OTG networking:**  
The Pi Zero 2 W supports USB gadget networking also which lets it act as a network adapter over a USB cable connected directly to a laptop. I enabled this by adding two lines to the SD card before booting:

```powershell
# On the SD card (D:)
Add-Content D:\config.txt "`ndtoverlay=dwc2"
$cmd = (Get-Content D:\cmdline.txt -Raw).Trim()
[System.IO.File]::WriteAllText("D:\cmdline.txt", $cmd + " modules-load=dwc2,g_ether")
```

Then connected the USB cable to the Pi's **middle port** and SSHed into it:

```powershell
ssh x@pisoc.local
```

When inside, I connected to WiFi manually with ;

```bash
sudo nmcli dev wifi connect "NETWORKNAME" password "yourpassword"
```



## Setting a static IP

By default the Pi gets a dynamic IP from DHCP. For a SOC device, a stable IP is essential. I set a static IP directly via NetworkManager:

```bash
sudo nmcli con mod "preconfigured" ipv4.addresses 192.168.x.x/24 \
  ipv4.gateway 192.168.1.x \
  ipv4.dns "127.0.x.x" \