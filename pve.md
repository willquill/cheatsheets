# PVE/Proxmox Cheatsheet

## Best Practices

Sourced [here](https://www.reddit.com/r/homelab/comments/5y92d1/what_are_proxmox_ve_44_best_practices_for_adding/deopty6/)

How I decided to do this:

Created the following directories in the CLI:
/tank/backups
/tank/images

And then in Datacenter > Storage > Add:

- Directory, ID backups, directory /tank/backups, content check all
- ZFS, ID images, directory /tank/images, content Disk image, Container
- Directory, ID template, directory /tank, content ISO image, Container template
- The reason I didnt choose /tank/template for ID template is that by selecting directory /tank, Proxmox VE automatically create the template directory inside /tank, and it also creates /tank/template/cache and /tank/template/iso automatically.

## Move container from one host to another

1. Backup the container.
2. Copy the tar.lzo backup file from host to another host.
3. On new host, "pct restore 101 /var/lib/vz/dump/vzdump-lxc-101-2017_06_19-03_01_55.tar.lzo -storage local-lvm" where local-lvm is where you store containers

`pct restore 114 /var/lib/vz/dump/vzdump-lxc-102-2017_06_19-03_04_16.tar.lzo -storage local-lvm`

Import qemu VM from backup

`qmrestore --storage local-zfs /mnt/willstuff/dump/vzdump-qemu-135-2017_07_22-13_24_43.vma.lzo 135`

## Shrink or resize container

Can confirm that the same error occurs with LVM backed containers. Manual (offline) resize works fine -

```sh
e2fsck -fy /dev/pve-store/vm-105-disk-2
resize2fs /dev/pve-store/vm-105-disk-2 2G
lvreduce -L 3G /dev/pve-store/vm-105-disk-2
resize2fs /dev/pve-store/vm-105-disk-2
```

With edit to /etc/pve/lxc/105.conf to correct size.

`pct restore 113 /var/lib/vz/dump/vzdump-lxc-101-2017_06_19-14_57_09.tar.lzo -storage local-lvm --rootfs 20`

## Edit properties of a VM

```sh
nano /etc/pve/nodes/gemini/qemu-server/112.conf
qemu-img convert -f vmdk VA-DTE-SA-55339.1-VT-disk1.vmdk  -O qcow2 outfile
```

```sh
root@gemini:/etc/pve/nodes/gemini# find / -name "vm-112-disk-3"
/dev/zvol/tank/images/vm-112-disk-3
/dev/tank/images/vm-112-disk-3
```

```sh
./kvm-create-img.pl - -configFile kvm_va.conf - -vaImage VA-DTE-SA-55339.1-VT.img
./kvm-create-img.pl - -Memory 2G - -IntMapBridge br0 - -Consoletype telnet::9999,server,nowait
- -vaImage VA-SPE-SA-26686.1-SERIAL.img
```

## Troubleshooting

### No space left on device #1

SOLVE "Failed to add /run/systemd/  ask-password to directory watch: No space left on device"

https://proxmox-openvz.blogspot.com/2015/06/increase-open-file-limit-number-of-open.html

`nano /etc/security/limits.conf`

```sh
# wildcard does not work for root, but for all other users
*               soft     nofile           65536
*               hard     nofile           65536
```

```sh
# settings should also apply to root
root            soft     nofile           65536
root            hard     nofile           65536
```

`ulimit -n 65536`

You do not need to modify pam limits.

### No space left on device #2

```sh
root@grostruc:/# service ssh restart
Error: No space left on device
root@grostruc:/# sysctl fs.inotify.max_user_watches
fs.inotify.max_user_watches = 65536
root@grostruc:/# sysctl fs.inotify.max_user_watches=262144
fs.inotify.max_user_watches = 262144
root@grostruc:/# service ssh restart
```

### Avoiding "too many open files"

More info [here](https://bayton.org/docs/linux/lxd/lxd-zfs-and-bridged-networking-on-ubuntu-16-04-lts/).

```sh
nano /etc/sysctl.conf
fs.inotify.max_queued_events = 1048576
fs.inotify.max_user_instances = 1048576
fs.inotify.max_user_watches = 1048576
```

```sh
nano /etc/security/limits.conf
* soft nofile 100000
* hard nofile 100000
```

Then reboot!

### Passthrough disk

```sh
qm set 101 -virtio0 /dev/disk/by-id/ata-Hitachi_HUS724030ALE641_P8HDNARR
qm set 101 -virtio1 /dev/disk/by-id/ata-Hitachi_HUS724030ALE641_P8J6KY4R
```
