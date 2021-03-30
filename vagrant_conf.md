
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
                        <summary> Базовый конфиг вагрант-файла, на несколько машин версия 2  </summary>

Vagrant.configure(2) do |config|
            #config.vm.box = "ashum1976/centos7_kernel_5.10"
            config.vm.box = "centos/7"

            config.vm.define "prod_server" do |prod|
                #prod.vm.synced_folder "./sync_data_server", "/home/vagrant/mnt"
                prod.vm.network "private_network", ip: "192.168.50.10", virtualbox__intnet: "net1"
                prod.vm.hostname = "prod-server"
                config.vm.provider "virtualbox" do |v|
                    v.memory = 256
                    v.cpus = 1
                end
            #    prod.vm.provision "shell", path: "nfss_script.sh"
            end

            config.vm.define "adm_comp" do |adm|
                #adm.vm.synced_folder "./sync_data_client", "/home/vagrant/mnt"
                adm.vm.network "private_network", ip: "192.168.50.11", virtualbox__intnet: "net1"
                adm.vm.hostname = "admin-comp"
                config.vm.provider "virtualbox" do |v|
                    v.memory = 256
                    v.cpus = 1
                end
              #  adm.vm.provision "shell", path: "nfsc_script.sh"
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
                box.vm.network "private_network", ip: boxconfig[:ip_addr]

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





##         Конфиг параметры


1.   Проброс симлинков

-   Вам просто нужно добавить параметр setextradata для каждой общей папки mount:

*   config.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/k5", "1"]    < ---------- возможность пробрасывать симлинки из хостовой папки,  в папку которая монтируется в        гостевую систему


##    Команды управления

 1. Просмотр списка виртуальных машин

        [andrey@nix64amd /]$ vagrant global-status
        id       name          provider   state    directory
        ------------------------------------------------------------------------------------------------------------------------
        26be8b7  default       virtualbox poweroff /installs/vagrant/vagrant_proect/vb/centos8
        94be509  kernel-update virtualbox running  /storage/Study/OTUS/lesson_01/Homework/homework_manual_kernel_update
        66f6dc9  kernel-update virtualbox running  /storage/Study/OTUS/lesson_01/homework/homework_manual_kernel_update
        5d4ba60  raid-create   virtualbox running  /storage/Study/OTUS/lesson_02/homework_raid_create

        The above shows information about all known Vagrant environments    <---- Почистить vagrant окружение (environments), запустив команду "vagrant global-status --prune"
        on this machine. This data is cached and may not be completely
        up-to-date (use "vagrant global-status --prune" to prune invalid
        entries).


 2. Просмотр списка box-ов на основе которых создавались машины (vm)

        [andrey@nix64amd /]$ vagrant box list
        ashum1976/centos7_k5_raid_home (virtualbox, 03.02.2021v1)
        ashum1976/centos7_kernel_5.10  (virtualbox, 01.03.2021)
        ashum1976/centos7_kernel_5.11  (virtualbox, 03.03.2021)
        ashum1976/centos_7.5_home      (virtualbox, 0)

 3. Удаление определённого образа, и определённой версии

        [andrey@nix64amd /]$ vagrant box remove centos/7 --box-version 1804.02

        Box 'centos/7' (v1804.02) with provider 'virtualbox' appears           <---- Сообщение , что на базе этого образа созданы виртуальные машины, рекомендуется сначало их удалить
        to still be in use by at least one Vagrant environment. Removing
        the box could corrupt the environment. We recommend destroying
        these environments first:

        lvm (ID: 7bd10d6299c74e59b125287259c336b1)                              <---- Список VM
        hwproc (ID: 25dfcf410ab64485ad86276a1b6d9b9e)                           <---- Список VM
        prod_server (ID: 9b018bbe99ae424093546e810d24cc6c                   <---- Список VM
