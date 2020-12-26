# Setup OpenStack Cloud For Lab Purpose With 2-Nodes Configuration

## Overview And Requirements

Setup a OpenStack Cloud For Lab Purpose, should not use for production. Using following Components Provides By Virtualization Software as:
Virtual Machine Manager (Linux) Or VMWare Workstation (Windows):

![openstack-2-node-archiecture.png](./images/openstack-2-node-archiecture.png)

Controller node:

- Management IP: 192.168.10.11
- External IP: 192.168.175.11
- 3 Network Interfaces
- 2 Disk drives, one for Host OS, one for Cinder Storage
- Ubuntu Server 20.04

Compute node:

- Management IP: 192.168.10.31
- External IP: 192.168.175.31
- 3 Network Interfaces
- 1 Disk drives, for Host OS
- Ubuntu Server 20.04

Bastion node:

- Management IP: 192.168.10.5
- External IP: 192.168.175.5
- 2 Network Interfaces
- 1 Disk drives, for Host OS
- Ubuntu Server 20.04

## Prepare Bastion Node

Setup networking configuration:

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
          ens33:
                  addresses:
                          - 192.168.175.5/24
                  gateway4: 192.168.175.2
                  nameservers:
                          addresses: [8.8.8.8, 8.8.4.4]
          
          ens36:
                  addresses:
                          - 192.168.10.5/24
```

apply it

```bash
netplan apply
```

## Prepare Controller Node

### Prepare Networking

Setup networking configuration:

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
          ens33:
                  addresses:
                          - 192.168.175.11/24
                  gateway4: 192.168.175.2
                  nameservers:
                          addresses: [8.8.8.8, 8.8.4.4]
          
          ens36:
                  addresses:
                          - 192.168.10.11/24
```

apply it

```bash
netplan apply
```

### Prepare Storage

Configure LVM2

```bash
apt-get install lvm2
```

Create volume-group `cinder-volumes`:

```bash
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
```

Configure LVM to scan only /dev/sdb:

```bash
# file /etc/lvm/lvm.conf

devices {
...
filter = [ "a/sda/", "a/sdb/", "r/.*/"]
```

## Prepare Compute Node

### Prepare Networking

Setup networking configuration:

```yaml
# /etc/netplan/00-installer-config.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
          ens33:
                  addresses:
                          - 192.168.175.31/24
                  gateway4: 192.168.175.2
                  nameservers:
                          addresses: [8.8.8.8, 8.8.4.4]
          
          ens36:
                  addresses:
                          - 192.168.10.31/24
```

apply it

```bash
netplan apply
```

### Prepare Storage

Add configfs module to /etc/modules

```bash
# file /etc/modules
configfs
```

Rebuild initramfs using: `update-initramfs -u` command

Stop open-iscsi system service due to its conflicts with iscsid container.

```bash
systemctl stop open-iscsi; systemctl stop iscsid
systemctl disable open-iscsi; systemctl disable iscsid
```

Make sure configfs gets mounted during a server boot up process. There are multiple ways to accomplish it, one example:

```bash
mount -t configfs /etc/rc.local /sys/kernel/config
```
