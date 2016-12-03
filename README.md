
**If you want something easier than a script, look at the dedicated distro that will build an up-to-date wifi hotspot. 
Look at [this repository](https://github.com/pihomeserver/Kupiki-Hotspot)**


What is Pi Hotspot
==================

This project is the latest version of [the tutorial](http://www.pihomeserver.fr/2015/08/05/raspberry-pi-coovachilli-et-freeradius-pour-un-hotspot-wifi-avec-portail-captif/) created first on my blog [Pi Home Server](http://www.pihomeserver.fr)
Created on a Raspberry Pi 2, some functionalities and tools where not available for the Raspberry Pi 3. Also the tutorial was a little bit difficult to 
implement because of too many steps. That's why i decided to create a script that will help you to build your own hotspot automatically.

Once the script is executed, you will get :
- A Wifi hotspot using the integrated wifi chipset
- A captive portal based on coovachilli
- An authenticate control based on freeRadius
- An interface for freeRadius based on daloRadius

Requirements
============

What are the requirements ?
- A Raspberry Pi 3
- An ethernet cable
- A power supply for the Raspberry Pi
- A micro SD card with a raspbian-like OS installed. For this project i used [minibian](https://minibianpi.wordpress.com/) which optimized
for this project
- An internet access of course

Usage
=====

You just have to download the script, edit it to update it's parameters, execute and wait ... If the wifi on the Raspberry is not already configured, don't worry, the script will do it

- Download the script with the following command   
` git clone https://github.com/pihomeserver/Pi-Hotspot.git `
- Edit the script and update the first lines to define your own configuration (take care that an ethernet link is required)
- Execute the script using sudo (or as root but you already may know that it's not recommanded)

A log file named `pihotspot.log` will be created in the folder `/var/log`

Support
=======

Please use input your requests or issues in the GIT repository 

Any contribution is welcome !




How-to Rpi3 2016-11:
===

- Install minibian image to SD
- connecth with SSH and resize SD: ([source](https://minibianpi.wordpress.com/how-to/resize-sd/))
```
#connect to Rpi3 with SSH
fdisk /dev/mmcblk0
#p, d, 2, n, p, 2, (create a new primary partition, next you need to enter the start of the old main partition and then the size (enter for complete SD card). The main partition on Minibian image from 2016-11-27 starts at 125056, but the start of your partition might be different. Check the p output!)
#w
reboot
resize2fs /dev/mmcblk0p2
df -h #(to check size)
```
  
- onboard wifi, and bluetooth ([source](https://minibianpi.wordpress.com/how-to/rpi3/))
```
apt-get update
apt-get install firmware-brcm80211 pi-bluetooth wpasupplicant
```

- install editor  
```
apt-get install nano
```
- install external Ralink USB wifi ([source](https://minibianpi.wordpress.com/how-to/wifi/))
```
#to check USB wifi:
apt-get install usbutils 
#verify firmware name (something similar: Bus 001 Device 004: ID 148f:5370 Ralink Technology...)
apt-get install firmware-realtek
apt-get install wpasupplicant (is this optional if we only need hotspot?)
```
- modifie interfaces, comment out everything about wlan0 and insert this:
```
auto wlan1
allow-hotplug wlan1
iface wlan1 inet static
        address 192.168.10.1
        netmask 255.255.255.0
        network 192.168.10.0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
```

- run the script, from original source (or from this fork)
```
apt-get install git
git clone https://github.com/pihomeserver/Pi-Hotspot.git
cd Pi-Hotspot
chmod +x pihotspot.sh
nano pihotspot.sh #(modifie ssid, and interface)
./pihotspot.sh
```
- testing:
  - login to daloRadius -> managment/users/New User - quick add -> name: test1 password: test2
  - run this in ssh:
```
radtest test1 test2 127.0.0.1 0 testing123
```
  - if it's work you will see this:
```
Sending Access-Request of id 192 to 127.0.0.1 port 1812
        User-Name = "test1"
        User-Password = "test2"
        NAS-IP-Address = 127.0.1.1
        NAS-Port = 0
        Message-Authenticator = 0x00000000000000000000000000000000
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812, id=32, length=20
```
- now you can test with another device. Connect to the new wifi hotspot, and try to login with test1/test2
- change every password! (daloRadius admin, SSH, chilli secret-key, ...)

daloRADIUS extra:
===
- set permissions:
```
chown www-data:www-data /usr/share/nginx/html -R
chmod 644 /usr/share/nginx/html/daloradius/library/daloradius.conf.php
```
