# -*- mode: ruby -*-
# vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version '>= 1.6.0'
VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Specify provider order preference
  # For details see https://www.vagrantup.com/docs/providers/basic_usage.html#default-provider
  config.vm.provider "virtualbox"

  # use local ssh key to connect to remote vagrant box
  # if not Vagrant::Util::Platform.windows? then
  #   config.ssh.private_key_path = './ssh_keys/id_rsa'
  # end
  
  # Set virtualbox provider specific attributes
  config.vm.provider :virtualbox do |v|
    v.linked_clone = true
    v.customize ['modifyvm', :id, '--audio', 'none']
    v.check_guest_additions = false
    v.gui = false
  end

  # Create a machine to run Ansible

  config.vm.define "controller", primary: true do |subconfig|

    # Specify the hostname of the machine
    subconfig.vm.hostname = "controller"

    # Sync ansible folder to remote
    subconfig.vm.synced_folder "./ansible", "/vagrant/ansible", type: "rsync",
      rsync__exclude: [".git/"],
      rsync__args: ["--verbose", "--rsync-path='sudo rsync'", "--archive", "--delete", "-z"]
  
    subconfig.vm.box = "debian/buster64"

    subconfig.vm.network :private_network, ip: "192.168.7.2"
    
    # Set node specific VirtualBox configuration/overrides
    subconfig.vm.provider "virtualbox" do |vb|
      vb.check_guest_additions = false
      vb.gui = false
      vb.name = "controller"
      vb.memory = 1024
      vb.cpus = 1
      vb.linked_clone = true
      vb.customize ['modifyvm', :id, '--audio', 'none']
      # vb.customize ["modifyvm", :id, "--ioapic", "on"]
      # Enable NAT hosts DNS resolver
      # vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      # vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]    
    end

    # Copy private ssh key to controller
    subconfig.vm.provision "file", source: "ssh_keys/id_rsa", destination: "/home/vagrant/.ssh/id_rsa"

    # Install Ansible on contoller_node and provision services
    subconfig.vm.provision :ansible_local do |ansible|

      ansible.install_mode = "pip"
      # Ensure pip is installed for Python3
      ansible.pip_install_cmd = "sudo apt install -y python3-distutils && curl https://bootstrap.pypa.io/get-pip.py | sudo python3"
      ansible.version = "2.10.6"
      ansible.compatibility_mode = "2.0"
      ansible.install = true
      ansible.limit = "all"
      ansible.verbose = "v"

      ansible.config_file = "ansible/ansible.cfg"
      ansible.inventory_path = "ansible/hosts.yml"
      ansible.playbook = "ansible/site.yml"

      ansible.galaxy_role_file = "ansible/requirements.yml"
      ansible.galaxy_roles_path = "ansible/roles"

    end

    subconfig.vm.post_up_message = "Controller node spun up!"

  end

end