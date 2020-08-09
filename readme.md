# NeoGaze

neogaze	B8:27:EB:4D:50:D0	192.168.0.54/24	21.72	WI-FI 2.4G VM4826469

pi/raspbery


https://www.raspberrypi.org/documentation/raspbian/updating.md

sudo apt update

sudo apt full-upgrade



https://elinux.org/RPi-Cam-Web-Interface


sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get install git

git clone https://github.com/silvanmelchior/RPi_Cam_Web_Interface.git
cd RPi_Cam_Web_Interface

./install.sh

The scripts are

install.sh main installation as used in step 4 above
update.sh check for updates and then run main installation
start.sh starts the software. If already running it restarts.
stop.sh stops the software
remove.sh removes the software
debug.sh is same as start but allows raspimjpeg output to console for debugging

Camera:
![Alt text](./camera01.png)

https://www.stemmer-imaging.com/en-gb/knowledge-base/lens-mounts/

C-CS mount adapter

![Alt text](./CS-Mount-I0.png)

![Alt text](./C-Mount-I1.png)

Telescope eye-piece 1.25"

The first print is good - however.
It needs the insert into the CS-mount narrowing and lengthening.
Narrower by the width of the threads, and longer by 1 or 2 mm.

The telescope insert can be much longer.

I want to try and add tabs, so I can attached it to the camera board...

![Alt text](./PiHQSpec.png)

To add flanges, I think I might need to print it in two pieces.

Prototyping the flanges now as EyeTubeWithFlanges.

I had to make the eye holes bigger, and that spec sheet is wrong! Either than or I can't count.
It's only 30mm between corner hole center.

I'm also trying to print some spacers too...

Parts are saved in PrusaMini repo


I want to setup neoGaze to be a wireless access point.

https://thepi.io/how-to-use-your-raspberry-pi-as-a-wireless-access-point/

https://howtoraspberrypi.com/create-a-wi-fi-hotspot-in-less-than-10-minutes-with-pi-raspberry/

sudo apt-get install hostapd
sudo apt-get install dnsmasq
sudo systemctl disable hostapd
sudo systemctl disable dnsmasq

sudo nano /etc/hostapd/hostapd.conf

interface=wlan0
driver=nl80211
ssid=RPI3hot
hw_mode=g
channel=6
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=1234567890
wpa_key_mgmt=WPA‐PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP


sudo nano /etc/default/hostapd

Change:
#demon_conf=""
to
demon_conf="/etc/hostapd/hostapd.conf"


sudo nano /etc/dnsmasq.conf

#neogaze Config
#stop DNSmasq from using resolv.conf
no‐resolv
#Interface to use
interface=wlan0
bind‐interfaces
dhcp‐range=10.0.0.3,10.0.0.20,12h


sudo nano /etc/network/interfaces


sudo nano /usr/bin/autohotspot


#!/bin/bash
#
#Wifi config ‐ if no prefered Wifi generate a hotspot
#enter required ssids: ssids=('ssid1' 'ssid2')
ssids=('mySSID1' 'mySSID2')#Main script
createAdHocNetwork()
{
    ip link set dev wlan0 down
    ip a add 10.0.0.5/24 dev wlan0
    ip link set dev wlan0 up
    service dnsmasq start
    service hostapd start
}
connected=false
for ssid in "${ssids[@]}"
do
    if iw dev wlan0 scan ap‐force | grep $ssid > /dev/null
    then
        wpa_supplicant ‐B ‐i wlan0 ‐c /etc/wpa_supplicant/wpa_supplicant.conf > /dev/null 2>&1
        if dhclient ‐1 wlan0
        then
            connected=true
            break
        else
            wpa_cli terminate
            break
        fi
    else
        echo "Not in range, WiFi with SSID:" $ssid
    fi
done
if ! $connected; then
    createAdHocNetwork
fi


sudo chmod +x /usr/bin/autohotspot

sudo nano /etc/systemd/system/autohotspot.service

[Unit]
Description=Generates a non‐internet Hotspot for ssh when a listed ssid is not in range.
After=network‐online.target
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/autohotspot
[Install]
WantedBy=multi‐user.target


sudo systemctl enable autohotspot.service

sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd

I'm using the middle portion of the sensor only:

X: 24576 Y: 24576
W: 40960 H: 40960

FixedFPS and adjust shutter to set brightness.