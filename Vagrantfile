# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/bionic64"

  config.vm.define "k8master.mumasoft.nl"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.network "private_network", ip: "192.168.34.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
    # Customize the amount of memory on the VM:
    vb.memory = "1536"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
     # Fix the hostname
     echo k8s.mumasoft.nl > /etc/hostname
     hostname k8s.mumasoft.nl
     echo "192.168.34.10 k8s.mumasoft.nl" >> /etc/hosts

     apt-get update && apt-get install -y apt-transport-https curl ca-certificates gnupg-agent software-properties-common
     # Add kubernetes apt config. At this time (2019-12-24) xenial is the newest version
     curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
     echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" >> /etc/apt/sources.list.d/kubernetes.list
     # Add the docker apt config
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
     add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

     # Install docker and kubernetes
     apt-get update
     DEBIAN_FRONTEND=noninterface apt-get install -y kubelet kubeadm kubectl docker-ce docker-ce-cli containerd.io

     # Configure docker
     # The cgroup driver for docker should default to systemd.
     # See also https://kubernetes.io/docs/setup/production-environment/container-runtimes/
     cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
     # Pull the kubelet images
     kubeadm config images pull

     # Initialize the cluster
     kubeadm init --pod-network-cidr=10.43.0.0/16 --apiserver-advertise-address=192.168.34.10
     # Initialize the network driver
     KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
     curl -L  https://github.com/projectcalico/calicoctl/releases/download/v3.11.1/calicoctl > /usr/local/bin/calicoctl
     chmod 755 /usr/local/bin/calicoctl
     # Install the config into the vagrant user's homedir and fix the docker group
     mkdir ~vagrant/.kube
     cp /etc/kubernetes/admin.conf ~vagrant/.kube/config
     chown -R vagrant:vagrant ~vagrant/.kube
     usermod -a -G docker vagrant
     # And the dashboard
     KUBECONFIG=/etc/kubernetes/admin.conf kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml
     KUBECONFIG=/etc/kubernetes/admin.conf kubectl create serviceaccount dashboard -n default 
     KUBECONFIG=/etc/kubernetes/admin.conf kubectl create clusterrolebinding dashboard-admin -n default --clusterrole=cluster-admin --serviceaccount=default:dashboard
     # Access the dashboard at http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
     # Retrieve token for login:
     # kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
     # Get join command for cluster: kubeadm token create --print-join-command
  SHELL
end
