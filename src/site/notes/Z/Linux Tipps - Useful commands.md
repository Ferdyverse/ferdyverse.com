---
{"dg-publish":true,"dg-path":"HowTo/linux-tipps","permalink":"/how-to/linux-tipps/","tags":["notes/fern"],"noteIcon":"fern","created":"2024-10-07 09:46","updated":"2025-01-07 15:23"}
---

Additional Files:
- [[Z/Resize HDD\|Resize HDD]]
## Mount HDD on startup
Just a litte script to add a entry to the fstab file:
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
I use wsman to configure the Intel AMT access for my servers
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
This is useful if you want to expose something to the internet but only allow connections from CloudFlare and the local subnet
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
This small script will create random secrets and save them into a .env file
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
 The script is based of the openssl command to create the random value
 ```bash
 openssl rand -base64 24
 ```

## Use `.env`-file in local terminal
Sometimes it is required to run commands that are based on environment vars. To include them into your current session you can use the following snippet:
```bash
export $(echo $(cat .env | sed 's/#.*//g') | xargs | envsubst)
```

## Runuser
With runuser you can execute commands as a different user, even when the user does not have a console.
```bash
runuser -u <user> -- <COMMAND>

# Example usecase
runuser -u alloy -- ./alloy --config-file test.alloy
```

## Git client functions
It is possible to define functions as alias in the `.gitconfig` file.
```bash
[alias]
	acp = "!f() { git add -A && git commit -m \"$@\" && git push; }; f"
```
So I can run `git acp "fix(): Missing file type"` and it will run all the commands defined above. I use this feature also to override some repo settings if needed

```bash
[alias]
	priv = "!f() { git config user.name 'Ferdyverse' && git config user.email 'git@ferdyverse.de'; }; f"
```