Learning resources for Raspberry Pi, sample projects, helpful references

### Headless Setup  

1. Fresh install of Raspbian Lite on your MicroSD card, which should create 2 new partitions (rootfs and boot)  
2. Open the boot partition, and created an empty file simply named "ssh" (no extension)  
3. Created new file `wpa_supplicant.conf` in boot partition with the contents:  
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CA

network={
	ssid="SSID"
	psk="PASSWORD"
	key_mgmt=WPA-PSK
}
```  
Replace the SSID with your WiFi name, and PASSWORD with the password  
4. Set static IPs  
Open the "rootfs" partition as root. Edit /etc/dhcpcd.conf (append to end of file)  
> Make sure the indentations are done with tabs?  
```
interface eth0
    static ip_address=192.168.0.200/24
    static routers=192.168.0.1
    static domain_name_servers=192.168.0.1

interface wlan0
    static ip_address=192.168.0.200/24
    static routers=192.168.0.1
    static domain_name_servers=192.168.0.1
```  

5. Start up, ssh: `ssh pi@192.168.0.200`  
> If that doesn't work, try `ssh pi@raspberrypi.local` (the default hostname)  
6. Run `passwd`, change the default password  
7. Change hostname using `sudo raspi-config`, enable Camera and 1-Wire interfaces if you want to use them later, then reboot  
8. SSH back in, and run `sudo apt-get update -y && sudo apt-get upgrade -y`  
> Like in Step 5, if you can't connect to the right ip, try `ssh pi@<newhostname>.local` (the new hostname)  


### Enable Automatic Updates  
```  
sudo apt-get install unattended-upgrades  
dpkg-reconfigure -plow unattended-upgrades  
```  

## Favourite Applications  
### Pi-Hole  
Run Raspberry Pi as a local DNS server to block ads across entire local network  
[Pi-Hole home](https://pi-hole.net)  

#### Installation  
```
curl -sSL https://install.pi-hole.net | bash
```

#### My Blocklists   
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts	 
https://mirror1.malwaredomains.com/files/justdomains	 
http://sysctl.org/cameleon/hosts	 
https://zeustracker.abuse.ch/blocklist.php?download=domainblocklist	 
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt	 
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt	 
https://hosts-file.net/ad_servers.txt	 
https://gist.githubusercontent.com/anudeepND/adac7982307fec6ee23605e281a57f1a/raw/5b8582b906a9497624c3f3187a49ebc23a9cf2fb/Test.txt  

---
### PiVPN  
Local VPN server to connect to local internet from anywhere  
[PiVPN home](http://www.pivpn.io)  

#### Installation  
```
curl -L https://install.pivpn.io | bash
```

#### Having issues?  
If you're having trouble accessing local devices over vpn, try running
`pivpn debug`
It might automatically fix those issues, especially if the issue is due to incorrect masquerading (I don't actually know what that is!)

---
## Raspberry Pi Setup  

### Enable SSH  
###### SSH is what you will use to log in to your raspberry pi remotely and control it through your CLI. (command-line interface)  
Open raspberry pi settings and enable ssh on port 22, if your raspberry pi is connected to a monitor.  
Alternatively, you can run this:  
```
sudo raspi-config  
```
Select Interfacing Options (5)  
Select SSH and set it to enabled.  
  
### Set Device Name (Hostname)  
###### This is very helpful when you have more than 1 raspberry pi on the same network and you need to be able to distinguish between them.  
```
sudo raspi-config  
```
Select Network Settings (2)  
Set hostname to something unique, like raspi-uniquename  

### Set Static IP Address  
###### Setting a static ip address means you always know how to connect to your raspberry pi from another computer.  
```
sudo nano /etc/dhcpcd.conf
```
Append this to the end of the file:  
```
interface eth0
    static ip_address=192.168.0.100/24
    static routers=192.168.0.1
    static domain_name_servers=192.168.0.1
```
This will set your raspberry pi to the IP address 192.168.0.100 on interface eth0.  
What this means is if your raspberry pi connects over an ethernet (wired) connection, it will set its own address to 192.168.0.100.
To set a wireless IP address, change **eth0** to **wlan0**.  
You can stack an eth0 config and a wlan0 config on top of each other in the config file.  


### Split Memory  
###### If you run the Raspberry Pi without a monitor connected you can safely reduce the amount of ram used by the GPU. By default, the Pi has assigned 64MB RAM to the GPU.  
```
sudo raspi-config  
```
Select Advanced Options (7)  
Change 64 to 16  


### Speed up boot  
###### With many Linux installations, starting the OS and applications can be delayed during random number generation. haveged speeds that up.  
```
sudo apt-get install haveged -y 
```

### Crontab basics  
```  
# m h  dom mon dow   command  
* * * * * /home/pi/code/webcam/snap.sh >> /home/pi/code/webcam/snap.log 2>&1  
```  
This command above calls the snap.sh script every minute and outputs the execution details to snap.log  

### Camera basics
#### Local setup
Cameras are easy to set up, drivers aren't even usually required to set things up.  
`sudo apt-get install fswebcam`  
`fswebcam image.jpg`  <- take a snapshot  
`fswebcam -r 1280x720 --no-banner image.jpg`  <- take a snapshot with more pixels and no banner  

Script to take a snapshot (don't forget `chmod +x`):  
```
#!/bin/bash

DATE=$(date +"%Y-%m-%d_%H%M")

fswebcam -r 1280x720 --no-banner /home/pi/code/webcam/images/$DATE.jpg
```
 
 Stitch images together:  
 `mencoder mf://*.jpg -mf fps=24:type=jpg -ovc x264 -x264encopts bitrate=1200:threads=2 -o outputfile.mkv`  
   
   
 Here's a guide that shows how to use the `motion` application to quickly set up a raspi webcam server:  
 https://pimylifeup.com/raspberry-pi-webcam-server/
   
 This can easily be integrated into homekit!
 
 Summary here:
 #### Webcam Server

`sudo apt-get install libjpeg-dev gettext libmicrohttpd-dev libavformat-dev libavcodec-dev libavutil-dev libswscale-dev libavdevice-dev libwebp-dev mysql-common libmariadbclient18 libpq5`

`sudo wget https://github.com/Motion-Project/motion/releases/download/release-4.2.1/pi_stretch_motion_4.2.1-1_armhf.deb`  
`sudo dpkg -i pi_stretch_motion_4.2.1-1_armhf.deb`

`sudo nano /etc/motion/motion.conf`
daemon on
stream_localhost off

`sudo nano /etc/default/motion`
start_motion_daemon=yes

`sudo service motion start`
   
 ---
 
 ### HomeBridge Setup  
1. Node v4.3.2 or greater is required. Check by running: `node --version`. The plugins you use may require newer versions.  
2. `sudo apt-get install libavahi-compat-libdnssd-dev`  
3. `sudo npm install -g --unsafe-perm homebridge`
4. Install plugins using: `npm install -g <plugin-name>`  

A. Use this plugin for IP cameras: https://www.npmjs.com/package/homebridge-camera-ffmpeg  
NOTE: Camera plugins require ffmpeg: `sudo apt-get install libav-tools`
Do this: https://github.com/KhaosT/homebridge-camera-ffmpeg/wiki/Raspberry-PI  
See this too for configs: https://github.com/KhaosT/homebridge-camera-ffmpeg/wiki/Tested-Configurations#configjson  

B. Use this plugin for DS18B20 temp sensor: https://www.npmjs.com/package/homebridge-ds18b20  
5. Create the config.json file.  
 
Start for testing:  
`homebridge`

Set up automatic start (systemctl)  
`sudo nano /etc/systemd/system/homebridge.service` and paste this: 
```
[Unit]
Description=Node.js HomeKit Server
After=syslog.target network-online.target

[Service]
Type=simple
User=homebridge
EnvironmentFile=/etc/default/homebridge
ExecStart=/usr/bin/homebridge
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```
 
Create a user to run service: `sudo useradd --system homebridge`   
This copies your current user’s config. This assumes you have already added accessories etc.  
`sudo cp -r ~/.homebridge/persist /var/homebridge`  
`sudo chmod -R 0777 /var/homebridge`  
`sudo systemctl daemon-reload`  
`sudo systemctl enable homebridge`  
`sudo systemctl start homebridge`  
Type `sudo systemctl status homebridge` to check the status of the service.  
`sudo systemctl start/stop/restart homebridge` to start/stop/restart  
Type `sudo journalctl -fu homebridge` to view the logs  
 

 
TODO - Set up SSH keys  
