如何解决报错：Device /dev/sdc excluded by a filter

```bash
$  lsblk -d -o name,rota
NAME ROTA
sda     1
sdb     1
sdc     1
sr0     1

$ grep ^ /sys/block/*/queue/rotational
/sys/block/sda/queue/rotational:1
/sys/block/sdb/queue/rotational:1
/sys/block/sdc/queue/rotational:1
/sys/block/sr0/queue/rotational:1

$ fdisk -l /dev/sdc
Disk /dev/sdc: 500 GiB, 536870912000 bytes, 1048576000 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2287FE7F-E2C6-4D8B-96B5-9EA2317B7FA0


$ pvcreate /dev/sdc
  Device /dev/sdc excluded by a filter.

$  parted  /dev/sdc
GNU Parted 3.3
Using /dev/sdc
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel msdos                                                    
Warning: The existing disk label on /dev/sdc will be destroyed and all data on this disk will be lost. Do you
want to continue?
Yes/No? yes                                                               
(parted) quit                                                             
Information: You may need to update /etc/fstab.

$  pvcreate /dev/sdc                  
WARNING: dos signature detected on /dev/sdc at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdc.
  Physical volume "/dev/sdc" successfully created.

```

