# wifi connection on rasbian/debian (WPA Supplicant)
### Connect to Wi-Fi From Terminal on Debian 11/10 with WPA Supplicant


This tutorial is going to show you how to connect to Wi-Fi network from the command line on Debian 11/10 server and desktop using wpa_supplicant, which is an implementation of the supplicant component for the WPA protocol. A supplicant in wireless LAN is client software installed on end-user’s computer that needs to be authenticated in order to join a network.

Please note that you will need to install the wpa_supplicant software before connecting to Wi-Fi, so you need to connect to Wired Ethernet first, which is done for just one time. If you don’t like this method, please don’t be mad at me. Maybe someday Debian will ship wpa_supplicant out of the box.


## Step 1: Find The Name of Your Wireless Interface And Wireless Network

### Run iwconfig command to find the name of your wireless interface.

`iwconfig`

wlan0 is a common name for a wireless network interface on Linux systems. On systemd-based Linux distros, you might have a wireless interface named wlu1u2. wlu1u2 are used for usb devices for consitancey for being used in diffrent port and can have more then 1 usb can be present at same time. you can verfy this by using:
    
    dmesg | grep "renam"
    r8188eu 1-1.2:1.0 wlu1u2: renamed from wlan0

### prepare to connect 
As you can see, the wireless interface isn’t associated with any access point right now. Then run the following command to bring up the wireless interface.

`sudo ip link set dev wlu1u2 up`

If you encounter the following error,

`RTNETLINK answers: Operation not possible due to RF-kill`

you need to unblock Wi-Fi with the following command.

`sudo rfkill unblock wifi`

### list network present nearby 
Next, find your wireless network name by scanning nearby networks with the command below. Replace wlu1u2 with your own wireless interface name. ESSID is the network name identifier.

`sudo iwlist wlu1u2 scan | grep ESSID`


## Step 2: Connect to Wi-Fi Network With WPA_Supplicant

Now install wpa_supplicant on Debian 11/10 from the default software repository.

`sudo apt install wpasupplicant`

We need to create a file named wpa_supplicant.conf using the wpa_passphrase utility. wpa_supplicant.conf is the configuration file describing all networks that the user wants the computer to connect to. Run the following command to create this file. Replace ESSID (network name) and Wi-Fi passphrase with your own.

`wpa_passphrase your-ESSID your-wifi-passphrase | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf`

If your ESSID contains whitespace such as (linuxbabe WiFi), you need to wrap the ESSID with double-quotes ("linuxbabe WiFi") in the above command.

The output of wpa_passphrase command will be piped to tee, and then written to the /etc/wpa_supplicant/wpa_supplicant.conf file. Now use the following command to connect your wireless card to the wireless access point.

sudo wpa_supplicant -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlu1u2

The following output indicates your wireless card is successfully connected to an access point.

    Successfully initialized wpa_supplicant
    wlu1u2: SME: Trying to authenticate with c5:4a:21:53:ac:eb (SSID='CMCC-11802' freq=2437 MHz)
    wlu1u2: Trying to associate with c5:4a:21:53:ac:eb (SSID='CMCC-11802' freq=2437 MHz)
    wlu1u2: Associated with c5:4a:21:53:ac:eb
    wlu1u2: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
    wlu1u2: WPA: Key negotiation completed with c5:4a:21:53:ac:eb [PTK=CCMP GTK=CCMP]
    wlu1u2: CTRL-EVENT-CONNECTED - Connection to c5:4a:21:53:ac:eb completed [id=0 id_str=]

### desktop debian confict 
Note that if you are using Debian desktop edition, then you need to stop Network Manager with the following command, otherwise it will cause a connection problem when using wpa_supplicant.

`sudo systemctl stop NetworkManager`

And disable NetworkManager auto-start at boot time by executing the following command.

`sudo systemctl disable NetworkManager-wait-online NetworkManager-dispatcher NetworkManager`

By default, wpa_supplicant runs in the foreground. If the connection is completed, then open up another terminal window and run

`iwconfig`

You can see that the wireless interface is now associated with an access point.

You can press CTRL+C to stop the current wpa_supplicant process and run it in the background by adding the -B flag.

`sudo wpa_supplicant -B -c /etc/wpa_supplicant.conf -i wlu1u2`

Although we’re authenticated and connected to a wireless network, we don’t have an IP address yet. To obtain a private IP address from DHCP server, use the following command:

`sudo dhclient wlu1u2`

Now your wireless interface has a private IP address, which can be shown with:

`ip addr show wlu1u2`

you are done here 


## Step 3: Auto-Connect At System Boot Time

To automatically connect to wireless network at boot time, we need to edit the wpa_supplicant.service file. It’s a good idea to copy the file from /lib/systemd/system/ directory to /etc/systemd/system/ directory, then edit the file content, because we don’t want a newer version of wpa_supplicant to override our modifications.

`sudo cp /lib/systemd/system/wpa_supplicant.service /etc/systemd/system/wpa_supplicant.service`

Edit the file with a command-line text editor, such as Nano.

`sudo nano /etc/systemd/system/wpa_supplicant.service`

Find the following line.

`ExecStart=/sbin/wpa_supplicant -u -s -O /run/wpa_supplicant`

Change it to the following. Here we added the configuration file and the wireless interface name to the ExecStart command.

`ExecStart=/sbin/wpa_supplicant -u -s -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlu1u2`

or if you are using old driver

`ExecStart=/sbin/wpa_supplicant -B -D wext -c /etc/wpa_supplicant/wpa_supplicant.conf -i wlx10feed2673c7`

It’s recommended to always try to restart wpa_supplicant when failure is detected. Add the following right below the ExecStart line.

`Restart=always`

Save and close the file. (To save a file in Nano text editor, press Ctrl+O, then press Enter to confirm. To exit, press Ctrl+X.) Then reload systemd.

`sudo systemctl daemon-reload`

Enable wpa_supplicant service to start at boot time.

`sudo systemctl enable wpa_supplicant.service`

We also need to start dhclient at boot time to obtain a private IP address from DHCP server. This can be achieved by creating a systemd service unit for dhclient.

`sudo nano /etc/systemd/system/dhclient.service`

Put the following text into the file.
    
    [Unit]
    Description= DHCP Client
    Before=network.target
    After=wpa_supplicant.service
    
    [Service]
    Type=forking
    ExecStart=/sbin/dhclient wlu1u2 -v
    ExecStop=/sbin/dhclient wlu1u2 -r
    Restart=always
     
    [Install]
    WantedBy=multi-user.target

Save and close the file. Then enable this service.

`sudo systemctl enable dhclient.service`





read:
https://unix.stackexchange.com/questions/483678/debian-connect-to-wifi-automatically-when-in-range

ref1: https://wiki.debian.org/WiFi/HowToUse#Manual

ref2?: https://www.linuxbabe.com/debian/connect-to-wi-fi-from-terminal-on-debian-wpa-supplicant

ref3: https://wiki.debian.org/WiFi/HowToUse#Manual



### Simple methord (wifi-menu)
dont beark your head, use this
```
wifi-menu -o
cd /etc/netctl/
netctl list
netctl reenable <profile-listed in /etc/netctl>
```
Note:<br/>

`wifi-menu` dont come as in built tools, you need to install it, you can intall it using LAN / RNDIS (android)

[ref1](https://unix.stackexchange.com/questions/276844/how-do-i-automatically-execute-netctl-start-tq84-wifi-on-bootup)





