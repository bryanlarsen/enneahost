# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "private_network", ip: "192.168.33.100"
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision-vagrant.yml"
  end

  config.vm.synced_folder "../docker-consulkv-plugin", "/docker-consulkv-plugin"

  config.vm.post_up_message = "############ Setup your keys in order to use your enneahost ############\n\nvagrant ssh -c \"curl -L https://github.com/{your-username}.keys >> /home/vagrant/.ssh/authorized_keys\"\n\nThen:\n\ncat ~/.ssh/{your-key.pub} | ssh vagrant@192.168.33.100 \"sudo gitreceive upload-key {your-name}\n"
end
