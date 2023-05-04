---
layout: default
title: Routing
parent: Instructions
nav_order: 2
---
<h1>Routing</h1>
Once you have a solar server set up at home, you need to connect it to the wider internet and an URL. For that you need to follow several steps:
1. You need to have a static IP adress from your internet provider.
2. You change the DNS for your URL to point to that static adress.
3. You need to configure the router to allow port forwarding or set up the server as a DMZ. A DMZ (demilitiarized zone) exposes all ports of a specific local adress, then the ports can be closed at the device using a firewall.
![alt text](assets/images/dmz.jpg)
This works fine as long as the IP adress is static. If IP adress is not static a solution can be to update DNS dynamically using a cronjob and dynamic DNS and DynDNS protocol.
