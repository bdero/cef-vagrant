# -*- mode: ruby -*-
# vi: set ft=ruby :

MEMORY = 6144
CPU_COUNT = 4

unless Vagrant.has_plugin?("vagrant-vbguest")
  raise "Please install the vagrant-vbguest plugin by running `vagrant plugin install vagrant-vbguest`"
end

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
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

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    vb.name = "cef_vagrant_build"

    # Display the VirtualBox GUI when booting the machine
    #vb.gui = ENABLE_VBOX_GUI

    vb.memory = MEMORY
    vb.cpus = CPU_COUNT
  end
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
  config.vm.provision "shell", inline: <<-SHELL
    set -x

    # NOTE: These commands are adapted from
    #       https://bitbucket.org/chromiumembedded/cef/wiki/MasterBuildQuickStart#markdown-header-linux-setup

    mkdir -p ~/code/automate
    mkdir -p ~/code/chromium_git

    cd ~/code
    curl 'https://chromium.googlesource.com/chromium/src/+/master/build/install-build-deps.sh?format=TEXT' | base64 -d > install-build-deps.sh
    chmod 755 install-build-deps.sh
    sudo ./install-build-deps.sh

    sudo apt-get -y install libgtkglext1-dev

    cd ~/code
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

    export PATH=$HOME/code/depot_tools:$PATH

    cd ~/code/automate
    wget https://bitbucket.org/chromiumembedded/cef/raw/master/tools/automate/automate-git.py

    echo "#!/bin/bash
      set -x
      python ../automate/automate-git.py \
        --download-dir=$HOME/code/chromium_git \
        --depot-tools-dir=$HOME/code/depot_tools \
        --no-distrib --no-build \
      | tee $HOME/code/chromium_git/update.sh
    "

    cd ~/code/chromium_git
    chmod 755 update.sh
    ./update.sh

    cd ~/code/chromium_git/chromium/src/cef
    ./cef_create_projects.sh

    cd ~/code/chromium_git/chromium/src
    ninja -C out/Debug cefclient cefsimple cef_unittests chrome_sandbox

    # This environment variable should be set at all times.
    export CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox

    # This command only needs to be run a single time.
    cd ~/code/chromium_git/chromium/src
    sudo ./build/update-linux-sandbox.sh
  SHELL
end
