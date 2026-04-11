## Лабороторная работ 1. Обновление ядра системы.

### Цель.

Научиться обновлять ядро в ОС Linux.

Используем ОС Ubuntu 24.04

Проверим версию ОС.

```
evs@ubuntu-srv01:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

проверим текущую версию ядра
```
evs@ubuntu-srv01:~$ uname -r
6.8.0-107-generic
```
Проверим архитектуру ОС
```
evs@ubuntu-srv01:~$ uname -p
x86_64
```
На сайе https://kernel.ubuntu.com/mainline выберем свежее ядро. Будем обновляться до версии v6.19.10.

Создаём папаку `kernel` и качаем туда ядро.
```
evs@ubuntu-srv01:~$ mkdir kernel && cd kernel
evs@ubuntu-srv01:~/kernel$
evs@ubuntu-srv01:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
--2026-04-11 18:49:59--  https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.75, 185.125.189.74, 185.125.189.76
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.75|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 167178432 (159M) [application/x-debian-package]
Saving to: ‘linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb’

linux-modules-6.19.10-061910-generic_6.19.10- 100%[=================================================================================================>] 159.43M  1.03MB/s    in 2m 26s

2026-04-11 18:52:26 (1.09 MB/s) - ‘linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb’ saved [167178432/167178432]

evs@ubuntu-srv01:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb
--2026-04-11 18:54:05--  https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.76, 185.125.189.74, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14632414 (14M) [application/x-debian-package]
Saving to: ‘linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb’

linux-headers-6.19.10-061910_6.19.10-061910.2 100%[=================================================================================================>]  13.95M  1.03MB/s    in 13s

2026-04-11 18:54:18 (1.09 MB/s) - ‘linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb’ saved [14632414/14632414]

evs@ubuntu-srv01:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
--2026-04-11 18:54:34--  https://kernel.ubuntu.com/mainline/v6.19.10/amd64/linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.74, 185.125.189.76, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.74|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4043216 (3.9M) [application/x-debian-package]
Saving to: ‘linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb’

linux-headers-6.19.10-061910-generic_6.19.10- 100%[=================================================================================================>]   3.86M  1.12MB/s    in 3.5s

2026-04-11 18:54:38 (1.10 MB/s) - ‘linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb’ saved [4043216/4043216]

```

Проыерим скаченные пакеты
```
evs@ubuntu-srv01:~/kernel$ ls -al
total 198128
drwxrwxr-x 2 evs evs      4096 Apr 11 18:58 .
drwxr-x--- 5 evs evs      4096 Apr 11 18:43 ..
-rw-rw-r-- 1 evs evs  14632414 Mar 25 14:25 linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb
-rw-rw-r-- 1 evs evs   4043216 Mar 25 14:24 linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
-rw-rw-r-- 1 evs evs  17008832 Mar 25 14:24 linux-image-unsigned-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
-rw-rw-r-- 1 evs evs 167178432 Mar 25 14:24 linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb
```

Устанавливаем все пакеты сразу
```
evs@ubuntu-srv01:~/kernel$ sudo dpkg -i *.deb
[sudo] password for evs:
Selecting previously unselected package linux-headers-6.19.10-061910.
(Reading database ... 87540 files and directories currently installed.)
Preparing to unpack linux-headers-6.19.10-061910_6.19.10-061910.202603251147_all.deb ...
Unpacking linux-headers-6.19.10-061910 (6.19.10-061910.202603251147) ...
Selecting previously unselected package linux-headers-6.19.10-061910-generic.
Preparing to unpack linux-headers-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb ...
Unpacking linux-headers-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
Selecting previously unselected package linux-image-unsigned-6.19.10-061910-generic.
Preparing to unpack linux-image-unsigned-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb ...
Unpacking linux-image-unsigned-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
Selecting previously unselected package linux-modules-6.19.10-061910-generic.
Preparing to unpack linux-modules-6.19.10-061910-generic_6.19.10-061910.202603251147_amd64.deb ...
Unpacking linux-modules-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
Setting up linux-headers-6.19.10-061910 (6.19.10-061910.202603251147) ...
Setting up linux-headers-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
Setting up linux-modules-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
Setting up linux-image-unsigned-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
I: /boot/vmlinuz is now a symlink to vmlinuz-6.19.10-061910-generic
I: /boot/initrd.img is now a symlink to initrd.img-6.19.10-061910-generic
Processing triggers for linux-image-unsigned-6.19.10-061910-generic (6.19.10-061910.202603251147) ...
/etc/kernel/postinst.d/initramfs-tools:
update-initramfs: Generating /boot/initrd.img-6.19.10-061910-generic
/etc/kernel/postinst.d/zz-update-grub:
Sourcing file `/etc/default/grub'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.19.10-061910-generic
Found initrd image: /boot/initrd.img-6.19.10-061910-generic
Found linux image: /boot/vmlinuz-6.8.0-107-generic
Found initrd image: /boot/initrd.img-6.8.0-107-generic
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```
Проверяем что ядро появилось в `/boot`
```
evs@ubuntu-srv01:~/kernel$ ls -al /boot/
total 204548
drwxr-xr-x  4 root root     4096 Apr 11 19:01 .
drwxr-xr-x 23 root root     4096 Apr  5 18:30 ..
-rw-r--r--  1 root root   306720 Mar 25 11:47 config-6.19.10-061910-generic
-rw-r--r--  1 root root   287601 Mar 13 13:27 config-6.8.0-107-generic
drwxr-xr-x  5 root root     4096 Apr 11 19:01 grub
lrwxrwxrwx  1 root root       33 Apr 11 19:01 initrd.img -> initrd.img-6.19.10-061910-generic
-rw-r--r--  1 root root 80521703 Apr 11 19:01 initrd.img-6.19.10-061910-generic
-rw-r--r--  1 root root 76332312 Apr  5 18:33 initrd.img-6.8.0-107-generic
lrwxrwxrwx  1 root root       28 Apr  5 18:32 initrd.img.old -> initrd.img-6.8.0-107-generic
drwx------  2 root root    16384 Apr  5 18:30 lost+found
-rw-------  1 root root 10815061 Mar 25 11:47 System.map-6.19.10-061910-generic
-rw-------  1 root root  9125925 Mar 13 13:27 System.map-6.8.0-107-generic
lrwxrwxrwx  1 root root       30 Apr 11 19:01 vmlinuz -> vmlinuz-6.19.10-061910-generic
-rw-------  1 root root 16978432 Mar 25 11:47 vmlinuz-6.19.10-061910-generic
-rw-------  1 root root 15042952 Mar 13 17:46 vmlinuz-6.8.0-107-generic
lrwxrwxrwx  1 root root       25 Apr  5 18:32 vmlinuz.old -> vmlinuz-6.8.0-107-generic
```

Перезагружаемся и проверяем версию ядра после перезагрузки.
```
evs@ubuntu-srv01:~$ uname -r
6.19.10-061910-generic
```

Итог. Ядро успешно обновилось.
