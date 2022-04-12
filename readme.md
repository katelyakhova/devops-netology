# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.

**A:** Разреженный файл - такой, у которого вместо последовательности нулевых байт хранится информация о дырах. Полезно для торрентов, образы дисков для виртуальных машин

2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?

**A:** Нет, не могут, т.к. у них один и тот же inode. Повторила пример из лекции:

```bash
vagrant@vagrant:~$ touch test_file
vagrant@vagrant:~$ ln test_file test_link
vagrant@vagrant:~$ stat test_file ; stat test_link 
  File: test_file
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 1053986     Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
Access: 2022-04-11 18:52:32.459571138 +0000
Modify: 2022-04-11 18:52:32.459571138 +0000
Change: 2022-04-11 18:53:12.123394600 +0000
 Birth: -
  File: test_link
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 1053986     Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
Access: 2022-04-11 18:52:32.459571138 +0000
Modify: 2022-04-11 18:52:32.459571138 +0000
Change: 2022-04-11 18:53:12.123394600 +0000
 Birth: -
vagrant@vagrant:~$ ls -lh
total 0
-rw-rw-r-- 2 vagrant vagrant  0 Apr 11 18:52  test_file
-rw-rw-r-- 2 vagrant vagrant  0 Apr 11 18:52  test_link
vagrant@vagrant:~$ chmod o+x test_link 
vagrant@vagrant:~$ ls -lh
total 0
-rw-rw-r-x 2 vagrant vagrant  0 Apr 11 18:52  test_file
-rw-rw-r-x 2 vagrant vagrant  0 Apr 11 18:52  test_link
```

3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.

**A:** Готово

```bash
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.4M  1 loop /snap/core18/2128
loop1                       7:1    0 32.3M  1 loop /snap/snapd/12704
loop2                       7:2    0 70.3M  1 loop /snap/lxd/21029
loop3                       7:3    0 43.6M  1 loop /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop /snap/core20/1405
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
sdc                         8:32   0  2.5G  0 disk
```

4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.

**A:**
```bash
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.4M  1 loop /snap/core18/2128
loop2                       7:2    0 70.3M  1 loop /snap/lxd/21029
loop3                       7:3    0 43.6M  1 loop /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop /snap/lxd/22753
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
├─sdb1                      8:17   0    2G  0 part 
└─sdb2                      8:18   0  511M  0 part 
sdc                         8:32   0  2.5G  0 disk 
```

5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.

**A:**
```bash
vagrant@vagrant:~$ sudo sfdisk -d /dev/sdb | sudo sfdisk --force /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0xaef9f5d5.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0xaef9f5d5

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0 55.4M  1 loop /snap/core18/2128
loop2                       7:2    0 70.3M  1 loop /snap/lxd/21029
loop3                       7:3    0 43.6M  1 loop /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop /snap/lxd/22753
sda                         8:0    0   64G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   63G  0 part 
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm  /
sdb                         8:16   0  2.5G  0 disk 
├─sdb1                      8:17   0    2G  0 part 
└─sdb2                      8:18   0  511M  0 part 
sdc                         8:32   0  2.5G  0 disk 
├─sdc1                      8:33   0    2G  0 part 
└─sdc2                      8:34   0  511M  0 part 

```

6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.

```bash
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
loop3                       7:3    0 43.6M  1 loop  /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop  /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
```

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

```bash
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
loop3                       7:3    0 43.6M  1 loop  /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop  /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 


```

8. Создайте 2 независимых PV на получившихся md-устройствах.

```bash
vagrant@vagrant:~$ sudo pvcreate /dev/md0
  Physical volume "/dev/md0" successfully created.
vagrant@vagrant:~$ sudo pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.

```

9. Создайте общую volume-group на этих двух PV.

```bash
vagrant@vagrant:~$ sudo vgcreate vgmd0 /dev/md{0,1}
  Volume group "vgmd0" successfully created
```

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

```bash
vagrant@vagrant:~$ sudo lvcreate -L 100M vgmd0 /dev/md0
  Logical volume "lvol1" created.
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
loop3                       7:3    0 43.6M  1 loop  /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop  /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vgmd0-lvol1         253:1    0  100M  0 lvm   
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vgmd0-lvol1         253:1    0  100M  0 lvm   
```

11. Создайте `mkfs.ext4` ФС на получившемся LV.

```bash
vagrant@vagrant:~$ sudo mkfs.ext4 -F /dev/vgmd0/lvol1 
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

```bash
vagrant@vagrant:~$ mkdir -p /tmp/new
vagrant@vagrant:~$ sudo mount /dev/vgmd0/lvol1 /tmp/new/
```

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

```bash
vagrant@vagrant:~$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2022-04-12 19:02:23--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22356784 (21M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz    100%[===================>]  21.32M  19.1MB/s    in 1.1s    

2022-04-12 19:02:24 (19.1 MB/s) - ‘/tmp/new/test.gz’ saved [22356784/22356784]

vagrant@vagrant:~$ ls -l /tmp/new/
total 21852
drwx------ 2 root root    16384 Apr 12 19:01 lost+found
-rw-r--r-- 1 root root 22356784 Apr 12 17:36 test.gz
```

14. Прикрепите вывод `lsblk`.

```bash
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
loop3                       7:3    0 43.6M  1 loop  /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop  /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vgmd0-lvol1         253:1    0  100M  0 lvm   /tmp/new
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
    └─vgmd0-lvol1         253:1    0  100M  0 lvm   /tmp/new
```

15. Протестируйте целостность файла:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```

```bash
vagrant@vagrant:~$ gzip -t /tmp/new/test.gz
vagrant@vagrant:~$ echo $?
0
```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

```bash
vagrant@vagrant:~$ sudo pvmove -v /dev/md0
  Archiving volume group "vgmd0" metadata (seqno 3).
  Creating logical volume pvmove0
  activation/volume_list configuration setting not defined: Checking only host tags for vgmd0/lvol1.
  Moving 25 extents of logical volume vgmd0/lvol1.
  activation/volume_list configuration setting not defined: Checking only host tags for vgmd0/lvol1.
  Creating vgmd0-pvmove0
  Loading table for vgmd0-pvmove0 (253:2).
  Loading table for vgmd0-lvol1 (253:1).
  Suspending vgmd0-lvol1 (253:1) with device flush
  Resuming vgmd0-pvmove0 (253:2).
  Resuming vgmd0-lvol1 (253:1).
  Creating volume group backup "/etc/lvm/backup/vgmd0" (seqno 4).
  activation/volume_list configuration setting not defined: Checking only host tags for vgmd0/pvmove0.
  Checking progress before waiting every 15 seconds.
  /dev/md0: Moved: 20.00%
  /dev/md0: Moved: 100.00%
  Polling finished successfully.
vagrant@vagrant:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
loop0                       7:0    0 70.3M  1 loop  /snap/lxd/21029
loop2                       7:2    0 55.4M  1 loop  /snap/core18/2128
loop3                       7:3    0 43.6M  1 loop  /snap/snapd/15177
loop4                       7:4    0 55.5M  1 loop  /snap/core18/2344
loop5                       7:5    0 61.9M  1 loop  /snap/core20/1405
loop6                       7:6    0 67.8M  1 loop  /snap/lxd/22753
sda                         8:0    0   64G  0 disk  
├─sda1                      8:1    0    1M  0 part  
├─sda2                      8:2    0    1G  0 part  /boot
└─sda3                      8:3    0   63G  0 part  
  └─ubuntu--vg-ubuntu--lv 253:0    0 31.5G  0 lvm   /
sdb                         8:16   0  2.5G  0 disk  
├─sdb1                      8:17   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
│   └─vgmd0-lvol1         253:1    0  100M  0 lvm   /tmp/new
└─sdb2                      8:18   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 
sdc                         8:32   0  2.5G  0 disk  
├─sdc1                      8:33   0    2G  0 part  
│ └─md1                     9:1    0    2G  0 raid1 
│   └─vgmd0-lvol1         253:1    0  100M  0 lvm   /tmp/new
└─sdc2                      8:34   0  511M  0 part  
  └─md0                     9:0    0 1018M  0 raid0 

```

17. Сделайте `--fail` на устройство в вашем RAID1 md.

```bash
vagrant@vagrant:~$ sudo mdadm /dev/md1 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md1
vagrant@vagrant:~$ sudo mdadm --detail /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Tue Apr 12 18:58:21 2022
        Raid Level : raid1
        Array Size : 2094080 (2045.00 MiB 2144.34 MB)
     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Tue Apr 12 19:28:35 2022
             State : clean, degraded 
    Active Devices : 1
   Working Devices : 1
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : vagrant:1  (local to host vagrant)
              UUID : d570961f:fab5c6e5:b6973162:5950adef
            Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       33        0      active sync   /dev/sdc1
       -       0        0        1      removed

       1       8       17        -      faulty   /dev/sdb1
```

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

```bash
vagrant@vagrant:~$ dmesg | grep md1
[  468.321665] md/raid1:md1: not clean -- starting background reconstruction
[  468.321668] md/raid1:md1: active with 2 out of 2 mirrors
[  468.321696] md1: detected capacity change from 0 to 2144337920
[  468.327248] md: resync of RAID array md1
[  478.907103] md: md1: resync done.
[ 2280.645069] md/raid1:md1: Disk failure on sdb1, disabling device.
               md/raid1:md1: Operation continuing on 1 devices.
```

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

     ```bash
     root@vagrant:~# gzip -t /tmp/new/test.gz
     root@vagrant:~# echo $?
     0
     ```

Выполнено
```bash
vagrant@vagrant:~$ gzip -t /tmp/new/test.gz
vagrant@vagrant:~$ echo $?
0
```

20. Погасите тестовый хост, `vagrant destroy`.

```bash
$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
```