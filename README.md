# cloudcomp18
Cloud Computing 2K18

1. Buat vagrant virtualbox dan buat user 'awan' dengan password 'buayakecil'.

CONFIG VAGRANTFILE

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
VAGRANT_COMMAND = ARGV[0]
Vagrant.configure(2) do |config|
  if VAGRANT_COMMAND == "ssh"
        config.ssh.username = "awan"
        config.ssh.password = "buayakecil"
  end
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "hashicorp/precise64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 8443
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network", ip: "10.151.36.255"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "src/", "/var/www"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #  Customize the amount of memory on the VM:
     vb.memory = "1024"
     vb.cpus = 2
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

#  config.ssh.username="awan"
#  config.ssh.password = "buayakecil"
#  config.ssh.insert_key = false

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", path: "bootstrap.sh"
end

ISI FILE PROVISION (BOOTSTRAP.SH)

 #!/usr/bin/env bash
 apt-get update 
 apt-get install -y expect

expect /var/www/user.exp

ISI FILE EXPECT SETTING USER (USER.EXP)                                          

#!/usr/bin/expect -f

spawn sudo adduser awan

expect "password"
send "buayakecil\r"

expect "Retype new UNIX password"
send "buayakecil\r"


set timeout 1

expect {Full Name []: }
send "\r"

expect {Room Number []: }
send "\r"

expect {Work Phone []: }
send "\r"

expect {Home Phone []: }
send "\r"

expect {Other []: }
send "\r"

expect "Is the information correct? \[Y/n\] "
send "\r"





2. Buat vagrant virtualbox dan lakukan provisioning install Phoenix Web Framework

CONFIG VAGRANTFILE

# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "hashicorp/precise64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 8443
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network", ip: "10.151.36.255"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "src/", "/var/www"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #  Customize the amount of memory on the VM:
     vb.memory = "1024"
     vb.cpus = 2
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

#  config.ssh.username="awan"
#  config.ssh.password = "buayakecil"
#  config.ssh.insert_key = false

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", path: "bootstrap.sh"
end

ISI FILE PROVISION (BOOTSTRAP.SH)

 #!/usr/bin/env bash
 sudo apt-get update

 wget https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb 
 sudo dpkg -i erlang-solutions_1.0_all.deb
 sudo apt-get update
 sudo apt-get install -y esl-erlang elixir

 mix local.hex

 sudo apt-get update
 sudo apt-get install -y curl

 curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
 sudo apt-get update
 sudo apt-get install -y nodejs

 sudo apt-get install -y postgresql postgresql-client

 mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez --force

Run again in vagrant:

 mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez --force

