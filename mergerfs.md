# MergerFS

## Get /dev/sd*x* number for each capacity

sudo fdisk -l

## Get UUID of each /dev/sd*x*

sudo blkid
sudo vim /etc/fstab

## Add to fstab

```sh
UUID=d5de06a8-9d7b-4509-8985-a68020fbcad8      /mnt/disks/disk1  ext4    defaults        0       1
UUID=9e9dcee9-aad5-4f37-8e04-ceec18e9694f  /mnt/disks/disk2  ext4    defaults        0       1
UUID=50c94838-b8fa-48d9-9e30-9f7dee2bd665   /mnt/disks/disk3  ext4    defaults        0       1
UUID=2b9caa95-f811-40e1-a92e-4317c523cc4d    /mnt/disks/disk4  ext4    defaults        0       1
UUID=a1e30721-c65d-4fd8-89cd-9c6922b3380d     /mnt/disks/disk5  ext4    defaults        0       1
UUID=061984e8-d882-41c4-bc6d-f17b809f07f9    /mnt/disks/disk6  ext4    defaults        0       1
UUID=a51abc04-37dc-4a4f-a559-9703c29f05c3   /mnt/disks/disk7  ext4    defaults        0       1
UUID=2b0f905e-570b-4ecb-ae17-4d26a7427772    /mnt/disks/disk8  ext4    defaults        0       1
UUID=560fe5bb-9cf3-44f8-9a4d-5cc023830bac     /mnt/disks/disk9  ext4    defaults        0       1

/mnt/disks/* /storage fuse.mergerfs defaults,allow_other,direct_io,use_ino,category.create=lfs,moveonenospc=true,minfreespace=20G,fsname=mergerfsPool 0 0

/storage /srv/nfs4/storage  none   bind   0   0
```

## install nfs-common so nfs works

sudo apt install nfs-common

## then do this after saving fstab

sudo mount -a
