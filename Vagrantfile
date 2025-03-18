Vagrant.configure("2") do |config|
    # Define a reusable script block for all VMs
    script = <<-SHELL
      sudo swapoff -a
      echo "192.168.33.10 node1" | sudo tee -a /etc/hosts
      echo "192.168.33.11 node2" | sudo tee -a /etc/hosts
      echo "192.168.33.12 kubemaster" | sudo tee -a /etc/hosts
      sudo apt update -y
    SHELL
    
      # Define the first VM
      config.vm.define "node1" do |node1|
        node1.vm.box = "ubuntu/jammy64"
        node1.vm.network "private_network", ip: "192.168.33.10"
        node1.vm.provider "virtualbox" do |v|
          v.memory = 4096
        end
        node1.vm.provision "shell", inline: script
      end
    
      # Define the second VM
      config.vm.define "node2" do |node2|
        node2.vm.box = "ubuntu/jammy64"
        node2.vm.network "private_network", ip: "192.168.33.11"
        node2.vm.provider "virtualbox" do |v|
          v.memory = 4096
        end
        node2.vm.provision "shell", inline: script
      end
    # Define the second VM
    config.vm.define "kubemaster" do |kubemaster|
      kubemaster.vm.box = "ubuntu/jammy64"
      kubemaster.vm.network "private_network", ip: "192.168.33.12"
      kubemaster.vm.provider "virtualbox" do |v|
        v.memory = 4096
      end
      kubemaster.vm.provision "shell", inline: script
      end
    
    end
    