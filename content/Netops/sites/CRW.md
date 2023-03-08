---
title: CRW
---

CRW is a point-of-presence site at the City of Olive Branch, MS's water tower nearby Craft Rd and US-78.

# Physical Deployment

There are two locations for equipment at this site: cab2.2 on the 2nd floor and mast1 on the top of the tower.

## Cabinet 2.2 (cab2.2)

This is a square-hole 42 RU tall server cabinet. There are no locks, and the cabinet has no sides, a single glass front, and a single mesh back door. Due to a water leak reported in 2023-03, the cabinet is currently covered by 5 mil plastic on the top and open sides to reduce the amount of water entering. The front and rear were left uncovered to ensure adequate airflow.

|  RU | Use, Front   | Use, Rear          |
| --: | ------------ | ------------------ |
|  42 |              | pp1                |
|  41 |              | cable manager (2u) |
|  40 |              | cable manager (2u) |
|  39 |              | poe1               |
|  38 |              |                    |
|  37 |              | cable manager (2u) |
|  36 |              | r1.crw             |
|  35 |              |                    |
|  34 |              |                    |
|  33 |              |                    |
|  32 |              |                    |
|  31 |              |                    |
|  30 |              |                    |
|  29 |              |                    |
|  28 |              |                    |
|  27 |              |                    |
|  26 |              |                    |
|  25 |              |                    |
|  24 |              |                    |
|  23 |              |                    |
|  22 |              |                    |
|  21 |              |                    |
|  20 |              |                    |
|  19 |              |                    |
|  18 |              |                    |
|  17 |              |                    |
|  16 |              |                    |
|  15 |              |                    |
|  14 |              |                    |
|  13 |              |                    |
|  12 |              |                    |
|  11 |              |                    |
|  10 |              |                    |
|   9 |              |                    |
|   8 |              |                    |
|   7 |              |                    |
|   6 |              |                    |
|   5 |              |                    |
|   4 |              |                    |
|   3 | ups.crw (2u) | ups.crw (2u)       |
|   2 | ups.crw (2u) | ups.crw (2u)       |
|   1 | Shelf        | Shelf              |

Unlocated: 3x 1u full depth racks, 1x 2u part depth rack.

## Mast 1 (mast1)

The mast pipe is a x and is mounted to the structure using two saddle clamps x. Both were installed on x. Attached to this mast is a microwave dish, an omnidirectional antenna and radio, and a length of unistrut houding two switches

### Dish

mNAT 30 non precision. One of the mounting bolts threads have stretched and so it is not easily put on or off. Includes sleeve and radome. MikroTik Metal with flexguide cabling.

### Omni

Uibquiti AMO-5G13 with 921UAGS-5SHPacD in an rf elements stationbox. Mounted at the top of mast1.

### Cameras

Unistrut clamped to mast1. 6x6x4 box for two fixed cameras. 4x4x4 for zoom camera. PowerBOX devices are simply metal tied to the strut.

## Cabling Subsystem

Two cat5e shielded cables going from the cabinet to mast1. Two unshielded cat6 cables going to the obarc cabinet.

### Patchpanel 1 (pp1)

pp1 is a 24 port CAT5E patch panel mounted in the top of cab2.2.

| Port | Use                      |
| ---: | ------------------------ |
|    1 | r?                       |
|    2 | r?                       |
|    3 | x-con to cab 2-1 (obarc) |
|    4 | x-con to cab 2-1 (obarc) |
|    5 |                          |
|    6 |                          |
|    7 |                          |
|    8 |                          |
|    9 |                          |
|   10 |                          |
|   11 |                          |
|   12 |                          |
|   13 |                          |
|   14 |                          |
|   15 |                          |
|   16 |                          |
|   17 |                          |
|   18 |                          |
|   19 |                          |
|   20 |                          |
|   21 |                          |
|   22 |                          |
|   23 |                          |
|   24 |                          |

## Grounding

4 (or 6?) AWG THHN (black) is attaching the cabinet chassis to a grounded conduit on site. There is no dedicated lightning protection or grounding of the cabling interconnects currently. There is no bus bar in the cabinet or at the site.

## AC Power Subsystem

### apc.crw

Single 120VAC 15 Amp AC line running to the cabinet connected to an [APC SMT2200RM2U Smart-UPS](https://www.apc.com/us/en/product/SMT2200RM2U/apc-smartups-line-interactive-2200va-rackmount-2u-120v-6x-nema-515r+2x-nema-520r-outlets-smartslot-avr-lcd/). All devices connect to this currently. This has a remote management card. The batteries (RBC43) were last replaced in 2023-03.

## DC Power Subsystem

R1 is powered both by the DC power in jack on the back to a dedicated Meanwell PSU located on a shelf-mounted DIN rail and by poe1.

R? is powered by a MikroTik gigabit power injector which is connected to a COTS 24v power supply sitting on the 2u shelf.

### POE Injector 1 (poe1)

This rack-mount POE injector is a ws-poe-12-1u located in cab2.2. It provides 12 ports 24 VDC passive POE and is powered by a dedicated Meanwell PSU located on a shelf-mounted DIN rail.

|          Port | Use         |
| ------------: | ----------- |
| Data+Power 12 | r1.crw-eth1 |
| Data+Power 11 |             |
| Data+Power 10 |             |
|  Data+Power 9 | pp1.?? (r?) |
|  Data+Power 8 |             |
|  Data+Power 7 |             |
|  Data+Power 6 |             |
|  Data+Power 5 |             |
|  Data+Power 4 |             |
|  Data+Power 3 |             |
|  Data+Power 2 |             |
|  Data+Power 1 |             |
|       Data 12 | ?           |
|       Data 11 | ?           |
|       Data 10 |             |
|        Data 9 | r1.crw-eth4 |
|        Data 8 |             |
|        Data 7 |             |
|        Data 6 |             |
|        Data 5 |             |
|        Data 4 |             |
|        Data 3 |             |
|        Data 2 |             |
|        Data 1 |             |

## Networking Subsystems

MikroTik Powerbox 2 up top, one powering cameras, one powering PtP and omni. This leaves 2 100mbit ports available.

### r1.crw

r1.crw is located in cab2.2. It is an RB3011UiAS-RM.

| Physical Port | Use         |
| ------------: | ----------- |
|          sfp1 |             |
|          eth1 | poe in      |
|          eth2 |             |
|          eth3 | R?.crw      |
|          eth4 | pp1-??      |
|          eth5 |             |
|          eth6 | ups.crw     |
|          eth7 | pp1-??      |
|          eth8 | pp1-??      |
|          eth9 | coconut.crw |
|         eth10 | coconut.crw |

## Computing Subsystems

### coconut.crw

coconut.crw is located in cab2.2. It is an HP proliant server. It has a single ethernet port and a dedicated iLO port for OOB management. This device is currently on standby -- it has no workloads deployed. Long term, the intent is to deploy a k3s cluster to this hardware and have a 2nd availability zone for k3s workloads. This would provide redundancy for systems like DNS and NTP and allow us to migrate services as needed.

## Networking Subsystems

### Subnetting

- 44.34.129.??/28
- 44.34.129.??/28
- 44.34.129.??/28

# Wireless

The equipment on this site uses a mixture of licensed and unlicensed spectrum. There are two radios today.

## Radio Engineering

For an estimate of coverage, view the [coverage map](../../getting-started/coverage-map).

### hil.crw

hil.crw is a [921UAGS-5SHPacD-NM](https://mikrotik.com/product/RB921UAGS-5SHPacD-NM) point-to-point link to East Memphis providing backhaul for the CRW site. It is connected via MikroTik waveguide rpsma cabling to a mANT30 (non precision mount) that has a sleeve and radome kit installed. It is mounted about 2/3rds of the way up mast1.

### omn1.crw

omn1.crw is a point-to-multipoint device providing subscriber connection. It is an RB912 in an rf elements case. It is connected to a [Uibquiti AMO-5G13](https://store.ui.com/collections/operator-airmax-and-ltu-antennas/products/5ghz-airmax-omni-13dbi-rocket-kit) antenna and mounted on the top of mast1.

## Spectrum Engineering

| Frequency | Width  | Operations Regulations & License | Emission designator | Mode    | Location | EIRP | Radio    |
| --------- | ------ | -------------------------------- | ------------------- | ------- | -------- | ---- | -------- |
| 5825 mhz  | 20 mhz | Part 15, unlicensed              |                     | 802.11n | mast1    |      | hil.crw  |
| mhz       | 5 mhz  | Part 97, KM4ECM                  |                     | nv2     | mast1    |      | omn1.crw |

# Safety

This location has multiple hazards. They include:

- Fall risk associated with working on elevations and elevated platforms both with and without railing
- Falling objects striking personnel below, both associated with the work being performed and generally
- Weather exposure, as there are work areas that have no overhead covering and there are no temperature controlled areas on site

Also, there is no on-site restroom or accessible water.

Access to the top requires climbing multiple sectioned ladders with built-in fall arrest systems. Standard industrial safety harness and cable grip should be used. When working in areas without railing such as the top of the water tower, tethers should also be used. When working on top of the tower, anticipate that about 15 feet of rope will be required to travel from the top hatch to the work area.
