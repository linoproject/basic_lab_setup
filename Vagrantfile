vmconfig = (JSON.parse(File.read(".vagrant_param.json")))

Vagrant.configure("2") do |config| 
    config.vm.box = "bento/ubuntu-22.04"
    config.vm.network "public_network",
        ip: vmconfig["ip"]
    
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
    config.vm.provider "vmware_desktop" do |vmware|
        vmware.gui = true
        vmware.allowlist_verified = true
        vmware.vmx['displayname'] = vmconfig["vmname"]
        vmware.vmx['ethernet1.connectionType'] = "custom"
        vmware.vmx['ethernet1.vnet'] = vmconfig["vmnet"]
        vmware.vmx["memsize"] = vmconfig["memsize"]
        vmware.vmx["numvcpus"] = vmconfig["cpu"]
       

        
    end

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
    config.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_docker.yaml"
    end
    config.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_minikube.yaml"
    end
    config.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_haproxy.yaml"
    end

    config.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_terraform.yaml"
    end

    config.vm.provision "ansible_local" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_drone.yaml"
    end


end