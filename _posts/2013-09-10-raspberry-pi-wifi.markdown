---
layout: post
title:  "Raspberry Pi Wifi"
date:   2013-09-10 18:00:00
categories: debian raspbian rpi wifi
comments: true
---

The first thing I did when I got my Raspberry Pi was to buy a wifi dongle for
it so that I could put it anywhere in the house without the need for a wired
connection.  For this purpose I bought the [Edimax EW-7811Un][edimax] 802.11n
wifi dongle.  It was cheap and the docs said it was supported so I bought it.
Tonight I finally had a chance to set up the wifi dongle and I thought I'd
share my process and tips for setting it up.

I spent at least an hour trying to set up my wifi dongle before I realized that
my biggest problem was the version of Raspbian Wheezy that I had imaged on
my 4GM SD card.  I got the card at PyCon 2013 and have had a few issues with it
that I'd just dealt with.  I decided it made more sense to just re-image the
card and that fixed almost all of my up-front problems.  You can get the lastest
Raspbian Wheezy image from the [Raspberry Pi Downloads][rpi_downloads] page.
If, like me, you're on OSX then you can use the [OSX Installer][osx_installer]
script that I've linked to.

Before you go any further make sure that your dongle is recognized by your
system with commands <code>lsusb</code> and <code>lsmod</code>.

I started setting up the wifi dongle using a great [Wifi Tutorial][adafruit_wifi]
from Adafruit.  I highly recommend following along there and she has lots of
other available resources.  For me that got me started but I needed a few more
things to get going.  Also, there shouldn't be any need to restart your RPi
if you follow these instructions.  Generally if you have to restart a linux box
you're probably doing something wrong.  There's always a command to get things
going or to reset things.

The first file to set up is /etc/network/interfaces:

{% highlight bash %}
auto lo
                               
iface lo inet loopback         
iface eth0 inet dhcp

allow-hotplug wlan0            
iface wlan0 inet manual        
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
{% endhighlight %}

The second file to set up is /etc/wpa_supplicant/wpa_supplicant.conf

{% highlight bash %}
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1                
ap_scan=1
network={
    scan_ssid=1                
    key_mgmt=WPA-PSK           
    proto=RSN WPA
    pairwise=CCMP TKIP
    group=CCMP TKIP
    ssid="YOUR-NETWORK-SSID"
    #psk="YOUR-NETWORK-PASSWORD"
    psk=YOUR-UNQUOTED-HASHED-NETWORK-PASSWORD
}
{% endhighlight %}

The second file was harder to set up without help.  You should be able to
get started with this command:

{% highlight bash %}
$ sudo wpa_passphrase YOUR-NETWORK-SSID YOUR-NETWORK-PASSWORD > /etc/wpa_supplicant/wpa_supplicant.conf
{% endhighlight %}

This creates the last three lines of your <code>network</code>.  Then you can
fill in the remaining lines using:

{% highlight bash %}
$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
{% endhighlight %}

With these files set up all you have to do is restart the network and test that
your configuration works:

{% highlight bash %}
$ ifconfig wlan0
$ iwconfig wlan0
$ sudo iwlist wlan0 scan
$ sudo ifdown wlan0; sudo ifup wlan0
$ sudo dhclient wlan0
$ sudo iwconfig wlan0 essid YOUR-NETWORK-SSID
$ ifconfig
$ sudo apt-get update
{% endhighlight %}

You should see an assigned ip address in the <code>ifconfig</code> step and
the <code>apt-get update</code> should work without any errors.  Now you're
all set up!

[edimax]: http://www.amazon.com/EW-7811UN-IEEE-802-11n-draft-USB/dp/B005CLMJLU
[rpi_downloads]: http://www.raspberrypi.org/downloads
[osx_installer]: https://github.com/RayViljoen/Raspberry-PI-SD-Installer-OS-X
[adafruit_wifi]: http://learn.adafruit.com/adafruits-raspberry-pi-lesson-3-network-setup/
