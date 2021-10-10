---
layout: post
title: Home-grown Wireguard VPN for your own domain (with DDNS)
---

<abbr title="Virtual Private Network">VPN</abbr>s are everywhere, yep. They are flagged all over the place as some sort of pro-privacy marketing tactic. Sadly, this is the only side that is communicated to people about the beauty of them. For once, can't I have my own one and not have to constantly fork money over to these people? 

In this post, I show how I used a raspberry pi to become a Wireguard VPN server that also performs dynamic-DNS so it is always accessible whenever (and whenever) I am. Hopefully, you should be tempted to do the same.


**This post is absolutely not sponsored by NordVPN.**

<!-- More -->

-----

## The Briefest Definition
When you go browse to some place on the internet, whether this be some website or a peer on a network, it is carried through the beautiful <a href="https://datatracker.ietf.org/doc/html/rfc791" target="_blank">internet protocol.</a> Whether it's HTTP or HTTPS, the usual model comes as *client* and *server*. With all the mumbo-jumbo of routing, the data essentially flows through the internet; a big fat cloud where what you want is on the other side. 

<img class="container" width="100%" src="/assets/media/2021/10/05/vpn-1.png" alt="Client-Server connection diagram through the internet both HTTP and with VPN">

With our two entities, `Client = Alice` & `Server = Bob`, the internet can be rife of middle-people getting in the way causing all sorts of harm, snooping (you name it). HTTPS can certainly provide some confidentiality, but isn't a full solution. Why wade through a sewer? It would make much more sense to make a nice clean tunnel. That's the VPN: **an encrypted tunnel to wade through the sewer that is the internet.**

## Benefits

1. Localisation - From this encrypted tunnel, Alice can be apart of the network that Bob is also *(there's your virtual)* whilst seemingly within a stones throw. Access to devices within the network can be reached thanks to this VPN, whilst sealing off others. 

2. Obfuscation - Browsing the internet via Bob shall give websites the IP of Bob, which then gets relayed back to Alice. The diagram below shows the flow starting from Alice to Bob, then Charlie and then all the way back.

<img class="container" width="100%" src="/assets/media/2021/10/05/vpn-2.png" alt="Using a VPN to relay to a website data flow diagram">


## Downfalls

1. Authentication - In this case, the VPN public/private keys serve as credentials for authentication. If someone were to get a hold of a configuration file, confidentiality is compromised. Encryption *isn't* authentication.

2. Always middle-men - If the key exchange is passed haphazardly, a snoopy malicious person could grab the keys in the process and persistently insert themselves. 


## Setup

Enough with the chit-chat, let's get to work. Downloading pivpn is done with a nice easy command:

```shell
curl -L https://install.pivpn.io | bash
```

From the prompts, I use a static IP for the pi and chose the Wireguard protocol instead of OpenVPN as it is more lightweight and handshakes are done in a few packets (it's speedier). Other recommended port settings and TCP/UDP protocol was left as default


### Linking own domain
On the page of DNS provider, a subdomain of your own domain can help deal with the problem of your public IP changing over time. Thus, people trying to reach the VPN server will go to a url, say `vpn.yourdomain.com` instead of `122.134.170.176` for example. Following the rest of the installation and reboot, the other half of the connection needs to be fixed.


### Cloudflare DDNS
<a href="https://www.cloudflare.com" target="_blank">Cloudflare</a> can allow some *preeeeeeety cool* things, but the single use item where it can help with the VPN is through what is called *Dynamic Domain Name Service* (DDNS). It tells the internet the current IP of a domain, much like a phone-book has names to phone numbers. In this analogy, the phonebook can be updated as much as we need. Thanks to an awesome project on GitHub, the DDNS updater can be downloaded on the pi with:

```shell
git clone https://github.com/K0p1-Git/cloudflare-ddns-updater.git
```

Logging into cloudflare and adding your site can be quite tedious (<a href="https://support.cloudflare.com/hc/en-us/articles/201720164-Creating-a-Cloudflare-account-and-adding-a-website" target="_blank">here</a> is a great guide for setting one up for the first time), but once that is done the `Global API Key` is what gives the VPN the rights to update the `vpn.yourdomain.com` entry. We simply edit the file dubbed `cloudflare-template.sh` with the following items:

```shell
auth_email="<you@yourdomain.com>"
auth_method="global"
auth_key="<global key>"
zone_identifier=""			# Can be found in the "Overview" tab of your domain
record_name="vpn.yourdomain.com"
proxy=true					# Can be true or false
```

Note `<you@yourdomain.com>` and `<global key>` are items from your cloudflare account. Once the template is filled with the correct information. Running this in crontab will mean the pi will **constantly** check if it's IP has changed, if so, tell the internet where to find it. If not, we chill. Say if `cloudflare-template.sh` is kept under the path `/home/pi/Documents/cloudflare-template.sh` we could run the following:

```shell
sudo crontab -e
```

Then appending the following to the bottom of the file:
```shell
*/5 * * * * /bin/bash /home/pi/Documents/cloudflare-template.sh
```

The stars **\*** specify *how often* we want the script to run. It goes minutes, hours, days, month, day of week. Here, we want our script to update the DNS to check every 5 minutes.

### The Payoff
From all these various products, sign-ups and installs. A simple raspberry pi can become a surprisingly powerful VPN to let yourself back into your home network or browse the web safely when public WiFi is unencrypted and filled with potential bad actors.


----
**TL;DR:** VPN for cheap with good speed, gives you access to home network as well (if you want it to).




