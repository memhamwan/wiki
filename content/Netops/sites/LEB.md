---
title: LEB
---
# ESXi

Accessible 44.34.128.164, but TCP 80 and 443 are filtered at the edge. To administer this you must be within the network. Credentials are "root" and the standard password.

## Guests

### dns
44.34.128.177

### blueiris
44.34.128.169

# Durian

- k8s cluster host provisioned via RKE
- DL380g8
- Setup as ubuntu 20.04 on bare metal, then provisioned with memhamwan's linux base ansible script
- Then RKE installed
    - Note that https://rancher.com/docs/rke/latest/en/config-options/services/services-extras/#extra-binds had to be considered, see the cluster-management repo for details on this
- k8s management keys are not checked in for security; check in with Ryan for access
- Files are checked in to https://gitlab.com/memhamwan/cluster-management/-/blob/primary/rke/durian/
- 44.34.128.165