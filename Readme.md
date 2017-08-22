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
3. `ssh pi@192.168.0.135`
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

## Install Plex Server

1. Run commands below:

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install apt-transport-https -y
wget -O - https://dev2day.de/pms/dev2day-pms.gpg.key | sudo apt-key add -
echo "deb https://dev2day.de/pms/ jessie main" | sudo tee /etc/apt/sources.list.d/pms.list
sudo apt-get update
sudo apt-get install -t jessie plexmediaserver
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

1. `sudo apt-get install deluged deluge-web`
2. `deluged`
3. `deluge-web`
4. View deluge web ui at: `http://192.168.0.200:8112/`
5. Default password is: `deluge` (change that when prompted)
6. When you're done, stop deluged: `pkill deluged`

## TODO

- [x] Update password
- [x] Install Plex Server
- [ ] Configure Deluge Service
- [ ] Install Private Internet Access