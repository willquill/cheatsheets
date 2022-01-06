# ESXi Cheatsheet

I don't even know what I was doing here...it was a long time ago.

```sh
[root@indus:~] cd /vmfs/volumes/ds-ind1/
[root@indus:/vmfs/volumes/59e0a3c9-72a8fef8-2e7b-002590a1ad14
```

`esxcli software vib update -d "/vmfs/volumes/59e0a3c9-72a8fef8-2e7b-002590a1ad14/ESXi650-201710001.zip"`

http://www.virten.net/2016/11/usb-devices-as-vmfs-datastore-in-vsphere-esxi-6-5/

```sh
[root@wasat:~] ls /dev/disks/
mpx.vmhba34:C0:T0:L0                    vml.0000000000766d68626133323a303a30
mpx.vmhba34:C0:T0:L0:1                  vml.0000000000766d68626133323a303a30:1
mpx.vmhba34:C0:T0:L0:5                  vml.0000000000766d68626133343a303a30
mpx.vmhba34:C0:T0:L0:6                  vml.0000000000766d68626133343a303a30:1
mpx.vmhba34:C0:T0:L0:7                  vml.0000000000766d68626133343a303a30:5
mpx.vmhba34:C0:T0:L0:8                  vml.0000000000766d68626133343a303a30:6
mpx.vmhba34:C0:T0:L0:9                  vml.0000000000766d68626133343a303a30:7
naa.2020030102060804                    vml.0000000000766d68626133343a303a30:8
naa.2020030102060804:1                  vml.0000000000766d68626133343a303a30:9
```

```sh
naa.2020030102060804
1920 255 63 30851072 30844799
```

`partedUtil mklabel /dev/disks/naa.2020030102060804 gpt`

`partedUtil setptbl /dev/disks/naa.2020030102060804 gpt "1 2048 30844799 AA31E02A400F11DB9590000C2911D1B8 0"`

`vmkfstools -C vmfs6 -S ds-usb1 /dev/disks/naa.2020030102060804`

`partedUtil mklabel /dev/disks/naa.2020030102060804:1 gpt`

`partedUtil setptbl /dev/disks/naa.2020030102060804:1 gpt "1 2048 30844799 AA31E02A400F11DB9590000C2911D1B8 0"`

`vmkfstools -C vmfs6 -S ds-usb2 /dev/disks/naa.2020030102060804:1`

`partedUtil mklabel /dev/disks/t10.SanDisk_Cruzer_Blade____20060571710F3D10F9D0 gpt`

`partedUtil getptbl /dev/disks/t10.SanDisk_Cruzer_Blade____20060571710F3D10F9D0`

`eval expr $(partedUtil getptbl /dev/disks/t10.SanDisk_Cruzer_Blade____20060571710F3D10F9D0 | tail -1 | awk '{print $1 " \\* " $2 " \\* " $3}') - 1`

```sh
515969
15631244 #the right one with calculation?
```

`partedUtil setptbl /dev/disks/t10.SanDisk_Cruzer_Blade____20060571710F3D10F9D0 gpt "1 2048 15631244 AA31E02A400F11DB9590000C2911D1B8 0"`

`vmkfstools -C vmfs6 -S ds-usb1 /dev/disks/t10.SanDisk_Cruzer_Blade____20060571710F3D10F9D0:1`

`partedUtil mklabel /dev/disks/mpx.vmhba38\:C0\:T0\:L0 gpt`

`eval expr $(partedUtil getptbl /dev/disks/mpx.vmhba38:C0:T0:L0 | tail -1 | awk '{print $1 " \\* " $2 " \\* " $3}') - 1`

`partedUtil getptbl /dev/disks/mpx.vmhba36:C0:T0:L0`

`942 255 63 15133248`

`30282524`

`partedUtil setptbl /dev/disks/mpx.vmhba38\:C0\:T0\:L0 gpt "1 2048 30282524 AA31E02A400F11DB9590000C2911D1B8 0"`

`vmkfstools -C vmfs6 -S ds-usb2 /dev/disks/mpx.vmhba38\:C0\:T0\:L0:1`

`partedUtil delete "/vmfs/devices/disks/ " 0`
