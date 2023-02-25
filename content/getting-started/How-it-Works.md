---
weight: 1
---
*This article is a work-in-progress*

HamWAN Memphis Metro is building an wide area network in the midsouth for education, experimentation, and emergency use. The project can be split in three parts: the core network, network services, and user services. Together these make for a project and community supporting the intersection of computing and radios in and around Memphis.

This article seeks to provide high-level overviews of how these three parts operate and come together to create the project. It is useful for anyone wanting to learn more about the network, and consider it as a primer for people who want to volunteer to help as a network operator ("netop").

# Core Network

HamWAN in many ways is like an ISP: it has network service from upstream providers, provides regional transit, and ultimately has last-mile services for clients. At the same time, HamWAN is like an enterprise network: hosts are trusted, treated as the same tenant, and traffic is expected to be a mix of intra-network and inter-network (in both directions!).

## Design

### Physical Layout and Link

Cellular last mile, partial mesh between points of presence, infrastructure in the cloud

### Different Wireless Modes

nv2 for cellular/last mile, 802.11 for point to points (some AC, some AY, some N)

### Approach to Routing

OSPF, BGP, VRRP

### The Special Case that is Edge Routing

Cloud router

### Platforms, tools, and vendors

For the last mile, the NV2 TDMA mode was selected by the HamWAN organization, with is proprietary with MikroTik. This means that all last mile devices are manufactured by MikroTik. To avoid introduced unneeded complexity, we've chosen to use MikroTik devices for nearly all networking appliances. This includes switches, routers, and wireless radios. RouterOS has a lot of built in flexibility, and even more importantly, it is extremely capable. In fact, it is so capable that you won't find managed devices from other vendors.

If you're new to RouterOS, expect to access the devices via SSH (though we set SSH on port 222 whereever possible rather than 22 to avoid spam). The command line interface allows direct interaction to setting configuration and accessing various tools -- it is not a traditional shell. There is an extensive [manual](https://help.mikrotik.com/docs/display/ROS/RouterOS) that is good to reference as a nooby and a veteran.

For times when the device is not accessible over IP, but it might be available via MAC, protocols and tools like mactelnet and RoMON are helpful. Mactelnet is especially helpful for the initial provisioning of devices. To use these tools on Windows, the simplest thing to do is install MikroTik's Winbox application and use your favorite SSH client. On a POSIX machine, use ssh and [MAC Telnet](https://github.com/haakonnessjoen/MAC-Telnet).

## Development and Maintenance

### Use of CI/CD

GitLab CI running terraform and ansible, though initial provisioning is manual

