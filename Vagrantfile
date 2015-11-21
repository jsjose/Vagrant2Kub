# -*- mode: ruby -*-
# vi: set ft=ruby :
# Vagrantfile for atomic example http://www.projectatomic.io/docs/gettingstarted/

# Ugrade vm if necessary
$upgrade = <<SCRIPT
echo I am upgrading...
date > /etc/vagrant_provisioned_at
sudo atomic host upgrade
SCRIPT

# Local registry installation on master
$local_registry = <<SCRIPT1
echo I am provisioning master local registry... 
sudo docker create -p 5000:5000 \
-v /var/lib/local-registry:/srv/registry \
-e STANDALONE=false \
-e MIRROR_SOURCE=https://registry-1.docker.io \
-e MIRROR_SOURCE_INDEX=https://index.docker.io \
-e STORAGE_PATH=/srv/registry \
--name=local-registry registry 
sudo mkdir /var/lib/local-registry 
sudo chcon -Rvt svirt_sandbox_file_t /var/lib/local-registry
SCRIPT1

# Master configuration
$config_master = <<SCRIPT2  
echo I am configuring Atomic Master...
sudo cp ./sync/local-registry.service /etc/systemd/system/local-registry.service
sudo systemctl daemon-reload
sudo systemctl enable local-registry
sudo systemctl start local-registry
sudo cp ./sync/etcd.conf /etc/etcd/etcd.conf
sudo cp ./sync/config /etc/kubernetes/config
sudo cp ./sync/apiserver /etc/kubernetes/apiserver
sudo cp ./sync/controller-manager /etc/kubernetes/controller-manager
sudo cp ./sync/flanneld-conf.json ./
sudo systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler
SCRIPT2

$config_host = <<SCRIPT3
echo I am configuring Atomic host...
# copy config files
sudo cp ./sync/docker /etc/sysconfig/docker
sudo cp ./sync/flanneld /etc/sysconfig/flanneld
sudo cp ./sync/10-flanneld-network.conf /etc/systemd/system/docker.service.d/10-flanneld-network.conf
sudo cp ./sync/config.node /etc/kubernetes/config
sudo cp ./sync/docker /etc/sysconfig/docker
sudo mkdir -p /etc/systemd/system/docker.service.d/
# interface config file patching. Vagrant by default config miss NM_CONTROLLED parameter 
sudo awk '{gsub("NM_CONTROLLED=no", "NM_CONTROLLED=yes")}1' /etc/sysconfig/network-scripts/ifcfg-enp0s8 > ifcfg-enp0s8.bak && sudo cp ifcfg-enp0s8.bak /etc/sysconfig/network-scripts/ifcfg-enp0s8
# reload and enable services
sudo systemctl daemon-reload
sudo systemctl enable flanneld kubelet kube-proxy
SCRIPT3

# Config flannel overlay network
$config_flannel = <<SCRIPT4
echo Configuring the Flannel overlay network
curl -L http://localhost:2379/v2/keys/atomic01/network/config -XPUT --data-urlencode value@flanneld-conf.json
curl -L http://localhost:2379/v2/keys/atomic01/network/config | python -m json.tool
SCRIPT4

# configure kubelet config file for each node
$config_kubelet = <<SCRIPT5
echo I am configuring kubelet IP addr... 192.168.122.$1
sudo awk '{gsub("XXX", '$1')}1' sync/kubelet > kubelet.bak && sudo cp kubelet.bak /etc/kubernetes/kubelet
SCRIPT5

# Use https://github.com/exratione/vagrant-provision-reboot plugin for reboot
require './vagrant-provision-reboot-plugin.rb' 

Vagrant.configure(2) do |config|
  config.vm.define "master" do |master|
    master.vm.provision "shell", inline: "echo Starting Master"
    master.vm.box = "centos/atomic-host"
    master.vm.box_check_update = false
    master.vm.network "private_network", ip: "192.168.122.10"
    master.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
    master.vm.provision "shell", inline: $upgrade
    master.vm.provision "shell", inline: $local_registry
    master.vm.provision "shell", inline: $config_master
    master.vm.provision "shell", inline: $config_flannel
  end

  (11..14).each do |i|
    config.vm.define "atomic-#{i}" do |node|
      node.vm.provision "shell", inline: "echo Starting node Atomic-#{i}"
      node.vm.box = "centos/atomic-host"
      node.vm.box_check_update = false
      node.vm.network "private_network", ip: "192.168.122.#{i}" 
      node.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
      node.vm.provision "shell", inline: $upgrade
      node.vm.provision "shell" do |s|
         s.inline = $config_kubelet
         s.args   = "#{i}"
      end
      node.vm.provision "shell", inline: $config_host
      # It is necessary a reboot in order to get changes in services
      node.vm.provision :unix_reboot
      # Checking services are ok
      node.vm.provision "shell", inline: "sudo systemctl status flanneld docker kubelet kube-proxy"
    end
  end

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "1024"
  end
end
