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

## Containers

### Move container from one host to another

1. Backup the container.
2. Copy the tar.lzo backup file from host to another host.
3. On new host, "pct restore 101 /var/lib/vz/dump/vzdump-lxc-101-2017_06_19-03_01_55.tar.lzo -storage local-lvm" where local-lvm is where you store containers

`pct restore 114 /var/lib/vz/dump/vzdump-lxc-102-2017_06_19-03_04_16.tar.lzo -storage local-lvm`

Import qemu VM from backup

`qmrestore --storage local-zfs /mnt/willstuff/dump/vzdump-qemu-135-2017_07_22-13_24_43.vma.lzo 135`

### Shrink or resize container

Can confirm that the same error occurs with LVM backed containers. Manual (offline) resize works fine -

```sh
e2fsck -fy /dev/pve-store/vm-105-disk-2
resize2fs /dev/pve-store/vm-105-disk-2 2G
lvreduce -L 3G /dev/pve-store/vm-105-disk-2
resize2fs /dev/pve-store/vm-105-disk-2
```

With edit to /etc/pve/lxc/105.conf to correct size.

`pct restore 113 /var/lib/vz/dump/vzdump-lxc-101-2017_06_19-14_57_09.tar.lzo -storage local-lvm --rootfs 20`

## Virtual Machines

### Edit properties of a VM

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

### Move VM from LVM to ZFS

Link [here](https://pve.proxmox.com/pipermail/pve-user/2016-August/167334.html).

On target node:

1. Create volume on target storage

Format: `pvesm alloc <storage> <vmid> <filename> <size>`

Example: `pvesm alloc images 200 vm-200-disk-0 32G --format raw`

Output: `successfully created 'images:vm-200-disk-0'`

2. Get path to new volume

`pvesm path images:vm-200-disk-0`

Output: `/dev/zvol/tank/images/vm-200-disk-0`

3. On source node, get path to old volume

`pvesm path local-lvm:vm-200-disk-0`

Output: `/dev/pve/vm-200-disk-0`

4. Perform the copy

Format: `qemu-img convert -p -n -t writeback -f raw -O raw <source-path> <destination-path>`

Example: `qemu-img convert -p -n -t writeback -f raw -O raw /dev/pve/vm-200-disk-0 /dev/zvol/tank/images/vm-200-disk-0`

5. In PVE shell, modify 200.conf

`cp /etc/pve/nodes/pve/qemu-server/200.conf /etc/pve/nodes/pve/qemu-server/200.conf.bak`

`vi /etc/pve/nodes/pve/qemu-server/200.conf`

Replace `local-lvm:vm-200-disk-0,size=10G` with `images:vm-200-disk-0,size=32G`

6. Power up the VM

This still shows size 8.8 used 8.8:

`df -h`

`fdisk -l`

We can see that /dev/vda is 32G but it says GPT PMBR size mismatch.

7. Fix GPT PMBR size mismatch

```sh
gdisk
/dev/vda
x ## for experts menu
e ## to relocate backup data structures to end of the disk
w ## to write change
```

8. Remove partition, create it with a new last sector.

More info [here](https://gist.github.com/jornane/71edf07363706898bc78cf2032b124b2).

```sh
gdisk /dev/vda
p ## print start and end sectors
d ## delete
3 ## partition number
n ## new
<enter> ## default start sector is same as original partition
<enter> ## default end sector is much higher number now
8e00 ## original partition was 8300, new one is Linux LVM
w ## write changes
reboot
```

At this point in the Proxmox WebUI, I backed up the VM so if I mess things up, I don't have to redo the above work.

9. Try to resize

```sh
pvs # see physical volume size
vgs # see volume group size
lvs # see logical volume size
pvresize /dev/vda3 ## No space left on devize
```

10. Clear space

`apt clean`

`apt autoclean`

11. Try Resize again

`pvresize /dev/vda3`

This time it is successful.

12. Expand logical volume

`lvextend -l 100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv`

Now reboot.

13. Re-size file system

`resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv`

Finally, we have available space.

`df -h`

Output: `Used 8.8G, Avail 12G`

There's 9G free on the pvs/vgs so I found this command on askbuntu.com:

`sudo lvextend -rl +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv`

Now I'm finally using the entire 32GB for the Ubuntu VM.

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
