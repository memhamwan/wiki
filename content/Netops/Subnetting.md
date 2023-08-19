# Subnetting

## Aggregate range 44.34.128.0/21

### Prefixes

| Prefix           | Role                              |
|------------------|-----------------------------------|
| 44.34.128.32/27  | LAN, SCO                          |
| 44.34.128.48/28  | Users, SCO Sec 1                  |
| 44.34.128.64/28  | Users, SCO Sec 2                  |
| 44.34.128.80/28  | Users, SCO Sec 3                  |
| 44.34.128.96/28  | LAN, HIL                          |
| 44.34.128.112/28 | Users, HIL Sec 1                  |
| 44.34.128.128/28 | Users, HIL Sec 2                  |
| 44.34.128.144/28 | Users, HIL Sec 3                  |
| 44.34.128.160/27 | LAN, LEB                          |
| 44.34.128.192/28 |                                   |
| 44.34.128.208/28 | Users, LEB Omn 1                  |
| 44.34.128.224/28 |                                   |
| 44.34.128.240/28 | Users, MNO Sec 1                  |
| 44.34.129.0/28   | Users, MNO Sec 2                  |
| 44.34.129.16/28  | Users, MNO Sec 3                  |
| 44.34.129.32/27  | LAN, MNO                          |
| 44.34.129.64/28  | LAN, AZO                          |
| 44.34.129.80/28  | Users, AZO R1 (aka omn 1)         |
| 44.34.129.96/28  | Users, AZO Omn 2                  |
| 44.34.129.112/28 | LAN, CRW                          |
| 44.34.129.128/28 | LAN, SCO for IOT/Cameras          |
| 44.34.129.144/28 | LAN, CRW for IOT/Cameras          |
| 44.34.129.160/28 | LAN, RET                          |
| 44.34.129.176/28 | Users, CRW Ethernet OBARC         |
| 44.34.129.192/28 | LAN, FTN (DHCP pool on 196/31)    |
| 44.34.129.208/28 | LAN, ATL er1 cloud router         |
| 44.34.129.224/28 |                                   |
| 44.34.129.240/28 |                                   |
| 44.34.130.0/28   | User LAN (DHCP), HIL              | 
| 44.34.131.130/31 | PTP, between HIL and LEB          |
| 44.34.131.138/31 | PTP, between HIL and MNO          |
| 44.34.131.140/31 | PTP, between LEB and MNO          |
| 44.34.131.142/31 | PTP, between AZO and LEB          |
| 44.34.131.144/31 | PTP, between CRW and LEB          |
| 44.34.131.146/31 | PTP, between LEB and RET          |
| 44.34.131.156/31 | PTP, between SCO and LEB          |
| 44.34.131.164/31 | PTP, between RET and ATL (tunnel) |
| 44.34.131.168/31 | PTP, between FTN and ATL (tunnel) |
| 44.34.131.170/31 | PTP, between AZO and ATL (tunnel) |
| 44.34.132.0/23   | Anycast floating services         |
| 44.34.132.1/32   | Recursive DNS                     |
| 44.34.132.7/32   | APRS                              |
| 44.34.132.8/32   | K8S ingress services              |
| 44.34.132.18/32  | DRATS                             |
| 44.34.134.0/24   | OVPN on ATL                       |
