# Creating three local Virtual machines for K8S cluster

1. Open Windows Powershell and use following commands to create folder structure

```
cd \
mkdir k8s-cluster
notepad Vagrantfile
```

2. You should have a new file "Vagrantfile" open in Notepad. Add following lines in `Vagrantfile`

```vagrantfile
Vagrant.configure("2") do |config|
  # Define a reusable script block for all VMs
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
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
        v.memory = 5120
      end
      node1.vm.provision "shell", inline: script
    end
  
    # Define the second VM
    config.vm.define "node2" do |node2|
      node2.vm.box = "ubuntu/jammy64"
      node2.vm.network "private_network", ip: "192.168.33.11"
      node2.vm.provider "virtualbox" do |v|
        v.memory = 5120
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
```

3. Notepad would force an extension ".txt" to Vagrantfile, rename "Vagrantfile.txt" to "Vagrantfile"

```
cd \k8s-cluster
ren  .\Vagrantfile.txt   .\Vagrantfile
```

4. Use `vagrant up` command to launch three VMs at once.

```
vagrant up
```

5. Wait for 5-10 Minutes for VMs to be ready !

6. To Access `Master` node we would use this command:

```
cd \k8s-cluster
vagrant ssh kubemaster 
```

> Keep this Powershell Window Open for Future use

7. To Access `Node1` node we would use this command:

```
cd \k8s-cluster
vagrant ssh node1
```

> Keep this Powershell Window Open for Future use

8. To Access `Node2` node we would use this command:

```
cd \k8s-cluster
vagrant ssh node2 
```

> Keep this Powershell Window Open for Future use

## How to install `libcontainerd` (container runtime)

> These instructions are COMMON to all the machines (master, node1 and node2)

1. SSH into one of the machines (Step 6,7 and 8)

2. Use following commands to set linux kernel modules (container networking) for `libcontainerd`

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

3. Use following command to reload kernel modules configured just now.

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

```

4. Set kernel parameters for kubernetes packages (used later)

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOD
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOD

```

5. Apply all the changes to kernel parameters before installing packagses.

```bash
sudo sysctl --system

```

6. Install the pre-requisite packages for installing third party packages (containerd and kubernetes).

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

7. Install the GPG key for `docker` repository.

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
```

8. Add the `docker` repository. (press ENTER when prompted)

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

9. Install the `containerd` package.

```bash
sudo apt update -y && sudo apt install -y containerd.io
```

10. Using a new Powershell, test if all VMs have `containerd` installed.

```bash
cd \k8s-cluster
vagrant ssh node1 -c "apt list --installed | grep containerd"
vagrant ssh node2 -c "apt list --installed | grep containerd"
vagrant ssh node3 -c "apt list --installed | grep containerd"
```

11. Configure `containerd` to start using systemd cgroup driver (Required for latest kubernetes packages )

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

```

12. Restart and Enable `containerd` service (Using SystemD)

```bash
sudo systemctl enable containerd

sudo systemctl restart containerd
```

## Install Kubernetes packages using Ubuntu APT package manager

1. Add kubernetes package repository 

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update -y

```

2. Install the packages and make sure OS (Ubuntu) wont update them automatically.

```bash
sudo apt install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl
```

3. Verify if all packages are installed.

```bash
kubeadm version -o short
kubelet --version
kubectl version --client
```

