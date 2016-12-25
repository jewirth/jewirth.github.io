---
layout: post
title: "Raspberry Pi: Setting up a Wifi connection without keyboard, mouse and monitor connected"
date: 2013-07-21 20:00:00 +0200
---
Unlike Ethernet, which is configured to automatically use DHCP on the RPi (Raspberry Pi) by default, setting up a Wifi connection can be difficult to some users. First of all, the RPi can not and should not guess the name of your Wifi network and it surely can't do so with your Wifi passphrase. I hope you really do not want this.



Now, setting up Wifi can be simple by using the graphical interface and the provided application on the desktop named "Wifi Config". But this requires you to have a HDMI display and a USB keyboard/mouse ready to be connected. If not, then you can attach your RPi to your router via an Ethernet cable, then connect via ssh to the command line and then configure your Wifi remotely. Below, this procedure is described in more detail.

### Step 1 - Requirements
<ul><li>Raspberry Pi with <a href="http://www.raspberrypi.org/downloads">Raspbian</a> (tested with wheezy-Release) </li>
<li>Ethernet cable</li>
<li>Router</li>
<li>A computer which is already connected to your router</li>
<li>SSH client software (for Windows systems you can use <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/">PuTTY</a>, MacOS and GNU/Linux systems are already equipped with such a client software)</li></ul>

### Step 2 - Set Up
Connect your RPi to the router using the Ethernet cable and power on the RPi.

### Step 3 - Get the IP address of your RPi
Boot it up and figure out the IP address of your RPi. Since we don't have anything but power and ethernet connected to the RPi we can do so by checking the administration utilities of our router or by doing a <a href="http://en.wikipedia.org/wiki/Ping_sweep">ping sweep</a>. Another possibility <a href="http://blog.remibergsma.com/2013/05/03/howto-discover-the-ip-address-of-a-raspberry-pi-on-dhcp/">explained by Remi Bergsma</a> is to scan for the specific range of ethernet addresses used by the RPi boards.

### Step 4 - Connect via SSH
Use the SSH Client software to connect to your RPi. On MacOS X and GNU/Linux systems you can do so by opening the Terminal application and entering the command `ssh pi@192.168.1.101`. In this case the IP address is 192.168.1.101 and the username/password are the default ones (username: pi password: raspberry). The password will be requested by the SSH client while the connection is established.

    macy:~ jens$ ssh pi@192.168.1.101
    pi@192.168.1.101's password:

### Step 5 - Wifi Settings

Now you should have access to the command line of your RPi. Type `sudo nano /etc/network/interfaces` to edit the network interface configuration file. You should see something similar like this:

{% highlight plain %}
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

iface default inet dhcp
{% endhighlight %}


We need to put the information about our Wifi network name and it's WPA passphrase into this configuration file. Edit your file analogue to this example:

{% highlight plain %}
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0
auto wlan0
iface wlan0 inet dhcp
 wpa-ssid "MyHomeNetwork"
 wpa-psk "YourWpaPassphraseGoesHere"
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

iface default inet dhcp
{% endhighlight %}

Save the file and close it.

### Step 6 - Verify
Verify your changes by rebooting the system with
<PRE>sudo shutdown -r now</PRE>
On success, the RPi should automatically connect to your Wifi network on startup. Please note that your router's DHCP server won't assign the same IP address to the Wifi connection of the RPi as it did before with the ethernet connection. Therefore, repeat Step 3.

If you figured out the new IP address you should be able to talk to your RPi as usual (SSH, ping, and so on).

