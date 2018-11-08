# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANT_COMMAND = ARGV[0]

VAGRANT_HOSTNAME = ENV['VAGRANT_HOSTNAME'] || "starter-dev";

# VAGRANT_PRIVATE_IP = ENV['VAGRANT_PRIVATE_IP'] || "192.168.100.100";
VAGRANT_PUBLIC_INTERFACE = ENV['VAGRANT_PUBLIC_INTERFACE'] || "wlan0";
VAGRANT_PUBLIC_IP = ENV['VAGRANT_PUBLIC_IP'] || "192.168.1.100";

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"

  config.vm.provider "virtualbox" do |v|
    v.name = VAGRANT_HOSTNAME
    v.memory = 2048
  end

  config.vm.hostname = VAGRANT_HOSTNAME

  config.vm.synced_folder "../", "/data"

  # config.vm.network "private_network", ip: VAGRANT_PRIVATE_IP
  config.vm.network "public_network", bridge: VAGRANT_PUBLIC_INTERFACE, ip: VAGRANT_PUBLIC_IP

  config.vm.network "forwarded_port", guest: 5858, host: 5858 # Node.js Legacy Debugges
  config.vm.network "forwarded_port", guest: 9229, host: 9229 # Node.js Inspector (Chrome DevTools)

  [
    # Application ports
    3000,
    3010,
    4021,
  ].each { |port|
    config.vm.network "forwarded_port", guest: port, host: port
  }

  config.vm.provision "shell", privileged: false, inline: <<-SHELL

set -xeo pipefail

# START Essentials

sudo apt-get update

sudo apt-get install -y build-essential python

# END Essentials


# START Python

sudo apt-get install -y python-pip

sudo pip install --upgrade pip

sudo pip install pipenv

# END Python


# START Node.js

curl -sL https://deb.nodesource.com/setup_8.x | sudo bash
sudo apt-get install -y nodejs

sudo mkdir -p /opt/npm /usr/etc

sudo chown -R vagrant:vagrant /opt/npm
sudo chmod -R u+rw,g+rw,o+r /opt/npm

echo "export NPM_PREFIX=/opt/npm" | sudo tee -a /etc/bash.bashrc
echo 'export PATH=$NPM_PREFIX/bin:$PATH' | sudo tee -a /etc/bash.bashrc

source /etc/bash.bashrc
source ~/.bashrc

echo 'prefix=/opt/npm' | sudo tee -a /usr/etc/npmrc | tee -a /opt/npm/npmrc

npm install -g npm@^5.0.0 localtunnel

curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list

sudo apt-get update && sudo apt-get install -y yarn

# END Node.js


# START Meteor

curl https://install.meteor.com/ | sh

meteor --version

# END Meteor


# START Ruby

sudo apt-get install -y git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

mkdir ~/tmp
cd ~/tmp
wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.1.tar.gz
tar -xzf ruby-2.5.1.tar.gz
cd ./ruby-2.5.1/
./configure
make
sudo make install
cd ~
rm -fr ~/tmp/ruby*

ruby -v

sudo gem install bundler

bundler -v

# END Ruby


# START Rails

sudo gem install rails

rails -v

sudo apt-get install -y libpq-dev postgresql-client-* graphviz

# END Rails


# START Envirenment

echo 'export MAIL_URL=smtp://172.17.0.1:1025' >> ~/.bashrc
echo 'export POSTGRES_URL=postgresql://172.17.0.1:5432' >> ~/.bashrc
echo 'export MONGO_URL=mongodb://172.17.0.1:27017' >> ~/.bashrc
echo 'export REDIS_URL=redis://172.17.0.1:6379' >> ~/.bashrc

# END Envirenment

SHELL

end
