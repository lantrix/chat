# Synapse Install

## Further OpenBSD requirements

Install platform specific requirements https://matrix-org.github.io/synapse/latest/setup/installation.html#openbsd

Recommended to mount `/var/synapse` as a new disk

> The filesystem underlying the homeserver directory (defaults to `/var/synapse`) has to be mounted with wxallowed (cf. mount(8)), so creating a separate filesystem and mounting it to `/var/synapse` should be taken into consideration.

```shell
dmesg | grep -E '(sd[0-9]|wd[0-9])'
# assuming sd1 is the new disk - note the following commands are destructive on sd1
doas fdisk -iy sd1 # create partition
doas fdisk sd1 # disaply partition
doas disklabel -E sd1 # add an a partition with defaults for offset, size, and FS type
doas newfs sd0a # format
doas mkdir /var/synapse # make mountpoint
doas mount -o wxallowed /dev/sd1a /var/synapse # mount for synapse
sysctl hw.disknames # get DUID, e.g. b97c0e39d8cdbb8b
vim /etc/fstab # append new mount
```

Example mount line

```
b97c0e39d8cdbb8b.a /var/synapse ffs rw,wxallowed 1 2
```
