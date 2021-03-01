
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
                nfsc.vm.network "private_network", ip: "192.168.50.11", virtualbox__intnet: "net1"
                nfsc.vm.hostname = "nfscln"
                nfsc.vm.provision "shell", path: "nfsc_script.sh"
            end

end



</details>


##                                                              Конфиг параметры


1.   Проброс симлинков

-   Вам просто нужно добавить параметр setextradata для каждой общей папки mount:

*           config.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/k5", "1"]    < ---------- возможность пробрасывать симлинки из хостовой папки,  в папку которая монтируется в        гостевую систему

