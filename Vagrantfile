# -*- mode: ruby -*-
# vi:set ft=ruby sw=2 ts=2 sts=2:

# Define the number of master and worker nodes
# If this number is changed, remember to update setup-hosts.sh script with the new hosts IP details in /etc/hosts of each VM.
NUM_MASTER_NODE = 1

IP_NW = "192.168.5."
MASTER_IP_START = 10


Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_check_update = false

  (1..NUM_MASTER_NODE).each do |i|
      config.disksize.size = '50GB'
      config.vm.define "master-#{i}" do |node|
        # Name shown in the GUI
        node.vm.provider "virtualbox" do |vb|
            vb.name = "kubernetes-ha-master-#{i}"
            vb.memory = 6144
            vb.cpus = 4
            vb.gui = true
        end
        node.vm.hostname = "master-#{i}"
        node.vm.network :private_network, ip: IP_NW + "#{MASTER_IP_START + i}"
        node.vm.network "forwarded_port", guest: 22, host: "#{2710 + i}"

        node.vm.provision "setup-hosts", :type => "shell", :path => "setup-hosts.sh" do |s|
          s.args = ["enp0s8"]
        end

        node.vm.provision "setup-dns", type: "shell", :path => "update-dns.sh"
        node.vm.provision "setup-docker", type: "shell", :path => "install-docker.sh"
        node.vm.provision "setup-kubeadm", type: "shell", :path => "setup-kubeadm.sh"
        node.vm.provision "setup-gui", type: "shell", :path => "setup-gui.sh"

      end
  end
end
