# -*- mode: ruby -*-
# vi: set ft=ruby :
#

FEDORA = {
  box: "opscode-fedora-18",
  url: "https://opscode-vm-bento.s3.amazonaws.com/vagrant/opscode-fedora-18_provisionerless.box"
}

UBUNTU = {
  box: "opscode-ubuntu-12.04",
  url: "https://opscode-vm.s3.amazonaws.com/vagrant/opscode_ubuntu-12.04_provisionerless.box",
  prerequisites: "git python-pip python-dev libxml2-dev libxslt1-dev screen"
}

OS = UBUNTU

ADMIN_PASSWORD = "nomoresecrete"

options = {
  :admin_password => ADMIN_PASSWORD,
  :mysql_password => "stackdb",
  :rabbit_password => "stackqueue",
  :service_password => ADMIN_PASSWORD,
  :host_ip => "10.0.3.50",
  :verbose => "True",
  :logfile => "$DEST/logs/devstack-install.log",
  :screen_logdir => "$DEST/logs/screen",
  :log_color => "False",
  :service_password => ADMIN_PASSWORD,
  :service_token => "vagranttoken",
  :api_rate_limit => "False",
  :memory => 1536,
  :cores => 1
}

CONF_FILE = Pathname.new("vagrant-overrides.conf")

override_options = CONF_FILE.exist? ? Hash[*File.read(CONF_FILE).split(/[= \n]+/)] : {}
override_options.each { |key, value| options[key.to_sym] = value unless key.start_with?("#") }

puts "Computed options: " + options.to_s

$script = <<SCRIPT
  SEMAPHORE=/root/.devstack-provisioned
  if [ ! -f $SEMAPHORE ]; then
    echo "Installing Devstack prerequisites ..."
    /usr/bin/apt-get update -y
    /usr/bin/apt-get install -y #{OS[:prerequisites]}

    if [ "$?" -ne "0" ]; then
      echo "Devstack prerequisite installation failed."
      exit 1
    fi

    echo "Installing Devstack ..."
    /bin/su vagrant -c "cd /vagrant && ./stack.sh &>/dev/null"

    if [ "$?" -ne "0" ]; then
      echo "Devstack installation failed."
      exit 1
    fi

    /usr/bin/touch $SEMAPHORE
    echo "Devstack succesfully installed."
  fi

  echo "Horizon is now available at http://#{options[:host_ip]}/"
  echo "Keystone is serving at http://#{options[:host_ip]}:5000/v2.0/"
  echo "The password: #{options[:admin_password]}"
  echo "This is your host ip: #{options[:host_ip]}"
SCRIPT

Vagrant.configure("2") do |config|

  config.vm.box = OS[:box]
  config.vm.box_url = OS[:url]
  config.vm.provider(:virtualbox) { |v| v.customize [ "modifyvm", :id, "--memory", options[:memory].to_i, "--cpus", options[:cores].to_i ] }

  config.vm.hostname = "devstack"

  config.vm.network :private_network, ip: options[:host_ip]

  File.open("localrc", "w") { |f|
    options.each { |key, value|
      key = key.to_s.upcase
      seperator = "ENABLED_SERVICES".eql?(key) ? "+=," : "="
      f.puts "#{key}#{seperator}#{value}\n"
    }
    f.puts "FLAT_INTERFACE=eth1\n"
    f.puts "PUBLIC_INTERFACE=eth1\n"
  }

  config.vm.provision :shell, :inline => $script

end

