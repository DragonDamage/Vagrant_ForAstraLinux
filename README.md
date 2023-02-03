# Vagrant
## VagrantFile:
```ruby
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
```


## playbook.yaml:
```ruby
---
- name: Install Docker and Docker Compose
  hosts: all
  become: yes
  tasks:
  - name: Update package index
    apt:
      update_cache: yes
  - name: Install Docker
    apt:
      name: docker.io
      state: latest
  - name: Install Docker Compose
    command: sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    become: yes
    environment:
      PATH: /usr/local/bin:$PATH
  - name: Add execute permission to Docker Compose binary
    command: sudo chmod +x /usr/local/bin/docker-compose
    become: yes

- name: Install Node Exporter
  hosts: all
  become: yes
  tasks:
  - name: Download Node Exporter
    command: sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.0.0/node_exporter-1.0.0.linux-amd64.tar.gz
    become: yes
  - name: Extract Node Exporter
    command: sudo tar -xvzf node_exporter-1.0.0.linux-amd64.tar.gz
    become: yes
  - name: Start Node Exporter
    command: sudo ./node_exporter-1.0.0.linux-amd64/node_exporter &
    become: yes
    args:
      chdir: node_exporter-1.0.0.linux-amd64

- name: Start Docker Compose
  hosts: all
  become: yes
  tasks:
  - name: Start Prometheus and Grafana
    command: docker-compose up -d
    become: yes
    args:
      chdir: /vagrant
```
