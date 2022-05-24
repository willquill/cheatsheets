# LVM

## Using LVM in Ubuntu 18.04 when it only gives 4 GB starting size

```sh
root@lyra:/tank# sudo pvs
  PV         VG        Fmt  Attr PSize   PFree
  /dev/sda3  ubuntu-vg lvm2 a--  <79.00g <75.00g
root@lyra:/tank# sudo vgs
  VG        #PV #LV #SN Attr   VSize   VFree
  ubuntu-vg   1   1   0 wz--n- <79.00g <75.00g
root@lyra:/tank# sudo lvs
  LV        VG        Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- 4.00g
root@lyra:/tank# sudo lvextend -L +75G /dev/ubuntu-vg/ubuntu-lv
  Insufficient free space: 19200 extents needed, but only 19199 available
root@lyra:/tank# sudo lvextend -L +74.99G /dev/ubuntu-vg/ubuntu-lv
  Rounding size to boundary between physical extents: 74.99 GiB.
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 4.00 GiB (1024 extents) to 78.99 GiB (20222 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
root@lyra:/tank# resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 10
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 20707328 (4k) blocks long.
root@lyra:/tank# df -h /
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   78G  2.1G   73G   3% /
root@lyra:/tank#
```