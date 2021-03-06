# RPi Plex

## Introduction

This is the setup for a Raspberry Pi from scratch to setup Plex Server, a Torrent Client (Deluge), and a VPN client.

## Contents

- [Burning](#burning)
- [Enabling SSH](#enabling-ssh)
- [Using RPi](#using-rpi)
- [Change Password](#change-password)
- [Set Static IP](#set-static-ip)
- [Install Plex Server](#install-plex-server)
- [View Plex Server](#view-plex-server)
- [Install and Use Deluge](#install-and-use-deluge)
- [Install and Use PIA OpenVPN](#install-and-use-pia-openvpn)
- [Fix VPN Routing](#fix-vpn-routing)
- [Startup Script](#startup-script)
- [Fixing Your Router](#fixing-your-router)
- [Deluge](#deluge)
- [Disclaimer](#disclaimer)

## Burning

1. Download the latest raspbian using torrent (fastest)
2. Get etcher.io
3. Burn the latest raspbian using etcher

## Enabling SSH

1. Reinsert the flash card
2. `touch /Volumes/boot/ssh`
3. Eject and insert into your RPi
4. Plug your RPi into power and ethernet to router

## Using RPi

1. `brew install nmap`
2. `nmap -sn 192.168.0.1/24`
3. `ssh -o IdentitiesOnly=yes pi@192.168.0.135`
4. Password is `raspberry`

## Change Password

1. Type `passwd` and follow the prompt
2. Reboot: `sudo reboot`
3. SSH into the RPi with the new password: `ssh pi@192.168.0.135`

## Set Static IP

1. Identify the router's DHCP range
2. Select an arbitrary IP outside of the DHCP range
3. `sudo nano /boot/cmdline.txt`
4. Add the static IP at the end: `ip=192.168.0.200`
5. Reboot: `sudo reboot`
6. SSH into the RPi at the new static IP: `ssh pi@192.168.0.200`

## Disable IPv6

1. Run the command below:

```bash
if ! grep -q disable_ipv6 /etc/sysctl.conf; then printf "net.ipv6.conf.all.disable_ipv6 = 1\nnet.ipv6.conf.default.disable_ipv6 = 1\nnet.ipv6.conf.lo.disable_ipv6 = 1\n" | sudo tee -a /etc/sysctl.conf; fi
```

2. `sudo reboot`
6. SSH into the RPi: `ssh pi@192.168.0.200`

## Install Plex Server

1. Run commands below:

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install apt-transport-https -y
wget -O - https://dev2day.de/pms/dev2day-pms.gpg.key | sudo apt-key add -
echo "deb https://dev2day.de/pms/ jessie main" | sudo tee /etc/apt/sources.list.d/pms.list
sudo apt-get update
sudo apt-get install -t jessie plexmediaserver -y
sudo mkdir -p /srv/movies
```

2. Open the plex server config: `sudo nano /etc/default/plexmediaserver`
3. Change the user from `plex` to `pi`: `PLEX_MEDIA_SERVER_USER=pi`
4. Save and exit
5. Restart the server: `sudo service plexmediaserver restart`

## View Plex Server

1. Go to [http://192.168.0.200:32400/web](http://192.168.0.200:32400/web)
2. Sign up or Sign in
3. Follow the steps and use `/srv/movies` as the movie location

## Install and Use Deluge

1. `sudo apt-get install deluged deluge-web -y`
2. `deluged`
3. View deluge web ui at: `http://192.168.0.200:8112/`
4. Default password is: `deluge` (change that when prompted)
5. When you're done, stop deluged: `pkill deluged`

Alternatively, you can use `deluge-console` on the command line

## Install and Use PIA OpenVPN

Use an OpenVPN client for Private Internet Access (PIA) 

1. Run commands below:

```bash
# Install and setup OpenVPN for PIA
sudo apt-get install openvpn
cd /etc/openvpn
sudo wget https://www.privateinternetaccess.com/openvpn/openvpn.zip
sudo unzip openvpn.zip

# Replace with actual username and pass
printf "ACTUAL_PIA_USERNAME\nACTUAL_PIA_PASSWORD\n" | sudo tee auth.txt
sudo chmod 600 auth.txt
sudo mv US\ Silicon\ Valley.ovpn US-Silicon-Valley.ovpn
if ! grep -q /etc/openvpn US-Silicon-Valley.ovpn; then sudo sed -i 's|crl-verify crl.rsa.2048.pem|crl-verify /etc/openvpn/crl.rsa.2048.pem|g' US-Silicon-Valley.ovpn && sudo sed -i 's|ca ca.rsa.2048.crt|ca /etc/openvpn/ca.rsa.2048.crt|g' US-Silicon-Valley.ovpn; fi
if ! grep -q auth.txt US-Silicon-Valley.ovpn; then sudo sed -i 's|auth-user-pass|auth-user-pass /etc/openvpn/auth.txt|g' US-Silicon-Valley.ovpn; fi
if ! grep -q update-resolv-conf US-Silicon-Valley.ovpn; then printf "up /etc/openvpn/update-resolv-conf\ndown /etc/openvpn/update-resolv-conf\nscript-security 2\n" | sudo tee -a /etc/openvpn/US-Silicon-Valley.ovpn; fi

# Connect to PIA servers with OpenVPN
sudo nohup openvpn /etc/openvpn/US-Silicon-Valley.ovpn &
```

## Fix VPN Routing

PIA/OpenVPN will configure your route table `route -n` with:

```
10.94.10.5      0.0.0.0         255.255.255.255 UH    0      0        0 tun0
```

While your actual `tun0` ip address is `10.94.10.6` and has an `ifconfig` record of:

```
tun0: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.94.10.6  netmask 255.255.255.255  destination 10.94.10.5
```

Your route table `route -n` should look like:

```
10.94.10.5      10.94.10.6         255.255.255.255 UH    0      0        0 tun0
```

So, you can run these command on startup to clean and properly configure the VPN Client:

```bash
cd ~
cat << EOF > fix-route.sh
ip route del \$(ifconfig | grep -A 1 tun0 | grep inet | awk '{print \$6}') via default
ip route add \$(ifconfig | grep -A 1 tun0 | grep inet | awk '{print \$6}') via \$(ifconfig | grep -A 1 tun0 | grep inet | awk '{print \$2}')
EOF

chmod a+x fix-route.sh
sudo mv fix-route.sh /etc/openvpn/fix-route.sh
sudo bash /etc/openvpn/fix-route.sh
```

## Startup Script

```bash
cat << EOF > startup.sh 
echo "Restarting Plex"
sudo service plexmediaserver restart

echo "Quitting deluge"
pkill deluge

echo "Starting deluged"
deluged

echo "Starting OpenVPN"
cd /etc/openvpn
sudo nohup openvpn US-Silicon-Valley.ovpn &

sleep 5

echo "Fixing Route"
cd /etc/openvpn
sudo bash /etc/openvpn/fix-route.sh

echo "Startup Done"
EOF

chmod a+x startup.sh
```

Then on startup, run:

```bash
./startup.sh
```

## Fixing Your Router

So, at the end of the day, you've ran all of these commands. However, you're still not connecting to the internet. The issue is most likely that your router doesn't have a route setup for you to hit PIA/OpenVPN.

Log into your router and add this route:

```
Destination: 10.94.10.5
Netmask: 255.255.255.255
Gateway: 0.0.0.0
```

Then you should be all good to go.

## Deluge

```bash
deluge-console
```

## Disclaimer

**These techonologies are only intended for educational usage**
