---
{"dg-publish":true,"dg-path":"HowTo/linux-tipps","permalink":"/how-to/linux-tipps/","tags":["📝/🌿"],"noteIcon":"fern","created":"2024-10-07 09:46","updated":"2025-01-07 15:23"}
---

Additional Files:
- [[Z/Resize HDD\|Resize HDD]]
## Mount HDD on startup

```bash
# make a var for your mountpath -> Just to be able to copy the commands from this codeblock
mount=/data

# Create your target folder
sudo mkdir -p $mount

# Make it accessible for everone or your user
sudo chmod ugo+rwx $mount
sudo chown $USER:$USER $mount

# Add mountpoint to `/etc/fstab` -> Replace `/dev/nvme1n1p1` with your partition.
echo "UUID=$(lsblk -no UUID /dev/nvme1n1p1) $mount $(lsblk -no FSTYPE /dev/nvme1n1p1) defaults,noatime 0 2" | sudo tee -a /etc/fstab

# Mount everything from `/etc/fstab`
sudo mount -a
```

## Get network connection details

See if a connection is possible and get all the hops.
```bash
sudo traceroute -T -p <PORT> <TARGET_IP>
```

| Argument   | Description                                 |
| ---------- | ------------------------------------------- |
| -I  --icmp | Use ICMP ECHO for tracerouting              |
| -p  --port | Set the destination port to use.            |
| -T  --tcp  | Use TCP SYN for tracerouting                |
| -U  --udp  | Use UDP to particular port for tracerouting |
### MTR (My Traceroute)

```shell
mtr -s 1000 -r -c 200 <TARGET>
```

| Argument | Description                                             |     |
| -------- | ------------------------------------------------------- | --- |
| -4       | Use IPv4 only                                           |     |
| -6       | Use IPv6 only                                           |     |
| -c       | Count -> How many requests will be send                 |     |
| -j       | json -> Output stats in json                            |     |
| -r       | Report Mode -> generates Statistics                     |     |
| -s       | Packagesize                                             |     |
| -w       | Report wide -> Longer output, hostnames will not be cut |     |
## Flush IPTables
Run this as root in a script!
```bash
# Accept all traffic first to avoid ssh lockdown  via iptables firewall rules #
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
 
# Flush All Iptables Chains/Firewall rules #
iptables -F
 
# Delete all Iptables Chains #
iptables -X
 
# Flush all counters too #
iptables -Z 
# Flush and delete all nat and  mangle #
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -t raw -F
iptables -t raw -X

systemctl restart networking
```

## WSMAN

```shell
# Evoli
export AMT_HOST=192.168.1.14 export AMT_PASSWORD=<PASSWORD> export VNC_PASSWORD=<PASSWORD>
# Pikachu
export AMT_HOST=192.168.1.11 export AMT_PASSWORD=<PASSWORD> export VNC_PASSWORD=<PASSWORD>

# View settings
docker run --rm -it derekgottlieb/wsmancli wsman get http://intel.com/wbem/wscim/1/ips-schema/1/IPS_KVMRedirectionSettingData -h ${AMT_HOST} -P 16992 -u admin -p ${AMT_PASSWORD}

# Set the VNC password
docker run --rm -it derekgottlieb/wsmancli wsman put http://intel.com/wbem/wscim/1/ips-schema/1/IPS_KVMRedirectionSettingData -h ${AMT_HOST} -P 16992 -u admin -p ${AMT_PASSWORD} -k RFBPassword=${VNC_PASSWORD}

# Enable KVM redirection to port 5900
docker run --rm -it derekgottlieb/wsmancli wsman put http://intel.com/wbem/wscim/1/ips-schema/1/IPS_KVMRedirectionSettingData -h ${AMT_HOST} -P 16992 -u admin -p ${AMT_PASSWORD} -k Is5900PortEnabled=true
```

## Allow only CloudFlare

```shell
ufw default deny incoming # Block all incoming!!!
ufw default allow outgoing # Allow all outgoing
ufw allow ssh # Allow SSH

# Allow access for local subnet (preferably dedicated subnet for hosted services)
ufw allow from 192.0.0.0/8 to any port 443

# Allow CloudFlare IPs
wget -O- https://www.cloudflare.com/ips-v4 | \
while read line; do ufw allow from $line to any port 443; done

# Add IPv6 support # wget -O- https://www.cloudflare.com/ips-v6 | \
# while read line; do ufw allow from $line to any port 443; done
```

## Generate Secrets
```bash
#!/bin/bash
generate_secret() {
    local length=${1:-30}
    local generate_length=$((length + 4))
    openssl rand -base64 "$generate_length" | tr -d '+=/\n' | cut -c1-"$length"
}

[ -f .env ] && { echo ".env file already exists!"; exit 1; }

cat > .env << EOL
POSTGRES_PASSWORD=$(generate_secret)
JWT_SECRET=$(generate_secret 64)
SESSION_KEY=$(generate_secret 24)
REDIS_PASSWORD=$(generate_secret 20)
UNSAFE_PLACEHOLDER=__WARNING_REPLACE_RANDOM_TEXT__
EOL

echo "New .env file generated with secure random values!"
```