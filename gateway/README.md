# Gateway Set Up

## Table of contents
* [Hardware](#hardware)
* [Installation and configuration](#installation)
* [Access Point](#access_point)
* [Firewall](#firewall)
* [Fog computing](#fog_computing)


## Hardware  <a name="hardware"></a>

### Raspberry Pi 3
#### Specifications
* SoC: Broadcom BCM2837
* CPU: 4× ARM Cortex-A53, 1.2GHz
* GPU: Broadcom VideoCore IV
* RAM: 1GB LPDDR2 (900 MHz)
* Networking: 10/100 Ethernet, 2.4GHz 802.11n wireless
* Bluetooth: Bluetooth 4.1 Classic, Bluetooth Low Energy
* Storage: microSD
* GPIO: 40-pin header, populated
* Ports: HDMI, 3.5mm analogue audio-video jack, 4× USB 2.0, Ethernet, Camera Serial Interface (CSI), Display Serial Interface (DSI)

[Benchmarks](https://www.raspberrypi.org/magpi/raspberry-pi-3-specs-benchmarks/)

### Other
* Micro SD card (class 10 recommended).
* Power Supply- Micro USB.


## Installation <a name="installation"></a>

### Operative system

Raspbian Jessie Lite
* Minimal image based on Debian Jessie
* Version: January 2017
* Release date: 2017-01-11
* Kernel version: 4.4
* Release notes: Link
* Available at https://downloads.raspberrypi.org/raspbian_lite_latest

```bash
# Copy Raspbian image to the SD card.
dd bs=4M if=2017-01-11-raspbian-jessie-lite.img of=/dev/mmcblk0
```

### Minimum configuration
We are going to edit the files in the SD card.
In my system, it is located on
`/media/samuelmunoz/0aed834e-8c8f-412d-a276-a265dc676112`

#### Hostname
Set the name of your device in the file `etc/hostname`.
_Note: do **NOT** use `/etc/hostname`. That is your machine!_
```bash
echo "IOT-GW-01" > etc/hostname
```

#### WiFi Network (optional)
If you want to use a cable connection, you can skip this step. Just plug the cable and everything should work.
```bash
nano etc/wpa_supplicant/wpa_supplicant.conf
```
And add a configuration like this.

```
network={
 ssid="YOUR_NETWORK_NAME"
 psk="YOUR_NETWORK_PASSWORD"
 proto=RSN
 key_mgmt=WPA-PSK
 pairwise=CCMP
 auth_alg=OPEN
}
```

#### SSH
By default, Raspbian comes with SSH disabled. It is possible to enable it by running `raspi-config` on a terminal. But since it is a gateway, it does not have a screen nor a keyboard.

See: https://www.raspberrypi.org/blog/a-security-update-for-raspbian-pixel/
>The boot partition on a Pi should be accessible from any machine with an SD card reader, on Windows, Mac, or Linux. If you want to enable SSH, all you need to do is to put a file called ssh in the /boot/ directory. The contents of the file don’t matter: it can contain any text you like, or even nothing at all. When the Pi boots, it looks for this file; if it finds it, it enables SSH and then deletes the file. SSH can still be turned on or off from the Raspberry Pi Configuration application or raspi-config; this is simply an additional way to turn it on if you can’t easily run either of those applications.

_NOTE:_ this is not the previous partition. This is a FAT32 type partition, located in my system as:
`/media/samuelmunoz/boot`

```bash
touch /media/samuelmunoz/boot/ssh
```


### First connection
At this point, unmount the SD card, plug it into the Raspberry and wait 2 minutes or so to give it time to boot up and connect to the WiFi network.
If everything goes well, you should be able to connect to the device.
```bash
ssh pi@IOT-GW-01.local
```
_NOTE: the default password is **raspberry**._

#### Change the default passwords
It is highly insecure to leave a gateway with the default password. Change it!
```bash
passwd
```


#### Update system
Lets update the system.
```bash
apt-get update
apt-get upgrade
apt-get clean
```


## Access Point <a name="access_point"></a>
We are setting up the device to connect it to the internet through a wired connection while keeping the wireless interface as an Acces Point for our intranet of things.

Connect your device to a wired connection and SSH your device.
By default, it will get an IP address with DHCP, but if you need to set a static connection, you should edit `/etc/network/interfaces`

### Packages
* hostapd: Access Point
* dnsmasq: DHCP and DNS server
* Alternative: isc-dhcp-server bind9

```
apt-get install hostapd dnsmasq
```

### Set wlan0 as the AP
Do not use DHCP on the interface.
```
echo "denyinterfaces wlan0" >> /etc/dhcpd.conf
```
_Note: This must be above any added interface lines._

Edit the interface `wlan0` and modify its section
```
nano /etc/network/interfaces
```
Proposed configuration:
```
allow-hotplug wlan0
iface wlan0 inet static
    address 172.24.1.1
    netmask 255.255.255.0
    network 172.24.1.0
    broadcast 172.24.1.255
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```
Restart the service:
```
service dhcpcd restart
ifdown wlan0
ifup wlan0.
```

### Hostapd
* [Webpage](http://w1.fi/hostapd/)

_ NOTE: based on https://frillip.com/using-your-raspberry-pi-3-as-a-wifi-access-point-with-hostapd/ _
Create the config file:
```
nano /etc/hostapd/hostapd.conf
```

and put the following configuration.
_ NOTE: edit the values of `ssid=Pi3-AP` and `wpa_passphrase=raspberry` to your desired AP name and AP password. _

```
# This is the name of the WiFi interface we configured above
interface=wlan0

# Use the nl80211 driver with the brcmfmac driver
driver=nl80211

# This is the name of the network
ssid=Pi3-AP

# Use the 2.4GHz band
hw_mode=g

# Use channel 6
channel=6

# Enable 802.11n
ieee80211n=1

# Enable WMM
wmm_enabled=1

# Enable 40MHz channels with 20ns guard interval
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]

# Accept all MAC addresses
macaddr_acl=0

# Use WPA authentication
auth_algs=1

# Require clients to know the network name
ignore_broadcast_ssid=0

# Use WPA2
wpa=2

# Use a pre-shared key
wpa_key_mgmt=WPA-PSK

# The network passphrase
wpa_passphrase=raspberry

# Use AES, instead of TKIP
rsn_pairwise=CCMP
```

Edit the file
```
nano /etc/default/hostapd
```
and set the line `DAEMON_CONF="/etc/hostapd/hostapd.conf"`


### dnsmasq
* [Webpage](http://www.thekelleys.org.uk/dnsmasq/doc.html)

Backup the file:
```bash
mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

And create the new file:
```bash
nano /etc/dnsmasq.conf
```
With your configuration.
```
interface=wlan0      # Use interface wlan0
listen-address=172.24.1.1 # Explicitly specify the address to listen on
bind-interfaces      # Bind to the interface to make sure we aren't sending things elsewhere
server=8.8.8.8       # Forward DNS requests to Google DNS
domain-needed        # Don't forward short names
bogus-priv           # Never forward addresses in the non-routed address spaces.
dhcp-range=172.24.1.50,172.24.1.150,12h # Assign IP addresses between 172.24.1.50 and 172.24.1.150 with a 12 hour lease time
```

Restart the services
```
service hostapd start
service dnsmasq start
```

And you can see the access point and connect to it.


## Firewall <a name="firewall"></a>
* [Webpage](http://shorewall.org/)

```bash
apt-get install shorewall
```

### Configuration

#### Interfaces
* Map hardware network interfaces.
* File: `/etc/shorewall/interfaces`
```
#ZONE   INTERFACE       OPTIONS
wired   eth0
wifi    wlan0
```

#### Zones
* Define zones.
* File: `/etc/shorewall/zones`
```
#ZONE   TYPE            OPTIONS         IN                      OUT
#                                       OPTIONS                 OPTIONS
fw      firewall
wired   ip
wifi    ip
```

#### Policy
* Define general policy communications.
* File: `/etc/shorewall/policy`
```
#SOURCE DEST    POLICY          LOG     LIMIT:          CONNLIMIT:
#                               LEVEL   BURST           MASK
fw      all     ACCEPT
wifi    wired   ACCEPT          info
wifi    fw      ACCEPT          info
wired   fw      DROP            info
wired   wifi    DROP
```

#### Rules
* Define exceptions to the rules.
* File: `/etc/shorewall/rules`
```
#ACTION         SOURCE          DEST            PROTO   DEST    SOURCE          ORIGINAL        RATE            USER/   MARK    CONNLIMIT       TIME            HEADE$
#                                                       PORT    PORT(S)         DEST            LIMIT           GROUP
?SECTION ALL
?SECTION ESTABLISHED
?SECTION RELATED
?SECTION INVALID
?SECTION UNTRACKED
?SECTION NEW
Invalid(DROP)   wired           $FW             tcp
SSH(ACCEPT)     wired           $FW
Ping(ACCEPT)    wired           $FW
```




## Fog Computing <a name="fog_computing"></a>
apache nifi
