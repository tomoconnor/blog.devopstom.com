---
title: "VPN Technologies: A Primer"
date: "2014-01-12T19:59:58.000Z"
description: "Know your IPsec from your OpenVPN"
---
**What does VPN stand for?**

Virtual Private Network.  Moving on... 

**What is a VPN?**

A VPN is a mechanism to extend a private network (like your LAN [Local Area Network]) across a public network (like the Internet).  The upshot of this is, that you can connect two separate computers, each on their own LAN, across a VPN so that they appear to be on the same network; which, in a sense, they are. 

Typically, a VPN will include some form of encryption, so that the traffic traversing the public network isn't identifiable as being part of either of the interconnected private networks.

 

**What kinds of VPN are there?**

Well, that's a big question.  Basically, there's two main types.  

The ones that connect PCs to LANs (Like your laptop to a Work / Corporate network). (**Remote-Access**)

The ones that connect two LANs together (Like between a business and their supplier). (**Site-to-Site (or LAN-to-LAN)**)
 

There's more to it, though.

We can classify a VPN by the type of encryption and protocol used when the traffic traverses the public network, and also by the [OSI network layer](https://en.wikipedia.org/wiki/OSI_model) they present at. 


## Main types:

**Internet Protocol Security (IPsec)** - My personal favourite. 

**Transport Layer Security / Secure Sockets Layer (TLS/SSL)** Commonly referred to as SSL VPNs.  

**Datagram Transport Layer Security (DTLS)** - used in Cisco AnyConnect and OpenConnect ()

**Microsoft Point-to-Point Encryption (MPPE)** - provides encryption over PPTP connections.

**Microsoft Secure Socket Tunneling Protocol (SSTP)** - tunnels PPTP or L2TP traffic over a SSL connection.

**Secure Shell (SSH)** - OpenSSH provides a VPN mechanism to forward network connections over a SSH connection.

It occurs to me that there's two types of site-to-site VPN also, Policy Based, and Routed.

 

**Policy Based** VPNs are clever.  They have access lists, comprised of rules to match traffic on the LAN that should be sent over the VPN.  This might be something like "Match all VoIP traffic, and send it over the VPN to x.x.x.x", or "Match anything that looks like HTTP or HTTPS, and send it to y.y.y.y".

This is a bit like an implementation of [Split Tunneling](https://en.wikipedia.org/wiki/Split_tunneling) . 

**Routed VPNs** aren't anywhere near as clever, and basically replace the default route for your LAN with a path across the VPN, so that all traffic not destined for the LAN goes out across the VPN: "If destination is not on the LAN, send it to the VPN address", and so on,

 

There's a couple of protocols which *could* be described as a VPN, however, unlike protocols such as IPsec, offer no encryption. 

Firstly, **Point-to-Point Tunneling Protocol (PPTP)** - First proposed as an RFC in 1999, now widely considered as cryptographically broken and, hence, insecure.  PPTP leverages a Generic Routing Encapsulation (GRE) tunnel, and a non-standard packet format.  The GRE tunnel between the two networks carries Point-to-Point Protocol (PPP) traffic, which can theoretically encapsulate IPX as well as IP traffic - although, I'm willing to bet nobody's actually using IPX over PPTP..

PPTP is, however, well supported, with native clients on iOS, OSX, Windows and Android.   On the other hand, it's about as secure as a net curtain is against accidental nakedness exposure.  I wouldn't recommend it under any circumstances.

 

**Microsoft Point-to-Point Encryption (MPPE)** is probably worth a brief mention here.  It's a mechanism for encrypting traffic over a PPTP connection.  It uses RSA RC4 (*Danger Will Robinson*), and 40, 56 and 128 bit keys. 

I don't think I've found a use for it so far, so.. moving on..

 

There's also **Layer 2 Tunneling Protocol (L2TP)**:

L2TP alone does *not* provide authentication. It's merely an advanced tunnel (somewhat based on PPTP), which allows more support for transit over non IP networks (such as Frame Relay and ATM).

Internet Protocol Security (IPsec) is often used on top of L2TP to provide encryption, confidentiality and integrity. This is commonly known as L2TP/IPsec.

L2TP is common as "carrier-grade" tunneling when you have a reseller of a broadband (typically ADSL), service using somebody else's network.

Without the encryption (and so on) provided by IPsec, L2TP isn't *really* a VPN technology in it's own right, more a protocol used to enable VPNs to work.

**SSL VPNs**

Traditional SSL/TLS VPNs (excluding, for the moment, OpenVPN), utilise TLS/SSL encryption to provide either a portal (common for remote access to corporate web-based IT resources - like Webmail...), or a SSL encrypted tunnel, providing end-to-end security encrypted by the TLS/SSL protocol suite.

SSL Portals are commonly used for remote access to corporate IT resources, such as intranets (technically, this makes them Extranets), webmail, or similar applications.  The user accesses the portal via their web browser, and typically provides some form of 2-factor authentication, such as a RSA SecureID token's password in addition to their standard authentication information.

SSL Tunnels allow an initial connection via a web browser to open further, secured connections to remote resources via the use of Java Applets, or ActiveX controls.  This is commonly used to provide secure remote access to remote desktop sessions. 

Typically, SSL VPNs will be solely browser based, making them tricky to implement for distinctly non-browser protocols.

Personally, I don't like SSL VPNs, as they're no substitute for a proper, encrypted tunnel protected by IPsec, and they often use questionable key lengths for the SSL connection itself, which often makes me wonder whether it's worth encrypting at all.

**OpenVPN**

[OpenVPN](https://openvpn.net/community/) is cool.  It's open-source, and supported on pretty much every platform I can think of.. There's even an iOS client somewhere.

OpenVPN uses OpenSSL to provide encryption of the tunnel, and control for the tunnel.  In terms of authentication, there's a choice of pre-shared keys (a password known by both the client, and the VPN endpoint), a SSL x.509 certificate, and username and password.  It's also possible to combine them to require both a valid certificate and a username and password, if desired.

As OpenVPN runs on top of Linux, it's easy to deploy, and very configurable.  I've actually implemented it in a number of environments in the past, mostly for remote-access scenarios, although it is possible to use it in a site-to-site capability.

Clients are available for almost every operating sysem you can think of (however, I can't think of a way to connect it directly to a Cisco router).

 

On to my favourite VPN Technology:

**Internet Protocol Security (IPSec)**

Can be a right pain in the arse to configure, however, once it's up, it's typically rock solid. 

Provides both transparent Site-to-Site tunnels, as well as remote-access connections.

Most of my professional experience of VPNs has been dealing with IPSec, and the majority of that has been working on Cisco platforms.  

The key thing with IPSec, is that both ends must have the same configuration parameters, otherwise nothing works.  In some ways, this makes everything awkward, but on the other hand, it makes for better security, as the VPN endpoint (or server, if you'd rather), won't accept any old shit, it has to be an exact match for what it's expecting. 

Most major firewall vendors have an implementation of IPSec.  Cisco and Juniper are the two I understand the best, but there's also implementations for Linux and similar, in OpenSwan and StrongSwan.

 

Dishonourable Mentions: 

[Hamachi/LogMeIn](https://en.wikipedia.org/wiki/Hamachi_(software)) VPN:

Proprietary VPN technology.  Currently squatting on 25.0.0.0/8 (rightfully allocated by the MOD). Requires a "mediation server" provided by Hamachi - Fuck knows what this does.

I wouldn't trust it, frankly. 

**Footnotes**:

I'm not going to comment on whether various protocols have been backdoored by various agencies.  From my point of view, there's still a lot of FUD floating around post Snowden, and I really don't want to get involved.

That said, there are some things I will say.  `3DES` doesn't provide enough security really for me to want to use it.  Ditto `RC4`.  

