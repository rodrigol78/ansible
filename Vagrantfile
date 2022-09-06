
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"

  config.vm.provider "virtualbox" do |v|
	v.memory = 1024
  end

  config.vm.define "wordpress" do |m|
	m.vm.network "public_network", ip: "192.168.100.100"
  end

end
