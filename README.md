This tutorial is primarily aimed at PS4 jailbreak users, but it can be used for other consoles, too, and even for adblocking.

Currently many PS4 jailbreak users are in need of blocking Sony's firmware update servers. Public DNS servers made for this purpose turned out to be unreliable in the long term, so here is a short, simple tutorial that shows how to use a Raspberry Pi (or a similar device) to run a blocking DNS server in your own local network.
Keep in mind that there are many different possible network configurations out there, and this quick tutorial only handles one of the most common home network situations where you have exactly one router where all your devices are directly connected to. The tutorial cannot provide support for more complicated setups. With that said, let's begin:

# Prerequisites

- Raspberry Pi (or similar device), running a Debian-based Linux distribution and connected to a router via cable connection.
- The device and its Internet access are already set up, and you have basic knowledge of accessing and operating the Linux command line.
- PS4 (or other device) that is supposed to use the DNS server, connected to the router via cable or Wi-Fi.

The Raspberry Pi (or similar device) must be given a static IP address, which can be set up as described here: https://www.tomshardware.com/how-to/static-ip-raspberry-pi.

# 1. Install dnsmasq (and the text editor used in this tutorial)

    # sudo apt update && sudo apt install dnsmasq nano

# 2. Create a dnsmasq configuration file

    $ sudo rm /etc/dnsmasq.conf
    $ sudo nano /etc/dnsmasq.conf

Paste this content into the editor:

```
no-resolv
no-hosts
server=xxx.xxx.xxx.xxx
server=xxx.xxx.xxx.xxx

interface=eth0
domain-needed
bogus-priv
min-port=4096
cache-size=500
local-ttl=36000

#log-queries	
#log-facility=/var/log/dnsmasq.log

addn-hosts=/etc/ps4blocklist
```

Replace "server=xxx.xxx.xxx.xxx" with the (often two) DNS server IP addresses you were given by your ISP when you received your contract papers. If you don't know the IP addresses, use public ones, like Google's 8.8.8.8 or Cloudflare's 1.1.1.1. For some routers, you could also use the router's local network IP address.

You can edit other values if you want to. But you need to understand what they do. The official dnsmasq homepage describes all available options in detail: https://thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html.  
It is assumed that the Raspberry Pi (or similar device) is connected via Ethernet cable and that the Ethernet connection is called "eth0". If on your device it has a different name, the line "interface=..." must be adjusted accordingly.

Press Ctrl-x and confirm with "y" to save the file.

# 3. Create a block list

Now create the PS4 block file, which in this example is called "/etc/ps4blocklist". You could call it any way you want and save it in a different directory, but if you do, don't forget to adjust the line "addn-hosts=..." in dnsmasq.conf.

    $ sudo nano /etc/ps4blocklist

Paste a ready-made hosts file that contains all possible Sony update servers, for example this one: https://github.com/phoanglong/ps4-dns-block. This list looks complete to me, but if you spot any missing servers, please let me (or better, the list maintainer) know as soon as possible. There may be countries that use exotic servers that are missing. I don't want to take responsibility if an update should make it through, so make sure the list you use is complete. If I receive reports of a better list, I will update the link. The list will block firmware updates, but not game updates.

Note thay by using multiple "addn-hosts=" lines, you could add many more block lists that block other stuff, too (e.g. adblock hosts lists, block lists for other consoles, etc.)

Again, save the file by pressing Ctrl-x and confirming with "y".

# 4. Start dnsmasq

Make sure dnsmasq is started at boot:

    $ sudo systemctl enable dnsmasq

Each time the configuration file is changed or a block list is added/removed, dnsmasq must be restarted for the changes to take effect:

    $ sudo systemctl restart dnsmasq


# 5. Set up the PS4 (and/or other devices) to use the DNS server

In the device's custom network settings, enter the Raspberry Pi's static IP address twice, for both primary and secondary DNS entries. You may need to start the Internet configuration procedure once again for the necessary dialogs to show up. On the PS4, it's done by choosing "Set Up Internet Connection" and then "Custom".

# Optional: view log output

If you want see what happens while dnsmasq is running, you can remove the "#" before the two "#log-..." entries. If your device uses an SD card, make sure the log file location is inside the RAM (on a Raspberry Pi, /var/log usually is). Restart dnsmasq (see above) and then you can view the log file "live" by entering

    $ sudo tail -f /var/log/dnsmasq.log

