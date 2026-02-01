# Chapter 10: Networking

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand Linux networking fundamentals
- Configure network interfaces
- Use essential network diagnostic tools
- Transfer files securely over a network
- Connect to remote systems with SSH
- Configure and use firewalls
- Troubleshoot common network issues
- Work with DNS and hosts resolution

---

## 10.1 Networking Fundamentals

### Key Networking Concepts

**IP Address:** Unique identifier for each device on a network.
- **IPv4:** 192.168.1.100 (32-bit, four octets)
- **IPv6:** fe80::1234:5678:abcd:ef01 (128-bit, hexadecimal)

**Subnet Mask:** Defines which part of the IP is network vs host.
```
IP:      192.168.1.100
Mask:    255.255.255.0  (/24)
Network: 192.168.1.0    (first 24 bits)
Host:    .100           (last 8 bits)
```

**Common Subnet Masks:**
```
/8   255.0.0.0        16,777,214 hosts
/16  255.255.0.0      65,534 hosts
/24  255.255.255.0    254 hosts        ← Most common (home/small office)
/25  255.255.255.128  126 hosts
/30  255.255.255.252  2 hosts          ← Point-to-point links
```

**Gateway:** Router that connects your local network to the outside world.
```
Your PC: 192.168.1.100
Gateway: 192.168.1.1    ← Router
Internet: Everything else
```

**MAC Address:** Physical address of a network interface (e.g., 00:1A:2B:3C:4D:5E).

**Port Numbers:** Identify specific services on a host.
```
Port 22   → SSH
Port 80   → HTTP
Port 443  → HTTPS
Port 21   → FTP
Port 25   → SMTP (email)
Port 3306 → MySQL
Port 5432 → PostgreSQL
```

---

### Network Interface Types

```bash
# View all interfaces
$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    inet 127.0.0.1/8 scope host lo
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
3: wlan0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN
```

**Common Interfaces:**
- **lo** : Loopback (127.0.0.1) - for local testing
- **eth0** : First Ethernet adapter
- **enp0s3** : Predictable naming (PCI slot based)
- **wlan0** : First wireless adapter
- **docker0** : Docker virtual network
- **virbr0** : Virtual bridge (VMs)

---

## 10.2 Viewing and Configuring Networks

### ip Command - Modern Networking Tool

**View IP Addresses:**
```bash
# Full details
$ ip addr show
# or short form
$ ip addr
# or shortest
$ ip a

# Specific interface
$ ip addr show eth0

# Only IPv4
$ ip -4 addr

# Only IPv6
$ ip -6 addr
```

**View Routing Table:**
```bash
# Show routes
$ ip route
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100

# Short form
$ ip route show
# or
$ ip r
```

**View Network Interfaces:**
```bash
# Interface status
$ ip link show
# or
$ ip link
# or
$ ip l

# Bring interface up
$ sudo ip link set eth0 up

# Bring interface down
$ sudo ip link set eth0 down
```

**Assign IP Address (Temporary):**
```bash
# Add IP address
$ sudo ip addr add 192.168.1.50/24 dev eth0

# Remove IP address
$ sudo ip addr del 192.168.1.50/24 dev eth0

# Set default gateway
$ sudo ip route add default via 192.168.1.1 dev eth0

# These are TEMPORARY - lost on reboot!
```

---

### ifconfig - Legacy (Still Commonly Used)

```bash
# Install if missing
$ sudo apt install net-tools

# View all interfaces
$ ifconfig

# View specific interface
$ ifconfig eth0

# Set IP address (temporary)
$ sudo ifconfig eth0 192.168.1.50 netmask 255.255.255.0

# Bring interface up/down
$ sudo ifconfig eth0 up
$ sudo ifconfig eth0 down
```

---

### Persistent Network Configuration

**Ubuntu/Debian (Netplan - YAML based):**
```bash
# Configuration files
$ ls /etc/netplan/
01-netcfg.yaml

# Edit configuration
$ sudo nano /etc/netplan/01-netcfg.yaml
```

**Static IP Example:**
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

**DHCP Example:**
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

**Apply Configuration:**
```bash
# Test (reverts after 120 seconds if you lose connection)
$ sudo netplan try

# Apply permanently
$ sudo netplan apply
```

**Red Hat/CentOS (NetworkManager):**
```bash
# Configuration files
$ ls /etc/sysconfig/network-scripts/
ifcfg-eth0

# Edit
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-eth0
```

**Static IP (Red Hat):**
```
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
NAME="ens33"
UUID="..."
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.1.100"
NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"
DNS1="8.8.8.8"
DNS2="8.8.4.4"
```

**Restart Networking:**
```bash
# Ubuntu
$ sudo netplan apply
# or
$ sudo systemctl restart systemd-networkd

# Red Hat
$ sudo systemctl restart NetworkManager
# or
$ sudo nmcli connection reload
```

---

## 10.3 Network Diagnostic Tools

### ping - Test Connectivity

**Basic Usage:**
```bash
# Basic ping
$ ping google.com
PING google.com (142.250.80.46) 56(84) bytes of data.
64 bytes from lax17s55-in-f14.1e100.net (142.250.80.46): icmp_seq=1 ttl=118 time=12.3 ms
64 bytes from lax17s55-in-f14.1e100.net (142.250.80.46): icmp_seq=2 ttl=118 time=11.8 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 11.800/12.050/12.300/0.250 ms

# Limited ping count
$ ping -c 5 google.com

# Ping interval (every 2 seconds)
$ ping -i 2 google.com

# Ping with specific packet size
$ ping -s 1000 google.com

# Ping IP address
$ ping 192.168.1.1

# Ping localhost (test networking is working)
$ ping 127.0.0.1
$ ping localhost
```

**Interpreting ping Output:**
```
64 bytes        → Response size (healthy)
icmp_seq=1      → Sequence number
ttl=118         → Time to Live (hops remaining)
time=12.3 ms    → Round-trip time (latency)

0% packet loss  → Good connection
rtt avg=12 ms   → Average latency
```

**Troubleshooting with ping:**
```bash
# Check internet connectivity
$ ping -c 3 8.8.8.8        # Google DNS (IP)
$ ping -c 3 google.com     # Tests DNS too

# If IP works but domain doesn't → DNS issue
# If neither works → No internet

# Check local network
$ ping -c 3 192.168.1.1    # Gateway

# If gateway doesn't work → Network config issue
```

---

### traceroute / tracepath - Route Tracing

```bash
# Install if needed
$ sudo apt install traceroute

# Trace route to destination
$ traceroute google.com
traceroute to google.com (142.250.80.46), 30 hops max
 1  gateway (192.168.1.1)  1.234 ms  1.123 ms  1.345 ms
 2  isp-router (10.0.0.1)  5.678 ms  5.543 ms  5.789 ms
 3  backbone (172.16.0.1)  15.234 ms  15.543 ms  14.987 ms
 ...
 8  google.com (142.250.80.46)  12.456 ms  11.987 ms  12.123 ms

# tracepath (doesn't need root)
$ tracepath google.com

# UDP traceroute
$ traceroute -U google.com

# TCP traceroute (port 80)
$ sudo traceroute -T -p 80 google.com
```

**Interpreting traceroute:**
```
* * *           → Hop didn't respond (firewall or timeout)
increasing ms   → Normal latency increase
sudden jump     → Long-distance hop or congestion
```

---

### nslookup / dig / host - DNS Lookup

**nslookup:**
```bash
# Look up IP for domain
$ nslookup google.com
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Addresses: 142.250.80.46

# Reverse lookup (IP to domain)
$ nslookup 142.250.80.46

# Query specific DNS server
$ nslookup google.com 8.8.4.4
```

**dig (Domain Information Groper):**
```bash
# Install if needed
$ sudo apt install bind9-utils

# Basic lookup
$ dig google.com
; <<>> DiG 9.16.1 <<>> google.com
;; QUESTION SECTION:
;google.com.            IN  A
;; ANSWER SECTION:
google.com.     300     IN  A   142.250.80.46

# Short output
$ dig google.com +short
142.250.80.46

# MX record (email server)
$ dig google.com MX +short

# All records
$ dig google.com ANY

# Reverse lookup
$ dig -x 142.250.80.46

# Query specific nameserver
$ dig @8.8.8.8 google.com
```

**host:**
```bash
# Simple lookup
$ host google.com
google.com has address 142.250.80.46

# Reverse lookup
$ host 142.250.80.46

# MX records
$ host -t MX google.com
```

---

### netstat / ss - Network Connections

```bash
# Install netstat
$ sudo apt install net-tools

# All connections
$ netstat -an

# TCP connections
$ netstat -an | grep TCP

# Listening ports
$ netstat -tlnp
Proto  Local Address           Foreign Address        State
tcp    0.0.0.0:22              0.0.0.0:*               LISTEN      sshd
tcp    0.0.0.0:80              0.0.0.0:*               LISTEN      nginx
tcp    0.0.0.0:443             0.0.0.0:*               LISTEN      nginx

# ss (modern replacement for netstat)
$ ss -tlnp
State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1234))
LISTEN  0       128     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=2000))

# Established connections
$ ss -tn state established

# Show process using a port
$ sudo ss -tlnp | grep :80
```

---

### wget and curl - Download Files

**wget:**
```bash
# Download file
$ wget https://example.com/file.zip

# Save with different name
$ wget -O myfile.zip https://example.com/file.zip

# Resume interrupted download
$ wget -c https://example.com/largefile.iso

# Download in background
$ wget -b https://example.com/largefile.iso

# Download multiple files
$ wget -i urls.txt    # Each URL on its own line

# Limit download speed
$ wget --limit-rate=100k https://example.com/file.zip

# Download recursively
$ wget -r https://example.com/docs/

# Quiet mode (no progress)
$ wget -q https://example.com/file.zip
```

**curl:**
```bash
# Download to stdout
$ curl https://example.com

# Save to file
$ curl -o file.html https://example.com

# Save with original name
$ curl -O https://example.com/file.zip

# Follow redirects
$ curl -L https://example.com

# Show progress bar
$ curl -# -O https://example.com/file.zip

# Send POST request
$ curl -X POST -d "key=value" https://api.example.com/endpoint

# Send JSON
$ curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "age": 30}' \
  https://api.example.com/users

# Include headers in output
$ curl -v https://example.com

# Use API key
$ curl -H "Authorization: Bearer your-api-key" https://api.example.com

# Silent mode
$ curl -s https://api.example.com/data

# Test HTTP status code
$ curl -o /dev/null -s -w "%{http_code}" https://example.com
200
```

---

## 10.4 Secure Remote Access with SSH

### SSH - Secure Shell

SSH is the standard way to remotely access Linux systems securely.

**Basic Connection:**
```bash
# Connect to remote server
$ ssh user@hostname
$ ssh john@192.168.1.200
$ ssh john@myserver.example.com

# Connect on different port
$ ssh -p 2222 john@myserver.com

# Connect as specific user
$ ssh -l john myserver.com

# Connect and run command
$ ssh john@myserver.com "ls -la /home/john"

# Connect with identity file
$ ssh -i ~/.ssh/my_key john@myserver.com
```

---

### SSH Key Authentication (More Secure Than Password)

**Generate Key Pair:**
```bash
# Generate RSA key (recommended)
$ ssh-keygen -t rsa -b 4096 -C "john@laptop"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/john/.ssh/id_rsa): [press Enter]
Enter passphrase (empty for no passphrase): [enter passphrase]
Enter passphrase again: [confirm]
Your identification has been saved in '/home/john/.ssh/id_rsa'.
Your public key has been saved in '/home/john/.ssh/id_rsa.pub'.

# Or generate Ed25519 key (newer, shorter)
$ ssh-keygen -t ed25519 -C "john@laptop"

# View public key
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... john@laptop

# View key fingerprint
$ ssh-keygen -l -f ~/.ssh/id_rsa.pub
4096 SHA256:xxxxx (john@laptop) (RSA)
```

**Copy Public Key to Server:**
```bash
# Method 1: ssh-copy-id (easiest)
$ ssh-copy-id john@192.168.1.200

# Method 2: Manual
$ cat ~/.ssh/id_rsa.pub | ssh john@192.168.1.200 "cat >> ~/.ssh/authorized_keys"

# Method 3: scp then append
$ scp ~/.ssh/id_rsa.pub john@192.168.1.200:/tmp/
$ ssh john@192.168.1.200 "cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys && rm /tmp/id_rsa.pub"
```

**After copying, connect without password:**
```bash
$ ssh john@192.168.1.200
# No password prompt! (uses key authentication)
```

---

### SSH Configuration

**Client Configuration (~/.ssh/config):**
```bash
$ nano ~/.ssh/config

# Define shortcut for server
Host myserver
    HostName 192.168.1.200
    User john
    Port 2222
    IdentityFile ~/.ssh/id_rsa

Host webserver
    HostName web.example.com
    User deploy
    IdentityFile ~/.ssh/deploy_key

# Now connect with:
$ ssh myserver
# Instead of:
$ ssh -p 2222 john@192.168.1.200
```

**Server Configuration (/etc/ssh/sshd_config):**
```bash
$ sudo nano /etc/ssh/sshd_config

# Change default port (basic security)
Port 2222

# Disable root login (IMPORTANT!)
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Allow specific users
AllowUsers john alice

# Set idle timeout (seconds)
ClientAliveInterval 300
ClientAliveCountMax 3

# Restart SSH after changes
$ sudo systemctl restart sshd
```

---

### SSH Tunneling

**Port Forwarding:**
```bash
# Forward local port to remote server
# Access remote MySQL (3306) through local port 3307
$ ssh -L 3307:localhost:3306 john@myserver.com

# Forward remote port to local machine
$ ssh -R 8080:localhost:80 john@myserver.com

# Dynamic SOCKS proxy (like VPN)
$ ssh -D 1080 john@myserver.com
# Configure browser to use SOCKS proxy on port 1080

# Background tunnel (with -N for no shell and -f for background)
$ ssh -Nf -L 3307:localhost:3306 john@myserver.com
```

---

## 10.5 File Transfer

### scp - Secure Copy

```bash
# Copy file TO remote server
$ scp localfile.txt john@192.168.1.200:/home/john/

# Copy file FROM remote server
$ scp john@192.168.1.200:/home/john/file.txt ./

# Copy to specific path
$ scp file.txt john@192.168.1.200:/tmp/file.txt

# Copy directory recursively
$ scp -r ./project john@192.168.1.200:/home/john/

# Copy on different port
$ scp -P 2222 file.txt john@myserver.com:/home/john/

# Copy using identity file
$ scp -i ~/.ssh/my_key file.txt john@myserver.com:/tmp/

# Copy between two remote servers
$ scp john@server1:/file.txt alice@server2:/home/alice/

# Copy multiple files
$ scp file1.txt file2.txt john@192.168.1.200:/home/john/
```

---

### rsync - Remote Synchronization

**rsync** is better than scp for large transfers and synchronization.

```bash
# Copy file
$ rsync file.txt john@192.168.1.200:/home/john/

# Copy directory
$ rsync -avz ./project/ john@192.168.1.200:/home/john/project/
# -a : archive (preserves permissions, timestamps, etc.)
# -v : verbose
# -z : compress during transfer

# Sync directory (delete files not in source)
$ rsync -avz --delete ./project/ john@192.168.1.200:/home/john/project/

# Dry run (see what would happen)
$ rsync -avzn ./project/ john@192.168.1.200:/home/john/project/

# Exclude files
$ rsync -avz --exclude="*.log" --exclude=".git" ./project/ john@server:/project/

# Backup local directory to remote
$ rsync -avz --backup-dir=/backup/$(date +%Y%m%d) \
  /home/john/ john@backup-server:/backups/john/

# Progress bar
$ rsync -avz --progress ./largedir/ john@server:/backup/

# Limit bandwidth (100KB/s)
$ rsync -avz --bwlimit=100K ./project/ john@server:/project/

# Copy from remote to local
$ rsync -avz john@192.168.1.200:/remote/dir/ ./local/dir/

# Local sync (no network)
$ rsync -avz /source/dir/ /destination/dir/
```

---

## 10.6 Firewall Management

### ufw - Uncomplicated Firewall (Ubuntu)

```bash
# Check firewall status
$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443                        ALLOW       Anywhere

# Enable/disable firewall
$ sudo ufw enable
$ sudo ufw disable

# Default policies
$ sudo ufw default deny incoming    # Block all incoming by default
$ sudo ufw default allow outgoing   # Allow all outgoing by default

# Allow specific port
$ sudo ufw allow 80/tcp             # Allow HTTP
$ sudo ufw allow 443/tcp            # Allow HTTPS
$ sudo ufw allow 22/tcp             # Allow SSH

# Allow by service name
$ sudo ufw allow ssh
$ sudo ufw allow http
$ sudo ufw allow https

# Allow from specific IP
$ sudo ufw allow from 192.168.1.100 to any port 22

# Allow IP range
$ sudo ufw allow from 192.168.1.0/24

# Deny specific port
$ sudo ufw deny 23/tcp              # Block telnet

# Delete rule
$ sudo ufw delete allow 23/tcp

# Show rule numbers (for deletion)
$ sudo ufw status numbered
Status: active
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere

# Delete by number
$ sudo ufw delete 2

# Verbose status
$ sudo ufw status verbose
```

---

### iptables - Advanced Firewall

```bash
# View current rules
$ sudo iptables -L -v -n

# Allow SSH
$ sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
$ sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
$ sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
$ sudo iptables -A INPUT -i lo -j ACCEPT

# Block all other incoming
$ sudo iptables -A INPUT -j DROP

# Allow from specific IP
$ sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Block specific IP
$ sudo iptables -A INPUT -s 10.0.0.50 -j DROP

# Save rules (persist after reboot)
$ sudo iptables-save > /etc/iptables/rules.v4
# or
$ sudo ufw enable    # If using ufw, it manages iptables
```

---

## 💡 Key Concepts Summary

1. **IP Addressing**: IPv4 (192.168.1.x), subnets, gateways
2. **ip command**: Modern tool for all network configuration
3. **ping**: Test connectivity and latency
4. **traceroute**: Trace path to destination
5. **DNS**: nslookup, dig, host for name resolution
6. **SSH**: Secure remote access with key authentication
7. **scp/rsync**: Secure file transfer
8. **wget/curl**: Download files from web
9. **ufw/iptables**: Firewall for security
10. **netstat/ss**: Monitor network connections

---

## 🔥 Real-World Use Cases

### Use Case 1: Deploying to a Remote Server

```bash
# Connect to server
$ ssh deploy@production.example.com

# Check current state
$ ps aux | grep nginx
$ sudo systemctl status nginx

# Upload new files
$ rsync -avz --exclude=".git" ./myapp/ deploy@production:/opt/myapp/

# Restart service
$ sudo systemctl restart nginx

# Verify
$ curl -s -o /dev/null -w "%{http_code}" https://production.example.com
200
```

### Use Case 2: Setting Up SSH Keys for Team

```bash
# Each team member generates their key
$ ssh-keygen -t ed25519 -C "alice@company.com"

# Admin copies each key to server
$ ssh-copy-id alice@192.168.1.200
$ ssh-copy-id bob@192.168.1.200

# Disable password auth on server
$ sudo nano /etc/ssh/sshd_config
# PasswordAuthentication no

$ sudo systemctl restart sshd
```

### Use Case 3: Backup Server to Remote Location

```bash
# Automated backup with rsync
$ rsync -avz --delete \
  --exclude="*.tmp" \
  --log-file=/var/log/rsync.log \
  /var/www/html/ \
  backup@192.168.1.200:/backups/www/

# Add to cron for daily backup at 2 AM
$ crontab -e
0 2 * * * rsync -avz --delete /var/www/html/ backup@192.168.1.200:/backups/www/
```

### Use Case 4: Securing a New Server

```bash
# 1. Change SSH port
$ sudo nano /etc/ssh/sshd_config
# Port 2222

# 2. Set up firewall
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2222/tcp     # New SSH port
$ sudo ufw allow 80/tcp       # HTTP
$ sudo ufw allow 443/tcp      # HTTPS
$ sudo ufw enable

# 3. Disable root login
# PermitRootLogin no

# 4. Enable key-based auth only
# PasswordAuthentication no

# 5. Verify
$ sudo ufw status
$ sudo systemctl status sshd
```

### Use Case 5: API Testing with curl

```bash
# Test REST API
$ curl -s https://api.example.com/users | python3 -m json.tool

# POST new user
$ curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  -d '{"name":"John","email":"john@example.com"}' \
  https://api.example.com/users

# Check response code
$ curl -o /dev/null -s -w "%{http_code}\n" https://api.example.com/health
200

# Download with progress
$ curl -L --progress-bar -o ubuntu.iso \
  https://releases.ubuntu.com/22.04/ubuntu-22.04-desktop-amd64.iso
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Display your current IP address, gateway, and DNS server.

**Problem 2**: Ping google.com 5 times and note the average latency.

**Problem 3**: Use traceroute to find the path from your computer to 8.8.8.8.

**Problem 4**: Check which ports are currently open and listening on your system.

**Problem 5**: Use curl to display the HTTP status code of https://google.com.

### Intermediate Level

**Problem 6**: Generate an SSH key pair and display the public key.

**Problem 7**: Use wget to download a file from the web in background mode.

**Problem 8**: Set up ufw firewall to allow only SSH, HTTP, and HTTPS.

**Problem 9**: Use rsync to synchronize a local directory to another local directory (dry run first).

**Problem 10**: Use dig to find the MX records for gmail.com.

### Advanced Level

**Problem 11**: Create an SSH config file with shortcuts for three different servers.

**Problem 12**: Set up an SSH tunnel to forward a local port to a remote database.

**Problem 13**: Write a script that monitors a remote server's health using SSH and curl.

**Problem 14**: Configure persistent network settings using netplan for a static IP.

**Problem 15**: Create a comprehensive firewall ruleset using ufw for a web server.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
# IP address
$ ip addr show eth0 | grep "inet "
    inet 192.168.1.100/24 brd 192.168.1.255

# Or simpler
$ hostname -I
192.168.1.100

# Gateway
$ ip route show default
default via 192.168.1.1 dev eth0

# DNS server
$ cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
```

**Solution 2**:
```bash
$ ping -c 5 google.com
PING google.com (142.250.80.46) 56(84) bytes of data.
64 bytes from ... icmp_seq=1 ttl=118 time=12.1 ms
64 bytes from ... icmp_seq=2 ttl=118 time=11.8 ms
64 bytes from ... icmp_seq=3 ttl=118 time=12.4 ms
64 bytes from ... icmp_seq=4 ttl=118 time=11.9 ms
64 bytes from ... icmp_seq=5 ttl=118 time=12.2 ms

--- google.com ping statistics ---
5 packets transmitted, 5 received, 0% packet loss
rtt min/avg/max/mdev = 11.800/12.080/12.400/0.220 ms
# Average latency: 12.08 ms
```

**Solution 3**:
```bash
$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max
 1  gateway (192.168.1.1)  1.234 ms  1.123 ms  1.345 ms
 2  isp-hop1 (10.0.0.1)    5.678 ms  5.543 ms  5.789 ms
 ...
 8  dns.google (8.8.8.8)  12.456 ms  11.987 ms  12.123 ms
```

**Solution 4**:
```bash
$ ss -tlnp
State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22           0.0.0.0:*          sshd
LISTEN  0       128     0.0.0.0:80           0.0.0.0:*          nginx
LISTEN  0       128     0.0.0.0:443          0.0.0.0:*          nginx
```

**Solution 5**:
```bash
$ curl -o /dev/null -s -w "%{http_code}\n" https://google.com
301
# 301 = Redirect (google.com redirects to www.google.com)

$ curl -L -o /dev/null -s -w "%{http_code}\n" https://google.com
200
# -L follows redirects, so we get 200
```

### Intermediate Solutions

**Solution 6**:
```bash
$ ssh-keygen -t ed25519 -C "myuser@mypc"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/myuser/.ssh/id_ed25519):
Enter passphrase: 
Your identification has been saved in /home/myuser/.ssh/id_ed25519
Your public key has been saved in /home/myuser/.ssh/id_ed25519.pub

# Display public key
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIx... myuser@mypc
```

**Solution 7**:
```bash
# Download in background
$ wget -b https://releases.ubuntu.com/22.04/ubuntu-22.04-desktop-amd64.iso

Continuing in background in process 12345.
Logging to wget-log.

# Check progress
$ jobs
[1]+  Running   nohup wget https://... &

# Or check the log
$ tail -f wget-log

# Resume if interrupted
$ wget -c https://releases.ubuntu.com/22.04/ubuntu-22.04-desktop-amd64.iso
```

**Solution 8**:
```bash
$ sudo ufw default deny incoming
Default incoming policy changed to "deny"

$ sudo ufw default allow outgoing
Default outgoing policy changed to "allow"

$ sudo ufw allow 22/tcp
Rule added

$ sudo ufw allow 80/tcp
Rule added

$ sudo ufw allow 443/tcp
Rule added

$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup

$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

**Solution 9**:
```bash
# Create test directories
$ mkdir -p /tmp/source /tmp/destination

# Create some files
$ echo "File 1" > /tmp/source/file1.txt
$ echo "File 2" > /tmp/source/file2.txt
$ mkdir /tmp/source/subdir
$ echo "Sub file" > /tmp/source/subdir/file3.txt

# Dry run first
$ rsync -avzn /tmp/source/ /tmp/destination/
sending incremental file list
file1.txt
file2.txt
subdir/
subdir/file3.txt

sent 234 bytes  received 39 bytes  total size 24

# Actual sync
$ rsync -avz /tmp/source/ /tmp/destination/

# Verify
$ ls -R /tmp/destination/
/tmp/destination/:
file1.txt  file2.txt  subdir

/tmp/destination/subdir:
file3.txt
```

**Solution 10**:
```bash
$ dig gmail.com MX +short
5 gmail-smtp-in.l.google.com.
10 alt1.gmail-smtp-in.l.google.com.
20 alt2.gmail-smtp-in.l.google.com.
30 alt3.gmail-smtp-in.l.google.com.
40 alt4.gmail-smtp-in.l.google.com.

# Explanation:
# 5, 10, 20, 30, 40 are priorities (lower = try first)
# alt1, alt2... are backup mail servers
```

### Advanced Solutions

**Solution 11**:
```bash
$ mkdir -p ~/.ssh
$ nano ~/.ssh/config

Host webserver
    HostName 192.168.1.200
    User deploy
    Port 22
    IdentityFile ~/.ssh/deploy_key

Host dbserver
    HostName 192.168.1.201
    User dbadmin
    Port 2222
    IdentityFile ~/.ssh/db_key

Host devserver
    HostName dev.example.com
    User john
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes

# Set permissions
$ chmod 600 ~/.ssh/config

# Usage
$ ssh webserver       # Instead of: ssh deploy@192.168.1.200
$ ssh dbserver        # Instead of: ssh -p 2222 dbadmin@192.168.1.201
$ ssh devserver       # Instead of: ssh john@dev.example.com
```

**Solution 12**:
```bash
# Forward local port 3307 to remote MySQL on port 3306
$ ssh -L 3307:localhost:3306 john@dbserver.example.com

# In another terminal, connect to MySQL via tunnel
$ mysql -h 127.0.0.1 -P 3307 -u root -p

# Background tunnel (non-interactive)
$ ssh -Nf -L 3307:localhost:3306 john@dbserver.example.com
# -N : Do not execute a remote command
# -f : Fork to background

# Check tunnel is active
$ ss -tlnp | grep 3307
LISTEN  0  128  127.0.0.1:3307  0.0.0.0:*  users:(("ssh",...))
```

**Solution 13**:
```bash
#!/bin/bash
# monitor_server.sh

SERVER="john@192.168.1.200"
LOG_FILE="/var/log/server_monitor.log"
ALERT_EMAIL="admin@example.com"

check_ssh() {
    if ssh -o ConnectTimeout=5 -o BatchMode=yes $SERVER "exit" 2>/dev/null; then
        echo "[$(date)] SSH: OK" >> $LOG_FILE
        return 0
    else
        echo "[$(date)] SSH: FAILED" >> $LOG_FILE
        echo "SSH connection failed to $SERVER" | mail -s "Server Alert" $ALERT_EMAIL
        return 1
    fi
}

check_web() {
    local status=$(ssh $SERVER "curl -o /dev/null -s -w '%{http_code}' http://localhost")
    if [ "$status" == "200" ]; then
        echo "[$(date)] Web: OK (HTTP $status)" >> $LOG_FILE
    else
        echo "[$(date)] Web: FAILED (HTTP $status)" >> $LOG_FILE
        echo "Web server returning $status" | mail -s "Server Alert" $ALERT_EMAIL
    fi
}

check_disk() {
    local usage=$(ssh $SERVER "df / | tail -1 | awk '{print \$5}' | tr -d '%'")
    echo "[$(date)] Disk: ${usage}%" >> $LOG_FILE
    if [ "$usage" -gt 80 ]; then
        echo "Disk usage at ${usage}%" | mail -s "Server Alert: Disk Space" $ALERT_EMAIL
    fi
}

check_cpu() {
    local usage=$(ssh $SERVER "top -bn1 | grep 'Cpu(s)' | awk '{print \$2}' | cut -d. -f1")
    echo "[$(date)] CPU: ${usage}%" >> $LOG_FILE
}

# Run checks
check_ssh
if [ $? -eq 0 ]; then
    check_web
    check_disk
    check_cpu
fi

# Usage:
# chmod +x monitor_server.sh
# crontab -e
# */5 * * * * /path/to/monitor_server.sh
```

**Solution 14**:
```bash
$ sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
        search:
          - example.com

# Test first (reverts in 120s if SSH breaks)
$ sudo netplan try

# If working, apply permanently
$ sudo netplan apply

# Verify
$ ip addr show eth0
$ ip route show
$ cat /etc/resolv.conf
```

**Solution 15**:
```bash
#!/bin/bash
# setup_webserver_firewall.sh

# Reset rules
sudo ufw reset

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (from specific IP for extra security)
sudo ufw allow from 203.0.113.0/24 to any port 22 proto tcp comment "SSH from office"

# Allow HTTP and HTTPS from anywhere
sudo ufw allow 80/tcp comment "HTTP"
sudo ufw allow 443/tcp comment "HTTPS"

# Allow database from web server only (same machine)
sudo ufw allow from 127.0.0.1 to any port 3306 proto tcp comment "MySQL local"

# Rate limit SSH to prevent brute force
sudo ufw limit 22/tcp comment "SSH rate limit"

# Allow ping (ICMP)
sudo ufw allow proto icmp comment "Ping"

# Block known bad IPs (example)
sudo ufw deny from 10.0.0.50 comment "Known malicious"

# Enable firewall
sudo ufw enable

# Show final rules
sudo ufw status verbose

# Usage: chmod +x setup_webserver_firewall.sh && sudo ./setup_webserver_firewall.sh
```

---

## 🎓 Learning Tips

### Network Troubleshooting Order

When connectivity fails, follow this order:
1. **Physical**: Is cable plugged in? WiFi enabled?
2. **IP Address**: `ip addr` — do you have one?
3. **Gateway**: `ping 192.168.1.1` — can you reach router?
4. **DNS**: `ping 8.8.8.8` vs `ping google.com` — IP works but name doesn't?
5. **Remote**: `traceroute` — where does it fail?

### SSH Security Checklist
```
✓ Use key-based authentication
✓ Disable root login
✓ Disable password authentication
✓ Change default port
✓ Use fail2ban to block brute force
✓ Keep SSH updated
```

### Common Commands Quick Reference
```bash
ip addr          → Show IP addresses
ip route         → Show routing table
ping host        → Test connectivity
traceroute host  → Trace path
ss -tlnp         → Show listening ports
ssh user@host    → Remote login
scp file user@host:/path  → Copy file
rsync -avz src/ user@host:/dst/  → Sync files
curl -s URL      → Download/test URL
ufw status       → Check firewall
```

### Common Mistakes to Avoid

1. **Forgetting to save network config**
   ```bash
   # WRONG - temporary only
   $ sudo ip addr add 192.168.1.50/24 dev eth0
   
   # RIGHT - save to netplan config file
   $ sudo nano /etc/netplan/01-netcfg.yaml
   ```

2. **Not using key auth for SSH**
   ```bash
   # Generate keys once
   $ ssh-keygen -t ed25519
   # Copy to all servers
   $ ssh-copy-id user@server
   ```

3. **Blocking SSH in firewall before testing**
   ```bash
   # ALWAYS allow SSH first
   $ sudo ufw allow 22/tcp
   # THEN set other rules
   $ sudo ufw enable
   ```

4. **Not following redirects with curl**
   ```bash
   # WRONG - gets 301/302
   $ curl https://google.com
   
   # RIGHT
   $ curl -L https://google.com
   ```

---

## 🔗 Connections to Future Chapters

- **Chapter 11**: Shell scripts for network automation
- **Chapter 13**: Network monitoring and system health
- **Chapter 14**: Network services management
- **Chapter 19**: Security hardening and SSH best practices
- **Chapter 21**: Network troubleshooting methodology

---

## 📚 Additional Resources

### Man Pages
```bash
man ip
man ping
man traceroute
man ssh
man scp
man rsync
man curl
man wget
man ufw
man netstat
man ss
```

### Important Files
```bash
/etc/netplan/           # Network configuration (Ubuntu)
/etc/hosts              # Local hostname resolution
/etc/resolv.conf        # DNS server configuration
/etc/ssh/sshd_config    # SSH server configuration
~/.ssh/config           # SSH client configuration
~/.ssh/authorized_keys  # Allowed SSH public keys
/etc/ufw/               # Firewall rules
```

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 11, ensure you can:
- [ ] View and understand IP addresses and network config
- [ ] Configure network interfaces (temporary and persistent)
- [ ] Use ping to test connectivity
- [ ] Use traceroute to diagnose path issues
- [ ] Perform DNS lookups with dig/nslookup
- [ ] Generate and use SSH keys
- [ ] Connect to remote servers with SSH
- [ ] Transfer files with scp and rsync
- [ ] Download files with wget and curl
- [ ] Configure firewall rules with ufw
- [ ] Troubleshoot common network issues
- [ ] Set up SSH tunneling

---

## 🚀 Next Steps

You now understand networking — the ability to connect, communicate, and secure Linux systems remotely.

In **Chapter 11**, you'll learn:
- Writing shell scripts from scratch
- Variables, operators, and control flow
- Loops, conditionals, and functions
- Reading input and handling arguments
- Building practical automation scripts

**Challenge**: Write a script that checks network connectivity to 5 hosts, logs results with timestamps, and alerts (to a file) if any host is unreachable!

---

*"Networking is the nervous system of Linux. Master it, and your systems can talk to each other securely and efficiently."*