# Vagrant
VagrantFile:

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "host01" do |host01|
   host01.vm.box = "generic/ubuntu2004"
   host01.vm.box_url = 'file:///C:/Users/rikit/Downloads/UbuntuForVagrant.box'
   host01.vm.synced_folder ".", "/vagrant", disabled: true
   host01.vm.communicator = "ssh"
   host01.vm.ignore_box_vagrantfile = true
   host01.vm.hostname = "host01"
  config.vm.network "private_network", ip: "192.168.56.2"
  config.ssh.forward_agent = true
  config.vm.network :forwarded_port, guest: 80, host: 8080,
    auto_correct: true
  config.vm.playbook :ansible do |ansible|
    ansible.limit = "all"
    ansible.playbook = "playbook.yaml"
  end
   host01.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
	vb.gui = true
	vb.memory = "1024"
	vb.name = "host01"
	vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
   end
end
