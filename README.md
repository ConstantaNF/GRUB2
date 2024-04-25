# ****Работа с загрузчиком**** # 

### Описание домашннего задания ###

1. Попасть в систему без пароля несколькими способами
2. Установить систему с LVM, после чего переименовать VG
3. Добавить модуль в initrd

### **Выполнение** ###

Задание выполняется на рабочей станции с ОС Ubuntu 22.04.4 LTS с заранее установленными Vagrant 2.4.1 и VirtualBox 7.0. Перед выполнением предварительно подготовлен репозиторий

### **Подготовка окружения** ###

Для развёртывания управляемой ВМ посредством Vagrant использую Vagrantfile из методички <https://github.com/Nickmob/vagrant_lvm1> с некоторыми изменениями: 

```
# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :grub2 => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.56.101',
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
                end
        end
    end
end
```

Данный Vagrantfile кладу в заранее подготовленный каталог `/home/adminkonstantin/GRUB2`.

Стартую ВМ:

```
adminkonstantin@2OSUbuntu:~/GRUB2$ vagrant up
```

```
Bringing machine 'grub2' up with 'virtualbox' provider...
==> grub2: Checking if box 'centos/7' version '1804.02' is up to date...
==> grub2: Clearing any previously set forwarded ports...
==> grub2: Clearing any previously set network interfaces...
==> grub2: Preparing network interfaces based on configuration...
    grub2: Adapter 1: nat
    grub2: Adapter 2: hostonly
==> grub2: Forwarding ports...
    grub2: 22 (guest) => 2222 (host) (adapter 1)
==> grub2: Running 'pre-boot' VM customizations...
==> grub2: Booting VM...
==> grub2: Waiting for machine to boot. This may take a few minutes...
    grub2: SSH address: 127.0.0.1:2222
    grub2: SSH username: vagrant
    grub2: SSH auth method: private key
==> grub2: Machine booted and ready!
==> grub2: Checking for guest additions in VM...
    grub2: No guest additions were detected on the base box for this VM! Guest
    grub2: additions are required for forwarded ports, shared folders, host only
    grub2: networking, and more. If SSH fails on this machine, please install
    grub2: the guest additions and repackage the box to continue.
    grub2: 
    grub2: This is not an error message; everything may continue to work properly,
    grub2: in which case you may ignore this message.
==> grub2: Setting hostname...
==> grub2: Configuring and enabling network interfaces...
==> grub2: Rsyncing folder: /home/adminkonstantin/GRUB2/ => /vagrant
==> grub2: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> grub2: flag to force provisioning. Provisioners marked to run always will still run.
```

### **Попасть в систему без пароля несколькими способами** ###

Открываем GUI VirtualBox, запускаем виртуальную машину и при выборе ядра для загрузки нажимаем e - в данном контексте edit. Попадаем в окно, где мы можем изменить параметры загрузки:

Ubuntu
![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/d5a8aead-4ba6-417c-8e23-6eab8e094bdc)


Centos![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/2a7b6f14-c172-43cd-964b-3c48e135bc74)



### **Способ 1. init=/bin/bash** ###

Данный способ актуален для Debian/Ubuntu дистрибутивов.
В конце строки, начинающейся с linux добавляем init=/bin/bash и нажимаем сtrl-x для загрузки в систему

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/16499765-1033-41b0-8fd5-1bf41beeee71)

Мы попали в систему:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/0b15247e-b1b2-4988-9ecd-834a3ae2a125)


Рутовая файловая система при этом монтируется в режиме Read-Only. Если мы хотим перемонтировать ее в режим Read-Write, можно воспользоваться командой:

```
mount -o remount,rw /
```

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/6cd05caf-2827-46af-9454-4e30b872ec04)

Убедимся в этом записав данные в любой файл:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/21a39080-5204-4cd3-9ffe-0a7baaf13234)

### **Способ 2. rd.break** ###

Данный способ актуален для Centos.
В конце строки, начинающейся с linux16, добавляем rd.break console=tty0 и нажимаем сtrl-x для загрузки в систему:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/09caa030-11c3-4ce4-bd6e-bd6cc5fb171f)

 Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же в режиме Read-Only, но мы не в ней). 

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/364ea54b-9168-4e16-8a37-0d3d78d7d709)

Далее пробуем попасть в нее и поменять пароль администратора:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/5d5f7c41-5eb5-4c62-8a07-cf297ac7fdc8)

Перезагружаемся и заходим с новым паролем.

### **Способ 3. rw init=/sysroot/bin/bash** ###

Альтернативный сопсоб для дистрибутива Centos 7,8.
В строке, начинающейся с linux16, заменяем ro на rw init=/sysroot/bin/bash, в конце строки добавляем console=tty0 и нажимаем сtrl-x для загрузки в систему:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/f71d657e-f86a-47df-9c5c-4c79183de7e9)

Мы в системе и файловая система сразу смонтирована в режим Read-Write:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/16155898-486c-4f72-a382-837edb64b105)

### **Установить систему с LVM, после чего переименовать VG** ###

Первым делом посмотрим текущее состояние системы:

```
[root@grub2 ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 
```

Приступим к переименованию:

```
[root@grub2 ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"
```

Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое.

Пересоздаем initrd image, чтобы он знал новое название Volume Group:

```
[root@grub2 ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```

Перезагруаемся и проверяем результат:

```
[root@grub2 ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0 
```

### **Добавить модуль в initrd** ###

Для того, чтобы добавить свой модуль, создаем папку в каталоге /usr/lib/dracut/modules.d/ с именем 01test:

```
[root@grub2 ~]# mkdir /usr/lib/dracut/modules.d/01test
```

В нее поместим два скрипта:

1. module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh
2. test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик

```
[root@grub2 ~]# cd /usr/lib/dracut/modules.d/01test
```

```
[root@grub2 01test]# touch module-setup.sh
```

```
[root@grub2 01test]# nano module-setup.sh 
```

```
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






                                                                                         [ Read 13 lines ]
^G Get Help                     ^O WriteOut                     ^R Read File                    ^Y Prev Page                    ^K Cut Text                     ^C Cur Pos
^X Exit                         ^J Justify                      ^W Where Is                     ^V Next Page                    ^U UnCut Text                   ^T To Spell
```

```
[root@grub2 01test]# touch test.sh
```

```
[root@grub2 01test]# nano test.sh 
```

```
  GNU nano 2.3.1                                                     File: test.sh                                                                                                                 


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









^G Get Help                     ^O WriteOut                     ^R Read File                    ^Y Prev Page                    ^K Cut Text                     ^C Cur Pos
^X Exit                         ^J Justify                      ^W Where Is                     ^V Next Page                    ^U UnCut Text                   ^T To Spell
```

```
[root@grub2 01test]# chmod +x module-setup.sh test.sh 
```

```
[root@grub2 01test]# ls -l
total 8
-rwxr-xr-x. 1 root root 126 Apr 25 16:14 module-setup.sh
-rwxr-xr-x. 1 root root 333 Apr 25 16:15 test.sh
```

Пересобираем образ initrd:

```
[root@grub2 01test]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
Executing: /sbin/dracut -f -v /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
dracut module 'busybox' will not be installed, because command 'busybox' could not be found!
dracut module 'crypt' will not be installed, because command 'cryptsetup' could not be found!
dracut module 'dmraid' will not be installed, because command 'dmraid' could not be found!
dracut module 'dmsquash-live-ntfs' will not be installed, because command 'ntfs-3g' could not be found!
dracut module 'mdraid' will not be installed, because command 'mdadm' could not be found!
dracut module 'multipath' will not be installed, because command 'multipath' could not be found!
*** Including module: bash ***
*** Including module: test ***
*** Including module: nss-softokn ***
*** Including module: i18n ***
*** Including module: drm ***
*** Including module: plymouth ***
*** Including module: dm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 60-persistent-storage-dm.rules
Skipping udev rule: 55-dm.rules
*** Including module: kernel-modules ***
Omitting driver floppy
*** Including module: lvm ***
Skipping udev rule: 64-device-mapper.rules
Skipping udev rule: 56-lvm.rules
Skipping udev rule: 60-persistent-storage-lvm.rules
*** Including module: qemu ***
*** Including module: resume ***
*** Including module: rootfs-block ***
*** Including module: terminfo ***
*** Including module: udev-rules ***
Skipping udev rule: 40-redhat-cpu-hotplug.rules
Skipping udev rule: 91-permissions.rules
*** Including module: biosdevname ***
*** Including module: systemd ***
*** Including module: usrmount ***
*** Including module: base ***
*** Including module: fs-lib ***
*** Including module: shutdown ***
*** Including modules done ***
*** Installing kernel module dependencies and firmware ***
*** Installing kernel module dependencies and firmware done ***
*** Resolving executable dependencies ***
*** Resolving executable dependencies done***
*** Hardlinking files ***
*** Hardlinking files done ***
*** Stripping files ***
*** Stripping files done ***
*** Generating early-microcode cpio image contents ***
*** No early-microcode cpio image needed ***
*** Store current command line parameters ***
*** Creating image file ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***
```

Проверим, какие модули загружены в образ:

```
[root@grub2 01test]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```

Редактируем grub.cfg, убрав опции rghb и quiet.
Результат:

![изображение](https://github.com/ConstantaNF/GRUB2/assets/162187256/6ef2b213-6754-44a9-bba5-ae92ec86310f)
