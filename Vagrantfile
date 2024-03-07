vmconfig = (JSON.parse(File.read(".vagrant_param.json")))

Vagrant.configure("2") do |config| 


    def configure_vmware_provider(config, vmname, vmnet, memsize, cpu, ip)
        config.vm.provider "vmware_desktop" do |vmware, override|

            up_message = <<-MSG
                VM Fusion/Workstation
            MSG
            override.vm.post_up_message = up_message

            
            override.vm.network "private_network",
                ip: ip,
                adapter: 1

            vmware.gui = true
            vmware.allowlist_verified = true
            vmware.vmx['displayname'] = vmname
            vmware.vmx['ethernet0.connectionType'] = "bridge" ## pvt network use bridge or nat depending on your network
            
            vmware.vmx['ethernet1.connectionType'] = "custom"
            vmware.vmx['ethernet1.vnet'] = vmnet
            
            vmware.vmx["memsize"] = memsize
            vmware.vmx["numvcpus"] = cpu

            
        end
    end 

    def configure_vbox_provider(config, vmname, vmnet, memsize, cpu, ip)
        config.vm.provider "virtualbox" do |vbox, override|

            up_message = <<-MSG
                VirtualBox
            MSG
            override.vm.post_up_message = up_message

            vbox.check_guest_additions = true
            vbox.gui = true
            vbox.name = vmname
            vbox.memory = memsize
            vbox.cpus = cpu

            vbox.customize ["modifyvm", :id, "--nictype1", "virtio"]

            if Vagrant.has_plugin?("vagrant-vbguest")
                override.vbguest.auto_update = false
                override.vbguest.no_install = true
                override.vbguest.no_remote = true
            end
            
            override.vm.network "private_network",
                adapter: 2,
                ip: ip,
                name: vmnet
            
            

            override.vm.network "public_network",
                adapter: 3,
                type: "dhcp",
                bridge: "bridge_net"
            
        end
    end

    config.vm.provider :virtualbox do |_, override|
        # some code
        configure_vbox_provider(
            config, 
            vmconfig["vmname"],
            vmconfig["vmnet"],
            vmconfig["memsize"],
            vmconfig["cpu"],
            vmconfig["ip"]
        )
    end
    config.vm.provider :vmware_desktop do |_, override|
        # some code
        configure_vmware_provider(
            config,
            vmconfig["vmname"],
            vmconfig["vmnet"],
            vmconfig["memsize"],
            vmconfig["cpu"],
            vmconfig["ip"]
        )
    end
    
    config.vm.box = "bento/ubuntu-22.04"

    if vmconfig["disk_ext_size"] != nil
        up_message = <<-MSG
            Disk extension
        MSG
        config.vm.post_up_message = up_message
        config.vm.disk :disk, name: vmconfig["disk_ext_size"], size: vmconfig["disk_ext_name"]
    else
        up_message = <<-MSG
            No Disk Extension
        MSG
        config.vm.post_up_message = up_message
    end

    config.vm.synced_folder vmconfig["homedir"], vmconfig["mountdir"]
    

    #### Ansible configurationa (all providers)

    $script = <<-'SCRIPT'

       
        if [ ! -d "/vagrant" ]; then 
            mkdir /vagrant
        fi  

        if [ ! -d "/ansible" ]; then 
            mkdir /ansible 
            apt-get -y update && apt-get -y upgrade
            apt-get install -y python3 python3-pip python3.10-venv
            apt-add-repository ppa:ansible/ansible && apt update && apt install -y ansible
        fi
        if [ ! -d "/home/vagrant/.ansible" ]; 
            then mkdir /home/vagrant/.ansible 
        fi 
        chown vagrant:vagrant /home/vagrant/.ansible


        default_gateways=$(ip route | grep default | wc -l)
        if [ $default_gateways -gt 1 ]; then
            echo "Warning: Multiple default gateways detected remove the NAT gateway"
            ETH0_GATEWAY=$(ip route show default | grep "dev eth0" | awk '{print $3}')
            ip route del default via $ETH0_GATEWAY
        else
            echo "OK: Single default gateway detected"
        fi        

    SCRIPT
    config.vm.provision "shell",
        inline: $script

    if vmconfig["disk_ext_size"] != nil
        # Disk resize
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_disk_data.yaml"
        end
    end

    ####### Configurations
    if vmconfig["config"].include?("D")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_docker.yaml"
        end
    end

    if vmconfig["config"].include?("D") && vmconfig["config"].include?("M")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_minikube.yaml"
        end
    end

    if vmconfig["config"].include?("H")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_haproxy.yaml"
        end
    end

    if vmconfig["config"].include?("T")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_terraform.yaml"
        end
    end

    if vmconfig["config"].include?("D") && vmconfig["config"].include?("R")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_drone.yaml"
        end
    end

    if vmconfig["config"].include?("A")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_awscli.yaml"
        end
    end
end