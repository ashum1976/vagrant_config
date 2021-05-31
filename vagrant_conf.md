
##                                                              Базовая настройка вагрант файла

<details>
                        <summary> Базовый конфиг вагрант-файла </summary>

        Vagrant.configure("2") do |config|
                config.vm.box = "ashum1976/centos7_kernel_5.10"        <------ Образ в vagrantclouds
                config.vm.synced_folder ".", "/vagrant", disabled: true     <--- отключаем проброс папки "./" с хостовой системы в гостевую ( "/vagrant" ) для всех создаваемых машин
                config.vm.synced_folder "./sync_data", "/home/vagrant/mnt"   <---- пробрасываем в гостевую систему, в папку "/home/vagrant/mnt",  папку "./sync_data" с хостовой системы

    # Провижинг, выполнение команд после запуска машины
                config.vm.provision "shell", inline: <<-SHELL    <----- провижинг, выполнение команд после запуска машины, в данном случае в shell. можно подцепить скрипт
                        mkdir -p ~root/.ssh
                        cp ~vagrant/.ssh/auth* ~root/.ssh
                    #  yum install -y redhat-lsb-core rpmdevtools rpm-build createrepo yum-utils wget
                    #  /vagrant/bash_rpm.sh
                SHELL

        end

</details>

___

<details>
                        <summary> Базовый конфиг вагрант-файла, на несколько машин  </summary>

    Vagrant.configure(2) do |config|
            config.vm.box = "ashum1976/centos7_kernel_5.10"
          #config.vm.box = "centos/7"


            config.vm.provider "virtualbox" do |v|
                v.memory = 256
                v.cpus = 1
            end

            config.vm.define "nfs_server" do |nfss|                                                         <------ задаём параметры box-a  "nfs_server"
                #nfss.vm.synced_folder "./sync_data_server", "/home/vagrant/mnt"
                nfss.vm.network "private_network", ip: "192.168.50.10", virtualbox__intnet: "net1"   <----- добавляем ещё сетевую карту с нужным IP
                nfss.vm.hostname = "nfssrv"                                                                        <-------- Задаём имя нашей создаваемой виртуальной машины hostname
                nfss.vm.provision "shell", path: "nfss_script.sh"                                            <------ Провижинг используя готовый скрипт, который будет находится в одной папке с Vagrant файлом
            end

            config.vm.define "nfs_client" do |nfsc|
                #nfsc.vm.synced_folder "./sync_data_client", "/home/vagrant/mnt"
                nfsc.vm.network "private_network", ip: "192.168.50.11", virtualbox__intnet: "net2"
                nfsc.vm.hostname = "nfscln"
                nfsc.vm.provision "shell", path: "nfsc_script.sh"
            end

end



</details>

___

<details>
                        <summary> Расширенный конфиг вагрант-файла, на несколько машин  </summary>
Этот файл использовался в уроке по запуску ansible (lesson_13 и lesson_15)

        home = ENV['HOME']
        MACHINES = {
        :'prod-nginx-01' => {
                :box_name => "centos/7",
                :ip_addr => '192.168.11.150',
        },
        :'prod-nginx-02' => {
                :box_name => "centos/7",
                :ip_addr => '192.168.11.151',
        },
        :'staging-nginx-01' => {
                :box_name => "centos/7",
                :ip_addr => '192.168.11.200',
        }
        }
    Vagrant.configure("2") do |config|

        MACHINES.each do |boxname, boxconfig|

           config.vm.define boxname do |box|
                box.vm.box = boxconfig[:box_name]
                box.vm.host_name = boxname.to_s
>Создадим сетевой интерфейс, внешний, будет доступен на хостовой машине, где тоже создастся сетевой интерфейс из этой же подсети ( 192.168.11.0/24):

                box.vm.network "private_network", ip: boxconfig[:ip_addr], virtualbox__extnet: "net1"
                box.vm.provider :virtualbox do |vb|
                vb.customize ["modifyvm", :id, "--memory", "256"]
                vb.name = boxname.to_s

                end

                box.vm.provision "shell", inline: <<-SHELL
                mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
            SHELL

            end

        end

    end

</details>

___
<details>
                        <summary> Расширенный конфиг вагрант-файла, на несколько дополнительных дисков  </summary>

**_Использовался в уроке по управлению RAID массивом _**

      # Describe VMs
      MACHINES = {
        # VM name "raid_create"
       :"raid-create" => {
                    # VM box
                    :box_name => "ashum1976/centos7_k5_raid_home",
                    # VM CPU count
                    :cpus => 3,
                    # VM RAM size (Mb)
                    :memory => 2048,
                    <!-- # networks
                    :net => {
                              :ip_addr => 192.168.10.15
                      }, -->
                    # forwarded ports
                    :forwarded_port => [],
                    :sync_path => "./sync_data",
                    #:sync_path => ,
                    :diskv => {
                              :sata1 => {
                                                  :dfile => "./hddvm/sata1.vdi",
                                                  :size => 1024,
                                                  :port => 1,
                                                 },
                              :sata2 => {
                                                  :dfile => "./hddvm/sata2.vdi",
                                                  :size => 1024,
                                                  :port => 2,
                                                  },
                              :sata3 => {
                                                  :dfile => "./hddvm/sata3.vdi",
                                                  :size => 1024,
                                                  :port => 3,
                                                  },
                              :sata4 => {
                                                  :dfile => "./hddvm/sata4.vdi",
                                                  :size => 1024,
                                                  :port => 4
                                                  },
                               :sata5 => {
                                                  :dfile => "./hddvm/sata5.vdi",
                                                  :size => 1024,
                                                  :port => 5
                                                  },
                               :sata6 => {
                                                  :dfile => "./hddvm/sata6.vdi",
                                                  :size => 1024,
                                                  :port => 6
                                                  },
                               :sata7 => {
                                                  :dfile => "./hddvm/sata7.vdi",
                                                  :size => 1024,
                                                  :port => 7
                                                  }
                                          }
                                      }
                                  }
      Vagrant.configure("2") do |config|
        MACHINES.each do |boxname, boxconfig|
          # Disable shared folders
                      config.vm.synced_folder ".", "/vagrant", disabled: true  # - отключаем проброс папок с хостовой системы в гостевую для всех создаваемых машин, но можем включить
                      # Apply VM config
                          config.vm.define boxname do |box|
                              # Set VM base box and hostname
                                      box.vm.box = boxconfig[:box_name]
                                      box.vm.host_name = boxname.to_s
                              # Additional network config if present
                                      if boxconfig.key? (:net) # - () это наличие такой переменной (значения) в массиве
                                           boxconfig [:net].each do |etconf, ipconf| # - цикл по значению переменно [:net], т.е :eth1 => { :ipaddr => '192.168.10.15'}
                                          #     "#{ipconf}" - получить строку находящуюся в переменной ipconf (:ipaddr => '192.168.10.15')
                                           box.vm.network :private_network, ip: ipconf[:ipaddr]
                                          end
                                      end
                              # Port-forward config if present
                                      if boxconfig.key?(:forwarded_port)
                                          boxconfig[:forwarded_port].each do |port|
                                          box.vm.network "forwarded_port", port
                                          end
                                      end
                              #Включение директорий для проброса с хостовой машины на гостевую
                                      if boxconfig.key?(:sync_path)
                                    #      boxconfig[:sync_path].each do |path|
                                    #      config.vm.synced_folder path
                                           config.vm.synced_folder boxconfig[:sync_path], "/vagrant"
                                         end

                                    #  end

                                      # VM resources config
                          box.vm.provider "virtualbox" do |v|
                              # Set VM RAM size and CPU count
                                      v.memory = boxconfig[:memory]
                                      v.cpus = boxconfig[:cpus]
                                      needsController = false
                                      boxconfig[:diskv].each do |dname, dconf|
                                                  unless File.exist?(dconf[:dfile])
                                                  v.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                                  needsController = true
                                          end
                                      end

                                      if needsController == true
                                                 v.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                                                 boxconfig[:diskv].each do |dname, dconf|
                                                 v.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium',dconf[:dfile]]
                                           end
                                      end
                          end

                          box.vm.provision "shell", inline: <<-SHELL
                          #          mkdir -p ~root/.ssh
                          #          cp ~vagrant/.ssh/auth* ~root/.ssh
                                      yum install -y mdadm smartmontools hdparm gdisk
                                      /vagrant/bash.sh
                                      SHELL


                          end
          end
      end


</details>


___


<details>
                        <summary> Расширенный конфиг вагрант-файла, на несколько машин, создание  дополнительного диска для одной и отдельный провижинг для одной машины  </summary>

      # -*- mode: ruby -*-
      # vim: set ft=ruby :
      MACHINES = {
        # VM name "srvbackup"
       :"srvbackup" => {
                    # VM box
                    :box_conf => "centos/8",
                    # VM CPU count
                    :cpus => 1,
                    # VM RAM size (Mb)
                    :memory => 2048,
                    # networks
                    :ip_addr => '192.168.10.11',
                    # forwarded ports
                    #:forwarded_port => [],
                    #:sync_path => "./sync_data",
                    #:sync_path => ,
                    :diskv => {
                              :sata1 => {
                                          :dfile => './hddvm/sata1.vdi',
                                          :size => 2048,
                                          :port => 1
                                        }
                              }
                        },
        :"client" => {
                    # VM box
                    :box_conf => "centos/8",
                    # VM CPU count
                    :cpus => 1,
                    # VM RAM size (Mb)
                    :memory => 256,
                    # networks
                    :ip_addr => '192.168.10.10'
                  }
      }

      Vagrant.configure("2") do |config|

        MACHINES.each do |boxname, boxconfig|
            if Vagrant.has_plugin?("vagrant-timezone")
                  config.timezone.value = "Europe/Minsk"
            end
            config.vm.define boxname do |box|
                  box.vm.box = boxconfig[:box_conf]
                  box.vm.host_name = boxname.to_s
                  box.vm.network "private_network", ip: boxconfig[:ip_addr], virtualbox__intnet: "net1"
                  box.vm.provider "virtualbox" do |v|
                              # Set VM RAM size, CPU count, add disks
                                      v.memory = boxconfig[:memory]
                                      v.cpus = boxconfig[:cpus]
                                      if boxconfig.key?(:diskv)
                                      needsController = false
                                              boxconfig[:diskv].each do |dname, dconf|
                                                    unless File.exist?(dconf[:dfile])
                                                    v.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                                    needsController = true
                                                  end
                                                end
                                            if needsController == true
                                                     v.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                                                     boxconfig[:diskv].each do |dname, dconf|
                                                     v.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                                                 end
                                            end
                                      end
                  end
                  if boxname.to_s == "srvbackup"
                    box.vm.provision "shell", path: "srvbackup.sh"
                  end
                  box.vm.provision "shell",  inline: <<-SHELL
                        mkdir -p /root/.ssh
                        cp ~vagrant/.ssh/auth* /root/.ssh
                        yum install -y --nogpgcheck epel-release
                        SHELL

            end
        end
      end

</details>

___



**ДЗ по теме NETWORK 25 lesson**

[Vagrantfile Network 25lesson](vagrantfile_repo/lesson25_Network/Vagrantfile)


Расширеный вариант ДЗ с использованием ansible

      box.vm.provision :ansible_local do |ansible|
       #Установка  коллекции community.general, для использования community.general.nmcli (nmcli) управление сетевыми устройствами.
       ansible.galaxy_command = 'ansible-galaxy collection install community.general'
       ansible.verbose = "vv"
       ansible.install = "true"
       #ansible.limit = "all"    <----- По умолчанию запускается только для конфигурируемой машины, но при параметре "ansible.limit = "all", для всех
       ansible.tags = boxname.to_s
       ansible.inventory_path = "./ansible/inventory/"
       ansible.playbook = "./ansible/playbooks/routers.yml"
       ansible.playbook = "./ansible/playbooks/servers.yml"
       end

**_ansible.tags = boxname.to_s_** - тегируем в файле task/main.yml, роли ansible, команду на запуск скрипта, для конфигурации сетевой настройки тестового хоста:

      - name: 'Конфигурация office2Server'
        script: office2Server.sh
        tags: **office2Server**   <---- тегируем по имени хоста, чтоб при запуске  Vagrant использовать имя хоста как тег и запускать конфигурации только нужного хоста.

___

**ДЗ по теме IPTABLES 27 lesson**

  [Vagrantfile Network 27lesson](vagrantfile_repo/lesson27_IPTABLES/Vagrantfile)

    _**Отключаем звук в настройках VM, при появлении ошибки. Всё просто, при запуске вагрантфайла и конфигурации машины: **_

        Vagrant.configure("2") do |config|

        config.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--audio", "none"]   <---- Отключили звук, тут же и настроим другие параметры VM если нужно:
        v.memory = 256
        v.cpus = 1
          end**
          .......
          ........
        MACHINES.each do |boxname, boxconfig|
          config.vm.synced_folder "./", "/vagrant", type: "rsync", rsync__auto: true, rsync__exclude: ['./hddvm, README.md']


##             Конфиг параметры



### Общая конфигурация виртуальной машины в вагрант-е

- Плагин для настройки часового пояса, при старте Vagrantfile

          vagrant plugin install vagrant-timezone

Пример настройки:

          Vagrant.configure("2") do |config|
              if Vagrant.has_plugin?("vagrant-timezone")
                config.timezone.value = "Europe/Minsk" или config.timezone.value = "UTC" <---- Выбрать временную зону UTC или Europe/Minsk
              end
              # ... other stuff
            end



**vagrant reload --no-provision**


### Монтирование папок с хостовой системы

1.  Дополнителные опции синхронизации каталога с хостовой машины в гостевую.

Дополнительные опции:

**_disabled_** - если указать True, то синхронизация будет отключена. Удобно, если нам не нужна «изкоробочная» синхронизация.
**_mount_options_** - дополнительные параметры, которые будут переданы команде mount при монтировании
**_type_** - полезная опция, которая позволяет выбрать тип синхронизации. Доступны следующие варианты:
- NFS (тип NFS доступен только для Linux-host!)
- rsync
- SMB (тип SMB доступен только для Windows-host!)
- VirtualBox

>для Linux-гостей использовать rsync - этот тип не требует дополнений гостевых систем,  автоматически установить rsync на всех гостей. Также, доступны дополнительные плюшки, такие как vagrant rsync и vagrant rsync-auto

>Для rsync есть куча опций,  одна из самых полезных - rsync_exclude. (аналог gitignore) Опция позволяет исключить из списка синхронизации, которые не нужны

**_id_** - имя, которое будет показываться при команде mount в гостевой

Подробнее вариант rsync:

Первое - этот тип работает только в одну сторону. Каталоги, которые синхронизированы через rsync синхронизируются автоматически только один раз - при инициализации машины (vagrant up\vagrant reload).
 Принудительно синхронизировать можно двумя путями:

      vagrant rsync
      vagrant rsync-auto
"vagrant rsync" --  вариант запускается один раз (синхронизировал и всё).

>Имейте ввиду, что если вы сделали изменения в Vagrantfile в области синхронизации, то вам необходимо перед vagrant rsync сделать vagrant reload.

"vagrant rsync-auto" -- работает в режиме демона и отслеживает изменения на хосте. Это удобно, так как один каталог можно шарить на несколько машин сразу, передавая изменения на всех гостей. При запуске перевести в фоновый режим.

>Если вы хотите сделать изменения в Vagrantfile, для начали остановите vagrant rsync-auto, внесите изменения, и потом перезапустите.

Пример проброса папки в гостевую машину:

    config.vm.synced_folder "scripts/", "/vagrant", type: "rsync", rsync__auto: true, rsync__exclude: ['./hddvm, README.md']

    Пробрасываем папку scripts с хостовой системы, лежащую в папке с Vagrantfile, тип синхронизации rsync, разрешить rsync__auto, исключить каталог hddvm, файл README.md лежащие в той же директории.

2.  Проброс симлинков

-   Вам просто нужно добавить параметр setextradata для каждой общей папки mount:

*           config.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/k5", "1"]    < ---------- возможность пробрасывать симлинки из хостовой папки,  в папку которая
                                                                                                                                  монтируется в        гостевую систему

___

### Сетевые настройки


- vb.customize ['modifyvm', :id, '--nicbootprio2', '1'] - модификация ( проверено в virtualbox) загрузки по сети,  со второй сетевой карты

- vb.customize ['modifyvm', :id, '--cableconnected2', 'on'] - подключение или отключение сетевого кабеля при загрузке, на сетевую карточку


### Vagrant+Ansible

При использовании Ansible в качестве провижинга, может быть два варианта запуска:

Ansible выполняется на хостовой машине:
- **_:ansible_ ( box.vm.provision :ansible do |ansible| )**

Ansible выполняется на гостевой системе:
- **:ansible_local (box.vm.provision :ansible_local do |ansible|)**

Запуск только теггированной задачи:
- **ansible.tags = boxname.to_s**
