# CIFS cheatsheet

Mount CIFS/smb/samba at boot:

The first two examples keep SMB credentials in `/home/paul/.smb`

`sudo vim /etc/fstab`

```fstab
//192.168.1.153/tv /mnt/tv cifs iocharset=utf8,credentials=/home/paul/.smb,dir_mode=0775,uid=1420,gid=1420 0 0
//192.168.1.153/tv /mnt/tv cifs iocharset=utf8,credentials=/home/paul/.smb,dir_mode=0775,uid=1420,gid=1420 0 0
//10.2.200.20/storage /mnt/storage cifs iocharset=utf8,dir_mode=0775,uid=1420,gid=1420 0 0
```

Manually mount SMB CIFS

`sudo mount -t cifs -o user=nova //10.2.200.10/nova /mnt/nova`