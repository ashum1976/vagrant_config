
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
              # networks
              :net => [],
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




##                                                              Конфиг параметры


1.   Проброс симлинков

-   Вам просто нужно добавить параметр setextradata для каждой общей папки mount:

*           config.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/k5", "1"]    < ---------- возможность пробрасывать симлинки из хостовой папки,  в папку которая  
                                                                                                                                  монтируется в        гостевую систему
2. Плагин для настройки часового пояса, при старте Vagrantfile

          vagrant plugin install vagrant-timezone

Пример настройки:

          Vagrant.configure("2") do |config|
              if Vagrant.has_plugin?("vagrant-timezone")
                config.timezone.value = "Europe/Minsk" или config.timezone.value = "UTC" <---- Выбрать временную зону UTC или Europe/Minsk
              end
              # ... other stuff
            end
