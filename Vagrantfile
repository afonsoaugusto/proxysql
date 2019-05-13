# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "master1" do |subconfig|
    subconfig.vm.box = "debian/jessie64"
    subconfig.vm.hostname = "master1"
    subconfig.vm.network "public_network",
        use_dhcp_assigned_default_route: true
  end

  config.vm.define "master2" do |subconfig|
    subconfig.vm.box = "generic/alpine38"
    subconfig.vm.hostname = "master2"
    subconfig.vm.network "public_network",
        use_dhcp_assigned_default_route: true
  end  
end