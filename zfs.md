# ZFS Cheatsheet

## Pools

### Ashift info

Read [here](https://louwrentius.com/zfs-performance-and-capacity-impact-of-ashift9-on-4k-sector-drives.html) for info on ashift9 on 4k sector drives.

So if you really care about sequential write performance, ashift=12 is the better option. If storage capacity is more important, ashift=9 seems to be the best solution for 4K drives.

The performance of ashift=9 on 4K drives is always described as 'horrible' but I think it's best to run your own benchmarks and decide for yourself.

More on ashift [here](https://github.com/zfsonlinux/zfs/wiki/FAQ)

### Create aliases for the disks

More info [here](https://pve.proxmox.com/wiki/ZFS:_Tips_and_Tricks).

1. Find disk IDs in /dev/disk/by-id/

2. Edit /etc/zfs/vdev_id.conf to create aliases (probably use serial number)

Populate `/etc/zfs/vdev_id.conf` with aliases reflecting the output of the above command.

Example with six 3TB disks:

```sh
cat /etc/zfs/vdev_id.conf
alias 3T-P8J6KY4R       ata-Hitachi_HUS724030ALE641_P8J6KY4R
alias 3T-P8HDNARR       ata-Hitachi_HUS724030ALE641_P8HDNARR
alias 3T-P8HPEJSP       ata-Hitachi_HUS724030ALE641_P8HPEJSP
alias 3T-P8G21DUP       ata-Hitachi_HUS724030ALE641_P8G21DUP
alias 3T-P8J1VW9P       ata-Hitachi_HUS724030ALE641_P8J1VW9P
alias 3T-P8H2NXEP       ata-Hitachi_HUS724030ALE641_P8H2NXEP
```

Example with three 1TB disks:

```sh
cat /etc/zfs/vdev_id.conf
alias 1T-Z1W2WQPC       ata-ST1000NM0033-9ZM173_Z1W2WQPC
alias 1T-Z1W42F3X       ata-ST1000NM0033-9ZM173_Z1W42F3X
alias 1T-Z1W42RKM       ata-ST1000NM0033-9ZM173_Z1W42RKM
```

3. **Make sure you run this command after updating vdev_id.conf**: `sudo udevadm trigger`

#### Create the pools

`zpool create tank -o ashift=12 raidz 3T-P8J6KY4R 3T-P8HDNARR 3T-P8HPEJSP 3T-P8G21DUP 3T-P8J1VW9P 3T-P8H2NXEP`

`sudo zfs set atime=off compression=lz4 tank`

### Create stripe with two disks

`vi /etc/zfs/vdev_id.conf`

Add the below:

```sh
alias 1T-Z1W2WQPC       ata-ST1000NM0033-9ZM173_Z1W2WQPC
alias 1T-Z1W42F3X       ata-ST1000NM0033-9ZM173_Z1W42F3X
```

`sudo udevadm trigger`

`sudo zpool create -f backup 1T-Z1W2WQPC`

`sudo zpool add -f backup 1T-Z1W42F3X`

### Create mirror

```sh
sudo vim /etc/zfs/vdev_id.conf
alias 8T-JEH71NBN      ata-WDC_WD80EZAZ-11TDBA0_JEH71NBN
alias 8T-JEHEM2TN      ata-WDC_WD80EZAZ-11TDBA0_JEHEM2TN

zpool create -f tank mirror 8T-JEH71NBN 8T-JEHEM2TN
sudo udevadm trigger
```

### Create pool with two mirrored vdevs

This particular pool has 3x14TB and 1x12TB. It will have a capacity of 24 TB. Later, I can replace the 12TB with a 14TB, and it will have a capacity of 28TB.

```sh
sudo vim /etc/zfs/vdev_id.conf
alias 14T-WDC-9KG38U5L       ata-WDC_WUH721414ALE6L4_9KG38U5L
alias 14T-WDC-9KG81HRL       ata-WDC_WUH721414ALE6L4_9KG81HRL
alias 14T-WDC-9RGG5ZDC       ata-WDC_WUH721414ALE6L4_9RGG5ZDC
alias 12T-WDC-5PGHSZJC       ata-WDC_WD120EMAZ-11BLFA0_5PGHSZJC

sudo udevadm trigger

sudo zpool create -f tank -o ashift=12 mirror 14T-WDC-9KG38U5L 14T-WDC-9KG81HRL mirror 14T-WDC-9RGG5ZDC 12T-WDC-5PGHSZJC 
```

And then turn atime off and set the compression for the new pool:

```sh
sudo zfs set atime=off compression=lz4 tank
```

### Replacing drive in pool

First take the disk offline:

`zpool offline tank2 /dev/sdb2`

Before you can rebuild the ZFS pool, you need to partition the new disk. Proxmox uses a GPT partition table for all ZFS-root installs, with a protective MBR, so we want to clone a working disk's partition tables, copy the GRUB boot partition, copy the MBR, and rerandomize the GUIDs before letting ZFS at the disk again.

Copy the partition table from /dev/sdc to /dev/sdb:

`sgdisk --replicate=/dev/sda /dev/sdb`

Ensure the GUIDs are randomized otherwise the kernel and ZFS will confused:

`sgdisk --randomize-guids /dev/sda`

Install the Grub on the new disk, to ensure that it will boot.

`grub-install /dev/sda`

Then replace the disk in the ZFS pool, assuming the new disk has also shown up as /dev/sdb:

`zpool replace tank2 /dev/sda`

Ensure the RAIDZ array is rebuilding

`zpool status -v`

If you have problems, see [here](https://stackoverflow.com/questions/20494452/zpool-replace-erroring-with-does-not-contain-an-efi-label-but-it-may-contain-par).

More help [here](http://docs.oracle.com/cd/E19253-01/819-5461/gbcet/).

To make the new drive usable to replace the failed one, use parted and mklabel GPT.

## Volumes

```sh
zfs create -o recordsize=1M compression=off tank/media
zfs create -o recordsize=16K tank/plexdata
zfs create -o recordsize=16k tank/downloads
zfs create tank/data
```

## ZFS Properties

Get only local sources for all datasets

```sh
root@gemini:~# zfs get -r -s local all tank
NAME            PROPERTY              VALUE                  SOURCE
tank            compression           lz4                    local
tank            atime                 off                    local
tank            aclinherit            passthrough            local
tank/media      recordsize            1M                     local
tank/media      compression           off                    local
tank/plexdata   recordsize            16K                    local
```

Get only inherited sources for all datasets

```sh
root@gemini:~# zfs get -r -s inherited all tank
NAME            PROPERTY              VALUE                  SOURCE
tank/backups    compression           lz4                    inherited from tank
tank/backups    atime                 off                    inherited from tank
tank/backups    aclinherit            passthrough            inherited from tank
tank/downloads  compression           lz4                    inherited from tank
tank/downloads  atime                 off                    inherited from tank
tank/downloads  aclinherit            passthrough            inherited from tank
tank/media      atime                 off                    inherited from tank
tank/media      aclinherit            passthrough            inherited from tank
tank/nextcloud  compression           lz4                    inherited from tank
tank/nextcloud  atime                 off                    inherited from tank
tank/nextcloud  aclinherit            passthrough            inherited from tank
tank/plexdata   compression           lz4                    inherited from tank
tank/plexdata   atime                 off                    inherited from tank
tank/plexdata   aclinherit            passthrough            inherited from tank
```

## NFS

[Source](https://www.hiroom2.com/2016/05/18/ubuntu-16-04-share-zfs-storage-via-nfs-smb/)

`sudo apt-get install -y nfs-kernel-server`

`sudo zfs set sharenfs="rw=@10.1.20.20/32" tank/ds-lyra`

`sudo zfs share -a`

Confirm with this:

`showmount -e localhost`

Export list for localhost:

`/tank/ds-lyra 10.1.20.20/32`

## Miscellaneous Notes

### Create Nextcloud database pool

Source: [here](http://assets.en.oreilly.com/1/event/21/Optimizing%20MySQL%20Performance%20with%20ZFS%20Presentation.pdf)

```sh
zfs create -o recordsize=4k tank/nextdb
zfs set primarycache=metadata tank/nextdb
```

### Create Palo Alto Panorama pool

```sh
zfs create -o recordsize=4k tank/logpano
zfs set primarycache=metadata tank/logpano
```

### Record of degraded drive replacement

```sh
# I recently used the below method (about May 2017)
# zpool offline hardcore ata-ST2000DM001-1CH164_S1E1AL17
# cfgadm | grep c1t3d0
sata1/3::dsk/c1t3d0            disk         connected    configured   ok
# cfgadm -c unconfigure sata1/3
Unconfigure the device at: /devices/pci@0,0/pci1022,7458@2/pci11ab,11ab@1:3
This operation will suspend activity on the SATA device
Continue (yes/no)? yes
# cfgadm | grep sata1/3
sata1/3                        disk         connected    unconfigured ok
<Physically replace the failed disk c1t3d0>
# cfgadm -c configure sata1/3
# cfgadm | grep sata1/3
sata1/3::dsk/c1t3d0            disk         connected    configured   ok
# zpool online tank c1t3d0
# zpool replace tank c1t3d0
root@gemini:~# zpool clear hardcore
root@gemini:~# zpool offline hardcore 5036312895249712868
root@gemini:~# zpool status hardcore
  pool: hardcore
 state: DEGRADED
status: One or more devices has been taken offline by the administrator.
        Sufficient replicas exist for the pool to continue functioning in a
        degraded state.
action: Online the device using 'zpool online' or replace the device with
        'zpool replace'.
  scan: resilvered 64K in 0h0m with 0 errors on Tue May 23 22:10:27 2017
config:

        NAME                                           STATE     READ WRITE CKSUM
        hardcore                                       DEGRADED     0     0     0
          raidz1-0                                     DEGRADED     0     0     0
            ata-Hitachi_HUS724030ALE641_P8H2NXEP       ONLINE       0     0     0
            ata-ST2000DM001-1CH164_S1E1AL17            OFFLINE      0     0     0
            ata-WDC_WD2001FASS-00U0B0_WD-WMAUR0368524  ONLINE       0     0     0

errors: No known data errors
root@gemini:~# parted
(parted) select /dev/sdi
Using /dev/sdi
(parted) mklabel GPT
(parted) quit
root@gemini:~# ls -la /dev/disk/by-id
lrwxrwxrwx 1 root root    9 May 23 22:14 ata-Hitachi_HUS724030ALE641_P8J1VW9P -> ../../sdi
root@gemini:~# zpool replace hardcore ata-ST2000DM001-1CH164_S1E1AL17 ata-Hitachi_HUS724030ALE641_P8J1VW9P
root@gemini:~# zpool status hardcore
  pool: hardcore
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Tue May 23 22:17:58 2017
    1.57G scanned out of 3.02T at 146M/s, 6h0m to go
    534M resilvered, 0.05% done
config:

        NAME                                           STATE     READ WRITE CKSUM
        hardcore                                       DEGRADED     0     0     0
          raidz1-0                                     DEGRADED     0     0     0
            ata-Hitachi_HUS724030ALE641_P8H2NXEP       ONLINE       0     0     0
            replacing-1                                OFFLINE      0     0     0
              ata-ST2000DM001-1CH164_S1E1AL17          OFFLINE      0     0     0
              ata-Hitachi_HUS724030ALE641_P8J1VW9P     ONLINE       0     0     0  (resilvering)
            ata-WDC_WD2001FASS-00U0B0_WD-WMAUR0368524  ONLINE       0     0     0

errors: No known data errors
```

## Troubleshooting

### Unrecoverable error

Sample output of `sudo zpool status`:

```
  pool: tank
 state: ONLINE
status: One or more devices has experienced an unrecoverable error.  An
        attempt was made to correct the error.  Applications are unaffected.
action: Determine if the device needs to be replaced, and clear the errors
        using 'zpool clear' or replace the device with 'zpool replace'.
   see: https://openzfs.github.io/openzfs-docs/msg/ZFS-8000-9P
  scan: scrub repaired 3.50M in 19:00:21 with 0 errors on Sun Jul 14 19:24:35 2024
config:

        NAME              STATE     READ WRITE CKSUM
        tank              ONLINE       0     0     0
          raidz1-0        ONLINE       0     0     0
            12T-5PGJ4A0D  ONLINE       0     0     0
            12T-Z2J26EBT  ONLINE       0     0     0
            12T-5PGHSZJC  ONLINE       0     0     0
          raidz1-1        ONLINE       0     0     0
            14T-9KG38U5L  ONLINE       0     0     0
            14T-9KG81HRL  ONLINE       0     0     0
            14T-9RGG5ZDC  ONLINE       7     0     5

errors: No known data errors
```

First, clear the errors. This will clear the error counters. Check the status again afterward with zpool status to see if the errors return.

```sh
sudo zpool clear tank
```

Initiate a scrub to ensure data integrity and attempt to repair any additional errors:

```sh
sudo zpool scrub tank
```

After the scrub completes, check the status again with zpool status.

To check integrity of the disk, install smartmontools and execute:

```sh
sudo smartctl -a /dev/disk/by-id/your-disk-id
```
