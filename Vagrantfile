Vagrant.configure("2") do |config|

  config.vm.define "ipsec" do |ipsec|
    
    ipsec.vm.box = "centos/7"
    ipsec.vm.hostname = "ipsec.local"
    ipsec.vm.network "private_network" ,  ip:"192.168.50.10"
    ipsec.vm.network "private_network" ,  ip:"10.11.13.10", netmask:"255.255.255.0",virtualbox__intnet: "net2" , auto_config: true

  end


  config.vm.define "client" do |client|
    
    client.vm.box = "centos/7"
    client.vm.hostname = "clientside.local"
    client.vm.network "private_network" ,  ip:"192.168.50.20"
    client.vm.network "private_network" , ip:"10.11.12.10", netmask:"255.255.255.0",  virtualbox__intnet: "net2"

  end
end
