# -*- mode: ruby -*-
# vi: set ft=ruby :

ip_address = "172.22.22.22"
project_name = "my-project"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Enable Berkshelf support
  config.berkshelf.enabled = true

  # Use the omnibus installer for the latest Chef installation
  config.omnibus.chef_version = :latest

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "ubuntu/trusty64"

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

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./public" , "/var/www/" + project_name + "/",  :nfs => true, :mount_options => ['actimeo=2']


  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider :virtualbox do |v|
      v.gui = false
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "95"]
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
      v.customize ["modifyvm", :id, "--memory", "1024"]
      v.customize ["modifyvm", :id, "--cpus", 2]
  end

  config.vm.provision :shell, :inline => "sed -i 's#archive.ubuntu.com#mirrors.163.com#' /etc/apt/sources.list"
  config.vm.provision :shell, :inline => "sed -i 's#security.ubuntu.com#mirrors.163.com#' /etc/apt/sources.list"
  config.vm.provision :shell, :inline => "apt-get update"
  # Use hostonly network with a static IP Address and enable
  # hostmanager so we can have a custom domain for the server
  # by modifying the host machines hosts file
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  
  config.vm.define project_name do |node|
    node.vm.hostname = project_name + ".local"
    node.vm.network :private_network, ip: ip_address
    node.hostmanager.aliases = [ "www." + project_name + ".local" ]
  end
  config.vm.provision :hostmanager


  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end


  config.vm.provision "chef_solo" do |chef|
    
    
    chef.custom_config_path = "Vagrantfile.chef"
    chef.add_recipe "apt"
    chef.add_recipe "git"
    chef.add_recipe "nginx"
    chef.add_recipe "php-fpm"
    chef.add_recipe "mysql::server"
    chef.add_recipe "mysql::client"
    chef.add_recipe "app::virtual_host"
    chef.json = {
      :app => {
        :name           => project_name
      },
      :php => {
        # Customize PHP modules here
        :packages                => %w{ php5 php5-dev php5-cli php-pear php5-apcu php5-mysql php5-curl php5-mcrypt php5-memcached php5-gd php5-json },

        # It is necessary to specify a custom conf dir as we are using Apache 2.4
        :ext_conf_dir            => "/etc/php5/mods-available"
      },  
      'versions' => {
        "php5" => '5.5.*'
      }
    }

  end

end
