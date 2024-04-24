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

    ![Альтернативный текст](/home/adminkonstantin/Изображения/Снимки\ экрана/Screen1.png)

### **Способ 1. init=/bin/sh** ###

В конце строки, начинающейся с linux16, добавляем init=/bin/sh и нажимаем сtrl-x для загрузки в систему

