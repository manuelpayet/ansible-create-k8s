# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

def add_ssh_pub_key(vm)
  vm.provision "file", source: "./ssh_keys/id_rsa.pub", destination: "~/.ssh/me.pub"
  

  vm.provision "shell", inline: <<-SHELL
  cat /home/vagrant/.ssh/me.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL

end

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "debian/stretch64"
  
  host_file = "#{File.dirname(__FILE__)}/computed_host"

  File.write(host_file, '', mode: 'w')



  (0..2).each {|idx|
    config.vm.define "master-#{idx}" do |master|
      add_ssh_pub_key master.vm
      master.vm.provider "virtualbox" do |virtualbox|
        virtualbox.cpus = 4
        virtualbox.memory = 3072
        virtualbox.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      end
      master.vm.hostname = "master-#{idx}"
      ip = "10.20.30.2#{idx}"
      master.vm.network "private_network", ip: "#{ip}",
        virtualbox__intnet: true
      File.write(host_file, "#{ip} #{master.vm.hostname}\n", mode: 'a')
      #@ansibleref.vm.provision "shell", inline: "echo #{ip} #{master.vm.hostname} | sudo tee -a /etc/hosts"
    end

  }


  (0..1).each {|idx|
    config.vm.define "worker-#{idx}" do |worker|
      add_ssh_pub_key worker.vm
      worker.vm.provider "virtualbox" do |virtualbox|
        virtualbox.cpus = 4
        virtualbox.memory = 3072
        virtualbox.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
      end
      worker.vm.hostname = "worker-#{idx}"

      ip = "10.20.30.3#{idx}"
      worker.vm.network "private_network", ip: "#{ip}",
        virtualbox__intnet: true
      
      File.write(host_file, "#{ip} #{worker.vm.hostname}\n", mode: 'a')
      # @ansibleref.vm.provision "shell", inline: "echo #{ip} #{worker.vm.hostname}  | sudo tee -a /etc/hosts"
    end

  }

  config.vm.define "ansible", primary: true do |ansible|
    ansible.vm.hostname = "ansible"
    ansible.vm.network "private_network", ip: "10.20.30.10",
      virtualbox__intnet: true
    
    ansible.vm.provision "shell", inline: "echo deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main | sudo tee -a /etc/apt/sources.list"
    
    ansible.vm.provision "shell", inline: "sudo apt install -y dirmngr --install-recommends"
    ansible.vm.provision "shell", inline: "sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367"
    ansible.vm.provision "shell", inline: "sudo apt update"
    ansible.vm.provision "shell", inline: "sudo apt install -y ansible"
    ansible.vm.provision "shell", inline: "sudo apt install -y jq"
    
    ansible.vm.provision "file", source: "./autocomplete/ansible-completion.bash", destination: "~/ansible-completion.bash"
    ansible.vm.provision "shell", inline: "sudo mv ~vagrant/ansible-completion.bash /etc/bash_completion.d/ansible-completion.bash && sudo chmod +x /etc/bash_completion.d/ansible-completion.bash"

    ansible.vm.provision "shell", inline: "cp /vagrant/ssh_keys/id_rsa ~vagrant/.ssh && chown vagrant ~vagrant/.ssh/id_rsa"


    ansible.vm.provision "file", source: host_file, destination: "/vagrant/computed_host"
    ansible.vm.provision "shell", inline: <<-SHELL
      set -e
      cat /vagrant/computed_host | sudo tee -a /etc/hosts
    SHELL

    ansible.vm.provision "shell", inline: <<-SHELL
      set -e
      cd /vagrant/ansible/kubernetes
      sudo -u vagrant ansible-playbook kubernetes.playbook.yml
    SHELL
    
end

  
  
end
