# NFS Cheatsheet

## NFS share from Proxmox ([source](https://www.hiroom2.com/2016/05/18/ubuntu-16-04-share-zfs-storage-via-nfs-smb/))

`zfs create -o recordsize=1M tank/security`

`apt-get install -y nfs-kernel-server`

`zfs set sharenfs="rw=@10.1.20.90/32" tank/cam_archive`

On Ubuntu client that will mount it:

`sudo apt install nfs-common`

`sudo vim /etc/fstab`

```fstab
10.1.20.20:/tank/security /storage nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

## NFS Debugging ([source](https://kerneltalks.com/config/nfs-logs-in-linux/))

Enable debugging

`rpcdebug -m nfsd all`

`service nfs-kernel-server restart`

`tail -f /var/log/syslog`

Disable debugging

`rpcdebug -m  nfsd -c  all`

`service nfs-kernel-server restart`

## NFS Share from Ubuntu 18.04 ([source](https://linuxize.com/post/how-to-install-and-configure-an-nfs-server-on-ubuntu-18-04/))

`sudo mkdir -p /srv/nfs4/storage`

`sudo mount --bind /storage /srv/nfs4/storage`

`sudo vim /etc/fstab`

```fstab
/storage /srv/nfs4/storage  none   bind   0   0
```

`sudo vim /etc/exports`

```fstab
/srv/nfs4         10.0.0.0/8(rw,sync,no_subtree_check,insecure,crossmnt,fsid=0)
/srv/nfs4/storage 10.0.0.0/8(rw,sync,no_subtree_check,insecure,fsid=1) 172.16.0.0/12(rw,sync,no_subtree_check,insecure,fsid=1) 192.168.0.0/16(rw,sync,no_subtree_check,insecure,fsid=1)
```

*Above, I'm using the insecure option because of a random high number port*

`sudo exportfs -ra`

`sudo exportfs -v`

## Mount NFS Shares

`mkdir /mnt/media`

`mount -t nfs 10.1.102.8:/mnt/tank/media /mnt/media`

`mkdir /mnt/downloads`

`mount -t nfs 10.1.102.8:/mnt/tank/downloads /mnt/downloads`

## Mount at boot in CentOS

`sudo vim /etc/fstab`

```fstab
// <file system> <mount point> <type> <options> <dump> <pass>
10.1.20.90:/volumeUSB1/usbshare /mnt/usb1 nfs rw,hard,tcp,intr
10.1.20.90:/volumeUSB2/usbshare /mnt/usb2 nfs rw,hard,tcp,intr
10.1.20.90:/volumeUSB3/usbshare /mnt/usb3 nfs rw,hard,tcp,intr
10.1.20.90:/volume1/willstuff /mnt/gwace nfs rw,hard,tcp,intr
```

If you can't see NFS mount files, run this:

`sudo yum -y install nfs-utils`

## Mount at boot in Ubuntu

`sudo apt install nfs-common`

`sudo vim /etc/fstab`

```fstab
// <file system> <mount point> <type> <options> <dump> <pass>
10.1.20.10:/tank/media /mnt/media nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
10.1.20.10:/tank/downloads /mnt/downloads nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

If you need spaces in a line in `/etc/fstab`, like if the directory is `Application Support`

```fstab
10.1.102.8:/mnt/tank/plexdata/config/Library/Application\040Support /mnt/plexdata nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```
