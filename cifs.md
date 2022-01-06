# CIFS Cheatsheet

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

Configuring Samba server

`sudo vi /etc/samba/smb.conf`

Example:

```samba
[User]
   path = /home/nova
   browsable = yes
   read only = no
   directory mask = 0700
   create mask = 0700
   valid users = will nova

[Movies]
   path = /tank/media/movies
   browsable = yes
   read only = no
   directory mask = 0775
   create mask = 0775
   valid users = will nova

[TV]
   path = /storage/media/tv
   browsable = yes
   read only = no
   directory mask = 0775
   create mask = 0775
   valid users = will nova
   
[Data]
   path = /tank/data
   browsable = yes
   read only = no
   directory mask = 0700
   create mask = 0700
   valid users = will

[Backups]
   path = /storage/backup
   browsable = yes
   read only = no
   directory mask = 0777
   create mask = 0777
   valid users = will
```

Create a password for the `will` samba user (this can be a different password than is used for the `will` linux user)

`sudo smbpasswd -a will`

Restart SMB

`sudo systemctl restart smb`
