# Running H2R Graphics V2 output on a Raspberry Pi in kiosk mode
 
## Overview

This article gives a step-by-step guide to getting an [H2R Graphics V2](https://h2r.graphics/) output display appearing on a Raspberry Pi automatically at boot. Some of the steps outlined here are optional and it's assumed you are starting from a clean Pi using the latest image.

Whilst these instructions have been developed for H2R Graphics specifically, they will work for any URL you want to open in kiosk (i.e. fullscreen) mode when a Raspberry Pi boots up.

## Step-by-Step

The left-most HDMI port on the Pi should be used for your output.

Download the latest Raspberry Pi OS and write it to an SD Card

On the Pi, change the default password and configure some options:

1. Raspberry Pi logo | Preferences | Raspberry Pi Configuration
2. Change the password
3. Change hostname as required
4. Set “Network at Boot” to “Wait for network”
5. Switch to “Display” tab
6. Set “Screen Blanking” to “Disabled”
7. Set “Headless Resolution” to “1920×1080” (this is a “just in case” thing)
8. Switch to “Interfaces” and enable SSH and VNC as required
9. Click “OK”

If prompted to reboot, do this.

If a wireless network is required, set this up

Download the latest updates and install them – reboot when finished (icon indicates updates in the top right) – repeat if required

To set a static IP (optional), enter the following in a terminal:

```bash
sudo nano /etc/dhcpcd.conf
```

Add the following lines with your appropriate values:

```
interface NETWORK
static ip_address=STATIC_IP/24
static routers=ROUTER_IP
static domain_name_servers=DNS_IP
```

Where:

`NETWORK` = your network connection type: eth0 (Ethernet) or wlan0 (wireless)
`STATIC_IP` = the static IP address you want to set for the Raspberry Pi
`ROUTER_IP` = the gateway IP address for your router on the local network
`DNS_IP` = the DNS IP address (typically the same as your router's gateway address)

Reboot the Pi and don't forget to add a DNS entry if you wish to identify your Pi by IP.

Run the following command:

```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

Add the following line:

```
@/usr/bin/chromium-browser --kiosk  --disable-restore-session-state http://h2r.machine.name:4001/output/ABCD
```

Replace `h2r.machine.name` with the name of the machine running H2R Graphics V2. Note that the host machine may need firewall changes to allow remote access.

Reboot and confirm

Note, to quit kiosk mode, use Ctrl+F4.

## Dual Screens

If you have the inputs to spare on you switcher, you can output in dual screen mode. The three most obvious options are key & fill, preview and output, or output 1 & output 2. Below are the lines to add to the autostart file instead of the above.

### Preview & Output

```
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/0" --window-position=0,0 http://h2r.machine.name:4001/preview/ABCD
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/1" --window-position=1920,0 http://h2r.machine.name:4001/output/ABCD
```

### Key & Fill

```
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/0" --window-position=0,0 http://h2r.machine.name:4001/output/ABCD/?bg=%23000&key=true
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/1" --window-position=1920,0 http://h2r.machine.name:4001/output/ABCD/?bg=%23000
```

### Output 1 and Output 2

```
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/0" --window-position=0,0 http://h2r.machine.name:4001/preview/ABCD
@/usr/bin/chromium-browser --kiosk --disable-restore-session-state --user-data-dir="/home/pi/Documents/Profiles/1" --window-position=1920,0 http://h2r.machine.name:4001/output/ABCD/2
```

Having tried this on a Raspberry Pi 4 Model B+ (8GB) running in Key & Fill mode, I'd say it works but it's on the edge of being smooth, especially when you add a few graphics.