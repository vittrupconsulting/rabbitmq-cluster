# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  config.vm.synced_folder ".", "/vagrant"
  
  config.vm.define "node01" do |node01|
	node01.vm.network "private_network", ip: "172.16.1.20", virtualbox__intnet: true
    node01.vm.provision "shell", inline: <<-SHELL
      sudo su -
	  hostname "node01"
	  echo "node01" > /etc/hostname
      echo "172.16.1.20    node01" >> /etc/hosts
	  echo "172.16.1.21    node02" >> /etc/hosts
      echo "172.16.1.22    node03" >> /etc/hosts
	  apt-get update
	  apt-get install rabbitmq-server -y
	  cp /var/lib/rabbitmq/.erlang.cookie /vagrant/cluster.cookie
	  sleep 5
	  rabbitmqctl add_user admin AdminPassRabbitMQ
	  rabbitmqctl set_user_tags admin administrator
	  rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
	  rabbitmqctl delete_user guest
	  sleep 5
	  rabbitmq-plugins enable rabbitmq_management
    SHELL
  end
 
  config.vm.define "node02" do |node02|
	node02.vm.network "private_network", ip: "172.16.1.21", virtualbox__intnet: true
    node02.vm.provision "shell", inline: <<-SHELL
      sudo su -
	  hostname "node02"
	  echo "node02" > /etc/hostname
      echo "172.16.1.20    node01" >> /etc/hosts
	  echo "172.16.1.21    node02" >> /etc/hosts
      echo "172.16.1.22    node03" >> /etc/hosts
	  apt-get update
	  apt-get install rabbitmq-server -y
	  systemctl stop rabbitmq-server
	  cp /vagrant/cluster.cookie /var/lib/rabbitmq/.erlang.cookie
	  sleep 5
	  systemctl start rabbitmq-server
	  sleep 5
	  rabbitmqctl stop_app
	  sleep 5
	  rabbitmqctl join_cluster rabbit@node01
	  sleep 5
	  rabbitmqctl start_app
	  sleep 5
	  rabbitmq-plugins enable rabbitmq_management
    SHELL
  end

  config.vm.define "node03" do |node03|
	node03.vm.network "private_network", ip: "172.16.1.22", virtualbox__intnet: true
    node03.vm.provision "shell", inline: <<-SHELL
      sudo su -
	  hostname "node03"
	  echo "node03" > /etc/hostname
      echo "172.16.1.20    node01" >> /etc/hosts
	  echo "172.16.1.21    node02" >> /etc/hosts
      echo "172.16.1.22    node03" >> /etc/hosts
	  apt-get update
	  apt-get install rabbitmq-server -y
	  systemctl stop rabbitmq-server
	  sleep 5
	  cp /vagrant/cluster.cookie /var/lib/rabbitmq/.erlang.cookie
	  sleep 5
	  systemctl start rabbitmq-server
	  sleep 5
	  rabbitmqctl stop_app
	  sleep 5
	  rabbitmqctl join_cluster rabbit@node01
	  sleep 5
	  rabbitmqctl start_app
	  sleep 5
	  rabbitmq-plugins enable rabbitmq_management
    SHELL
  end

  config.vm.define "proxy" do |proxy|
	proxy.vm.network "private_network", ip: "172.16.1.23", virtualbox__intnet: true
	proxy.vm.network "forwarded_port", guest: 15672, host: 15672
    proxy.vm.provision "shell", inline: <<-SHELL
      sudo su -
      echo "172.16.1.20    node01" >> /etc/hosts
	  echo "172.16.1.21    node02" >> /etc/hosts
      echo "172.16.1.22    node03" >> /etc/hosts
	  apt-get update
	  apt-get install haproxy -y
	  cp /vagrant/haproxy.cfg /etc/haproxy/haproxy.cfg
	  systemctl restart haproxy
    SHELL
  end

end
