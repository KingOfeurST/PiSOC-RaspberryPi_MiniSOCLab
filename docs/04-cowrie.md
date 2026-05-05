# 04  Cowrie SSH Honeypot Installation

## What is Cowrie?

Cowrie is an SSH honeypot. It listens on a port and simulates a real Linux server. When an attacker (or Scanner try to) connects, they get dropped into a fake shell where they can run commands, try to download files, and explore while Cowrie silently logs everything in  JSON format.

This produces real attack data: IP addresses, usernames and passwords tried, commands executed, files requested. It shows what real attackers do once they get in.

Cowrie runs on port **2222** in this setup. The real SSH server stays on port 22.

---

## Installation

### 1. Install dependencies

```bash
sudo apt install -y git python3-virtualenv libssl-dev libffi-dev build-essential python3-dev
```

### 2. Create a dedicated user

Cowrie should not run as root or as the main user:

```bash
sudo adduser --disabled-password cowrie
sudo su - cowrie
```

### 3. Clone Cowrie and set up the virtual environment with

```bash
git clone https://github.com/cowrie/cowrie
cd cowrie
python3 -m virtualenv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
```

### 4. Fix dependency conflicts

Cowrie's latest version pins `twisted==25.5.0`, which has incompatibilities with newer versions of `incremental` and `constantly`. The fix is to use an older compatible Twisted version:

```bash
sed 's/twisted\[conch\]==25.5.0/twisted[conch]==24.3.0/' requirements.txt > requirements-fixed.txt
pip install -r requirements-fixed.txt
```

### 5. Install Cowrie as a package

This registers the Cowrie plugin with twistd:

```bash
pip install -e .
```

### 6. Copy the configuration file

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
```


## Running Cowrie

### Test run foreground

```bash
source cowrie-env/bin/activate
twistd -n cowrie
```

Expected output:
```
CowrieSSHFactory starting on 2222
Ready to accept SSH connections
```


## Running as a systemd Service

To make Cowrie start automatically on boot, create a systemd service as the `UserName` user:

```bash
sudo nano /etc/systemd/system/cowrie.service
```

Paste:

```ini
[Unit]
Description=Cowrie SSH Honeypot
After=network.target

[Service]
User=cowrie
WorkingDirectory=/home/cowrie/cowrie
ExecStart=/home/cowrie/cowrie/cowrie-env/bin/twistd -n cowrie
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
sudo systemctl status cowrie
```

---

## Log Files

All activity is logged to:

```
/home/cowrie/cowrie/var/log/cowrie/cowrie.json
```

Read the latest entries:

```bash
sudo tail -f /home/cowrie/cowrie/var/log/cowrie/cowrie.json
```

### Example log entries

**New connection:**
```json
{"eventid":"cowrie.session.connect","src_ip":"192.168.1.30","dst_port":2222,"protocol":"ssh"}
```

**Login attempt:**
```json
{"eventid":"cowrie.login.success","username":"root","password":"123","message":"login attempt [root/123] succeeded"}
```

**Command executed:**
```json
{"eventid":"cowrie.command.input","input":"cat /etc/passwd","message":"CMD: cat /etc/passwd"}
```

Every field is structured JSON making it easy to parse, search, and feed into a SIEM.

---

## Verification

From another machine on the network:

```powershell
ssh -p 2222 root@192.168.x.x
```

Any password will be accepted. You will land in a fake shell with hostname `svr04`. Type `exit` to disconnect, then check the logs on the Pi to confirm the session was captured :)