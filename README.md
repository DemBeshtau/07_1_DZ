# Работа с загрузчиком #
1. Попасть систему без пароля несколькими способами;<br/>
2. Установить систему с LVM, после чего перименовать VG;<br/>
3. Добавить модуль в initrd.<br/>
### Исходные данные ###
&ensp;&ensp;ПК на Linux c 8 ГБ ОЗУ или виртуальная машина (ВМ) с включенной Nested Virtualization.<br/>
&ensp;&ensp;Предварительно установленное и настроенное ПО:<br/>
&ensp;&ensp;&ensp;Hashicorp Vagrant (https://www.vagrantup.com/downloads);<br/>
&ensp;&ensp;&ensp;Oracle VirtualBox (https://www.virtualbox.org/wiki/Linux_Downloads);<br/>
&ensp;&ensp;&ensp;Все действия проводились с использованием Vagrant 2.4.0, VirtualBox 7.0.14 и образа<br/> 
&ensp;&ensp;CentOS 7 версии 1804_2
### Ход решения ###
### Попасть в систему несколькими способами ###
&ensp;&ensp;Для выполнения задания необходимо сконфигурировать Vagrant таким образом, чтобы после старта ВМ<br/>
открывалось окно Virtual Box с работающей системой. Необходимо в блок provider Vagrant-файла добавить строку<br/> 
vm.gui = true.
Способ № 1<br/>
 
