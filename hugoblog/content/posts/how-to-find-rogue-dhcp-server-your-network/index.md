---
title: "How To: Find a rogue DHCP server on your network"
date: "2013-09-27T19:59:58.000Z"
description: "Pesky things."
warning: ancient
---


**Symptoms:** Some clients are unable to connect to the internet. Some clients report a different IP address, subnet mask and default gateway, compared to others.

**Caveats:** Without a managed switch fabric, this is considerably more difficult.

**Diagnosis:**

1. Allow a device to get an IP address from the rogue server.  You might need to disable the main DHCP server to allow this to happen, as DHCP is a broadcast protocol, so it's really a case of the early bird getting the worm.

Kyle Gordon pointed out that this initially assumes that the DHCP server is the same as the default gateway.  He suggests:  Usually /var/lib/dhcpcd/*.info or /var/lib/NetworkManager/* will contain the dhcp-server-identifier info on Linux, and something in the Windows Event Logs will show similar :-)

2.  Once you've got an IP from the rogue, look at the ethernet adaptor's status, and get the IP of the default gateway.  For this example, we'll call it 10.1.1.1

3. Ping the default gateway for a few seconds.  We need to do this to populate the ARP table.

4. In a Powershell/Cmd/Terminal window, run the command to view the ARP table.  On windows, this is `arp -a`.

What you're looking for is the mapping between the IP address and the Physical (MAC) address.
```
IP address 		Physical Address
10.1.1.1 		20:c9:d0:17:20:a4
```

5. Go to http://www.coffer.com/mac_find/ and paste the found Physical/MAC address of the rogue.  This will tell you who made the device.
This works because all MAC address prefixes are registered by IANA, so you can look up a MAC address and it'll tell you who made the thing (roughly).

6. From the client with the rogue-assigned IP address, set up a long-running ping to the default gateway.  We'll need this to confirm that it's been killed when we start unplugging/disabling ports.

7. Next, you want to open the management pages of all of the switches on the fabric of your network.  Every switch has a MAC Address Table where it keeps track of physical switchports, and the learned MAC addresses it's seen on those ports.

8. Looking at the list of address tables (I find it's helpful to copy/paste them into a text editor, then do a search on the MAC of the rogue.) see if you can track down a port that has *only* that MAC assigned to it.   If there's a single port on a managed device, you can disable/shutdown the port.

9. Failing that, if you find that the MAC is in the table, but on a port with other devices too, say, port 1 has 5 other things, and the rogue is one of them, then that indicates that there's another distribution switch on port 1, and the rogue is connected to that.

10. Hopefully, you might have some clue as to what is on each port, distribution switch wise, especially if you have managed under-desk distribution switches, although this is generally unlikely.

11.  Start hunting.  You know that it's on the network, and can ping it (so you can tell when it's been disconnected).  You know something about the device, the manufacturer.  

12.  As you unplug devices, check whether the ping stops. 

13. When the ping stops, you've found the rogue.  

14.  Congratulate yourself by having a coffee, beer, or a non-stimulating beverage.

I actually worked through this process with one of my Astound Wireless customers, last night, over a VPN.  

It's really relatively straight forward, but is made considerably easier with a managed switch fabric. 

In this case, the rogue turned out to be an Apple Airport Extreme, which do tend to cause havok if misconfigured, or misconnected, as their default is to broadcast DHCP on the 3 LAN ports, which aren't obvious that they're LAN ports, as they have the mysterious `<->` symbol. 

I suspect whomever plugged it into the network, should've connected the link to the building's switch fabric to the WAN port of the Airport Extreme, rather than the LAN.  Or at the very least, disabled the DHCP server on the Airport.


An ideal solution for preventing this kind of mishap is [DHCP snooping](https://en.wikipedia.org/wiki/DHCP_snooping), but that *does* require a fully managed switch fabric, and a non-insignificant amount of management overhead.