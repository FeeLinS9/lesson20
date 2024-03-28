MACHINES = {
  :inetRouter => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "inetRouter",
        :net => [   
                    ["192.168.255.1", 2, "255.255.255.252",  "router-net"], 
                    ["192.168.56.10", 8, "255.255.255.0"],
                ]
  },

  :inetRouter2 => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "inetRouter2",
        :net => [
                   ["192.168.245.1",  2, "255.255.255.252",  "router2-net"],
                   ["192.168.56.11",  8, "255.255.255.0"],
                ]
  },

  :centralRouter => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "centralRouter",
        :net => [
                   ["192.168.0.1",    2, "255.255.255.240",  "central-net"],
                   ["192.168.245.2",  3, "255.255.255.252",  "router2-net"],
                   ["192.168.255.2",  4, "255.255.255.252",  "router-net"],
                   ["192.168.56.12",  8, "255.255.255.0"],
                ]
  },

  :centralServer => {
        :box_name => "generic/ubuntu2204",
        :vm_name => "centralServer",
        :net => [
                   ["192.168.0.2",  2,  "255.255.255.240",  "central-net"],
                   ["192.168.56.13",   8,  "255.255.255.0"],
                ]
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      
      box.vm.provider "virtualbox" do |v|
        v.memory = 768
        v.cpus = 1
       end

      boxconfig[:net].each do |ipconf|
        box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])
      end

      if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
      end

    #  box.vm.provision "shell", inline: <<-SHELL
    #    mkdir -p ~root/.ssh
    #    cp ~vagrant/.ssh/auth* ~root/.ssh
    #  SHELL
      if boxconfig[:vm_name] == "centralServer"
        box.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/playbook.yml"
            ansible.inventory_path = "ansible/inventory"
            ansible.host_key_checking = "false"
            ansible.limit = "all"
        end
      end
    end
  end
end
