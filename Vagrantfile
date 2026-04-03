Vagrant.configure("2") do |config|

    os = "generic/rocky9"
    host_net = "192.168.100"
    config.vm.synced_folder '.', '/vagrant', disabled: true

    config.vm.define :app do |app_config|
        app_config.vm.provider "virtualbox" do |vb|
            vb.memory = "2048"
            vb.cpus = 2
            vb.name = "FiiPractic-App-1"
        end
        app_config.vm.host_name = 'app.fiipractic.lan'
        app_config.vm.box = "#{os}"
        app_config.vm.network "private_network", ip: "#{host_net}.10"
        app_config.vm.provision "shell", inline: <<-SHELL
            echo "root:fiipractic" | chpasswd
            echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/99-vagrant.conf
            echo "PermitRootLogin yes" >> /etc/ssh/sshd_config.d/99-vagrant.conf
            systemctl restart sshd
            yum install -y git wget vim
        SHELL
    end

    config.vm.define :gitlab do |gitlab_config|
        gitlab_config.vm.provider "virtualbox" do |vb|
            vb.memory = "4096"
            vb.cpus = 4
            vb.name = "FiiPractic-GitLab"
        end
        gitlab_config.vm.host_name = 'gitlab.fiipractic.lan'
        gitlab_config.vm.box = "#{os}"
        gitlab_config.vm.disk :disk, size: "20GB", primary: true
        gitlab_config.vm.network "private_network", ip: "#{host_net}.20"
        gitlab_config.vm.provision "shell", inline: <<-SHELL
            echo "root:fiipractic" | chpasswd
            echo "PasswordAuthentication yes" > /etc/ssh/sshd_config.d/99-vagrant.conf
            echo "PermitRootLogin yes" >> /etc/ssh/sshd_config.d/99-vagrant.conf
            systemctl restart sshd
            yum install -y git wget vim
        SHELL
    end

end