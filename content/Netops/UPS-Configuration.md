We tend to use APC UPSes with the network line cards. The 1500 VA models are good fits for our usages.

There's a standard configuration that we use, however given the nature of them they must be manually configured via the GUI. Below are the standards that we typically use the settings below.

# General

## Mode

- Set it to synchronize with an NTP server, value `ntp.memhamwan.net` for both primary and secondary; UTC-6 CT timezone; check the box for update now
- Set DNS to use our on-network DNS server, 44.34.128.190

# Notification

## Event Actions

- Set "by group" entries to have informational events *not* send an email
- Set "by group" entries to have warning and critical events *do* send an email

## E-mail

- SNMP address should be `smtp-relay.gmail.com`
- From address should be `apc@$hostname`
- Recipients should be `netops@memhamwan.org`

# Network

## Console

- Disable fully

## SNMP

- Enable v1
- Set "public" community to read, specifically for the 44/8 network; generally this is done by setting the NMS value to `44.255.255.255`
- Disable any other entries under v1
- Disable v3

## All other services

- Disable FTP
- Disable WAP