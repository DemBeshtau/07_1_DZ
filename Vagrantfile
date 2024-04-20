# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
	:"grub-test" => {
		:box_name => "centos/7",
		:box_version => "0",
		:cpus => 1,
		:memory => 1024,
	} 
}

	Vagrant.configure("2") do |config|
	  MACHINES.each do |boxname, boxconfig|
	    config.vm.synced_folder ".", "/vagrant", disabled: true
	    config.vm.define boxname do |box|
	      box.vm.box = boxconfig[:box_name]
	      box.vm.box_version = boxconfig[:box_version]
	      box.vm.host_name = boxname.to_s
	      box.vm.provider "virtualbox" do |v|
	        v.gui = true
			v.memory = boxconfig[:memory]
		    v.cpus = boxconfig[:cpus]
	      end
	    end	  
	  end	

	  config.vm.provision "shell", inline: <<-SHELL
	  sudo yum install -y nano
	  SHELL
	end
	
