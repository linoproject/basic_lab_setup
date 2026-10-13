vmconfig = (JSON.parse(File.read(".vagrant_param.json")))

Vagrant.configure("2") do |config| 


    def configure_vmware_provider(config, vmname, vmnet, memsize, cpu, ip)
        config.vm.provider "vmware_desktop" do |vmware, override|

            up_message = <<-MSG
                VM Fusion/Workstation
            MSG
            override.vm.post_up_message = up_message

            override.vm.network "public_network",
                type: "dhcp",
                adapter: 1
            
            override.vm.network "private_network",
                ip: ip,
                adapter: 2
            
           
            vmware.gui = true
            vmware.allowlist_verified = true
            vmware.vmx['displayname'] = vmname
            vmware.vmx['ethernet0.connectionType'] = "nat" ## pvt network use bridge or nat depending on your network
            
            vmware.vmx['ethernet1.connectionType'] = "bridge"

            vmware.vmx['ethernet2.connectionType'] = "custom"
            vmware.vmx['ethernet2.vnet'] = vmnet
            
            vmware.vmx["memsize"] = memsize.to_s
            vmware.vmx["numvcpus"] = cpu.to_s

            
        end
    end 

    def configure_vbox_provider(config, vmname, vmnet, memsize, cpu, ip, bridge = nil)
        config.vm.provider "virtualbox" do |vbox, override|

            up_message = <<-MSG
                VirtualBox
            MSG
            override.vm.post_up_message = up_message

            vbox.check_guest_additions = true
            vbox.gui = true
            vbox.name = vmname
            vbox.memory = memsize.to_i
            vbox.cpus = cpu.to_i

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
            
            

            # Host interface to bridge (e.g. "en0: Wi-Fi"); if unset Vagrant asks at `up`
            bridge_opts = bridge ? { bridge: bridge } : {}
            override.vm.network "public_network",
                adapter: 3,
                type: "dhcp",
                **bridge_opts
            
        end
    end

    configure_vbox_provider(
        config,
        vmconfig["vmname"],
        vmconfig["vmnet"],
        vmconfig["memsize"],
        vmconfig["cpu"],
        vmconfig["ip"],
        vmconfig["bridge"]
    )
    configure_vmware_provider(
        config,
        vmconfig["vmname"],
        vmconfig["vmnet"],
        vmconfig["memsize"],
        vmconfig["cpu"],
        vmconfig["ip"]
    )

    config.vm.box = "bento/ubuntu-22.04"

    if vmconfig["disk_ext_size"] != nil
        up_message = <<-MSG
            Disk extension
        MSG
        config.vm.post_up_message = up_message

        config.vm.provider "virtualbox" do |_, override|
            override.vm.disk :disk, name: vmconfig["disk_ext_name"], size: vmconfig["disk_ext_size"]
        end

        # vagrant-vmware-desktop (<= 3.0.5) crashes in config.vm.disk on the bento box
        # (its vmx has a "scsi0:0.redo" entry without a filename), so for VMware the
        # disk is created with vmware-vdiskmanager and attached through the vmx instead
        ext_disk_path = File.join(__dir__, ".vagrant", "disks", "#{vmconfig["disk_ext_name"]}.vmdk")

        config.vm.provider "vmware_desktop" do |vmware|
            vmware.vmx["sata0:2.present"] = "TRUE"
            vmware.vmx["sata0:2.fileName"] = ext_disk_path
        end

        config.trigger.before :up do |trigger|
            trigger.ruby do |env, machine|
                if machine.provider_name == :vmware_desktop && !File.exist?(ext_disk_path)
                    vdiskmanager = "/Applications/VMware Fusion.app/Contents/Library/vmware-vdiskmanager"
                    vdiskmanager = "vmware-vdiskmanager" unless File.exist?(vdiskmanager)
                    FileUtils.mkdir_p(File.dirname(ext_disk_path))
                    system(vdiskmanager, "-c", "-s", vmconfig["disk_ext_size"], "-a", "sata", "-t", "0", ext_disk_path) ||
                        raise("vmware-vdiskmanager failed to create #{ext_disk_path}")
                end
            end
        end

        config.trigger.after :destroy do |trigger|
            trigger.ruby do |env, machine|
                if machine.provider_name == :vmware_desktop
                    Dir.glob(ext_disk_path.sub(/\.vmdk$/, "*.vmdk")).each { |f| File.delete(f) }
                end
            end
        end
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
            apt-add-repository -y ppa:ansible/ansible && apt update && apt install -y ansible
        fi
        if [ ! -d "/home/vagrant/.ansible" ]; 
            then mkdir /home/vagrant/.ansible 
        fi 
        chown vagrant:vagrant /home/vagrant/.ansible


        default_gateways=$(ip route | grep default | wc -l)
        if [ $default_gateways -gt 1 ]; then
            echo "Warning: Multiple default gateways detected remove the NAT gateway"
            ETH0_GATEWAY=$(ip route show default | grep "dev eth0" | awk '{print $3}')
            if [ -n "$ETH0_GATEWAY" ]; then
                ip route del default via $ETH0_GATEWAY
            fi
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

    if vmconfig["config"].include?("O")
        config.vm.provision "ansible_local" do |ansible|
            ansible.verbose = "v"
            ansible.playbook = "ansible_opencode.yaml"
        end
    end
end