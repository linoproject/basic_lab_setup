# basic_lab_setup with Vagrant
The Vagrant configuration is compatible with:
* VMware Fusion
* VMware Workstation
* VirtualBox

## Pre-requisites
* Virtualization platform VMware Fusion, Workstation or VirtualBox
* Vagrant
* For VMware Workstation
* For VMware Fusion
  * vagrant plugin install vagrant-vmware-fusion
* For Virtualbox
  * vagrant plugin install vagrant-vbguest

### Alternative way to install vagrant plugins
* Download the following gem files [https://rubygems.org/gems/](https://rubygems.org/gems/)
  * micromachine
  * vagrant-vbguest

## Params options general
| Option      | Description |
| ----------- | ----------- |
| vmname      | The name of the virtual machine |
| cpu   | Number of vCPU        |
| mem   | Mem capacity in GB        |

## Params options disk
| Option      | Description |
| ----------- | ----------- |
| disk_ext_name | Extension disk name eg.: data |
| disk_ext_size   | Extension capacity in GB |

## Params options networking

| Option      | Description |
| ----------- | ----------- |
| vmnet      | Networking name eg: vmnet2 (VMware) or HostNetwork (VirtualBox) |
| ip   | fixed ip address related to vmnet network |

## Params options configs
| Option      | Description          |
| ----------- | ----------- |
| homedir      | Directory to share (source)|
| mountdir   | Mountpoint related to the shared directory (target)        |
| config   | Default configurations (eg: "DMH_R_TA" ):       |
|          | D: Docker installation        |
|          | M: Minikube and Helm installation (need also D) |
|          | H: HAProxy installation |
|          | R: Drone automation tool installation (need also D) |
|          | T: Terraform OSS/CLI installation |
|          | A: AWS CLI installation | 

## Vagrant Execution
Depending on the virtual machine system it's possible to deploy this virtual machine in the following ways:
* vagrant up --provider vmware_desktop
* vagrant up --provider virtualbox


## WIP
Here are the feature requests and the working pipeline. For further please DM me

| Feature      | Description |
| ----------- | ----------- |
| doc with examples | This documentation with some configuration examples |
| custom_ansible      | Specify a custom ansible file location to be executed at the end of the "standard" config process |
