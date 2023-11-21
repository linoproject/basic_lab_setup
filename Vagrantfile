vmconfig = (JSON.parse(File.read(".vagrant_param.json")))

Vagrant.configure("2") do |config| 
    config.vm.box = "bento/ubuntu-22.04"
    config.vm.network "public_network",
        ip: vmconfig["ip"]

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
    

    ####### Configurations
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_docker.yaml"
    end
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_minikube.yaml"
    end
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_haproxy.yaml"
    end

    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_terraform.yaml"
    end

    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.playbook = "ansible_drone.yaml"
    end


    ###### Start Minikube


    ###### Deploy Jenkins via Helm
end