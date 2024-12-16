# Lab Vagrant et Ansible

## Objectif
Mettre en place un lab local avec **Vagrant** et **Ansible** comprenant :
1. Une machine de contrôle (manager) où Ansible est installé.
2. Deux machines cibles (node1 et node2) à configurer via Ansible.
3. Configurer automatiquement les clés SSH et les empreintes.

---

## Prérequis
- **Vagrant** installé ([Télécharger Vagrant](https://developer.hashicorp.com/vagrant/downloads)).
- **VirtualBox** ou un autre fournisseur de VM ([Télécharger VirtualBox](https://www.virtualbox.org/)).
- Un éditeur de texte (comme `nano`, `vim`, ou Visual Studio Code).

---

## Étape 1 : Configuration du répertoire Vagrant

1. Créez un répertoire pour le lab et naviguez dedans :
   ```bash
   mkdir ansible-lab
   cd ansible-lab
   ```
2. Initialisez Vagrant :
   ```bash
   vagrant init
   ```

---

## Étape 2 : Configuration du `Vagrantfile`

1. Modifiez le fichier `Vagrantfile` :
   ```bash
   nano Vagrantfile
   ```
2. Collez la configuration suivante :

   ```ruby
   Vagrant.configure("2") do |config|
     # Utilisation de l'image Ubuntu 22.04
     config.vm.box = "ubuntu/jammy64"

     # Génération de la clé SSH
     config.vm.provision "shell", inline: <<-SHELL
       if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
         ssh-keygen -t rsa -b 2048 -f /home/vagrant/.ssh/id_rsa -q -N ""
         cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
         chmod 600 /home/vagrant/.ssh/authorized_keys
       fi
     SHELL

     # Manager (machine Ansible)
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
   ```

3. Sauvegardez et quittez (`Ctrl+O`, puis `Ctrl+X`).
4. Lancez les machines :
   ```bash
   vagrant up
   ```

---

## Étape 3 : Configuration d'Ansible

### 1. Découverte des IP des machines

1. Utilisez la commande `vagrant ssh-config` pour récupérer les informations de connexion SSH :
   ```bash
   vagrant ssh-config manager
   vagrant ssh-config node1
   vagrant ssh-config node2
   ```
2. Notez les adresses IP (`HostName`) des machines et mettez-les à jour dans le fichier `inventory.ini`.

### 2. Création du fichier `inventory.ini`

1. Créez le fichier d’inventaire :
   ```bash
   nano inventory.ini
   ```
2. Ajoutez la configuration suivante (remplacez `<IP_NODE1>` et `<IP_NODE2>` par les IPs des nodes) :

   ```ini
   [managers]
   manager ansible_host=127.0.0.1 ansible_connection=local

   [nodes]
   node1 ansible_host=<IP_NODE1> ansible_user=vagrant
   node2 ansible_host=<IP_NODE2> ansible_user=vagrant

   [all:vars]
   ansible_ssh_private_key_file=~/.ssh/id_rsa
   ansible_python_interpreter=/usr/bin/python3
   ansible_ssh_common_args='-o StrictHostKeyChecking=no'
   ```

3. Sauvegardez et quittez (`Ctrl+O`, puis `Ctrl+X`).

### 3. Tester la connectivité
1. Testez la connexion Ansible avec le module `ping` :
   ```bash
   ansible all -i inventory.ini -m ping
   ```
   Si tout est correctement configuré, vous devriez voir des réponses **SUCCESS** pour chaque machine.

---

## Étape 4 : Exemple d'un Playbook Ansible

### 1. Création du Playbook

1. Créez un fichier `playbook.yml` :
   ```bash
   nano playbook.yml
   ```
2. Ajoutez un exemple pour installer **Nginx** sur les nodes :
   ```yaml
   ---
   - name: Installer et configurer Nginx
     hosts: nodes
     become: yes
     tasks:
       - name: Mettre à jour les paquets
         apt:
           update_cache: yes
       - name: Installer Nginx
         apt:
           name: nginx
           state: present
       - name: Vérifier que Nginx est actif
         service:
           name: nginx
           state: started
   ```

### 2. Exécuter le Playbook
1. Exécutez le Playbook avec Ansible :
   ```bash
   ansible-playbook -i inventory.ini playbook.yml
   ```

---

## Résultat attendu
- Les machines nodes sont accessibles via Ansible.
- Le Playbook installe et configure Nginx sur **node1** et **node2**.

---

## Conseils supplémentaires
- **Gestion des erreurs** : Si Ansible retourne des erreurs, vérifiez les permissions des fichiers SSH ou les IPs dans `inventory.ini`.
- **Extensibilité** : Ajoutez d'autres machines ou services en adaptant le `Vagrantfile` et l'inventaire Ansible.

---

## Conclusion
Ce lab fournit un environnement pratique pour apprendre et tester Ansible dans un cadre isolé. Si vous avez des problèmes, n'hésitez pas à demander de l'aide ! 🚀
