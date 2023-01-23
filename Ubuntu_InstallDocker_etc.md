# How to set up Ubuntu

## Install Docker in Ubuntu

<https://docs.docker.com/engine/install/ubuntu/>

```bash
cat > install_docker.sh <<EOF

sudo apt-get update
echo y | sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -p /etc/apt/keyrings
sudo rm -f /etc/apt/keyrings/docker.gpg
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
echo y | sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo chmod 666 /var/run/docker.sock
EOF

chmod 777 install_docker.sh
./install_docker.sh
```

## DNS

Make sure there are valid nameserver entries in `/etc/resolv.conf`

```bash
cat /etc/resolv.conf
```

If not -

```bash
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

Add below entries to `/etc/resolv.conf`

```bash
nameserver 8.8.8.8 
nameserver 8.8.4.4
```

## ifconfig

```bash
apt update && apt upgrade
apt install net-tools
```

## ping

```bash
apt update && apt upgrade
apt install iputils-ping
```
