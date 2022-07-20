# Linux Cheatsheet

## `find`

Find the largest file in /var

`find /var/log -type f -exec du -m {} \;|sort -nr|head`

Find largest directories in root

`sudo du -h / --max-depth=1`

Find largest directories in current directory

`sudo du -h . --max-depth=1`

Show largest "N" files in a directory

`du -ak /var | sort -rn | head -n <N>`

`du -ak . | sort -rn | head -n 15`

Find the largest file in /var:

`find /var -type f -exec du -m {} \;|sort -nr|head`

*`{}` expands the name of file and `\;` ends the expression*

This moves the first 91 files from current directory to `/mnt/media/movies`

`find . -maxdepth 1 -type f |head -91|xargs cp -t "/mnt/media/movies"`

Find and remove files larger than 16 GB

`find . -size +16G -delete`

## `du`

Check size of current directory

`du -sh`

Disk space usage by directory

`du / |sort -nr > diskusage.txt`

Exclude a directory from `du -sh`

`du -sh --exclude=./tank/media/tv/Stargate\ SG-1 /tank/media/tv`

## `rsync`

Copy files recursively, showing progress

`rsync -Pavr /tank/media/movies /mnt/media/movies`

Copy files recursively, showing progress, do not copy a source if it already exists at destination

`rsync -Pavr --ignore-existing /mnt/media/movies /mnt/seagate6/`

## `screen`

Create a screen session named `rsync`

`screen -S rsync`

Detach screen with Ctrl+a + `d`

Re-attach with `screen -d -r`

Kill screen

`screen -X -S [session # you want to kill] quit`

## `diff`

Find the difference between two directories method #1

`diff <(cd /mnt/media/movies; find .) <(cd /mnt/seagate6/movies; find .)`

Find the difference between two directories method #2

`find /mnt/media/movies/ -type f -exec md5sum {} \; > dir1.txt`

`find /mnt/seagate6/movies/ -type f -exec md5sum {} \; > dir2.txt`

Then compare the result two files with "diff":

`diff dir1.txt dir2.txt`

This strategy is very useful when the two directories to be compared are not in the same machine and you need to make sure that the files are equal in both directories.

## `nmap`

Verify which ports are listening

`sudo nmap -sT -O localhost`

## `top`

To view only the processes owned by a specific user, use the following command:

`top -U [USERNAME]`

## SSH

### Generating ECDSA key

`ssh-keygen -f ~/.ssh/nov19_ecdsa -t ecdsa -b 521`

### Permissions of key directories

```txt
.ssh directory: 700 (drwx------)
public key (.pub file): 644 (-rw-r--r--)
private key (id_rsa): 600 (-rw-------)
```

Lastly your home directory should not be writeable by the group or others (at most 755 (drwxr-xr-x)).

Home folder: group can read, user can write.

```sh
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/*.pub
```

### Disable root SSH login

For all of the below commands, you can preface it with `sed -i` and omit `> temp.txt` in order to perform an "in place" modification.

Find PermitRootLogin, remove octothorpe if there is one, replace after with no

`sed 's/#\?\(PermitRootLogin\s*\).*$/\1 no/' /etc/ssh/sshd_config > temp.txt`

`mv -f temp.txt /etc/ssh/sshd_config`

Set GSSAPIAuthentication to no

`sed 's/#\?\(GSSAPIAuthentication\s*\).*$/\1 no/' /etc/ssh/sshd_config > temp.txt`

`mv -f temp.txt /etc/ssh/sshd_config`

Set interval to 120

`sed 's/#\?\(ClientAliveInterval\s*\).*$/\1 120/' /etc/ssh/sshd_config > temp.txt`

`mv -f temp.txt /etc/ssh/sshd_config`

Set countmax to 720

`sed 's/#\?\(ClientAliveCountMax\s*\).*$/\1 720/' /etc/ssh/sshd_config > temp.txt`

`mv -f temp.txt /etc/ssh/sshd_config`

Do all of the above all at once

`sudo sed 's/#\?\(PermitRootLogin\s*\).*$/\1no/; s/#\?\(GSSAPIAuthentication\s*\).*$/\1no/; s/#\?\(ClientAliveInterval\s*\).*$/\1120/; s/#\?\(ClientAliveCountMax\s*\).*$/\1720/' \/etc/ssh/sshd_config > temp.txt`

`sudo mv -f temp.txt /etc/ssh/sshd_config && sudo systemctl restart sshd`

## Networking

### Bond Interfaces

#### Debian Bond

`sudo vi /etc/network/interfaces`

Create a bond0 network interface

```txt
address 10.1.20.24
gateway 10.1.20.1
netmask 255.255.255.0
dns-nameservers 10.1.20.1
dns-search paw.blue
bond-slaves enp1s0f1 enp1s0f0
bond-mode 4
bond-miimon 100
bond-downdelay 200
bond-updelay 200
```

`auto bond0`

`iface bond0 inet static`

`post-up ifenslave bond0 enp1s0f0 enp1s0f1`

`pre-down ifenslave -d bond0 enp1s0f0 enp1s0f1`

`auto enp1s0f0`

`iface enp1s0f0 inet manual`

`bond-master bond0`

`auto enp1s0f1`

`iface enp1s0f1 inet manual`

`bond-master bond0`

#### Ubuntu 16.04 Bond

`sudo ip link show`

`sudo nano /etc/network/interfaces`

```sh
# The loopback network interface
auto lo
iface lo inet loopback

# The aggregated link
auto bond0
iface bond0 inet static
        address 10.1.20.20
        gateway 10.1.20.1
        netmask 255.255.255.0
        dns-nameservers 10.1.20.1
        dns-search paw.blue
        bond-mode 4
        bond-miimon 100
        bond-downdelay 50
        bond-updelay 50
        bond-lacp-rate fast
        bond-slaves enp1s0f0 enp1s0f1
        bond-xmit_hash_policy 1

auto enp1s0f0
iface enp1s0f0 inet manual
bond-master bond0

auto enp1s0f1
iface enp1s0f1 inet manual
bond-master bond0
```

#### Ubuntu 18.04 Bond

[Source](https://cli.pignat.org/server-18.04-network-bond.html)

`sudo ip link show`

`sudo vi /etc/netplan/whatever-it-is.yml`

```yml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0f0:
    dhcp4: false
    dhcp6: false
    optional: true
  enp1s0f1:
    dhcp4: false
    dhcp6: false
    optional: true
  bonds:
    bond0:
      dhcp4: false
    dhcp6: false
      interfaces: 
      - enp1s0f0
      - enp1s0f1
      addresses: [10.1.20.24/24]
      gateway4: 10.1.20.1
    nameservers:
      search: paw.blue
        addresses: [10.1.20.1, 1.1.1.1]
      parameters:
        mode: 802.3ad
```

Update the network interface with the following:

`cloud-init clean -reboot`

### Network interface troubleshooting

You can do the following to remove an IP from an interface. For example, I accidentally assigned 10.1.20.24/24 to enp1s0f1 but I want that IP on enp1s0f0 instead:

`sudo ip address del 10.1.20.20/24 dev enp1s0f1`

You can do the following to restart an interface:

`sudo ip link set enp1s0f1 down && sudo ip link set enp1s0f1 up`

Fix 127.0.0.53 being in resolv.conf:

`sudo rm -f /etc/resolv.conf`

## Samba

`sudo apt install samba-common-bin samba samba-common python-glade2 system-config-samba -y`

`sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak`

`sudo nano /etc/samba/smb.conf`

```sh
#============================ Global definition ================================

[global]
workgroup = LYON
server string = Samba Server %v
netbios name = lyra
security = user
map to guest = bad user
name resolve order = bcast host
dns proxy = no
bind interfaces only = yes

#============================ Share Definitions ==============================

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

Then do this after doing that file:

`sudo smbpasswd -a will`

`sudo service smbd restart`

## SNMP

### Ubuntu SNMP

`sudo apt install snmpd`

Edit community and edit agent string

`sudo vim /etc/snmp/snmpd.conf`

`sudo /etc/init.d/snmpd restart`

### Centos SNMP

`sudo yum install net-snmp`

Edit community

`sudo vim /etc/snmp/snmpd.conf`

`systemctl enable snmpd`

`sudo service snmpd restart`

## Partitioning

### Create ext4 partition

```sh
sudo umount /dev/sda1
sudo gdisk
/dev/sda
o
n
w
sudo mkfs.ext4 /dev/sda1
fsck -N /dev/sda1
```

## Permissions

Check UID and GID of a user

`id nova`

Change UID of a user to 1420

`usermod -u 1420 nova`

Change GID of a group to 1420

`groupmod -g 1420 nova`

Replace file ownership old UID (1000) and old GID (1234) with the new one for both (1420)

`find / -user 1000 -exec chown -h 1420 {} \;`

`find / -group 1000 -exec chgrp -h 1420 {} \;`

Create new group and assign UID of 1420

`groupadd -g 1420 nova`

Make UID of media user be 1420

`usermod -u 1420 media`

Change current directory user/group recursively:

`chown -R media:media .`

If you truly want any user to have full permissions on a directory recursively

`chmod -R a+rwx /mnt/downloads` or `chmod -R 777 /mnt/downloads`

Create a new group and add `auth` user to the new `media` group

`sudo groupadd media`

`sudo usermod -aG media auth` or `sudo adduser auth media`

If you want to change the user owning this file or directory, you will have to use the command `chown`. For instance, if you run:

`sudo chown username: myfolder`

Then the user owning myfolder will be username. Then you can execute `sudo chmod u+w myfolder` to add the write permission for the user who owns the folder.

If you want to add this user to the group associated with "myfolder", you can run:

`sudo usermod -a -G groupname username`

...and then execute `sudo chmod g+w myfolder` to add the write permission to the group.

This sets permissions for specific users, without changing the ownership of the directory.

`setfacl -m u:nova:rwx /etc/openvpn`

### Change permissions with user group all values

```table
user group global
rwx  rwx   rwx
```

current:

`rwx --- ---`

change to:

`rw- r-- r--`

what is changing:

```txt
user is losing execute
group is adding read
global is adding read
```

`sudo chmod -R u-x,g+r,a+r /my/path/`

Disable firewall in Centos

`systemctl disable firewalld`

`systemctl stop firewalld`

Verify: `systemctl status firewalld`

Show all groups

`cut -d: -f1 /etc/group`

wrap lines

`journalctl -xe --no-pager`

## Miscellanous

### Replacing

Replaces spaces in the file names with periods

```sh
for file in *; do mv "$file" `echo $file | tr ' ' '.'` ; done
```

This replaces homelab.city with will.mx

`egrep -lRZ 'homelab.city' . | xargs -0 -l sed -i -e 's/homelab.city/will.mx/g`

### Finding

Find text in a file ([source](https://stackoverflow.com/questions/16956810/how-do-i-find-all-files-containing-specific-text-on-linux))

`grep -rnw '/var/www/html/' -e 'punkwolf'`
