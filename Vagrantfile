Vagrant.configure("2") do |config|
    # Define a reusable script block for all VMs
    script = <<-SHELL
      sudo hostnamectl set-hostname $(hostname)
      sudo swapoff -a
      echo "192.168.33.10 node1" | sudo tee -a /etc/hosts
      echo "192.168.33.11 node2" | sudo tee -a /etc/hosts
      echo "192.168.33.12 kubemaster" | sudo tee -a /etc/hosts
    SHELL
      # Define the first VM
      config.vm.define "node1" do |node1|
        node1.vm.box = "ubuntu/jammy64"
        node1.vm.hostname = "node1"
        node1.vm.network "private_network", ip: "192.168.33.10"
        node1.vm.provider "virtualbox" do |v|
          v.memory = 5120
          v.cpus = 2
        end
        node1.vm.provision "shell", inline: script
      end
    
      # Define the second VM
      config.vm.define "node2" do |node2|
        node2.vm.box = "ubuntu/jammy64"
        node1.vm.hostname = "node2"
        node2.vm.network "private_network", ip: "192.168.33.11"
        node2.vm.provider "virtualbox" do |v|
          v.memory = 5120
          v.cpus = 2
        end
        node2.vm.provision "shell", inline: script
      end
    # Define the second VM
    config.vm.define "kubemaster" do |kubemaster|
      kubemaster.vm.box = "ubuntu/jammy64"
      node1.vm.hostname = "kubemaster"
      kubemaster.vm.network "private_network", ip: "192.168.33.12"
      kubemaster.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 2
      end
      kubemaster.vm.provision "shell", inline: script
      end
    
    end
    