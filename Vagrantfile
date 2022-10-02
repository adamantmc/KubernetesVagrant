Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end

    config.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("./keys/id_rsa.pub").first.strip
        s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        SHELL
    end

    config.vm.define "master" do |master|
        master.vm.network "private_network", ip: "192.168.56.10"
        master.vm.hostname = "master"
    end

    config.vm.define "w1" do |w1|
        w1.vm.network "private_network", ip: "192.168.56.11"
        w1.vm.hostname = "worker1"
    end

    config.vm.define "w2" do |w2|
        w2.vm.network "private_network", ip: "192.168.56.12"
        w2.vm.hostname = "worker2"
    end

end
