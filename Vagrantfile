# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

$num_nodes = (ENV['NUM_NODES'] || 2).to_i
$num_glusters = (ENV['NUM_NODES'] || 2).to_i

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true


  def set_vbox(vb, cfg, men)
    vb.gui = false
    vb.memory = men
    vb.cpus = 1
  end

  glusters = Array.new()

  $num_glusters.times do |i|
    name = "gluster-node-#{i+1}"
    glusters.push(name)
    config.vm.define "#{name}" do |machine|
      machine.vm.hostname = name
 #    machine.vm.box = "geerlingguy/ubuntu1404"
      machine.vm.network "private_network", ip: "10.168.10.#{100+i}", :netmask => "255.255.255.0"
      machine.vm.provider :virtualbox do |vb, override|
        set_vbox(vb, override, 512)
      end
    end
  end 

  config.vm.define "kube-master" do |n|
    n.vm.hostname = "kube-master"
    n.vm.network "private_network", ip: "10.168.10.80", :netmask => "255.255.255.0"
    n.vm.provider :virtualbox do |vb, override|
     set_vbox(vb, override, 1024)
    end
  end

  config.vm.define "es" do |n|
    n.vm.hostname = "es"
    n.vm.network "public_network"
    n.vm.network "forwarded_port", guest: 9200, host: 9200
    n.vm.network "forwarded_port", guest: 9300, host: 9300
    n.vm.network "private_network", ip: "10.168.10.70", :netmask => "255.255.255.0"
    n.vm.provider :virtualbox do |vb, override|
     set_vbox(vb, override, 2048)
    end
  end

  nodes = Array.new()

  $num_nodes.times do |i|
    name = "kube-node-#{i+1}"
    nodes.push(name)
    config.vm.define "#{name}" do |n|
      n.vm.hostname = name
      n.vm.network "private_network", ip: "10.168.10.#{10+i}", :netmask => "255.255.255.0"
      n.vm.provider :virtualbox do |vb, override|
        set_vbox(vb, override, 2048)
      end
    end
  end

  groups = {
    :dev => ["es"],
    :etcd => ["kube-master"],
    :masters => ["kube-master"],
    :nodes => nodes,
    :glusters => glusters,
    :'all_groups:children' => ["dev", "etcd", "masters", "nodes", "glusters"]
  }

  config.vm.provision :ansible do |ansible|
    ansible.groups = groups
    ansible.playbook = "connection.yml"
    ansible.limit = "all"
    ansible.extra_vars = { private_network_iface: "eth1" }
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
