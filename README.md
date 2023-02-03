# Vagrant
## VagrantFile:

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  config.vm.network "forwarded_port", guest: 3000, host: 3000
  
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
end
```


## playbook.yml:
```yml
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


## docker-compose.yml:
```yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:v2.21.0
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana:7.3.0
    volumes:
      - ./grafana:/var/lib/
```
