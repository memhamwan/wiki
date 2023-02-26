---
title: FTN
---
# Devices

## r1.ftn - https://gitlab.com/memhamwan/bizops/inventory/-/issues/37

- Mirkotik HAP AC
- Dual uplink: one port is consumer broadband, the other is to the SCO site

## Port configuration
- ether5 is for the wireless link connecting to SCO; there’s no need for a power injector anymore because we’re switching to using the PoE output here
- ether4 is for the consumer broadband
- ether1-3 are for any on-net devices; these will receive DHCP to public IP address space, no NAT
- special note for ether1: this is Poe in, which we can use to give redundancy to the power supply for the router. We can reuse the existing power injector to feed power here in addition to the new ac adapter in order to get redundancy.
- wlan interfaces are also for any on-net devices, but these are all assumed to be convenience access and will be put behind NAT

# Work Orders

## 2021-12-??

Installing the new r1.ftn device.

0. Place r1.ftn near the existing equipment, and install its power adapter; power it on
1. disconnect the ethernet cable from 145.45’s computer; note that this is broadband wan
2. Attach the broadband wan to port #4 on the new router
3. Locate the network cable with a power injector attached to it; note that this is the hamwan node; remove the Ethernet cable from the power injector, but leave the injector powered on and nearby
4. Attach the hamwan node Ethernet cable to port #5 on the new router; it will now be powered directly from the router
5. Attach an Ethernet cable to port #1 on the new router
6. Attach the other end of this cable to the power injector, and then attach the power injector to the 145.45 computer
7. Confirm that the redundant power is working by unplugging the power adapter from the new router; it should remain powered on via the backup adapter on the power injector in port 1; reattach the power adapter to the new router
8. Check to see if outbound linking on 145.45 works; this should be configured by default to use the broadband internet; inbound linking won’t work yet but will be remotely enabled later; if this step fails this is Ok as long as all 3 ports show activity on the router
