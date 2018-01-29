# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|

  config.vm.define "indigo" do |indigo|
    indigo.vm.box = "generic/fedora27"
    indigo.vm.provision :shell, path: "bootstrap_fedora.sh"
    indigo.vm.synced_folder ".", "/vagrant"


    indigo.vm.network "private_network", ip: "192.168.50.4"
  end
  
end
