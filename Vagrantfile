# -*- mode: ruby -*-
# vi: set ft=ruby :

# Every Vagrant development environment requires a box. You can search for
# boxes at https://atlas.hashicorp.com/search.
BOX_IMAGE = "centos/7"
NODE_COUNT = 1

Vagrant.configure("2") do |config|
  config.vm.define "master" do |subconfig|
    subconfig.vm.box = BOX_IMAGE
    subconfig.vm.hostname = "master"
    subconfig.vm.network :private_network, ip: "10.0.0.10"
    
    #install nfs server
    config.vm.provision "file", source: "./exports", destination: "~/exports"
    config.vm.provision "shell", inline: <<-SHELL
      yum install -y nfs-utils
      mkdir /nfs
      touch /nfs/test.file
      mv /home/vagrant/exports /etc/exports
      systemctl enable nfs-server
      systemctl enable rpcbind
      systemctl start nfs-server
      systemctl start rpcbind
      exportfs -arv
    SHELL

  end
  
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |subconfig|
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "node#{i}"
      subconfig.vm.network :private_network, ip: "10.0.0.#{i + 10}"
    end
  end

  # Add repos config
  config.vm.provision "file", source: "./kubernetes.repo", destination: "~/kubernetes.repo"
  config.vm.provision "file", source: "./k8s.conf", destination: "~/k8s.conf"
  config.vm.provision "file", source: "./api.conf", destination: "~/api.conf"

  # Install docker on all instances and nfs client
  config.vm.provision "shell", inline: <<-SHELL
    yum update 
    yum install -y nfs-utils
    yum install -y yum-utils device-mapper-persistent-data lvm2
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce
    systemctl start docker
    systemctl enable docker
    groupadd docker
    usermod -aG docker vagrant
 
    #general kubernetes config
    mv /home/vagrant/kubernetes.repo /etc/yum.repos.d/kubernetes.repo
    mv /home/vagrant/k8s.conf /etc/sysctl.d/k8s.conf
    mkdir /etc/systemd/system/docker.service.d
    mv /home/vagrant/api.conf /etc/systemd/system/docker.service.d/api.conf
    
    systemctl daemon-reload
    systemctl restart docker

    swapoff -a
    sysctl --system
    setenforce 0
    yum install -y kubelet kubectl kubeadm
    systemctl enable kubelet && systemctl start kubelet
    
    #kubeadm init --pod-network-cidr=10.244.0.0/16
    #kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
    
  SHELL
end
