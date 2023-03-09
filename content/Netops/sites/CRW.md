---
title: CRW
---

CRW is a point-of-presence site at the City of Olive Branch, MS's water tower nearby Craft Rd and US-78.

# Standards

This deployment is treated as a TIA-606 class 1 deployment.

# Physical Deployment

There are two locations for equipment at this site: cabinet 2.2 on the 2nd floor and mast1 on the top of the tower.

## Cabinet 2.2 (2.2)

This is a square-hole 42 RU tall server cabinet. There are no locks, and the cabinet has no sides, a single glass front, and a single mesh back door. Due to a water leak reported in 2023-03, the cabinet is currently covered by 5 mil plastic on the top and open sides to reduce the amount of water entering. The front and rear were left uncovered to ensure adequate airflow.

|  RU | Use, Front   | Use, Rear          |
| --: | ------------ | ------------------ |
|  42 |              | 2.2-42 (patch)     |
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

The mast is a [P272 2-3/8" x 72" schedule 40 galvanized pipe](/engineering-references/valmont-Pxxx.pdf) and is mounted to the structure using two [crossover plates SCX7-u](/engineering-references/valmont-SCX7-U.pdf). Attached to this mast is a microwave dish (hil.crw), an omnidirectional antenna (omn1.crw), and a length of unistrut houding two switches (r2.crw, r3.crw) and two conduit boxes with network cameras (cam1.crw, cam2.crw, cam3.crw).

### Unistrut

Unistrut clamped to mast1. To it there is a 6x6x4 conduit box holding cam2.crw and cam3.crw. There is also attached a 4x4x4 conduit box for cam1.crw. r2.crw and r3.crw are affixed to the unistrut using metal cable tie.

## Cabling Subsystem

Two cat5e shielded cables going from 2.2-42 to mast1. Two unshielded cat6 cables going from 2.2-42 to 2.1 (OBARC).

### Patchpanel 1 (2.2-42)

2.2-42 is a 24 port CAT5E patch panel mounted in the top of cabinet 2.2.

| Port | Use                      |
| ---: | ------------------------ |
|    1 | r?                       |
|    2 | r?                       |
|    3 | x-con to cab 2.1 (obarc) |
|    4 | x-con to cab 2.1 (obarc) |
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

## Power Subsystem

```goat
                                    .------. 24v .--------. poe .--------.     .----------.
                              .---->+ psu1 +---->+  poe1  +---->+ r3.crw +-+-->+ omn1.crw |
                              |     .------.     .---+----.     .--------. |   .----------.
                              |                      | poe                 |
.-----------.                 |                      v                     |   .----------.
|  120vac   |    .---------.  |     .------. 24v .---+----.                .-->+ hil.crw  |
| building  +--->| ups.crw +--+---->+ psu2 +---->+ r1.crw |                    .----------.
|  power    |    .---------.  |     .------.     .--------.
.-----------.                 |
                              |     .------. 24v .--------. poe .----------.
                              .---->+ psu3 +---->+ r2.crw +-+-->+ cam1.crw |
                              |     .------.     .--------. |   .----------.
                              |                             |
                              |     .-------------.         |   .----------.
                              .---->+ coconut.crw |         .-->+ cam2.crw |
                              |     .-------------.         |   .----------.
                              |                             |
                              |     .-------------.         |   .----------.
                              .---->+ coconut.crw |         .-->+ cam3.crw |
                                    .-------------.             .----------.
```

### UPS (ups.crw)

Single 120VAC 15 Amp AC line running to the cabinet connected to an [APC SMT2200RM2U Smart-UPS](/engineering-references/apc-SMT2200RM2U.pdf). All devices connect to this currently. This has a remote management card. The [APC RBC43](/engineering-references/apc-RBC43.pdf) batteries were last replaced in 2023-03.

### DC Power Subsystem (psu1, psu2, psu3)

psu1 and psu2 are [Meanwell MDR-100-24](https://www.meanwell.com/productPdf.aspx?i=133) PSUs located on a shelf-mounted DIN rail in cat2.2. psu3 is a mikrotik 24vdc power supply.

### POE Injector 1 (poe1)

This rack-mount POE injector is a ws-poe-12-1u located in cabinet 2.2. It provides 12 ports 24 VDC passive POE and is powered by a dedicated Meanwell PSU located on a shelf-mounted DIN rail.

|          Port | Use           |
| ------------: | ------------- |
| Data+Power 12 | r1.crw:ether1 |
| Data+Power 11 |               |
| Data+Power 10 |               |
|  Data+Power 9 | 2.2-42:? (r3) |
|  Data+Power 8 |               |
|  Data+Power 7 |               |
|  Data+Power 6 |               |
|  Data+Power 5 |               |
|  Data+Power 4 |               |
|  Data+Power 3 |               |
|  Data+Power 2 |               |
|  Data+Power 1 |               |
|       Data 12 | ?             |
|       Data 11 | ?             |
|       Data 10 |               |
|        Data 9 | r1.crw:ether4 |
|        Data 8 |               |
|        Data 7 |               |
|        Data 6 |               |
|        Data 5 |               |
|        Data 4 |               |
|        Data 3 |               |
|        Data 2 |               |
|        Data 1 |               |

## Networking Subsystems

### r1.crw

r1.crw is located in cabinet 2.2. It is an [MikroTik RB3011UiAS-RM](/engineering-references/mikrotik-RB3011UiAS-RM.pdf).

| Physical Port | Use              |
| ------------: | ---------------- |
|          sfp1 |                  |
|          eth1 | poe in           |
|          eth2 |                  |
|          eth3 | r2.crw           |
|          eth4 | 2.2-42:?         |
|          eth5 |                  |
|          eth6 | ups.crw          |
|          eth7 | 2.2-42:?         |
|          eth8 | 2.2-42:?         |
|          eth9 | ilo.coconut.crw  |
|         eth10 | eth0.coconut.crw |

### r2.crw

[MikroTik PowerBOX](/engineering-references/mikrotik-RB750P-PBr2.pdf) mounted on the unistrut on mast1.

| Physical Port | Use               |
| ------------: | ----------------- |
|          eth1 | 2.2-42:? (r1.crw) |
|          eth2 |                   |
|          eth3 |                   |
|          eth4 |                   |
|          eth5 |                   |

Unlocated: cam1.crw, cam2.crw, cam3.crw

### r3.crw

[MikroTik PowerBOX](/engineering-references/mikrotik-RB750P-PBr2.pdf) mounted on the unistrut on mast1.

| Physical Port | Use               |
| ------------: | ----------------- |
|          eth1 | 2.2-42:? (r1.crw) |
|          eth2 |                   |
|          eth3 |                   |
|          eth4 |                   |
|          eth5 |                   |

Unlocated: hil.crw, omn1.crw

### Backhaul to Hilton site (hil.crw)

[MikroTik 921UAGS-5SHPacD](/engineering-references/mikrotik-netmetal.pdf). hil.crw is a point-to-point link to East Memphis providing backhaul for the CRW site. It is connected via MikroTik waveguide rpsma cabling to a mANT30 (non precision mount) that has a sleeve and radome kit installed. It is mounted about 2/3rds of the way up mast1. mNAT 30 non precision. One of the mounting bolts threads have stretched and so it is not easily put on or off. Includes sleeve and radome.

| Physical Port | Use      |
| ------------: | -------- |
|          eth1 | r3.crw:? |

### Omni for subscribers (omn1.crw)

[Uibquiti AMO-5G13](/engineering-references/ubiquiti-amo.pdf) with [MikroTik RB912UAG-5HPnD](/engineering-references/mikrotik-rb912.pdf) in an [RF Elements SBX-S-CC-2SMA](/engineering-references/rfelements-sbx-s-cc-2sma.pdf). Mounted at the top of mast1.

| Physical Port | Use      |
| ------------: | -------- |
|          eth1 | r3.crw:? |

## Computing Subsystems

### coconut.crw

coconut.crw is located in cabinet 2.2. It is an HP proliant server. It has a single ethernet port and a dedicated iLO port for OOB management. This device is currently on standby -- it has no workloads deployed. Long term, the intent is to deploy a k3s cluster to this hardware and have a 2nd availability zone for k3s workloads. This would provide redundancy for systems like DNS and NTP and allow us to migrate services as needed.

## Networking Subsystems

### Subnetting

See entries tagged "CRW" on the [Subnetting page](../../Subnetting).

# Wireless

The equipment on this site uses a mixture of licensed and unlicensed spectrum. There are two radios today.

## Radio Engineering

For an estimate of coverage, view the [coverage map](../../../getting-started/coverage-map).

## Spectrum Engineering

| Frequency               | Width  | Operations Regulations & License | Emission designator | Mode    | Location | EIRP | Radio    |
| ----------------------- | ------ | -------------------------------- | ------------------- | ------- | -------- | ---- | -------- |
| 5825 mhz (802.11an-165) | 20 mhz | Part 15, unlicensed              |                     | 802.11n | mast1    |      | hil.crw  |
| 5865 mhz (Omni1-5)      | 5 mhz  | Part 97, KM4ECM                  |                     | nv2     | mast1    |      | omn1.crw |

# Safety

This location has multiple hazards. They include:

- Fall risk associated with working on elevations and elevated platforms both with and without railing
- Falling objects striking personnel below, both associated with the work being performed and generally
- Weather exposure, as there are work areas that have no overhead covering and there are no temperature controlled areas on site

Also, there is no on-site restroom or accessible water.

Access to the top requires climbing multiple sectioned ladders with built-in fall arrest systems. Standard industrial safety harness and cable grip should be used. When working in areas without railing such as the top of the water tower, tethers should also be used. When working on top of the tower, anticipate that about 15 feet of rope will be required to travel from the top hatch to the work area.
