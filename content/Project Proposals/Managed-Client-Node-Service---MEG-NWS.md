This wiki page is to document the current understanding of the MEG NWS Amateur Radio Liaisons group and HamWAN Memphis Metro. The purpose of this installation is to enable amateur radio operators working at the amateur radio desk at the Memphis NWS Office to be able to access various amateur radio resources and the internet (except portions of the internet that do not conform to Part 97 regulations when transmitted).

## Purposes of this agreement

A connection to HamWAN may be used by licensed amateurs, or users under the supervision of license holders, for experimentation, education, and emergency use. A client-node configuration like proposed here generally presents as:

- A small microwave integrated radio-dish, pointed to one of HamWAN's cellular sites
- CAT6 cabling, carrying both data and power
- A consumer-grade wireless router to provide basic data network routing, firewall, and 802.11 wireless connectivity

With this device, amateur radio users at the NWS MEG office will be able to connect to a WiFi access point using equipment like a computer, tablet, cellphone, radio hotspot, or even wifi-enabled radio transceiver to perform routine amateur radio activities. This network can be used with a preference for amateur radio related activities, as it will specifically enable optimized connectivity to resources also hosted on the HamWAN network.

## General Terms

This service is provided as a best-effort basis, supported by a community of volunteers similar to all other amateur radio projects.

This service is not intended as an internet replacement. It is suitable to augment existing networking capabilities and provide an auxiliary means of communication. HamWAN Memphis Metro offers no assurances or warranty of the level of service, and usages consistent with general broadband internet consumption are explicitly prohibited.

Additionally, connectivity to any existing NWS, NOAA, or Agricenter International owned or operated network are explicitly prohibited. The purpose of this network is specifically to augment the existing networks, in a configuration which exists wholly independently. Routine audits may be performed to ensure this is not occurring, and should evidence of this be found, service may be suspended.

## Managed Client Node Service

### Administration

1. Who is administering the device: Shared with HamWAN Memphis Metro and MEG NWS Amateur Radio Liaisons
2. Who is responsible for patches: HamWAN Memphis Metro
3. Who is responsible for abuse: HamWAN Memphis Metro
4. Who is responsible for end-user supervision: MEG NWS Amateur Radio Liaisons
5. Who is the owner of the equipment: ownership is transferred to MEG NWS upon installation

### Network Configuration

1. Is the device NATing: yes
2. Is the device providing DHCP: yes
3. Is there static IP(s) to be routed: no
4. Ports to be forwarded: none

## Technical Data Sheet for Communication Sites

### Equipment

1. FCC Call Sign: KM4ECM
2. Date FCC License Issued: 09/08/2014 
3. Equipment Manufacturer: MikroTik
4. Model Number: RBSEXTANTG-5HPnD, RB951Ui-2HnD
5. Class of Service: HA
6. Type of Unit: Microwave
7. Is power on continuously?: Yes
8: Is this unit packet or voice?: Packet

### Units

1. Point-to-point link
   Transmit frequency: 5900
   Channels: digital, 10 mhz width
2. Consumer WiFi device
   Transmit frequency: auto 2.4 GHz channel (based on what's free)
   Channels: digital, 20 mhz width
   SSID: HamWAN
   WPA2 Key: wififorhams

### Dish Antennas

1. Point-to-point link
   Diameter: 250 mm
   Location: on rooftop nearby existing HF antennas
   Gain: 18 dBi
2. Consumer WiFi device
   Diameter: n/a
   Location: at amateur radio operator desk
   Gain: 2.5 dBi