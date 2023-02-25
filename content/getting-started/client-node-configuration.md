We're excited that you want to join us! There are a few steps to getting connected to be aware of. Big picture, the phases are: opening the management interface, configuring the modem and router, and then aiming the antenna. The following steps offer advice for each. Note that this assumes that you already have your hardware acquired and on hand.

As a reminder, HamWAN seeks to be highly efficient in our use of 5.9 GHz spectrum. This requires using abnormally-narrow channels that are not supported by MikroTik's 802.11 AC equipment. Before moving forward, please ensure that your equipment is _not_ 802.11 AC, at it is not compatible.

*Microwave exposure is dangerous! When powered on, always make sure they're connected to an antenna or suitable dummy load, and do not point them at living beings.*

# Opening the management interface

- Follow MikroTik's [First time startup](https://wiki.mikrotik.com/wiki/Manual:First_time_startup) guide to familiarize yourself with the device; we prefer SSH, but there is no one true way


# Configuring the modem and router

This section is based on [hamwan.org](http://hamwan.org/Standards/Network%20Engineering/Client%20Node%20Configuration.html)'s documentation, though it has been simplified and has added support for our region-specific omni sites.

Run these commands in your CLI to configue it as our network expects:

```
/system package update install
/system reset-configuration no-defaults=yes
/system identity set name=URCALL
/interface wireless set 0 radio-name=UCALL
/user set admin password=URPASSWORD
/console clear-history
/user add name=turnrye group=full
/user add name=seichold group=full
/file print file=turnrye.txt
/file print file=seichold.txt
/file set turnrye.txt contents="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEZXA4eMYwloBRhd7bvYVF8o+xBUHLCldQcSfwunhE+OP8g13a5/DsUL+XkM14MNxLgIZaTKwLQxN8Q0O7WyVy9ZRCS29ybGvjhzMi3YJENd9G/URkQpSA7Jt6l4M+pI5J6Az9Vpw0ASboxpVSaBWOL1u5Kg0+SDGsea1KtoU+gMi91Fy5dNdiuWVpjvtRJAkQ0Kukidzj/HxYZvetelXy6CwDqI2JBYHGWlO8gMfd3x5ObRW4RR7oyWsV3q+f3V2723GLre5W1owqvEZqhfUK1N3RBBEtAUCRM0WoQW0RNoLEqVAsXfy1rkyRgi0FXm+te9xQ3355+tTdwqioCoAj ryan_turner@Ryans-MacBook-Pro.local"
/file set seichold contents="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtNLtPE1FDaMZ3stcq/lMralo4gMvV3WgAFUq71Zc5rKXSfzYVgSYcPUMEVoEN8OPPa3x5pYW6QSTD7jNVIDtPjROfyv1ATZXnTkdMmyF4BDdHkBPMYjx1By6lsOhRts1QMo6uYsF0xXLciaZIEKQYck0oAKckXYf+slIUQ9MV7GXW6OolCYtdu9x64M8uvqQ+KblbulhGlB64pI1QXizcSr78wo+AhXZN3RiYVn+gawmdnP9AVqIxXEvIVjwDjRZY6HQWdeSMijFY+et1E7yWRRPPJGwPSjZ2bK+lUpT/7tN8MKVevRi9XjlvjtkVBr2B/Ldm97MyUzK4L92p5mqN sam@sam-HP-EliteBook-840-G1"
/user ssh-keys import public-key-file=turnrye.txt user=turnrye
/user ssh-keys import public-key-file=seichold.txt user=seichold
/system routerboard settings set boot-device=try-ethernet-once-then-nand
/snmp community set name=public addresses=44.0.0.0/8 read-access=yes write-access=no numbers=0
/system ntp client set enabled=yes server-dns-names=ntp.memhamwan.net
/ip firewall filter remove [find dynamic=no]
/ip dhcp-server remove [find]
/ip dhcp-server network remove [find]
/ip address remove [find interface~"^wlan1"]
/ip dns set allow-remote-requests=no
/ip service disable telnet,ftp,api
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 0 degrees (north)" frequency=5920 list=HamWAN name=Sector1-5 width=5
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 120 degrees (south-east)" frequency=5900 list=HamWAN name=Sector2-5 width=5
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 240 degrees (south-west)" frequency=5880 list=HamWAN name=Sector3-5 width=5
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 0 degrees (north)" frequency=5920 list=HamWAN name=Sector1-10 width=10
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 120 degrees (south-east)" frequency=5900 list=HamWAN name=Sector2-10 width=10
/interface wireless channels add band=5ghz-onlyn comment="Cell sites radiate this at 240 degrees (south-west)" frequency=5880 list=HamWAN name=Sector3-10 width=10
/interface wireless set 0 disabled=no country=no_country_set frequency-mode=superchannel band=5ghz-onlyn mode=station scan-list="HamWAN" ssid=HamWAN wireless-protocol=nv2
/ip dhcp-client add add-default-route=yes dhcp-options=hostname,clientid disabled=no interface=wlan1
/ip address add address=192.168.88.1/24 interface=ether1
/ip pool add name=dhcp-pool ranges=192.168.88.100-192.168.88.199
/ip dhcp-server network add address=192.168.88.0/24 dns-server=44.24.244.1,44.24.245.1 gateway=192.168.88.1
/ip dhcp-server add address-pool=dhcp-pool interface=ether1 name=dhcp disabled=no
/ip firewall nat add chain=srcnat action=masquerade out-interface=wlan1
/ip service set ssh port=222
```

# Aiming the antenna

- Check the [coverage map](../coverage-map) to see what's suitable and calculate the azimuth
- Point it in the general direction and power the device on
- Aim it using the signal indicators on your device
