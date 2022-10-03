# Vagrant
### VagrantFile:

-*- mode: ruby -*-
vi: set ft=ruby :

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



playbook.yml:
- hosts: all
  become: true
  tasks:
    - name: Install unzip
     apt: name=unzip state=present
     - name: Install Python 3
     apt: pkg=python3-minimal state=installed

    - name: Install aptitude using apt
      apt: name=aptitude state=latest update_cache=yes force_apt_get=yes

    - name: Install required system packages
      apt: name={{ item }} state=latest update_cache=yes
      loop: [ 'apt-transport-https', 'ca-certificates', 'curl', 'software-properties-common', 'python3-pip', 'virtualenv', 'python3-setuptools']

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Update apt and install docker-ce
      apt: update_cache=yes name=docker-ce state=latest

    - name: Install Docker Module for Python
      pip:
        name: docker

    - name: Pull default Docker image
      docker_image:
        name: "{{ default_container_image }}"
        source: pull

    - name: Create default containers
      docker_container:
        name: "{{ default_container_name }}{{ item }}"
        image: "{{ default_container_image }}"
        command: "{{ default_container_command }}"
        state: present
      with_sequence: count={{ create_containers }}
      #  docker-compose
    - name: Install docker-compose
      get_url: 
      url : https://github.com/docker/compose/releases/download/1.25.1-rc1/docker-compose-Linux-x86_64
      dest: /usr/local/bin/docker-compose
      mode: 'u+x,g+x'
      #  exporter + prometheus
    - name: create opt directory for prometheus
      file:
        path: /opt/prometheus
        state: directory
    - name: download node exporter
      get_url:
        url: https://github.com/prometheus/node_exporter/releases/download/v{{ node_exporter_version }}/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz
        dest: /opt/prometheus
    - name: unarchive node exporter
      unarchive:
        remote_src: yes
        src: /opt/prometheus/node_exporter-{{ node_exporter_version }}.linux-amd64.tar.gz
        dest: /opt/prometheus
    - name: create symlink to node exporter
      file:
        path: /usr/bin/node_exporter
        state: link
        src: /opt/prometheus/node_exporter-{{ node_exporter_version }}.linux-amd64/node_exporter
    - name: install unit file to systemd
      template:
        src: node_exporter.service
        dest: /etc/systemd/system/node_exporter.service
    - name: configure systemd to use service
      systemd:
        daemon_reload: yes
        enabled: yes
        state: started
        name: node_exporter.service
