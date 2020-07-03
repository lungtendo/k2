# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu1804"
  config.vm.box_check_update = false

  # Master node
  config.vm.define "master" do |m|
    config.vm.provider :libvirt do |v|
      v.memory = "2048"
      v.cpus = 2
      v.storage_pool_name = "home"
    end
    m.vm.network "private_network", ip: "192.168.33.110"
    m.vm.hostname = "master.localdomain"
  end

  # Worker node
  config.vm.define "worker" do |w|
    config.vm.provider :libvirt do |v|
      v.memory = "2048"
      v.cpus = 2
      v.storage_pool_name = "home"
    end
    w.vm.network "private_network", ip: "192.168.33.120"
    w.vm.hostname = "worker.localdomain"
  end

  # Ansible provisioner
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/add-user.yaml"
  end

end
