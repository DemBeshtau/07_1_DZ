# Работа с загрузчиком #
1. Попасть систему без пароля несколькими способами;<br/>
2. Установить систему с LVM, после чего переименовать VG;<br/>
3. Добавить модуль в initrd.<br/>
### Исходные данные ###
&ensp;&ensp;ПК на Linux c 8 ГБ ОЗУ или виртуальная машина (ВМ) с включенной Nested Virtualization.<br/>
&ensp;&ensp;Предварительно установленное и настроенное ПО:<br/>
&ensp;&ensp;&ensp;Hashicorp Vagrant (https://www.vagrantup.com/downloads);<br/>
&ensp;&ensp;&ensp;Oracle VirtualBox (https://www.virtualbox.org/wiki/Linux_Downloads).<br/>
&ensp;&ensp;&ensp;Все действия проводились с использованием Vagrant 2.4.0, VirtualBox 7.0.14 и образа<br/> 
&ensp;&ensp;CentOS 7 версии 1804_2
### Ход решения ###
### 1. Попасть в систему несколькими способами ###
&ensp;&ensp;Для выполнения задания необходимо сконфигурировать Vagrant таким образом, чтобы после старта ВМ<br/>
открывалось окно Virtual Box с работающей системой. Для этого в блок provider Vagrant-файла добавить строку<br/> 
vm.gui = true. После старта ВМ в меню GRUB, отображающем строки вариантов загрузки системы, необходимо <br/>
нажать "e" (edit) для того, чтобы отредактировать параметры загрузки. <br/>

- Способ № 1<br/>
&ensp;&ensp; В конце строки, начинающейся с linux 16, удалить параметры rhgb и quiet, затем добавить rd.break<br/>
(переход в командную строку в конце обработки initramfs) и enforcing=0 (загрузка SELinux в permissive режиме).<br/>
Для запуска системы с текущей конфигурацией необходимо нажать ctrl+x.<br/>
&ensp;&ensp;После загрузки консоли, необходимо перемонтировать корневую фаловую систему в режим чтения-записи<br/>
войти в корневую файловую систему, задать пароль root, обновить параметры системы SELinux и перезагрузить систему:<br/>
```shell
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
...
touch /.autoreleable
exit
```
&ensp;&ensp;Чтобы осуществлять вход в систему, необходимо восстановить контекст безопасности файла /etc/shadow.<br/>
Для этого необходимо при загрузке снова перевести SELinux в режим permissive (enforcing=0) и далее выполнить<br/>
ряд команд:
```shell
restorecon /etc/shadow
setenforce 1
```
- Способ № 2<br/>
&ensp;&ensp; В конце строки, начинающейся с linux 16, удалить параметры rhgb и quiet, затем вместо параметра ro<br/>
указать,rw (монтирование /sysroot в режиме чтение-запись) и после него добавить init=/sysroot/bin/sh (запуск консоли).<br/>
Для запуска системы с текущей конфигурацией необходимо нажать ctrl+x.<br/>
&ensp;&ensp;После загрузки консоли, необходимо войти в корневую файловую систему, задать пароль root, обновить<br/>
параметры системы SELinux и перезагрузить систему:<br/>
```shell
chroot /sysroot
passwd root
...
touch /.autoreleable
exit
```
&ensp;&ensp;Чтобы осуществлять вход в систему, необходимо восстановить контекст безопасности файла /etc/shadow.<br/>
Для этого необходимо при загрузке снова перевести SELinux в режим permissive (enforcing=0) и далее выполнить<br/>
ряд команд:
```shell
restorecon /etc/shadow
setenforce 1
```

&ensp;&ensp;Стоит отметить что, способ с добавлением в параметры загрузки только init=/bin/sh пригоден для debian-like <br/>
дистрибутивов (работатет с Ubuntu 22.04), и, возможно, других. Использование данного способа для запуска CentOS <br/>
вызывало сбой в работе ядра (kernel panic).<br/>
&ensp;&ensp;Перечисленные способы в целом идентичны, отличия заключаются в режимах монтирования корневой файловой системы. <br/>
Если используется режим ro, то после загрузки консоли, необходимо перемонтировать корень в режим rw, для того, чтобы все<br/>
планируемые изменения были применены. При использовании CentOS необходимо учитывать аспекты, связанные с работой SELinux.
### 2. Установить систему с LVM, после чего переименовать VG ###
2.1. Просмотр исходной дисковой инфраструктуры:<br/>
```shell
[root@grub-test ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
|-sda1                    8:1    0    1M  0 part 
|-sda2                    8:2    0    1G  0 part /boot
`-sda3                    8:3    0   39G  0 part 
  |-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  `-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

[root@grub-test ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 
```
2.2. Переименование Volume Group и просмотр изменений:<br/>
```shell
[root@grub-test ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"

[root@grub-test ~]# lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   40G  0 disk 
|-sda1                  8:1    0    1M  0 part 
|-sda2                  8:2    0    1G  0 part /boot
`-sda3                  8:3    0   39G  0 part 
  |-OtusRoot-LogVol00 253:0    0 37.5G  0 lvm  /
  `-OtusRoot-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

[root@grub-test ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0 
```
2.3. Правка /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg:<br/>
```shell
[root@grub-test ~]# nano /etc/fstab
...

[root@grub-test ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/OtusRoot-LogVol01 swap                    swap    defaults        0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
#VAGRANT-END

[root@grub-test ~]# nano /etc/default/grub 
...
[root@grub-test ~]# cat /etc/default/grub 
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb"
GRUB_DISABLE_RECOVERY="true"

[root@grub-test ~]# nano /boot/grub2/grub.cfg 
...
[root@grub-test ~]# cat /boot/grub2/grub.cfg
...
	linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/OtusRoot-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb 
	initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
...
```
2.4. Подготовка нового образа initramfs и перезагрузка системы:<br/>
```shell
[root@grub-test ~]# cd /boot

[root@grub-test boot]# mv initramfs-3.10.0-862.2.3.el7.x86_64.img initramfs-3.10.0-862.2.3.el7.x86_64.img.old

[root@grub-test boot]# dracut initramfs-`uname -r`.img
/sbin/dracut: line 655: warning: setlocale: LC_MESSAGES: cannot change locale (ru_RU.UTF-8): No such file or directory
/sbin/dracut: line 656: warning: setlocale: LC_CTYPE: cannot change locale (ru_RU.UTF-8): No such file or directory

[root@grub-test boot]# ll
total 40296
-rw-------. 1 root root  3409102 May  9  2018 System.map-3.10.0-862.2.3.el7.x86_64
-rw-r--r--. 1 root root   147823 May  9  2018 config-3.10.0-862.2.3.el7.x86_64
drwxr-xr-x. 3 root root       17 May 12  2018 efi
drwxr-xr-x. 2 root root       27 May 12  2018 grub
drwx------. 5 root root       97 May 12  2018 grub2
-rw-------. 1 root root 14658285 Apr 17 13:45 initramfs-3.10.0-862.2.3.el7.x86_64.img
-rw-------. 1 root root 16506787 May 12  2018 initramfs-3.10.0-862.2.3.el7.x86_64.img.old
-rw-r--r--. 1 root root   304926 May  9  2018 symvers-3.10.0-862.2.3.el7.x86_64.gz
-rwxr-xr-x. 1 root root  6225056 May  9  2018 vmlinuz-3.10.0-862.2.3.el7.x86_64

[root@grub-test boot]# reboot
```
2.5. Просмотр результатов выполнения задания:<br/>
```shell
[vagrant@grub-test ~]$ lsblk
NAME                  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                     8:0    0   40G  0 disk 
|-sda1                  8:1    0    1M  0 part 
|-sda2                  8:2    0    1G  0 part /boot
`-sda3                  8:3    0   39G  0 part 
  |-OtusRoot-LogVol00 253:0    0 37.5G  0 lvm  /
  `-OtusRoot-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]

[vagrant@grub-test ~]$ exit
```
### 3. Добавить модуль в initrd ###
3.1. Подготовка модулей:<br/>
```shell
[vagrant@grub-test ~]$ sudo -i

[root@grub-test ~]# cd /usr/lib/dracut/modules.d

[root@grub-test modules.d]# mkdir 01test

[root@grub-test modules.d]# cd 01test/

[root@grub-test 01test]# touch module-setup.sh

[root@grub-test 01test]# touch test.sh

[root@grub-test 01test]# chmod a+x module-setup.sh test.sh 

[root@grub-test 01test]# nano module-setup.sh 
...

[root@grub-test 01test]# cat module-setup.sh 
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}

[root@grub-test 01test]# nano test.sh 
...

[root@grub-test 01test]# cat test.sh 
#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'
Hello! You are in dracut module!
 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
```
3.2. Пересборка модуля initrd:<br/>
```shell
[root@grub-test 01test]# cd /boot

[root@grub-test boot]# mv initramfs-3.10.0-862.2.3.el7.x86_64.img initramfs-3.10.0-862.2.3.el7.x86_64.img  .old

[root@grub-test boot]# dracut initramfs-`uname -r`.img
/sbin/dracut: line 655: warning: setlocale: LC_MESSAGES: cannot change locale (ru_RU.UTF-8): No such file or directory
/sbin/dracut: line 656: warning: setlocale: LC_CTYPE: cannot change locale (ru_RU.UTF-8): No such file or directory

[root@grub-test boot]# ll
total 40296
-rw-------. 1 root root  3409102 May  9  2018 System.map-3.10.0-862.2.3.el7.x86_64
-rw-r--r--. 1 root root   147823 May  9  2018 config-3.10.0-862.2.3.el7.x86_64
drwxr-xr-x. 3 root root       17 May 12  2018 efi
drwxr-xr-x. 2 root root       27 May 12  2018 grub
drwx------. 5 root root       97 May 12  2018 grub2
-rw-------. 1 root root 14658055 Apr 17 15:20 initramfs-3.10.0-862.2.3.el7.x86_64.img
-rw-------. 1 root root 16506787 May 12  2018 initramfs-3.10.0-862.2.3.el7.x86_64.img.old
-rw-r--r--. 1 root root   304926 May  9  2018 symvers-3.10.0-862.2.3.el7.x86_64.gz
-rwxr-xr-x. 1 root root  6225056 May  9  2018 vmlinuz-3.10.0-862.2.3.el7.x86_64

[root@grub-test boot]# lsinitrd -m initramfs-3.10.0-862.2.3.el7.x86_64.img | grep test
test
```
3.3. Удаление опций rhgb и quiet в файле grub.cfg и перезагрузка системы:<br/>
```shell
[root@grub-test boot]# nano /boot/grub2/grub.cfg 
...

[root@grub-test boot]# cat /boot/grub2/grub.cfg
...
	linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 
	initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
...

[root@grub-test boot]# reboot
```
3.4. Просмотр результата выполнения задания:<br/>
![изображение](https://github.com/DemBeshtau/07_1_DZ/assets/149678567/3659fd57-8521-44d7-8804-5988920a91e2)
