# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'vagrant-hostmanager plugin is not installed!'
end

$ip_prefix = "192.168.99"

Vagrant.configure("2") do |config|
  config.vm.box = "jumperfly/centos-7-ansible"
  config.vm.box_version = "1804.02.01"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.hostmanager.include_offline = true
  config.hostmanager.enabled = false
  config.vm.provision :hostmanager

  config.vm.provider "virtualbox" do |v|
    v.memory = 256
    v.cpus = 1
  end

  (1..2).each do |i|
    config.vm.define vm_name = "node#{i}" do |config|
      config.vm.hostname = vm_name
      config.vm.network "private_network", ip: "#{$ip_prefix}.20#{i}"
      config.vm.provision "shell", inline: <<-SHELL
        cp /vagrant/vagrant_insecure_key /home/vagrant/.ssh/id_rsa
        chmod 600 /home/vagrant/.ssh/id_rsa
        chown vagrant:vagrant /home/vagrant/.ssh/id_rsa
      SHELL

      if i == 2
        config.vm.provision "shell", inline: <<-SHELL
          mkdir -p /etc/ansible/roles
          ln -snf /vagrant/ /etc/ansible/roles/jumperfly.intermediate_ca
          ansible-galaxy install --ignore-errors -r /vagrant/tests/requirements.yml -p /etc/ansible/roles
        SHELL

        config.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook = "tests/test.yml"
          ansible.limit = "all"
          ansible.groups = {
            "root_ca_nodes": [ "node1" ],
            "intermediate_ca_nodes": [ "node2" ]
          }
        end
      end
    end
  end
end
