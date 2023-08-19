---
title: FTN
---

# Physical Deployment

## r1.ftn - https://gitlab.com/memhamwan/bizops/inventory/-/issues/37

r1.hil is located in a cabinet owned by Tri-State Repeater Association. It is an [MikroTik RouterBOARD 962UiGS-5HacT2HnT](/engineering-references/mikrotik-hAP-ac.pdf).

| Physical Port | Use                                                 |
| ------------: | --------------------------------------------------- |
|          sfp1 |                                                     |
|        ether1 | User port w/ DHCP (wb4kog145450)                    |
|        ether2 | User port w/ DHCP (unused)                          |
|        ether3 | User port w/ DHCP (wb4kog444775)                    |
|        ether4 | OOB WAN                                             |
|        ether5 | PTP, poe out (sco.hil:ether1)                       |
|         wlan1 | Convenience w/ NAT, ssid "HamWAN" key "wififorhams" |
|         wlan2 | Convenience w/ NAT, ssid "HamWAN" key "wififorhams" |
