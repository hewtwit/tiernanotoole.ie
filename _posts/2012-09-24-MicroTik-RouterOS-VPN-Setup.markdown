---
layout: post
title: MicroTik RouterOS VPN Setup
tags:
- RouterOS
- Networking
---
I have been running a [MicroTik RouterBoard][1] in the house for a couple of months now (the [RB750G][2]) and I am very much loving the thing. But one thing you may need to do is setup VPN connections... Here are some tips on how to create a VPN Server and Client on your RouterBoard.

Client Setup
------------

to setup a client, you need to do the following:

* /interface l2tp-client add name="vpn" max-mtu=1460 max-mru=1460 mrru=disabled connect-to=servername user="username" password="password" profile=default-encryption add-default-route=no dial-on-demand=no allow=pap,chap,mschap1,mschap2 
* /interface l2tp-client enable [/interface l2tp-client find name ="vpn"]
* /ip firewall mangle add chain=prerouting action=mark-routing new-routing-mark=vpn passthrough=yes src-address=networkaddress/24 dst-address=!networkaddress/24 
* /ip route add dst-address=0.0.0.0/0 gateway=vpn gateway-status=vpn distance=1 scope=30 target-scope=10 routing-mark=vpn
* /ip firewall nat add chain=srcnat action=masquerade out-interface=vpn

What does that all do? the first line creates an l2tp-client interface, pointing at "servername" with the username and password set. encruption, etc is enabled... Line 2 then enables the client. Line 3 sets all traffic comming from networkaddress/24 (for example, 192.168.0.1/24) to be sent though the VPN. any traffic going into networkaddress (same example) is not sent though the VPN. Line 4 creates a gateway, for all addresses (0.0.0.0/0) to use the VPN address. finally, NAT Masquerading is enabled on the VPN interface. 

there are more advanced things you can set above... some examples i can think of are as follows:


* There should be no good reason to limit the amount of VPN connections you have... in theory, you could have multiple...
* for the mangle rule, set the src-address to a single machine in your network. that way, it gets VPN only connections. Also, you could set the dst-address to a single address or network to send only traffic going to a given server though the VPN... Example would be Netflix US traffic to a US VPN server, BBC iPlayer traffic to a UK VPN, etc.
* the example above uses L2TP, but PPTP, SSTP and OpenVPN are also available.


Server Setup
-------------
As mentioned above, L2TP, PPTP, SSTP and OpenVPN servers are available on RouterOs. Detals on setting them up are availabe on MicroTik's Wiki at the following locations:

* [OpenVPN][3]
* [PPTP][4]
* [L2TP][5]
* [SSTP][6]

[1]:http://www.routerboard.com
[2]:http://www.routerboard.com/RB750G
[3]:http://wiki.mikrotik.com/wiki/Manual:Interface/OVPN
[4]:http://wiki.mikrotik.com/wiki/Manual:Interface/PPTP
[5]:http://wiki.mikrotik.com/wiki/Manual:Interface/L2TP
[6]:http://wiki.mikrotik.com/wiki/Manual:Interface/SSTP
