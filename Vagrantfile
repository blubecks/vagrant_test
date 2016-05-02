VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "centos/7"
    config.vm.hostname = "vagrantbox"
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.synced_folder "./html/", "/var/www/html"
    config.vm.provider "virtualbox" do |vb|
        vb.customize ['modifyvm', :id, '--memory', '1024']
    end
    config.vm.provider :virtualbox do |vb|
      vb.gui = false
    end
    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
        ansible.sudo = true
    end
    config.vm.provision :shell, inline: "echo Good job, now enjoy your new vbox"
end
