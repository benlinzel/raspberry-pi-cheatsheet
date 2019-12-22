# Set up PiHole, PiVPN, Homebridge, and Monitoring  

1. fresh install of Raspbian Lite  
2. Created an empty file simply named "ssh" (no extension) in the boot partition  
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
4. Set static IPs  
in the rootfs partition, edit /etc/dhcpcd.conf (append to file)  
Make sure the indentations are done with tabs?  
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
> If that doesn't work, try `ssh pi@raspberrypi.local`  
6. `passwd`, then reboot  
7. change hostname using `sudo raspi-config` then reboot  
8. `sudo apt-get update -y && sudo apt-get upgrade -y`  
9. `sudo apt-get install git -y`  

Install PiHole  
`curl -sSL https://install.pi-hole.net | bash`  
###### save admin password after installer completes  
go to `http://raspi-hue.local/admin/`  

## Install Homebridge  
`sudo apt-get install libavahi-compat-libdnssd-dev`  
`sudo apt-get install nodejs npm`  
`sudo npm install -g --unsafe-perm homebridge`  

`sudo nano /etc/systemd/system/homebridge.service`  
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
`sudo nano ~/.homebridge/config.json`  

Paste this:  
```
{
  "bridge": {
    "name": "Homebridge",
    "username": "B6:17:AB:C6:4B:45",
    "port": 51826,
    "pin": "111-22-333"
  },

  "description": "My Homebridge Setup",

  "accessories": [
    {
      "accessory": "HTTP-TEMPERATURE",
      "name": "Pi 1",
      "getUrl": "http://raspi-little-cato.local:8000/"
    },
    {
      "accessory": "HTTP-TEMPERATURE",
      "name": "Pi 2",
      "getUrl": "http://192.168.0.178:8000/"
    }   
  ],

  "platforms": [
    {
      "platform": "Camera-ffmpeg",
      "cameras": [
        {
          "name": "Example Webcam",
          "videoConfig": {
            "source": "-re -i http://192.168.0.175:8081/video",
            "stillImageSource": "-i http://192.168.0.175:8081/image",
            "maxStreams": 2,
            "maxWidth": 640,
            "maxHeight": 480,
            "maxFPS": 20
          }
        }
      ]
    }
  ]
}

```

`sudo npm install -g homebridge-camera-ffmpeg`  
`sudo npm install -g homebridge-http-temperature-sensor`  
`sudo systemctl daemon-reload`   
`sudo systemctl enable homebridge`   
`sudo systemctl start homebridge`   
`sudo systemctl status homebridge` to check the status of the service.   
`sudo systemctl start homebridge` to start (or stop or restart)   
`sudo journalctl -fu homebridge` to view the logs   

## Install PiVPN  
`curl -L https://install.pivpn.io | bash`  

(Choose Cloudflare for DNS)  
`pivpn add`  

Router:  
Open Port 1194 (UDP) for IP, forward to port 1194  

Download the `.ovpn` file from ~/ovpns to client devices

## System Monitoring  
###### Warning: This takes the longest time to set up!  

`bash <(curl -Ss https://my-netdata.io/kickstart.sh)`  
More info here: (https://github.com/netdata/netdata/blob/master/README.md)  

http://raspi-hue.local:19999/  
