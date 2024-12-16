Vagrant.configure("2") do |config|
  # Utilisation de l'image Ubuntu 22.04 pour toutes les machines
  config.vm.box = "ubuntu/jammy64"

  # Génération de la clé SSH
  config.vm.provision "shell", inline: <<-SHELL
    if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
      ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
      cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    fi
  SHELL

  # Manager (où Ansible sera installé)
  config.vm.define "manager" do |manager|
    manager.vm.hostname = "manager"
    manager.vm.network "private_network", type: "dhcp"
    manager.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end
    manager.vm.provision "shell", inline: <<-SHELL
      sudo apt update
      sudo apt install -y ansible
    SHELL
  end

  # Node 1
  config.vm.define "node1" do |node1|
    node1.vm.hostname = "node1"
    node1.vm.network "private_network", type: "dhcp"
    node1.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    # Provision : copie la clé publique du manager
    node1.vm.provision "shell", inline: <<-SHELL
      mkdir -p /home/vagrant/.ssh
      echo '$(cat /home/vagrant/.ssh/id_rsa.pub)' >> /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL
  end

  # Node 2
  config.vm.define "node2" do |node2|
    node2.vm.hostname = "node2"
    node2.vm.network "private_network", type: "dhcp"
    node2.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    # Provision : copie la clé publique du manager
    node2.vm.provision "shell", inline: <<-SHELL
      mkdir -p /home/vagrant/.ssh
      echo '$(cat /home/vagrant/.ssh/id_rsa.pub)' >> /home/vagrant/.ssh/authorized_keys
      chmod 600 /home/vagrant/.ssh/authorized_keys
    SHELL
  end
end
