# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

config.vm.define :server1 do |node_conf|
  vm_name= "server1"
  node_conf.vm.host_name = "#{vm_name}.farm"
  node_conf.vm.box = "alphainternational/centos-6.5-x64"

  node_conf.vm.network "forwarded_port", guest: 8082, host: 9902

  node_conf.vm.network "private_network", ip: "77.77.77.56"

   node_conf.vm.provision "puppet" do |puppet|
     puppet.manifests_path = "manifests"
     puppet.manifest_file  = "server1.pp"
     puppet.module_path = "modules"
     puppet.options = "--debug"
     puppet.hiera_config_path = "hiera.yaml"

   end
  end

config.vm.define :server2 do |node_conf|
  vm_name= "server2"
  node_conf.vm.host_name = "#{vm_name}.farm"
  node_conf.vm.box = "cargomedia/debian-7-amd64-default"

  node_conf.vm.network "forwarded_port", guest: 8082, host: 9903

  node_conf.vm.network "private_network", ip: "77.77.77.57"

   node_conf.vm.provision "puppet" do |puppet|
     puppet.manifests_path = "manifests"
     puppet.manifest_file  = "server2.pp"
     puppet.module_path = "modules"
     puppet.options = "--debug"
     puppet.hiera_config_path = "hiera.yaml"

   end
  end
end
