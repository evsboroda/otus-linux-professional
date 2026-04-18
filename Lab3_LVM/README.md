## Лабороторная работ 3. работа с LVM.

### Задание.

- Уменьшить том под / до 8G.
- Выделить том под /home.
- Выделить том под /var - сделать в mirror.
- /home - сделать том для снапшотов.
- Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
- Работа со снапшотами:
- сгенерить файлы в /home/;
- снять снапшот;
- удалить часть файлов;
- восстановиться со снапшота.

#### 1. Создание LM.

Проверим диски.
```
evs@ubuntu-srv01:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
```
```
root@ubuntu-srv01:~# lvmdiskscan
  /dev/sda2 [       1.77 GiB]
  /dev/sda3 [     <18.23 GiB] LVM physical volume
  /dev/sdb  [      10.00 GiB]
  /dev/sdc  [       2.00 GiB]
  /dev/sdd  [       1.00 GiB]
  /dev/sde  [       1.00 GiB]
  4 disks
  1 partition
  0 LVM physical volume whole disks
  1 LVM physical volume
```
у нас есть четыре диска: sdb, sdc, sdd и sde.

Создадим PV на диске sdb командой `pvcreate`.
```
root@ubuntu-srv01:~# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```
Создадим Volume group с именем lab3 на созданом PV.
```
root@ubuntu-srv01:~# vgcreate lab3 /dev/sdb
  Volume group "lab3" successfully created
```
Создадим Logical Volume  размером 80% от Volume group *lab3* с именем *test* командой `lvcreate -l+80%FREE -n test lab3`
```
root@ubuntu-srv01:~# lvcreate -l+80%FREE -n test lab3
  Logical volume "test" created.
```
Проверим созданный LV
```
root@ubuntu-srv01:~# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0  1.8G  0 part /boot
└─sda3                      8:3    0 18.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   10G  0 lvm  /
sdb                         8:16   0   10G  0 disk
└─lab3-test               252:1    0    8G  0 lvm
sdc                         8:32   0    2G  0 disk
sdd                         8:48   0    1G  0 disk
sde                         8:64   0    1G  0 disk
sr0                        11:0    1 1024M  0 rom
```
Выведим всю информацию о VG командой `vgdisplay`
```
root@ubuntu-srv01:~# vgdisplay test
  Volume group "test" not found
  Cannot process volume group test
root@ubuntu-srv01:~# vgdisplay lab3
  --- Volume group ---
  VG Name               lab3
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               4FHt9e-JZcx-3LPt-o2jH-nfYO-4Z7P-qN1Ozl

root@ubuntu-srv01:~# vgdisplay -v lab3
  --- Volume group ---
  VG Name               lab3
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <10.00 GiB
  PE Size               4.00 MiB
  Total PE              2559
  Alloc PE / Size       2047 / <8.00 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               4FHt9e-JZcx-3LPt-o2jH-nfYO-4Z7P-qN1Ozl

  --- Logical volume ---
  LV Path                /dev/lab3/test
  LV Name                test
  VG Name                lab3
  LV UUID                ARrksO-Unfy-IiOW-Ird5-3Kin-Hw8w-xts7dE
  LV Write Access        read/write
  LV Creation host, time ubuntu-srv01, 2026-04-18 14:18:22 +0000
  LV Status              available
  # open                 0
  LV Size                <8.00 GiB
  Current LE             2047
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     16384
  Block device           252:1

  --- Physical volumes ---
  PV Name               /dev/sdb
  PV UUID               9ffNLr-mogn-6vqa-yuKa-fhdE-ntHi-qUshCq
  PV Status             allocatable
  Total PE / Free PE    2559 / 512
```

Проверим диски которые входят в VG.
```
root@ubuntu-srv01:~# vgdisplay -v lab3 | grep  'PV Name'
  PV Name               /dev/sdb
```
Посмотрим информацию о LV.
```
root@ubuntu-srv01:~# lvdisplay /dev/lab3/test
  --- Logical volume ---
  LV Path                /dev/lab3/test
  LV Name                test
  VG Name                lab3
  LV UUID                ARrksO-Unfy-IiOW-Ird5-3Kin-Hw8w-xts7dE
  LV Write Access        read/write
  LV Creation host, time ubuntu-srv01, 2026-04-18 14:18:22 +0000
  LV Status              available
  # open                 0
  LV Size                <8.00 GiB
  Current LE             2047
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     16384
  Block device           252:1
```
В сжатом виде можно проверить командами `lvs` и `pvs`

Создаим ещё один LV с абсолютным значением 100M опцией -L от свободного остатка места на VG *lab3*
```
root@ubuntu-srv01:~# lvcreate -L100M -n small lab3
  Logical volume "small" created.
```
Проверим.
```
root@ubuntu-srv01:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  small     lab3      -wi-a----- 100.00m
  test      lab3      -wi-a-----  <8.00g
  ubuntu-lv ubuntu-vg -wi-ao----  10.00g
```
Создадим файловую систему *ext4* на LV *test*
```
oot@ubuntu-srv01:~# mkfs.ext4 /dev/lab3/test
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 2096128 4k blocks and 524288 inodes
Filesystem UUID: cf8b9ba1-e1a5-4953-b7fe-6801c24c3fab
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

#### 2. Расширение LM.

Расширим файловую систему на LV /dev/lab3/test за счет нового
блочного устройства /dev/sdc.

Создадим PV на диске *sdc*
```
root@ubuntu-srv01:~# pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```
Расширим Volume Group lab3 засчёт нового PV
```
root@ubuntu-srv01:~# vgextend lab3 /dev/sdc
  Volume group "lab3" successfully extended
```
Проверим
```
root@ubuntu-srv01:~# vgdisplay -v lab3 | grep 'PV Name'
  PV Name               /dev/sdb
  PV Name               /dev/sdc
```

```
root@ubuntu-srv01:~# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  lab3        2   2   0 wz--n- 11.99g <3.90g
  ubuntu-vg   1   1   0 wz--n- 18.22g  8.22g
```
- Видим что диск добавился и наша Volume Group увеличилась и её размер теперь *sdb+sdc=12G*



