# wifi connection on rasbian/debian (WPA Supplicant)
### Connect to Wi-Fi From Terminal on Debian 11/10 with WPA Supplicant


This tutorial is going to show you how to connect to Wi-Fi network from the command line on Debian 11/10 server and desktop using wpa_supplicant, which is an implementation of the supplicant component for the WPA protocol. A supplicant in wireless LAN is client software installed on end-user’s computer that needs to be authenticated in order to join a network.

Please note that you will need to install the wpa_supplicant software before connecting to Wi-Fi, so you need to connect to Wired Ethernet first, which is done for just one time. If you don’t like this method, please don’t be mad at me. Maybe someday Debian will ship wpa_supplicant out of the box.

Step 1: Find The Name of Your Wireless Interface And Wireless Network

### Run iwconfig command to find the name of your wireless interface.

`iwconfig`

wlan0 is a common name for a wireless network interface on Linux systems. On systemd-based Linux distros, you might have a wireless interface named wlp4s0. wlp4s0 are used for usb devices for consitancey for being used in diffrent port and can have more then 1 usb can be present at same time. you can verfy this by using:</br>
`dmesg | grep "renam"`</br> 
`r8188eu 1-1.2:1.0 wlu1u2: renamed from wlan0`

### prepare to connect 
As you can see, the wireless interface isn’t associated with any access point right now. Then run the following command to bring up the wireless interface.

`sudo ip link set dev wlp4s0 up`

If you encounter the following error,

`RTNETLINK answers: Operation not possible due to RF-kill`

you need to unblock Wi-Fi with the following command.

`sudo rfkill unblock wifi`

