
##                                                              Базовая настройка вагрант файла

<details>
                        <summary> Базовый конфиг вагрант-файла </summary>
                        
        Vagrant.configure("2") do |config|
                        config.vm.box = "ashum1976/centos7_kernel_5.10"
                        config.vm.synced_folder ".", "/vagrant", disabled: true
                        config.vm.synced_folder "./sync_data", "/home/vagrant/mnt"
                        
    # Провижинг, выполнение команд после запуска машины     
                        config.vm.provision "shell", inline: <<-SHELL    <----- провижинг, выполнение команд после запуска машины, в данном случае в shell. можно подцепить скрипт
                                mkdir -p ~root/.ssh
                                cp ~vagrant/.ssh/auth* ~root/.ssh
                            #  yum install -y redhat-lsb-core rpmdevtools rpm-build createrepo yum-utils wget
                            #  /vagrant/bash_rpm.sh
                        SHELL
                
        end

</details>






##                                                              Конфиг параметры


1.   Проброс симлинков

-   Вам просто нужно добавить параметр setextradata для каждой общей папки mount:

*           config.vm.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/k5", "1"]    < ---------- возможность пробрасывать симлинки из хостовой папки,  в папку которая монтируется в        гостевую систему

