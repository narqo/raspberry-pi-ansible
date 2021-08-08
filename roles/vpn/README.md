# TODO ansible'ify the steps below

## 1. Provision a VM

`t2.micro` (free tier) works fine.
Consider `t4.micro` after free tier expires.

### Security group

- allow SSH from anywhere
- allow UDP port 51820 from anywhere
- allow HTTP/HTTPS from anywhere

## 2. Setup Wireguard server (VM)

```
sudo yum update
sudo amazon-linux-extras list
sudo amazon-linux-extras install epel
sudo curl -Lo /etc/yum.repos.d/wireguard.repo https://copr.fedorainfracloud.org/coprs/jdoss/wireguard/repo/epel-7/jdoss-wireguard-epel-7.repo
sudo yum install wireguard-dkms wireguard-tools
wg genkey > server.key
wg pubkey < server.key > server.pub
sudo cp files/wg0-server.conf /etc/wireguard/wg0.conf
sudo chmod 0600 /etc/wireguard/wg0.conf
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```

## 3. Setup Wireguard client (Pi)

```
sudo apt install -y wireguard
wg genkey > pi-31.key
wg pubkey < pi-31.key > pi-31.pub
sudo cp files/wg0-client.conf /etc/wireguard/wg0.conf
sudo chmod 0600 /etc/wireguard/wg0.conf
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```

## Notes

https://keremerkan.net/posts/wireguard-mtu-fixes/
