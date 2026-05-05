# PiSOC Cheatsheet

Use this when you do the PiSOC stuff. Replace `192.168.x.x` with your Pi IP.

## Main links

- Pi-hole admin: `http://192.168.x.x/admin` - where you block stuff and see DNS.
- Netdata: `http://192.168.x.x:19999` - live health screen for the Pi.
- Cowrie SSH: `ssh -p 2222 root@192.168.x.x` - fake SSH box for catching attacks.

## Setup WiFi and DNS

```bash
sudo nmcli dev wifi connect "NETWORKNAME" password "yourpassword"
sudo nmcli con mod "preconfigured" ipv4.dns "1.1.1.1"
sudo nmcli con up "preconfigured"
sudo nmcli con mod "preconfigured" ipv4.dns "127.0.x.x"
sudo nmcli con up "preconfigured"
```

- First line joins the Pi to WiFi.
- Middle lines use Cloudflare for Pi-hole install.
- Last lines switch DNS back to the Pi after Pi-hole is ready.

## If WiFi is broken

Use USB gadget mode to reach the Pi directly.

```powershell
Add-Content D:\config.txt "`ndtoverlay=dwc2"
$cmd = (Get-Content D:\cmdline.txt -Raw).Trim()
[System.IO.File]::WriteAllText("D:\cmdline.txt", $cmd + " modules-load=dwc2,g_ether")
ssh x@pisoc.local
```

- Put the cable in the middle USB port.
- This lets you log in even when WiFi is not working.

## Pi-hole

```bash
curl -sSL https://install.pi-hole.net | sudo bash
pihole status
```

- First command installs Pi-hole.
- Second command checks if it is running.

## Netdata

```bash
wget -O /tmp/netdata.sh https://get.netdata.cloud/kickstart.sh && sudo bash /tmp/netdata.sh
sudo systemctl status netdata
```

- First command installs Netdata.
- Second command checks if Netdata is alive.

## Cowrie

```bash
sudo apt install -y git python3-virtualenv libssl-dev libffi-dev build-essential python3-dev
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/cowrie/cowrie
cd cowrie
python3 -m virtualenv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
sed 's/twisted\[conch\]==25.5.0/twisted[conch]==24.3.0/' requirements.txt > requirements-fixed.txt
pip install -r requirements-fixed.txt
pip install -e .
cp etc/cowrie.cfg.dist etc/cowrie.cfg
twistd -n cowrie
sudo nano /etc/systemd/system/cowrie.service
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
sudo systemctl status cowrie
sudo tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.json
```

- These commands set up Cowrie, start it, and show the logs.
- Cowrie listens on port `2222`.

## Use Cowrie

### Connect to the fake SSH

```powershell
ssh -p 2222 root@192.168.x.x
```

- Type any password.
- You are not on the real Pi, you are in the fake honey pot.

### What to type inside

```bash
whoami
pwd
ls
cat /etc/passwd
exit
```

- These are simple things attackers try.
- `exit` leaves the fake shell.

### Find the connection in the logs

```bash
sudo tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.json
sudo grep 'cowrie.session.connect' /home/cowrie/cowrie/var/log/cowrie/cowrie.json
sudo grep 'cowrie.login.success' /home/cowrie/cowrie/var/log/cowrie/cowrie.json
sudo grep 'cowrie.command.input' /home/cowrie/cowrie/var/log/cowrie/cowrie.json
```

- `cowrie.session.connect` means someone connected.
- `cowrie.login.success` means the username and password were captured.
- `cowrie.command.input` means a command was typed.
- `tail -f` is live view, like watching it happen now.

## Test and check

```powershell
nslookup google.com 192.168.x.x
ssh -p 2222 root@192.168.x.x
sudo reboot
sudo systemctl status cowrie
pihole status
sudo systemctl status netdata
```

- `nslookup` checks Pi-hole DNS.
- `ssh -p 2222` checks Cowrie.
- `reboot` restarts the Pi.
- The status commands check if everything came back up.

## Docs

- [Raspberry Pi setup](../docs/01-raspberry-pi-setup.md)
- [Pi-hole setup](../docs/02-pihole.md)
- [Netdata setup](../docs/03-netdata.md)
- [Cowrie setup](../docs/04-cowrie.md)
- [Testing guide](../docs/05-testing.md)
