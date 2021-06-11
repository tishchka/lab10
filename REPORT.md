## Laboratory work X

Данная лабораторная работа посвещена изучению процесса создания и конфигурирования виртуальной среды разработки с использованием **Vagrant**

```sh
$ open https://www.vagrantup.com/intro/index.html
```

## Tasks

- [x] 1. Ознакомиться со ссылками учебного материала
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
# Устанавливаем значение переменных окружения
# Указываем имя пользователя Github
$ export GITHUB_USERNAME=tishchka

# Указываем используемый пакетный менеджер
$ export PACKAGE_MANAGER=apt
```

```sh
# Скачиваем Virtualbox
# Взято с https://linuxvsem.ru/programs/virtualbox-obzor#i-3
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"
sudo apt update && sudo apt install virtualbox-6.0
```

```sh
# Переходим в рабочую директорию
$ cd ${GITHUB_USERNAME}/workspace
# Устанавливаем vagrant
$ sudo ${PACKAGE_MANAGER} install vagrant
```

```sh
# Выводим версию скаченного vagrant
$ vagrant version

Installed Version: 2.2.6

Vagrant was unable to check for the latest version of Vagrant.
Please check manually at https://www.vagrantup.com


# Создаем новую виртуальную машину (с ОС Ubuntu-19.10)
$ vagrant init bento/ubuntu-19.10

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.


# Выводим содержимое Vagrantfile
$ less Vagrantfile
# Создаем новый Vagrantfile, перезаписываем изначальный (флаг -f), находящийся в текущем пути. Причем информация в нем будет в минимальном объеме (флаг -m)
$ vagrant init -f -m bento/ubuntu-19.10

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

```

```sh
# Создаем директорию shared
$ mkdir shared
```

```sh
# В файл Vagrantfile записываем комманды для запуска скрипта 
$ cat > Vagrantfile <<EOF
\$script = <<-SCRIPT
sudo apt install docker.io -y
sudo docker pull fastide/ubuntu:19.04
sudo docker create -ti --name fastide fastide/ubuntu:19.04 bash
sudo docker cp fastide:/home/developer /home/
sudo useradd developer
sudo usermod -aG sudo developer
echo "developer:developer" | sudo chpasswd
sudo chown -R developer /home/developer
SCRIPT
EOF
```

```sh
# В файл Vagrantfile записываем конфигурацию для виртуальной машины
# vagrant-vbguest - это плагин, который автоматически обновляет гостевые дополнения VirtualBox
$ cat >> Vagrantfile <<EOF

Vagrant.configure("2") do |config|

  config.vagrant.plugins = ["vagrant-vbguest"]
EOF
```

```sh
# Продолжаем конфигурацию для виртуальной машины:
$ cat >> Vagrantfile <<EOF

  config.vm.box = "bento/ubuntu-19.10"                              # Указываем версию виртуальной машины: ubuntu-19.10
  config.vm.network "public_network"                                # Указываем настройки сети: public_network
  config.vm.synced_folder('shared', '/vagrant', type: 'rsync')      # Указываем связующие директории: 'shared', '/vagrant', type: 'rsync'

  config.vm.provider "virtualbox" do |vb|                           # Указываем тип виртуальной машины: virtualbox
    vb.gui = true                                                   # Указываем, что используется графический интерфейс: vb.gui = true 
    vb.memory = "2048"                                              # Указываем, сколько выделяем оперативной памяти под виртуальную машину: 2048МБ 
  end

  config.vm.provision "shell", inline: \$script, privileged: true   # config.vm.provision "shell" - задает встроенную команду оболочки для выполнения на удаленном компьютере

  config.ssh.extra_args = "-tt"                                     # config.ssh.extra_args - значение настроек передается непосредственно в исполняемый файл ssh#

end
EOF
```

```sh
# Для устронения неполадок
$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching vagrant-libvirt-0.5.0.gem
Fetching micromachine-3.0.0.gem
Fetching vagrant-vbguest-0.29.0.gem
Parsing documentation for vagrant-libvirt-0.5.0
Installing ri documentation for vagrant-libvirt-0.5.0
Parsing documentation for micromachine-3.0.0
Installing ri documentation for micromachine-3.0.0
Parsing documentation for vagrant-vbguest-0.29.0
Installing ri documentation for vagrant-vbguest-0.29.0
Done installing documentation for vagrant-libvirt, micromachine, vagrant-vbguest after 0 seconds
Installed the plugin 'vagrant-vbguest (0.29.0)'!


# проверка корректности файла Vagrantfile
$ vagrant validate
Vagrantfile validated successfully.

# посмотреть список вируальных машин и их статусы
$ vagrant status
Current machine states:

default                   not created (libvirt)

# запуск виртуальной машины                         
$ vagrant up # --provider virtualbox
default: Machine booted and ready! 

# информация о проброске портов      
$ vagrant port
22 (guest) => 2222 (host)

$ vagrant status 
Current machine states:

default                   running (virtualbox)

# подключение по ssh к запущенной виртуальной машине                      
$ vagrant ssh

# посмотреть список сохранённых состояний виртальной машины
$ vagrant snapshot list

# сделать снимок текущего состояния виртуальной машины
$ vagrant snapshot push

$ vagrant snapshot list

# остановить виртуальную машину
$ vagrant halt

# востановить состояние виртуальной машины по снимку
$ vagrant snapshot pop
```

```ruby
  # настраиваем Vagrant для работы с VMware
  config.vm.provider :vmware_esxi do |esxi|

    esxi.esxi_hostname = '<exsi_hostname>'
    esxi.esxi_username = 'root'
    esxi.esxi_password = 'prompt:'

    esxi.esxi_hostport = 22

    esxi.guest_name = '${GITHUB_USERNAME}'

    esxi.guest_username = 'vagrant'
    esxi.guest_memsize = '2048'
    esxi.guest_numvcpus = '2'
    esxi.guest_disk_type = 'thin'
  end
```

```sh
$ vagrant plugin install vagrant-vmware-esxi
$ vagrant plugin list
$ vagrant up --provider=vmware_esxi
```

## Report

```sh
$ cd ~/workspace/
$ export LAB_NUMBER=10
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER}.git tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Links

- [VirualBox](https://www.virtualbox.org/)
- [Vagrant providers](https://github.com/hashicorp/vagrant/wiki/Available-Vagrant-Plugins#providers)
- [Vagrant vbguest plugin](https://github.com/dotless-de/vagrant-vbguest)
- [Vagrant disksize plugin](https://github.com/sprotheroe/vagrant-disksize)
- [Vagrant vmware esxi plugin](https://github.com/josenk/vagrant-vmware-esxi)

```
Copyright (c) 2015-2021 The ISC Authors
```
